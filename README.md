# Access Modification

"Access modification" is a fancy sounding term, but it is pretty literal in its definition. Wikipedia says:

```
"Access modifiers are keywords in object-oriented languages that set the accessibility of classes,
methods, and other members. Access modifiers are a specific part of programming language syntax
used to facilitate the encapsulation of components."
```

In Ruby, the stand alone access modifiers keywords are `private`, `protected` and `public`. They specify the level of access our methods have to the outside world. Until a class's access level is changed, all methods that fall below these stand alone modifiers have the same specified access. Let's look at these with some examples to see what this really means, and why it is important.

## Examples

By default, every method in Ruby is `public`. Which is why we often do not see a stand alone `public` accesor modifier. 

This:

```ruby
class ContructionWorker
  def build_bridge
    # code that builds bridge
    "Bridge built!"
  end
end
```

is the same as this:

```ruby
class ContructionWorker
  public

  def build_bridge
    # code that builds bridge
    "Bridge built!"
  end
end
```

And both will successfully execute:

```ruby
ContructionWorker.new.build_bridge #=> "Bridge built!"
```

When we specify private access using the stand alone `private` access modifier, we can no longer call `build_bridge` outside the context of the class.

```ruby
class ContructionWorker
  private

  def build_bridge
    # code
    "Bridge built!"
  end
end

ContructionWorker.new.build_bridge #=> NoMethodError (private method `build_bridge' called for #<ContructionWorker:0x00007fb390842ab0>)
```

You maybe noticed I keep referring to these as "stand alone" modifiers. That is because there are alternative ways to specify access in Ruby. We can specify access to an instance method by passing the method name as a symbol:


```ruby
class ContructionWorker
  def build_bridge
    # code
    "Bridge built!"
  end
  private :build_bridge
end

ContructionWorker.new.build_bridge #=> NoMethodError (private method `build_bridge' called for #<ContructionWorker:0x00007fb392124cb8>)
```

The same is true for class methods. But stand alone accessor methods do not apply to class level methods unless the class scope has been opened up with `class << self`. These two examples successfully create private class methods:

```ruby
class ContructionWorker
  def self.standard_wage; end
  private_class_method :standard_wage
end

ContructionWorker.standard_wage #=> NoMethodError (private method `standard_wage' called for ContructionWorker:Class)
```

```ruby
class ContructionWorker
  class << self
    private

    def standard_wage; end
  end
end

ContructionWorker.standard_wage #=> NoMethodError (private method `standard_wage' called for ContructionWorker:Class)
```

However, this will not set private access, even though it looks like it should be valid. This is important to remember, as it is a common "gotcha" for Rubyists.

```ruby
class ContructionWorker
  private

  def self.standard_wage
    "$20/hour"
  end
end

ContructionWorker.standard_wage #=> "$20/hour"
```



## Two class example

## Expose what should be tested, and what shouldn't

## Testing incoming and outgoing messages

## Easier to refactor and expand class


Reference:

[Wikipedia - Access Modifiers](https://en.wikipedia.org/wiki/Access_modifiers)
[Declaring Visibility in Ruby Classes](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Classes#Declaring_Visibility)
