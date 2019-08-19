# Pattern Matching in Ruby

I use Ruby on daily basis, but as many people I've looked into Elixir.
Two concepts from Elixir really stuck in my head:

  1. tuples
  2. pattern matching

## Tuples
This one is easy if you don't mind using basic data structures instead of objects.
Basically instead of returning a result object from your services you can just use an array like so:

```ruby
[:some_status, value]

# ex:
[:success, user]

[:validation_error, user]

[:server_error, error_message]
```

If you stick to this convention you can end up with a simple and light weight way of passing data and metadata
from your services.
Sticking to one convention can allow you to craft neat ways of handling different outcomes.


## Implementing Pattern Matching

Pattern matchingesque behaviour can be achieved in a few ways:
1. using [dry-matcher](https://dry-rb.org/gems/dry-matcher/)
2. using case with matcher objects implementing `===` operator
3. using plain case
4. using guard clauses

The first two points are more interesting so i'll just focus on them.

## Dry-matcher

I was aware of [dry-matcher](https://dry-rb.org/gems/dry-matcher/) for some time,
but was a bit hesitant (wrongly) to actually dive into it and take a closer look how it works.

```ruby
success_matcher = Dry::Matcher::Case.new(
  match:   ->(tuple) { tuple.first == :success },
  resolve: ->(tuple) { tuple.last }
)

error_matcher = Dry::Matcher::Case.new(
  match:   ->(tuple) { tuple.first == :error },
  resolve: ->(_) { "RuntimeError" }
)

matcher = Dry::Matcher.new(success: success_matcher, error: error_matcher)

res = matcher.call(tuple) do |m|
  m.one { |v| v }
  m.two { |_| "RuntimeError" }
end
```

Basically we need to create different cases and then buid a matcher object with them and then just call the matcher object.

While this may look like it requires quite a lot code it is easily extendable and is backed by a maintained and stable gem from
a great initiative in Ruby world.


## Matcher objects implementing `===` operator

I was never a fan of `case`/`switch` statements, but with this i came to like them a bit.

In Ruby `case` uses `===` operator to match data against conditions. Knowing this we can use objects
implementing this operator and end up with a neat solution.

For sake of this post i'll just focus on using lambdas since in Ruby lambdas respond to `===`.

Here we have a few options:

1. using plain lambdas in `case` statement

  ```ruby
  res = case tuple
        when ->(t) { t[0] == :success } then tuple.last
        when ->(t) { t[0] == :error }   then "RuntimeError"
        end
  ```

2. define matchers and then use them in `case` statement

  ```ruby
  success       = ->(t) { t[0] == :success }
  runtime_error = ->(t) { t[0] == :error }

  res = case tuple
        when success       then tuple.last
        when runtime_error then "RuntimeError"
        end
  ```

  Or if you feel a bit more adventurous you can unpack the tuple in the lambda argument

  ```ruby
  success       = ->((v, _)) { v == :success }
  runtime_error = ->((v, _)) { v == :error }

  res = case tuple
        when success       then tuple.last
        when runtime_error then "RuntimeError"
        end
```


# The end

Hope you found this useful or at least interesting, feel free to let me know by sending me an email.
