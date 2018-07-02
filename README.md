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
class ConstructionWorker
  def build_bridge
    "Bridge built!"
  end
end
```

is the same as this:

```ruby
class ConstructionWorker
  public

  def build_bridge
    "Bridge built!"
  end
end
```

And both will successfully execute:

```ruby
ConstructionWorker.new.build_bridge #=> "Bridge built!"
```

When we specify private access using the stand alone `private` access modifier, we can no longer call `build_bridge` outside the context of the class.

```ruby
class ConstructionWorker
  private

  def build_bridge
    "Bridge built!"
  end
end

ConstructionWorker.new.build_bridge #=> NoMethodError (private method `build_bridge' called for #<ConstructionWorker:0x00007fb390842ab0>)
```

You maybe noticed I keep referring to these as "stand alone" modifiers. That is because there are alternative ways to specify access in Ruby. We can specify access to an instance method by passing the method name as a symbol:


```ruby
class ConstructionWorker
  def build_bridge
    "Bridge built!"
  end
  private :build_bridge
end

ConstructionWorker.new.build_bridge #=> NoMethodError (private method `build_bridge' called for #<ConstructionWorker:0x00007fb392124cb8>)
```

The same is true for class methods. But stand alone accessor methods do not apply to class level methods unless the class scope has been opened up with `class << self`. These two examples successfully create private class methods:

```ruby
class ConstructionWorker
  def self.standard_wage; end
  private_class_method :standard_wage
end

ConstructionWorker.standard_wage #=> NoMethodError (private method `standard_wage' called for ConstructionWorker:Class)
```

```ruby
class ConstructionWorker
  class << self
    private

    def standard_wage; end
  end
end

ConstructionWorker.standard_wage #=> NoMethodError (private method `standard_wage' called for ConstructionWorker:Class)
```

This approach will not set private access, even though it looks like it should be valid. This is important to remember, as it is a common "gotcha" for Rubyists.

```ruby
class ConstructionWorker
  private

  def self.standard_wage
    "$20/hour"
  end
end

ConstructionWorker.standard_wage #=> "$20/hour"
```

The `protected` access modifier works very similarly to the `private` modifier. `protected` methods can not be called outside of the class itself, just like private methods. However, public and protected methods can be called with an explicit recipient inside the class, while private methods cannot.

That looks like this:

```ruby
class Foo
  def bar
    self.public_method     #=> OK
    self.protected_method  #=> OK
    self.private_method    #=> raises NoMethodError
  end

  def public_method; end

  protected

  def protected_method; end

  private

  def private_method; end
end

Foo.new.public_method     #=> OK
Foo.new.protected_method  #=> raises NoMethodError
Foo.new.private_method    #=> raises NoMethodError
```

For more details on `protected` methods, [read this informative blog post](http://nithinbekal.com/posts/ruby-protected-methods/). Take note that this access level is used considerable less than public and private, although it can be found in Nitro.

## When to Use Access Modifiers

Let's take a look at a more flushed out `ConstructionWorker#build_bridge` implementation to find the value in modifying access.

```ruby
class ConstructionWorker
  def build_bridge
    buy_supplies
    analyze_soil
    construct_pylons
    assemble_bridge
  end

  private

  def buy_supplies
    # code accomplishing task
  end

  def analyze_soil
    # code accomplishing task
  end

  def construct_pylons
    # code accomplishing task
  end

  def assemble_bridge
    # code accomplishing task
  end
end
```

In the code above, `build_bridge` is publically accessible, but all of the methods that make building a bridge possible are private. This is because, for now, the only task we care about is building a bridge. We do not have any need for the worker to simply `analyze_soil`, nor any of it's other tasks in its private interface. Only within the class itself will this functionality be needed.

### Why is this helpful?

This makes it very clear what methods are safe to alter, without worrying about breaking our application somewhere else. For example, if our `construction_worker` needs to start buying supplies from another vendor, that should not affect anything except for building a bridge. Knowing that nowhere else relies on `ConstructionWorker#buy_supplies` means we can modify our code in confidence, as long as `build_bridge` still works properly.

### How do we know `build_bridge` still works properly?

You guessed it. TESTS! Our tests for the `ConstructionWorker` class should focus entirely on `build_bridge`, because `build_bridge` is the only publicly accessible method in this class.

## Test Public Interface

![Test Public Interface](https://raw.githubusercontent.com/powerhome/phrg-testing-public-api/master/testing_public_messages.jpg?raw=true "Test Public Interface")

The image above is from Sandi Metz's 2013 Rails Conf talk, sharable under the [creative commons license](https://creativecommons.org/licenses/by-sa/3.0/us/legalcode). While this is not a summary of her presentation, the image shows how an object should be tested. Private methods are an implementation detail that is hidden to the users of the object. Thus, they are stored below the dotted blue line. The tests for an object should focus on the messages it takes in and the messages it sends out, where "messages" are the bits of data the class processes.

Futhermore, if the public interface of an object is well tested, then its private interface is *implicitly* tested as well. This gives developers confidence that when they refactor, their changes do not cause anything to break.

### Refactoring

One common argument to test a private method is that its functionality is crucial to an application. However, when someone feels strongly about the need to test a private method, its normally a sign that the logic should be refactored into its own class.

For example, lets say our `ConstructionWorker` class had a lot of things to sort out to construct a pylon. Instead of leaving that logic in its private interface, it might be time to create a `Pylon` class. Then the `ConstructionWorker` can access `Pylon`'s public interface to build a bridge. And the messages sent between `ConstructionWorker` and `Pylon` should be covered by tests.

The same may be true for any of our private methods. `Supply` and `SoilAnalysis` classes could also be refactored out of `ConstructionWorker`. This type of refactoring is best accomplished using Test Driven Development, or TDD. TDD is a workflow that starts with writing a test, seeing it break, then writing the minimal code needed to get the test to pass. We will discuss TDD more in future lessons.

## Resources

- [Wikipedia - Access Modifiers](https://en.wikipedia.org/wiki/Access_modifiers)
- [Declaring Visibility in Ruby Classes](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Classes#Declaring_Visibility)
- [Ruby Protected Methods](http://nithinbekal.com/posts/ruby-protected-methods/)
