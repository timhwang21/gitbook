---
description: Pry tricks
---

# Pry

## `binding.pry` in external code

Simply monkey patch the method that you are interested in debugging:

```ruby
def SomeGem::method
  require 'pry'
  binding.pry
  super
end
```

You can even do this automatically for a list of methods you are interested in:

```ruby
[:foo, :bar, :baz].each do |cb|
  define_method cb do
    require 'pry'
    binding.pry
    super
  end
end
```
