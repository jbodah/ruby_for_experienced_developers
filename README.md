# Ruby for Experienced Developers

This section assumes:

* You are new to Ruby
* You have production experience in other languages
* You are comfortable with advanced programming concepts but just want a jump-start for Ruby
* I can go fast and you won't get lost/you can fill-in the gaps

This guide is not meant to be complete but instead to expose you to the most important and powerful APIs and design patterns in Ruby

## Contents
  * [Documentation](#documentation)
  * [Running Programs](#running-programs)
  * [Important Stuff About Ruby](#important-stuff-about-ruby)
  * [Documentation Conventions](#documentation-conventions)
  * [Syntax](#syntax)
  * [Data Types](#data-types)
  * [Data Structures](#data-structures)
  * [Control Flow](#control-flow)
  * [Loops](#loops)
  * [Classes](#classes)
  * [Modules](#modules)
  * [Closures](#closures)
  * [Enumerable](#enumerable)
  * [Enumerators](#enumerators)
  * [Lazy Enumerators](#lazy-enumerators)
  * [Reflection](#reflection)
  * [Metaprogramming](#metaprogramming)
  * [IO](#io)
  * [Concurrency](#concurrency)
  * [Gems](#gems)
  * [Debugging](#debugging)
  * [Idioms, Patterns, and Conventions](#idioms-patterns-and-convetions)
  * [Code Smells and Anti-Patterns](#code-smells-and-anti-patterns)
  * [Other Resources](#other-resources)

## Documentation

* [ruby-doc](https://ruby-doc.org/)
* [rubygems](https://rubygems.org/)

## Running Programs

`ruby` will invoke the Ruby interpreter. `irb` will invoke the Ruby REPL.

```sh
# Print "Hello world!\n" to stdout
ruby -e 'puts "Hello world!"'

# Require the 'json' library then print
ruby -rjson -e 'puts JSON.parse("{\"hello\": \"world\"}")'
```

You can also use `ruby` to run script files:

```rb
# hello.rb
class Hello
  def greet_me
    puts "Hello!"
  end
end

Hello.new.greet_me
```

```sh
ruby hello.rb
#=> Hello!
```

If you don't understand how something works at any time, I recommend opening up `irb` and experimenting

## Important Stuff About Ruby

* Ruby is an interpreted scripting language. When you run a program it will parse the code line-by-line and run it line-by-line.
* Ruby has late binding. There is no compiler. There is no type checking. Your program will break at runtime if it is incorrect
  * Because of this it is very important to write automated tests. Ruby's testing ecosystem is very, very good
* There is no formal namespacing Ruby. Everything is loaded into a single global namespace
* There are no interfaces in Ruby. The Ruby standard library has a few powerful examples of "duck-typing" which I'll highlight
* Ruby compiles to C and relies on several C libraries that are not thread-safe (C threads). Because of this, there is a large coarse grain lock which synchronizes all of the Ruby VM instructions. The consequence is that a single Ruby process cannot run on multiple cores. Ruby implements threads but they are green threads. This constraint should be familiar to anyone who know Python. If you want ot use multiple cores you must use multi-processing
* Ruby supports C extensions which *can* use multiple processes and do all sorts of unsafe (but often efficient) things.
* There are a number of Ruby interpreters. The default one is MRI and it is what we use. JRuby, TruffleRuby and Rubinius ("rbx") also have a small amount of adoption.

## Documentation Conventions

You'll see me (and much of the Ruby world) describe methods in a syntax like:

* `Class#method_name` (`method_name` is an instance method of `Class` called by `class_instance.method_name`)
* or `Class::method_name` (`method_name` is a class/singleton method of `Class` called by `Class.method_name`)
* or `Class.method_name` (same as `Class::method_name`)

## Syntax

* Semicolons are not required to terminate lines
* `return` is implicit; methods always return the value of their final line
* Parentheses are generally optional. They can still be useful to explicitly define an order of operations or clear up ambiguity

## Data Types

In Ruby everything is an `Object`. There are no low-level data types (like Java `int`). Characters are Strings of length 1. `nil` is Ruby's `null`. Symbols are immutable. Strings are mutable (unless explicitly frozen)

```rb
5.class
#=> Integer

5.0.class
#=> Float

"hello".class
#=> String

:hello.class
#=> Symbol

/^hello$/.class
#=> Regexp

nil.class
#=> NilClass

true.class
#=> TrueClass

false.class
#=> FalseClass

Object.new.class
#=> Object

Class.new.class
#=> Class
```

## Data Structures

Ruby's set of provided data structures is very small. There are gems (external packages) that provide more advanced and efficient data structures.
None of these data structures are thread-safe!

Arrays can work as direct access arrays, linked lists, stacks, and queues.
You can view all of the methods supported by Array in Ruby's documentation

```rb
arr = [1, :hello, 20.0]
arr[1]
#=> :hello
arr << "p"
#=> [1, :hello, 20.0, "p"]
arr
#=> [1, :hello, 20.0, "p"]
```

Hashes are associative key-value dictionaries with efficient access by key. One value per key and keys are unique.

```rb
h = {name: "greg", age: 40}
h[:name]
#=> "greg"

h[:name] = "bob"
h[:name]
#=> "bob"

# If keys are not symbols then you must use the rocket syntax
h2 = {"name" => "greg", "age" => 40}
```

Finally you have Sets which are provided by the standard library and are just sugar on top of Hash:

```rb
require 'set'
set = Set.new([1,2,1])
set.size
#=> 2
```

Gems are available that provide more advanced data structures (trees, graphs, thread-safe data structures, immutable data structures)

## Control Flow

Ruby has `if`, `unless`, ternary operators, and `case`

```rb
if applies?
  puts "it applies"
else
  puts "it doesn't apply"
end

# `unless` == not if
# You should generally avo
unless applies?
  puts "it doesn't apply"
end

# both `if` and `unless` can be used as a suffix for one-liners
puts "it applies" if applies?

# ternary
msg = applies? ? "it applies" : "it doesn't apply"
puts msg
```

`case` is a bit unique. The `#===` method of each expression in a `when` is called for comparison. This is a form of duck-typing. You can implement `#===` on a custom class to get some interesting behavior (Google it for more detail):

```rb
case object
when Integer
  :int
when String
  :string
else
  :no_idea
end
```

## Loops

Ruby has `for` loops but you should almost never use them. You can get the same behavior as a `for` loop with `.each` and `.with_index`

```rb
[1,2,3].each do |num|
  #...
end

[1,2,3].each.with_index do |num, idx|
  #...
end

while applies?
  #...
  break
end

do_something while applies?

until applies?
  #...
  break
end

do_something until applies?

loop do
  #...
  break
end
```

`next` is like `continue` in other languages

## Classes

Ruby is object-oriented and has classes

```rb
class CarMaker
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def name_length
    name.size
  end
end

honda = CarMaker.new('honda')
toyota = CarMaker.new('toyota')
honda.name
#=> "honda"
honda.name_length
#=> 5
```

`#initialize` is the constructor.
`@name` below refers to an instance variable.
The `attr_reader :name` line gives as a read-only accessor (defined as `#name`).
`attr_writer` exists for write-only (e.g. `#name=`) and `attr_accessor` defines both read and write.

We define the `name_length` method here.
Methods can take positional and keyword arguments (and a mix of both), can have default arguments, and can capture args splat-style.
We'll talk about blocks later, but every method can accept a block - an arbitrary chunk of code.

Here `SedanMaker` inherits from `CarMaker`

```rb
class SedanMaker < CarMaker
end
```

By default methods are public. You can add access modifiers (which are positional) to limit scope. `public`, `protected, and `private` are ther available modifiers

```rb
class CarMaker
  def i_am_public
  end

  private

  def i_am_private
  end

  def i_am_private_too
  end
end
```

Classes in Ruby are "open" meaning they can be amended at any time by "reopening" the class:

```rb
class Hello
  def greet
    "hello"
  end
end

h = Hello.new
h.greet
#=> hello

class Hello
  def greet
    "heyya"
  end
end

h.greet
#=> "heyya"
```

It's generally a good idea not to reopen classes but there are a few times where this will make sense to do.
Open classes enable us to easily write very powerful stubbing and mocking libraries.
They also let us modify code on the fly in a shell.

You can also have class-level methods which are bound to the class itself:

```rb
class Hello
  def self.greet
    "hello"
  end
end

Hello.greet
#=> "hello"
```

Although there is special syntax for classes, they really aren't all that different from instances.
Classes are also objects

```rb
class Hello
end

Hello.is_a?(Object)
#=> true
```

If they are objects then they must be an instance of some class. Which class? Every object in Ruby has what is called a "singleton class".
This is a unique, anonymous, dynamically allocated class of which the object is the only instance. When you `def self.greet` you are
defining a method on Hello's singleton class. All of these snippets do the same thing:

```rb
class Hello
  def self.greet
    "hello"
  end
end

class Hello
  class << self
    def greet
      "hello"
    end
  end
end

class Hello
  singleton_class.class_eval do
    def greet
      "hello"
    end
  end
end

class Hello
  instance_eval do
    def greet
      "hello"
    end
  end
end
```

What does this mean? This means you can dynamically add methods to any instance:

```rb
class Animal
  def hairy?
    true
  end
end

cat = Animal.new
mr_bigglesworth = Animal.new
mr_bigglesworth.instance_eval { def hairy?; false; end }
cat.hairy?
#=> true
mr_bigglesworth.hairy?
#=> false
```

How does this work? Inheritance. Ruby's inheritance chain works like this:

1. Look at the object's singleton class for the method
2. Look up the ancestor chain

```rb
Animal.ancestors
#=> [Animal, Object, Kernel, BasicObject]
```

Again, you don't want to do this in production code but it is a powerful tool that comes in handy especially when debugging

The last thing to mention is Ruby uses "message passing" to call methods.
In the above example we would say that we are passing `mr_bigglesworth` the `:hairy?` message.

Methods can be called dynamically with `send` (which ignores access modifiers) and `public_send`:

```rb
mr_bigglesworth.public_send(:hairy?)
#=> true

["h", "e", "l", "l", "o"].public_send(:first, 2)
#=> ["h", "e"]
```

You should try to avoid `send` in production and test code so you don't confuse someone who is reading or refactoring your code.
They might assume a `private` method is not used anywhere except internally but a `send` could make that untrue

We can also use `respond_to?` to check if an object can handle a message (else we would get an exception)

## Modules

In short Modules are essentially partial classes. They have a couple of important properties:

* They can define methods (e.g. `def some_method`). In fact, `Class` is a subclass of `Module` (see `Class.ancestors`)
* They are objects, so they have a singleton class which you can define methods on (thus `MyModule.some_method` is a thing)
* They cannot be instantiated (they are not Classes)

Modules can be "mixed in" a class's (or other module's) ancestor hierarchy. It's best to show how this works with examples:

```rb
# `include` and `prepend` affect the class's ancestor chain
MyClass = Class.new
MyClass.ancestors
#=> [MyClass, Object, Kernel, BasicObject]

MyClass.include(Module.new)
MyClass.ancestors
#=> [MyClass, #<Module:0x00007f97cb16de78>, Object, Kernel, BasicObject]

MyClass.prepend(Module.new)
MyClass.ancestors
=> [#<Module:0x00007f97cb11e918>, MyClass, #<Module:0x00007f97cb16de78>, Object, Kernel, BasicObject]
```

```rb
# `extend` affects the singleton class's ancestor chain
MyClass.singleton_class
MyClass.singleton_class.ancestors
#=> [#<Class:MyClass>, #<Class:Object>, #<Class:BasicObject>, Class, Module, Object, Kernel, BasicObject]

MyClass.extend(Module.new)
MyClass.singleton_class.ancestors
=> [#<Class:MyClass>, #<Module:0x00007f97cd0858b0>, #<Class:Object>, #<Class:BasicObject>, Class, Module, Object, Kernel, BasicObject]
```

This is an important example to understand. Method lookup always works the same way:
* when an object receives a message, it looks for a handler (a method) defined by some Module (or Class) in that object's ancestor chain
* first it checks the singleton class for a definition
* then it walks the ancestor chain looking for a handler

What does this mean practically speaking? It means that we can define common functionality in one module and "mix it in" to many classes:

```rb
module BloodTemp
  def blood_temp
    if mammal?
      :warm
    else
      :cold
    end
  end
end

class Dog
  include BloodTemp

  def mammal?
    true
  end
end

class Snake
  include BloodTemp

  def mammal?
    false
  end
end

Dog.new.blood_temp
#=> :warm

Snake.new.blood_temp
#=> :cold
```

Lastly, Modules can implement callbacks ("included", "prepended", "extended" all exist):

```rb
module BloodTemp
  def self.included(base)
    # `base` is a reference to the class that this received the `include` call
  end
end
```

The following is a common pattern:

```rb
module BloodTemp
  def self.included(base)
    base.extend(ClassMethods)
  end

  module ClassMethods
    def some_class_method
    end
  end

  def some_instance_method
  end
end

Dog.include(BloodTemp)
Dog.respond_to?(:some_class_method)
#=> true
Dog.new.respond_to?(:some_instance_method)
#=> true
```

## Closures

Ruby has a number of different types of closures and anonymous functions. In general, you can think of these as some code + some binding/context.

Every method in Ruby can accept a "block". This is basically arbitrary chunk of code:

```rb
[1,2,3].some_method { |a, b| do_something_here(a, b) }
```

The most common thing to do with a block would be to `yield` to it and maybe handle the result:

```rb
def some_method
  result = yield 20, "blah"
  process_result(result)
  :ok
end
```

Blocks give you a way to have two pieces of code interact in a loosely coupled way.

Blocks can be enclosed in curly braces or do/end. You usually curlies for single line and do/end for multiline:

```rb
do_something { |a, b| ... }

do_something do |a, b|
  ...
end
```

A block is not an `Object` but exists as an abstraction inside the VM. You can "capture" a block into a `Proc`:

```rb
def some_method(&block)
  # `block` is "captured" into a Proc object
end

def some_method
  # Proc.new will capture the block given to the current method (see Proc.new's documentation)
  block = Proc.new
end
```

Procs (and all of the rest of the closures that we'll learn about) can also be forwarded to another method using the `&` operator.
This operator basically calls `#to_proc` on the object and then converts that Proc into a block.

```rb
def some_method(&block)
  # Delegate the block to Handler.do_something
  # so it is called like Handler.do_something { ... }
  Handler.do_something(&block)
end
```

A block *is not* bounded to a context. It is just a chunk of code. Capturing it as a proc *does bound it to a context*. In short, anything that responds to `#call` is bound to a context.
This becomes a bigger deal when we talk about `Method` vs. `UnboundMethod`. In general, anything can become bound/unbound/rebound

Procs do not check arguments (extra arguments are ignored, missing arguments are set to `nil`). When a block or Proc `return`'s it exits its outer context as well making a jump. You can
define an ad-hoc proc with the following:

```rb
add = proc { |a, b| a + b }
add.call(1, 2)
#=> 3
```

A lambda is technically a `Proc` (if you check its class, it is `Proc` but `#lambda?` will return true). A lambda has the following properties:

* it checks number arguments (raises if too few/many)
* `return`ing from a lambda just returns from the lambda

Thus, a lambda is more like a true anonymous method. You can define them two ways:

```rb
add = lambda { |a, b| a + b }
add.call(1, 2)
#=> 3

add = -> (a, b) { a + b }
add.call(1, 2)
#=> 3
```

Procs and lambdas are anonymous functions, but what methods? Let's see:

```rb
Array.instance_method(:each)
#=> #<UnboundMethod: Array#each>

Array.method(:new)
#=> #<Method: Array.new>
```

`UnboundMethod` and `Method` are different than Procs and lambda because they have an `owner` (i.e. whatever class defined it), a `name` (i.e. the name used to define the method), and potentially a `super_method`

Lastly, `UnboundMethod` cannot be `call`ed. It has no context to evaluate within, it isn't bound to a context. It's just a floating method attached to the `Array` class waiting for an instance it can be evaluated on.
`Method` on the other hand is bound to `Array` (or more exactly, `Array` is the method's `receiver`) thus it can be `call`ed

## Enumerable

[Enumerable](https://ruby-doc.org/core-2.5.1/Enumerable.html) is one of the most important classes in Ruby. You should spend some time and try to become familiar with it.
Any class that implements the method `#each` can mixin in `Enumerable` to get all of these methods:

```rb
class Mailbox
  include Enumerable

  def initialize(messages)
    @messages = messages
  end

  def each
    @messages.each(&Proc.new)
  end
end

mailbox = Mailbox.new([1,2,3])
mailbox.count
#=> 3
```

Enumerable is used everywhere in Ruby. A lot of Ruby code tends to look like functional programming:

```rb
arr = [1,2,3]
arr.map { |n| n + 1 }.reduce(0) { |sum, n| n + sum }
#=> 9
```

## Enumerators

Enumerators are like Enumerables on a stick. They include Enumerable and implement their `each` method with the block that you give them

Here's an example:

```rb
def get_more_messages
  100.times.map { rand }
end

messages = Enumerator.new do |yielder|
  loop do
    msgs = get_more_messages
    msgs.each { |msg| yielder.yield msg }
  end
end

to_process = messages.take(20)
```

Enumerators are implemented with Fibers (which we'll learn about much later). Fibers are a context-switching mechanism implemented in Ruby. You can kind of think of them like threads that we control.
The reason I mention them here is that this Fiber-based implementation prevents our code above from looping forever in the `loop do`.
That is, even though our Enumerator has an infinite loop, Ruby is only context-switching to that code on-demand.

We don't need to write Enumerators explicitly all the time, but I wanted to call out this example because it less us decouple what and how we iterate a collection from when we iterate that collection as well as how we collect it.
For example, we can pass the Enumerator around and process it whenever and however we want to

Lastly I want to mention one more pattern with enumerators. Sometimes it is useful to pass a block and to a method and have that block enumerated and other times it is useful to have that method return an Enumerator so we can
decide how we want to process the results later. For example:

```rb
things.each { |thing| thing.add(1) }

# vs.

things.each.select { |thing| thing.even? }
```

Thus, when you write an enumerable method (like `#each`) you should make sure to support each case:

```rb
# Bad - doesn't support second use case
def each
  @things.each { |thing| yield thing }
end

# Bad - doesn't support first use case
def each
  @things.each
end

# Good - supports both use cases
def each
  if block_given?
    @things.each { |thing| yield thing }
  else
    @things.each
  end
end

# My preference - see `#to_enum`'s docs
def each
  return to_enum(:each) unless block_given?
  @things.each { |thing| yield thing }
end
```

This example is a bit contrived, but this pattern works well in more complex cases too

## Lazy Enumerators

Now we're getting to the good stuff. One of the huge benefits of abstracting things into Enumerators is that we can redefine the flow of our pipelines without very little effort

Let's use some examples. Let's say we have a processing pipeline using the same Enumerator we used above:

```rb
def get_more_messages
  100.times.map { rand }
end

messages = Enumerator.new do |yielder|
  loop do
    msgs = get_more_messages
    msgs.each { |msg| yielder.yield msg }
  end
end

messages
  .map { |msg| msg + 1 }
  .each { |msg| save_msg(msg) }
```

This code unfortunately will loop forever and save no messages. Why? Because each stage of the pipeline is evaluated eagerly. What do I mean by that? The `.map { |msg| msg + 1 }` part of the pipeline
will process the entire output of `messages` and return its entire output. Well... `messages` is an infinite sequence so it never returns and `save_msg` is never called!

I know what you're thinking: let's change the code to something like:

```rb
messages.each do |msg|
  new_msg = msg + 1
  save_msg(new_msg)
end
```

That's fine and would work, but I have some issues with it. Let's say that we then want to batch messages in groups of 20. Now it's starting to get complex:

```rb
buffer = []
messages.each do |msg|
  new_msg = msg + 1
  buffer << new_msg
  if buffer.size >= 20
    save_msg_batch(buffer)
    buffer = []
  end
end
```

The batching logic is all intertwined with the increment logic. Yuk! It's really easy to miss something here. Imagine if this had a few more concerns.

Luckily lazy enumeration allows us to break this apart to keep things simple and decoupled:

```rb
messages.lazy
  .map { |msg| msg + 1 }
  .each_slice(20) { |batch| save_msg_batch(batch) }
```

What happened here? Calling `lazy` turned our `Enumerator` into an `Enumerator::Lazy`. The `map` call then returned another `Enumerator::Lazy`. Finally `each_slice` caused the lazy Enumerators to
finally evaluate (i.e. on-demand). When a pipeline of lazy Enumerators evaluate then each element goes one-by-one through each stage of the pipeline. Personally I find this much more readable
than our other solution. Each stage of the processing is concerned with one independent part of the processing and decoupled from everything else that is going on. There are no temporary variables
and there's no tricky scope to work out. We can easily reorder and insert new processing stages to our pipeline

That's not what I like most though. What I like most is that we can very easily control how things flow through our pipeline. We can choose whether we want to process things stage-by-stage or element-by-element
and we can mix and match however we see fit. If one becomes an issue, it's easy to reconfigure things to work a little differently

## Reflection

One of Ruby's greatest strengths and one of the things that I think makes Ruby fun is its high level of reflection through various helpers

Here is a smattering of them to whet your palette:

```rb
class Person
  def self.create_bob
    new("bob", 40)
  end

  def initialize(name, age)
    @name = name
    @age = age
  end

  def say_hi
    "hello from #{@name}"
  end
end

Person.ancestors
#=> [Person, Object, Kernel, BasicObject]

Person.superclass
#=> Object

Person.singleton_methods
#=> [:create_bob]

Person.instance_method(:initialize).parameters
#=> [[:req, :name], [:req, :age]]

bob = Person.create_bob
bob.instance_variables
#=> [:@name, :@age]

bob.instance_variable_get(:@name)
#=> "bob"
```

## Metaprogramming

In a compiled language metaprogramming generally happens at compile time possibly through some sort of macro expansion during a preprocessing step.
You can think of it as a kind of text expansion on your code - your code in a some file is just a big text string and the compiler knows how to "expand" certain symbols with other strings that you define.
Thus you "variablize" or DRY up your code in a sense.

Ruby's metaprogramming works a little differently. We don't need a preprocessor for one. Instead metaprogramming in Ruby tends to happen when the script is loaded.
Recall that a Ruby program is just a script that is executed line-by-line.
Ruby gives us all sorts of neat tools like reflection and open classes, so why not also give us tools that allow us to programmatically define and modify code when a class is first loaded?

For example, we could write this class two ways:

```rb
# Approach #1
class HTTPClient < Base
  def initialize
    @client = client
  end

  def get(url)
    request(@client, :get, url)
  end

  def post(url)
    request(@client, :post, url)
  end

  def delete(url)
    request(@client, :delete, url)
  end

  def put(url)
    request(@client, :put, url)
  end

  def patch(url)
    request(@client, :patch, url)
  end
end

# Approach #2
class HTTPClient < Base
  def initialize
    @client = client
  end

  [:get, :post, :delete, :put, :patch].each do |sym|
    define_method(sym) do |url|
      request(@client, sym, url)
    end
  end
end
```

There's not a huge savings using `define_method` but when thing it definitely screams is "these things are the same". If we had a very long list of similar methods then it could save the reader the effort of having to check every
implementation. Things get really interesting when we're start considering dynamic situations.

Metaprogramming can be both a blessing and a curse, and there's a lot of material out there with regards to metaprogramming in Ruby so I won't go much further with it.
I would recommend reading up on the methods supported by Class, Module, Object, Method, and UnboundMethod if metaprogramming interests you

## IO

If you're familiar with IO in other languages then IO in Ruby should be pretty straight-forward. Check out the examples in the docs for `IO` and `File`. If possible try to use a block when doing things like opening Files as it will ensure you close the file afterwards

## Concurrency

A Ruby process on MRI cannot run parallely on mutliple cores due to the GIL which only allows Ruby to process a single VM instruction at a time.
This means that in order to get parallel CPU time you need to use multiple processes.
Ruby does however support concurrent IO (i.e. you can listen for multiple outstanding requests at a time).
If you are familiar with `select` and similar commands then these functions (which reside in `IO`) should feel familiar

Ruby does support green threads and will context switch between them. The `Queue` data structure is thread-safe and there are `ConditionVariable` and `Mutex` classes you can take advantage of.
One simple way to use threads to get parallel execution is either via IO or to have threads that spawn subprocesses (each thread waits on a subprocess).
You should assume by default that your code in not thread safe, and many gems are not thread safe.
With a small bit of work using the tools listed above you can create a simple synchronized implementation of any class though using something like `MonitorMixin` (see docs)

Related to Threads are Fibers which are like threads but which don't have automatic context switching. You must explictly switch between Fibers. These aren't commonly used but they are very powerful

Multi-processing works similar to other languages. Take a look at `Open3.popen3` for an excellent "do-it-all" helper. Otherwise you have access to `fork` and `exec` as well a bunch of other helpers.
See [this overview](https://medium.com/zendesk-engineering/running-a-child-process-in-ruby-properly-febd0a2b6ec8)

If you still don't want to use multiple processes, want good concurrency primitives (actors, channels, thread safety, pooling), and are okay with the caveats about the GIL then take a look at [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby)

## Gems

Gems are Ruby's version of packages. Gems can be public or private. Some projects break things into gems just to break things up. Recall that Ruby uses global namespacing though, so unless you have tests then you are just moving code around and not drawing boundaries.

In terms of writing gems, take a peek at [the guides on RubyGems.org](https://guides.rubygems.org/). Make one the hard way once. After that use Bundler's `bundle gem my_gem` helper to bootstrap your gem. There are a bunch of helpful tasks included in the generated Rakefile

## Debugging

Ruby has a good number of debugging tools. Here are some of my favorites:

* `byebug` is a step debugger inspired by `gdb`. When your code hits a `byebug` it starts a REPL which you can use to step through the stacktrace

```rb
def my_method(arg)
  byebug
  arg + 1
end
```

* `pry` is a code inspection tool. Our setup integrates pry and byebug. Similar to byebug, when your code hits a `binding.pry` you can do things like `show-source my_method` and navigate to any object's context

* When in doubt, `Method#source_location` is great. Don't know where some Rails thing is defined? Call `method(:my_weird_method).source_location` and open up that gem

* `bundle info gem_name` shows you stuff like where a gem is installed in your system so you can hack on it

## Idioms, Patterns, and Conventions

* Caching something? The following pattern is used often:

```rb
def method_to_cache
  @method_to_cache ||= cold_cache_hit_code_here
end
```

* `method_missing` can be used for dynamic dispatch and is a catch-all that can be used to build a proxy:

```rb
class Proxy
  def initialize(delegate, handler)
    @delegate = delegate
    @handler = handler
  end

  def respond_to_missing?(sym, incl_private = false)
    @delegate.respond_to?(sym, incl_private)
  end

  def method_missing(sym, *args, &block)
    @handler.call(@delegate, sym, *args, &block)
  end
end
```

* `class << self` is a bit of an arcane syntax but is a useful way to ensure that class methods are grouped and that they respect any access modifiers:

```rb
# Bad
class MyClass
  def self.some_public_method
  end

  def some_instance_method
  end

  private

  def self.not_actually_private
  end
end

# Good
class MyClass
  class << self
    def some_public_method
    end

    private

    def actually_private
    end
  end

  def some_instance_method
  end
end
```


* Want to create a mixin that adds instance and class methods? Use this common idiom:

```rb
module Mixin
  def self.included(base)
    base.extend(ClassMethods)
  end

  module ClassMethods
    def some_new_class_method
    end
  end

  def some_new_instance_method
  end
end
```

## Code Smells and Anti-Patterns

* A lot of Java-based design patterns aren't necessary in Ruby due to Ruby's powerful reflective and dynamic helpers
* Breaking a monolithic class into a lot of Mixins (ala [ActiveRecord::Base](https://github.com/rails/rails/blob/94b5cd3a20edadd6f6b8cf0bdf1a4d4919df86cb/activerecord/lib/active_record/base.rb#L277)) generally makes it more complex rather than simpler

## Other Resources

* [Ruby Performance Optimization](https://www.amazon.com/Ruby-Performance-Optimization-Why-Slow/dp/1680500694/ref=sr_1_10_sspa?s=books&ie=UTF8&qid=1544718431&sr=1-10-spons&keywords=ruby&psc=1)
