---
title: A shallow dive into Elixir's Enum.frequencies
description: We look at what Enum.frequencies is doing
date: 2025-06-29
tags: elixir
---

A string is called an anagram of another string if they have the same letters but are arranged differently. For example, “hello” is an anagram of “elloh”. Can you write a program to check this?[^1]

[^1]: This problem comes from Leetcode 242: Valid Anagram.

Here's an elegant solution in Elixir:

```elixir
def is_anagram(s, t) do
	count(s) == count(t)
end

defp count(s) do
	s
	|> String.to_charlist()
	|> Enum.frequencies()
end
```

This is quite self-explantory apart from `Enum.frequencies()`. What is it actually doing? Looking inside the source code:

```elixir
@doc since: "1.10.0"
@spec frequencies(t) :: map
def frequencies(enumerable) do
  reduce(enumerable, %{}, fn key, acc ->
    case acc do
      %{^key => value} -> %{acc | key => value + 1}
      %{} -> Map.put(acc, key, 1)
    end
  end)
end
```

`^` is called [the pin operator](https://hexdocs.pm/elixir/1.18.1/pattern-matching.html#the-pin-operator). It matches against a variable’s existing value

```elixir
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
```

`%{}` is a key-value map in Elixir. So in this case

```elixir
case acc do
  %{^key => value} -> %{acc | key => value + 1}
  %{} -> Map.put(acc, key, 1)
end
```

is saying: If the key already has a value in the map, update the map with

`key => value + 1` .

Otherwise, update the map with `key => 1`.
