---
title: Reduce
author: Patrick Struthers
---

# Reduce in General
Reduce is the process of transforming some collection into an accumulator by applying a reducer function to each element.  Unlike Map, Reduce remembers the result of its previous call and uses it as the accumulator value for the next call.

The final result is the last accumulator value returned by calling the reducer on the last element.

## Reduce in English

My current height is the result (final accumulator) of:
- My inital height (initlal accumulator)
- My preceding height (accumulator) added to (reducer) changes in my height (collection)

## Aliases for Reduce
Reduce is known by different names in different languages:

- fold: Kotlin, F#
- foldl/foldr: Erlang, Racket 
---
# Enum.reduce() in Elixir
```
reduce(Enumerable.t(), (element(), acc() -> acc())) :: acc()
```
- first argument: An enumerable composed of any type.
- second argument: A function that takes any type as its first argument and an accumulator as its second argument and returns an accumulator. 
- return value: The final accumulator value 

You can also pass in an optional starting accumulator as the second argument.

``` elixir
# Implements Enum.sum
# Evaluates to 6
Enum.reduce([1,2,3], fn elem, acc -> elem + acc end)

# Tracks the position of a point as we apply changes to its x and y
# coordiantes
# Evaluates to {1,1}
Enum.reduce([{1,1},{1,0}, {0,-1}], {0,0}, fn {dx, dy}, {x, y} -> 
    {x + dx, y + dy}
end)

# Finds the seventh fibonacci number (1 indexed)
# We can use the accumulator to 'remember' more than one value
# Evaluates to 8
Enum.reduce(1..5, {1,0} fn 
    _, {last, lastlast} -> {last + lastlast, last}
end)
|> elem(0)
```
---
# Reduce with Comprehensions
We can use the :reduce option in list comprehensions to change their behavior from resembling map to resembling reduce.
``` elixir 
# Will sum all points whose product is even. 
for x <- 1..100, y <-1..100, reduce: %{} do
    acc -> 
        if rem(x * y, 2) == 0 do
            Map.put({x,y}, x + y)
        else
            acc
        end
end
```
---
# Reduce While
reduce_while works like reduce except it expects the reducer to return {:cont, acc} to continue the reduction or {:halt, acc} to stop reducing and return the accumulator. 

``` elixir
# sum numbers until the sum is greater than or equal to 100
Enum.reduce_while(1..100, 0, fn 
    _elem, acc when acc >= 100 -> {:halt, acc}
    elem, acc -> {:cont, acc + elem} 
end)
```

reduce_while is useful in cases where you cannot exhaustively iterate and must recognize a stopping point based on the state of your accumulator.

``` elixir
defmodule Prime do
  def divisor_found?(number, divisors) do
    Enum.reduce_while(divisors, false, fn div, _ ->
      if rem(number, div) == 0 do
        {:halt, true}
      else
        {:cont, false}
      end
    end)
  end

  def first_primes(prime_count) do
    Stream.iterate(2, &(&1 + 1))
    |> Enum.reduce_while([2], fn
      _elem, divisors when length(divisors) == prime_count ->
        {:halt, divisors}

      elem, divisors ->
        if divisor_found?(elem, divisors) do
          {:cont,  divisors}
        else
          {:cont, [elem | divisors]}
        end
    end)
  end
end

```
---
# Map Reduce
map_reduce will apply a map transformation on an enumerable and a reduction at the same time.
The reducer function must return a tuple {mapped_value, accumulator_value} where mapped_value will be used to build the mapped enumerable, and the accumulator will be used to build the final accumulator. 

``` elixir
defmodule Puppy do
  defstruct [:growth_rate, weight: 1] 
  def new(growth_rate), do: %__MODULE__{growth_rate: growth_rate}
  def feed(%__MODULE__{growth_rate: growth_rate, weight: weight} = puppy) do
    Map.update!(puppy, :weight, & &1 + growth_rate)
  end

  defmodule Farm do
    defstruct [:puppies, :total_weight]
    def new(puppies), do: %__MODULE__{puppies: puppies}
    def feed_farm(%__MODULE__{puppies: puppies}) do
      #the value of the accumulator will have the weigh tof the puppies before they are fed!
      {updated_puppy_farm, total_weight} =  Enum.map_reduce(puppies, 0, fn puppy, total_weight -> {Puppy.feed(puppy), total_weight + puppy.weight} end)
      %__MODULE__{puppies: updated_puppy_farm, total_weight: total_weight}
    end
  end
end

starting_farm = Puppy.Farm.new([Puppy.new(1), Puppy.new(4), Puppy.new(2)])
starting_farm
|> Puppy.Farm.feed_farm()
```
--- 

# Power of Reduce
Reduce is the most powerful general purpose abstraction availabe in Elixir. 
- Reduce is used to build almost all other function in the Enum module.
- GenServer is essentially a reduction that preserves the value of the accumlator between reductions.
- In the most general case any state can be represented as a reduction where:
    - The initial state is the inital accumulator.
    - The Enumerable represents instructions for transforming the state.
    - The reducer applies instructions from the Enumerable to create a new state.

## Some Pseudocode 

```
#Playing a game given a list of moves
Enum.reduce ([player_moves], game_state,(move, game_state) -> updated_game_state))

#Cracking a password hash given a list of guesses
Enum.reduce_while( [password_guesses], password_hash,(guess, password_hash) -> if hash(password) == password_hash, do: {:halt, guess}, else: {:cont, passsword_hash})

#Solving a rubiks cube with a stream of twists 
Enum.reduce_while([twists], cube, (twist, cube) -> 
if win?(twist(cube)) do 
    {:halt, twist(cube)}
else
    {:cont, twist(cube)}
)
```

