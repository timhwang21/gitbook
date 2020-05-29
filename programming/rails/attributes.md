# Attributes

`attributes` acts as a way to add a typecast layer to a model. The attribute may be backed by a database column, or it can be a "transient" attribute that only exists for the lifetime of a model instance.

## Use cases

### Structured JSON

JSON columns can be very lightweight and flexible ways to represent data when you are not sure exactly what you schema will look like in the future, and can be a way to defer decision making until more information is known. Adding a custom type and an attribute to represent a JSON column can provide more robustness.

This can look like this:

```ruby
# In some model...
attribute :my_custom_attribute, Attributes::MyCustomAttribute.new
```

```ruby
# The "bridge" between the attribute definition and the custom attribute type
class Attributes::MyCustomAttribute < ActiveRecord::Type::Json
  def deserialize(value)
    return value unless value.is_a?(String)

    parsed = JSON.parse(value)

    if parsed.blank?
      return MyCustomAttribute.new
    end

    MyCustomAttribute.new(**parsed.symbolize_keys)
  end

  def serialize(value)
    value&.to_json
  end
end
```

```ruby
# An ActiveModel-enabled class that can run validations, define custom comparators, etc.
class MyCustomAttribute
  include ActiveModel::Model
  include ActiveModel::Validations

  # ...definitions
end
```

## Sources

- [`ActiveRecord::Attributes::ClassMethods`](https://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html)
- [Introduction to ActiveRecord and ActiveModel Attributes API](https://karolgalanciak.com/blog/2016/12/04/introduction-to-activerecord-and-activemodel-attributes-api/)
