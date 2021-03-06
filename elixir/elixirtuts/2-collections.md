# List functions
Implemented as linked list, so excellent for looping
For large datasets, some operations will be slow (like access by index)

Most of list is covered in basics, functions, recursion chapters
```elixir
# Folding, Accumulating, Reducing whatever you may call this
List.foldl([1, 2, 3], "", fn value, acc -> "#{value}(#{acc})" end)
# -> "3(2(1()))"

List.foldr([1, 2, 3], "", fn value, acc -> "#{value}(#{acc})" end)
# -> "1(2(3()))"
```

> Note: Replace is a costly operation since lists are linked lists
```elixir
List.replace_at([10, 20, 30, 40, 40, 60], 4, 50)
# -> [10, 20, 30, 40, 50, 60]
```
List.keyfind only gives the first occurence
```elixir
List.keyfind([{:rane, 34, "python"}, {:amitu, 36, "rust"}, {:deepak, 35, "python"}], :rane, 0)
# -> {:rane, 34, "python"}

List.keyfind([{:rane, 34, "python"}, {:amitu, 36, "rust"}, {:deepak, 35, "python"}], "python", 2)
# -> {:rane, 34, "python"}
```
Others: `List.keyreplace`, `List.keydelete`

# Data structures:

    - Maps: Fast key-value lookup (most used)
    - Keyword: Non-unique keys, Ordered (used as named tuples in python, more structured than maps)
    - Struct: Fixed number of fields

# Keywords:
- Since they are ordered and lookups work only for atoms, they are often used as context to functions

```elixir
defmodule Canvas do
    @defaults [ fg: "black", bg: "white", font: "Merriweather" ]

    def draw_text(text, options \\ []) do
        options = Keyword.merge(@defaults, options)

        IO.puts "Drawing text #{inspect(text)}"
        IO.puts "Foreground: #{options[:fg]}"
        IO.puts "Background: #{Keyword.get(options, :bg)}"
        IO.puts "Font: #{Keyword.get(options, :font)}"
        IO.puts "Pattern: #{Keyword.get(options, :pattern, "solid")}"
        IO.puts "Style: #{inspect Keyword.get_values(options, :style)}"
    end
end
Canvas.draw_text("hello", fg: "red", style: "italic", style: "bold")
```

# Map:
```elixir
map = %{ name: "Dave", likes: "Programming", where: "Dallas" }
Map.keys map          # -> [:likes, :name, :where]
Map.values map        # -> ["Programming", "Dave", "Dallas"]
map[:name]            # -> "Dave"
map.name              # -> "Dave"
map1 = Map.drop map, [:where, :likes]     # -> %{name: "Dave"}
map2 = Map.put map, :also_likes, "Ruby"
# -> %{also_likes: "Ruby", likes: "Programming", name: "Dave", where: "Dallas"}

Map.keys map2         # -> [:also_likes, :likes, :name, :where]
Map.has_key? map1, :where     # -> false
{ value, updated_map } = Map.pop map2, :also_likes
# -> {"Ruby", %{likes: "Programming", name: "Dave", where: "Dallas"}}
Map.equal? map, updated_map   # -> true


person = %{ name: "Dave", height: 1.88 }
%{ name: a_name } = person   # ->  assigns a_name to value of person.name
%{ something: a_name } = person   # -> MatchError

%{ name: _, height: _ } = person  # ->  matches, does not give MatchError
```
> NOTE: You cannot BIND keys, only values can be bound via pattern matching, this also happens to be a good practice in other languages

# List Comprehension
```elixir
people = [
    %{ name: "abhas", department: "aero", city: "kota"  },
    %{ name: "shyam", department: "aero", city: "patna" },
    %{ name: "hemant", department: "chem", city: "vizag" },
    %{ name: "rahul", department: "chem", city: "indore" },
    %{ name: "akhil", department:  "chem", city: "kota" },
]
```

```elixir
aero_guys = for person = %{ department: dept} <- people, dept == "aero", do: person
```

- The generator clause bind each map in the list to person and binds the department to dept
- the filter selector only those dept where dept value is "aero", do returns the matches

```elixir
item = List.first people
# -> item = %{city: "kota", department: "aero", name: "abhas"}

# Matching key value
%{ :city => v } = item  # -> v = "kota"

for key <- [:city, :name] do
  %{ ^key => value } = item
  value
  end
# -> ["kota", "abhas"]
```

Updates on Map:
```elixir
new_map = %{ old_map | key1 => value1,  key2 = value2, ...}
```
> Note: This doesn't support adding new keys, can only update old keys

# Struct:

- They are like typed maps, keys cannot be added later and has default values and behaviours

```elixir
defmodule Attendee do

    defstruct name: "", paid: false, over_18: true

    def may_attend_after_party(attendee = %Attendee{}) do
        attendee.paid && attendee.over_18
    end

    def print_vip_badge(%Attendee{name: name}) when name !=
        IO.puts "Very cheap badge for #{name}"
    end

    def print_vip_badge(%Attendee{}) do
        raise "missing name for badge"
    end

end
```
```elixir
s1 = %Subscriber{}
# ->  %Subscriber{name: "", over_18: true, paid: false}

s2 = %Subscriber{ name: "Dave" }
# ->  %Subscriber{name: "Dave", over_18: true, paid: false}

s3 = %Subscriber{ name: "Mary", paid: true }
# -> %Subscriber{name: "Mary", over_18: true, paid: true}
s3.name       #-> "Mary
```

- Updating
```elixir
s4 = %Subscriber{ s3 | name: "Marie"}"
# -> %Subscriber{name: "Marie", over_18: true, paid: true}
Attendee.may_attend_after_party(s4)
```

# Nested Structs

```elixir
defmodule Customer do
    defstruct name: "", company: ""
end

defmodule BugReport do
    defstruct owner: %Customer{}, details: "", severity: 1
end
```
```elixir
report = %BugReport{owner: %Customer{name: "Dave", company: "Pragmatic"}, details: "broken" }
```
- Accessing attributes
```elixir
report.owner.company
```

- Modifying a nested attribute is PAINFUL
```elixir
report = %BugReport{ report | owner: %Customer{ report.owner | company: "PragProg" }}
```

- Elixir gives a macro to do the above
> NOTE: Macros generate the same erlang code in elixir during compile time

```elixir
put_in(report.owner.company, "PragProg")
# OR
put_in(report[:owner][:company], "PragProg")

update_in(report.owner.name, &("Mr." <>  &1))
# -> runs a lambda on the variable
# OR
update_in(report[:owner][:name], &("Mr." <> &1))
```

put_in and update_in also offer non-macro runtime code to access fields in runtime
```elixir
get_in(report, [:owner, :name])
put_in(report, [:owner, :name, :first], "Dev")
```

# Access module (awesome module to put/update)

```elixir
cast = [
    %{
        character: "Buttercup",
        actor: %{
            first: "Robin",
            last: "Wright"
        },
        role: "princess"
    },
    %{
        character: "Westley",
        actor: %{
            first: "Cary",
            last: "Elwes"
        },
        role: "farm boy"
    }
]
```

```elixir
get_in(cast, [Access.all(), :character])
# -> ["Buttercup", "Westley"]
> get_in(cast, [Access.at(1), :role])
# -> "farm boy"

get_and_update_in(cast, [Access.all(), :actor, :last], fn (val) -> {val, String.upcase(val)} end)
# -> Makes last name all caps

get_in(cast, [Access.all(), :actor, Access.elem(1)])
# -> ["Wright", "Elwes"]

IO.inspect get_and_update_in(cast, [Access.key(:buttercup), :role], fn (val) -> {val, "Queen"} end)
# -> Changes buttercup's role to Queen

{popped_value, rest_} = Access.pop(%{name: "Elixir", creator: "Valim", type: "Superhuman"}, :name)
# -> popped_value = "Elixir", rest = %{creator: "Valim", type: "Superhuman"}
```

# Sets (implemented as MapSet)

```elixir
set1 = 1..5 |> Enum.into(MapSet.new)
set2 = 3..8 |> Enum.into(MapSet.new)
MapSet.member? set1, 3    # -> true
MapSet.union set1, set2   # -> MapSet<[1, 2, 3, 4, 5, 6, 7, 8]>
```

- MapSet.intersection, MapSet.subset etc.

> Note: Dont use structs a lot, its more of a OOP concept and can/should be avoided in Elixir


# Functionality Collection

- Enum implements tons of functionality around list, and work on anyone who implements a "Eumerable" Protocol

- Enum.to_list, Enum.map, Enum.concat, Enum.at, Enum.filter, Enum.reject, Enum.sort, Enum.take, Enum.take_every, Enum.take_while etc.

```elixir
l = 1..10
Enum.take l, 3
# -> [1, 2, 3]

Enum.take_every l, 3
# -> [1, 3, 5]     Take every 3rd element

Enum.take_while [1,2,3,4,5,6,2,1], &(&1 < 4)
# -> [1, 2, 3]     Take till a condition matches

Enum.all? l, &(&1 < 4)
# -> false

Enum.any? l, &(&1 < 4)
# -> true

Enum.zip [:a, :b, :c], [1, 2, 3]
# -> [a: 1, b: 2, c: 3]

Enum.reduce 1..100, &(&1 + &2)
# -> 5050

Enum.reduce ["Lets", "find", "the", "longest", "word", "here"], fn word, longest ->
    if String.length longest >= String.length word do
        longest
    else
        word
```

- Deal a hand of cards
```elixir
import Enum
deck = for rank <- '23456789TJKQA', suit <- 'CDHS', do [suit, rank]
hands = deck |> shuffle |> chunk(13)
```

# Streams

- Streams are list implemented like yeild in python
- Great when data is too large to fit in memory or arriving in due time
- Best fit for data from IO operations
- Use in scenarios where all data in a Enumerable is not required for procession at one time
- Bad when data is small and can be processed in a List easily without major memory overhead


To find the longest word in dictionary, without streams
```elixir
IO.puts File.read!("/usr/share/dict/words")
# -> reads all file in one go
    |> String.split
    |> Enum.max_by(&String.length/1)

# with Streams
IO.puts File.open!("/usr/share/dict/words")         # -> creates a file handle (doesn't read yet)
    |> IO.stream(:line)                             # -> creates a stream, little slow, but good on memory
    |> Enum.max_by(&String.length/1)

# Better, directly use file stream to read shortcut
IO.puts File.stream!("/usr/share/dict/words") |> Enum.max_by(&String.length/1)
```

- Behaves exactly like Enumerables
```elixir
[1,2,3,4]           # -> This can be a stream or a Enum. Larger the better, or IO stream
    |> Stream.map(&(&1*&1))
    |> Stream.map(&(&1+1))
    |> Stream.filter(fn x -> rem(x,2) == 1 end)
    |> Enum.to_list
```

Streams works best for large dataset. e.g.

```elixir
Enum.map(1..10_000_000, &(&1+1)) |> Enum.take(5)      # -> Take more than 10 seconds to run
Stream.map(1..10_000_000, &(&1+1)) |> Enum.take(5)    # -> Runs instantly
```

Stream methods:
- Cycle
```elixir
Stream.cycle(~w{ green white }) |>        # -> generator to create a infinite list of ["green", "white", "green", "white", "green"....]
    Stream.zip(1..5) |>                   # -> zips it to list [1,2,3,4,5] till the stream is over
    Enum.map(fn {class, value} ->
        "<tr class='#{class}'><td>#{value}</td></tr>\n" end) |>
    IO.puts
```
Output:
```html
<tr class="green"><td>1</td></tr>
<tr class="white"><td>2</td></tr>
<tr class="green"><td>3</td></tr>
<tr class="white"><td>4</td></tr>
<tr class="green"><td>5</td></tr>
```

- Stream.repeatedly
Takes a function and invokes it each time a new value is wanted.
```elixir
Stream.repeatedly(&:random.uniform/0) |> Enum.take(3)
# -> Generates 3 random numbers
```
- Stream.unfold
Uses functions to generate a list, functions takes 2 values to generate the 3rd and so on (like fibonacci)
> fn state -> { stream_value, new_state } end

```elixir
Stream.unfold({0,1}, fn {f1,f2} -> {f1, {f2, f1+f2}} end) |> Enum.take(8)
```

- Stream.resource
To create stream from a resource, e.g. a file
```elixir
Stream.resource(
    # create and return a resource handle
    fn -> File.open!("/tmp/t") end,

    # take the resource object and process chunks one by one
    # return a Enum and the resource handle
    fn file -> case IO.read(file, :line) do
        data when is_binary(data) -> {
            [String.to_integer(String.trim(data)) + 200], file
        }
        _ -> {:halt, file}
        end
    end,

    # close the resource
    fn file -> File.close(file) end
) |> Enum.take(5)
```

To create a timer, which counts down from 10 seconds to 4 seconds using a stream:
```elixir
Stream.resource(
    fn -> 10 end,                 # start with 10

    fn seconds -> case seconds do
        data when
        0 -> {:halt, 0}           # stop at 0
        _ ->
            Process.sleep(1000)   # sleep for a second
            { [Integer.to_string(seconds)], seconds - 1 }
        end
    end,

    # close the resource
    fn _ -> nil end
) |> Enum.take(4)
```
Collectable Protocol:
Helps you create a collection by inserting elements
```elixir
Enum.into 4..7, [1, 2, 3]   # insert range 4..7 to a collection
# -> [1, 2, 3, 4, 5, 6, 7]

# Streams also implement collectable, lazily copy one stream to another
Enum.into IO.stream(:stdio, :line), IO.stream(:stdio, :line)
```

Comprehensions:
```elixir
# result = for <generator or filter> [, into: txfn ], do: expression
# into txfn is a tranformation function applied to output of expression (not exactly, but similar. :into takes values that implement Collectable protocol)

for x <- [ 1, 2, 3, 4, 5 ], do: x * x
# -> [1, 4, 9, 16, 25]

for {key, val} <- %{"a" => 1, "b" => 2}, do: {key, val * val}
# -> [{"a", 1}, {"b", 4}]

# If you want a Map instead of a list of tuples, use into
for {key, val} <- %{"a" => 1, "b" => 2}, into: Map.new, do: {key, val * val}
# -> %{"a" => 1, "b" => 4}

# filtering
for x <- 1..10, rem(x, 2) == 0, do: x*x
# -> [4, 16, 36, 64, 100]

# Multiple loop (i x j iterations)
for i <- 1..3, j <- 1..3, do: {i, j}
# -> [{1, 1}, {1, 2}, {1, 3}, {2, 1}, {2, 2}, {2, 3}, {3, 1}, {3, 2}, {3, 3}]

# reusing first loop vars in internal loops
for {min, max} <- min_max, x <- min..max, do: x
# -> [1, 2, 3, 4, 5, 6]

# multiple filter criterias
for x <- 0..5, y <- 0..5, x < y, rem(x, 2) == 0, do: {x, y}
# [{0, 1}, {0, 2}, {0, 3}, {0, 4}, {0, 5}, {2, 3}, {2, 4}, {2, 5}, {4, 5}]

# iterate list (represented as string)
for c <- 'hello', do: c + 1
# -> 'ifmmp'

# Convert BitString to List (weird syntax)
for <<c <- "hello">>, do: c
# -> 'hello'
for <<c <- "hello">>, do:  <<c>>
# ["h", "e", "l", "l", "o"]
```

> Note: Variables used in comprehension are scoped inside the comprehension and do not affect global scope
