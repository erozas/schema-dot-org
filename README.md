[![Gem Version](https://badge.fury.io/rb/schema_dot_org.svg)](https://badge.fury.io/rb/schema_dot_org) [![Maintainability](https://api.codeclimate.com/v1/badges/e0c60b4cbc998563a484/maintainability)](https://codeclimate.com/github/dogweather/schema-dot-org/maintainability)

# SchemaDotOrg

Easily create [Structured Data](https://developers.google.com/search/docs/guides/intro-structured-data) with **correct syntax and semantics**.
Good structured data [helps enhance a website's search result appearance](https://developers.google.com/search/docs/guides/enhance-site):

> Google Search works hard to understand the content of a page. You can help us by providing explicit clues about the meaning of a page . . .

## Usage

Let's say you have a Rails app. First write plain-ruby code in a controller. Just instantiate
the structured data object you want in your web page:

```ruby
@my_org = Organization.new(
  name:             'Public.Law',
  founder:           Person.new(name: 'Robb Shecter'),
  founding_date:     Date.new(2009, 3, 6),
  founding_location: Place.new(address: 'Portland, OR'),
  email:            'support@public.law',
  telephone:        '+1 123 456 7890',
  url:              'https://www.public.law',
  logo:             'https://www.public.law/favicon-196x196.png',
  same_as: [
    'https://twitter.com/law_is_code',
    'https://www.facebook.com/PublicDotLaw'
    ]
  )
```

...and then output it in a template:

```html
<%= @my_org %>
```

...you'll get this perfectly formatted structured data in your HTML:

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org",
  "@type": "Organization",
  "name": "Public.Law",
  "email": "support@public.law",
  "telephone": "+1 123 456 7890",
  "url": "https://www.public.law",
  "logo": "https://www.public.law/favicon-196x196.png",
  "foundingDate": "2009-03-06",
  "founder": {
    "@type": "Person",
    "name": "Robb Shecter"
  },
  "foundingLocation": {
    "@type": "Place",
    "address": "Portland, OR"
  },
  "sameAs": [
    "https://twitter.com/law_is_code",
    "https://www.facebook.com/PublicDotLaw"
  ]
}
</script>
```

### Principle: No silent failures

We coded the library this way because the data is embedded in the HTML - and it's a
pain in the butt to manually check for errors. In my case, I manage 500,000 unique
pages in my Rails app. There's _no way_ I could place error-free structured data in
them without automatic validation.

`SchemaDotOrg` will validate your Ruby code, and if it's correct, will generate Schema.org JSON-LD markup when `#to_s`
is called. If you, e.g., didn't add the correct attributes, you'll get a descriptive error message pointing
you to the problem.

Notice how the `foundingDate` is in the required ISO-8601 format. In the same way, the `foundingLocation` is a `Place`
which adds the proper `@type` attribute. All Ruby snake-case names have been converted to the Schema.org standard camel-case.
Etc., etc.

### You are prevented from creating invalid markup

If your page loads, you know your markup is good.

If you use the wrong type or try to set an unknown attribute, SchemaDotOrg will
refuse to create the incorrect JSON-LD. Instead, you'll get a message explaining
the problem:

```ruby
Place.new(address: 12345)
# => ArgumentError: Address is class Integer, not String

Place.new(
  address: '12345 Happy Street',
  author:  'Hemmingway'
)
# => NoMethodError: undefined method `author'
```

In my experience, I never get errors from the lib. I code it once, and then move on to other things.

> This automatic validation comes from my [ValidatedObject gem](https://github.com/dogweather/validated_object), which in turn,
> is a thin wrapper around ActiveRecord::Validations. So there's nothing magical going on here.

## Supported Schema.org Types

AggregateOffer, ContactPoint, ItemList, ListItem, Offer, Organization, Person, Place,
Product, SearchAction, and WebSite.

Here are a few examples. [The source code for these is extremely easy to read.](https://github.com/dogweather/schema-dot-org/tree/master/lib/schema_dot_org)  Check them out to see all the available attributes.

### WebSite

Example with only the required attributes:

```ruby
WebSite.new(
  name: 'Texas Public Law',
  url:  'https://texas.public.law',
)
```

With the optional `SearchAction` to enable a [Sitelinks Searchbox](https://developers.google.com/search/docs/data-types/sitelinks-searchbox):

```ruby
WebSite.new(
  name: 'Texas Public Law',
  url:  'https://texas.public.law',
  potential_action: SearchAction.new(
    target: 'https://texas.public.law/?search={search_term_string}',
    query_input: 'required name=search_term_string'
  )
)
```

### Organization

Example:

```ruby
Organization.new(
  name:             'Public.Law',
  founder:           Person.new(name: 'Robb Shecter'),
  founding_date:     Date.new(2009, 3, 6),
  founding_location: Place.new(address: 'Portland, OR'),
  email:            'say_hi@public.law',
  url:              'https://www.public.law',
  logo:             'https://www.public.law/favicon-196x196.png',
  same_as: [
    'https://twitter.com/law_is_code',
    'https://www.facebook.com/PublicDotLaw'
  ]
)
```


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'schema_dot_org'
```

## Development

The coding is as DRY as I could possibly make it. I think it's really
easy to create and add to. For example, here's `Product`:

```ruby
class Product < SchemaType
  validated_attr :description,  type: String, allow_nil: true
  validated_attr :image,        type: Array,  allow_nil: true
  validated_attr :name,         type: String
  validated_attr :offers,       type: SchemaDotOrg::AggregateOffer
  validated_attr :url,          type: String
end
```

The attributes are from the [Schema.org Product spec](https://schema.org/Product).

All Rails validations are available. These are just the attributes we've felt like
adding. PR's are welcome if you want to add more. Also for more Schema.org types.


## Contributing

Bug reports and pull requests are welcome on GitHub.

## License

[The](The) gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
