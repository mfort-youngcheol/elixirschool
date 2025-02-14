%{
  version: "1.1.0",
  title: "কম্প্রিহেনশান",
  excerpt: """
  লিস্ট কম্প্রিহেনশান হল সিন্ট্যাক্টিক সুগার যা এলিক্সির এনুমেরাবলে লুপিং করতে সহায়তা করে। এই অধ্যায়ে আমরা দেখব কি করে লিস্ট কম্প্রিহেনশানকে ব্যবহার করতে হয় ইটারেশান ও জেনারেশানের জন্য।
  """
}
---

## বেসিক

`Enum` এবং `Stream` এর মধ্যে লুপিং করতে প্রায়েই কম্প্রিহেনশান ব্যবহৃত হয় কারণ এটি লিখিত কোডকে সংক্ষিপ্ত করে। একটি সহজ কম্প্রিহেনশান দেখা যাক:

```elixir
iex> list = [1, 2, 3, 4, 5]
iex> for x <- list, do: x*x
[1, 4, 9, 16, 25]
```

প্রথমেই যা আমাদের চোখে পরে তা হল `for` এবং তারপর জেনারেটর। 
জেনারেটর কি জিনিস?
জেনারেটর হল `x <- [1, 2, 3, 4]` এক্সপ্রেশানগুলি যা লিস্ট কম্প্রিহেনশানের সময় দেখা যায়। এদের কাজ হল পরের ভ্যালু প্রদান করা।

লিস্ট কম্প্রিহেনশান শুধুমাত্র লিস্টের মধ্যে সীমাবদ্ধ নয়। এরা যে কোন এনুমেরেবলের সাথেই কাজ করতে পারে।

```elixir
# Keyword Lists
iex> for {_key, val} <- [one: 1, two: 2, three: 3], do: val
[1, 2, 3]

# Maps
iex> for {k, v} <- %{"a" => "A", "b" => "B"}, do: {k, v}
[{"a", "A"}, {"b", "B"}]

# Binaries
iex> for <<c <- "hello">>, do: <<c>>
["h", "e", "l", "l", "o"]
```

প্যাটার্ন ম্যাচিং এলিক্সিরের একটি বিশেষত্ব যার উপর এর বহু সিনট্যাক্সিয় ফিচার নির্ভরশীল। জেনারেটর এর ব্যতিক্রম নয়। জেনারেটররা তাদের ইনপুট সেটকে বাম পার্শ্বীয় ভেরিয়েবলের সাথে ম্যাচ করে, ম্যাচ পাওয়া না গেলে তা ইগনর করা হয়।

```elixir
iex> for {:ok, val} <- [ok: "Hello", error: "Unknown", ok: "World"], do: val
["Hello", "World"]
```

একাধিক জেনারেটর ব্যবহার করা সম্ভব, ঠিক নেস্টেড লুপের মত।

```elixir
iex> list = [1, 2, 3, 4]
iex> for n <- list, times <- 1..n do
...>   String.duplicate("*", times)
...> end
["*", "*", "**", "*", "**", "***", "*", "**", "***", "****"]
```

আভ্যন্তরীণ লুপিংকে আরও ভালভাবে বুঝানোর জন্য আমরা `IO.puts` দিয়ে জেনারেটেড ভ্যালু দেখাতে পারি। 

```elixir
iex> for n <- list, times <- 1..n, do: IO.puts "#{n} - #{times}"
1 - 1
2 - 1
2 - 2
3 - 1
3 - 2
3 - 3
4 - 1
4 - 2
4 - 3
4 - 4
```

লিস্ট কম্প্রিহেনশান হল সিন্ট্যাক্টিক সুগার যা শুধুমাত্র প্রয়োজনভেদে ও যথাস্থানে ব্যবহার করা উচিৎ।

## ফিল্টার

ফিল্টারকে আপনি মনে করতে পারেন লিস্ট কম্প্রিহেনশানের গার্ডস্বরপ। ফিল্টারকৃৎ ভ্যালু যদি `false` অথবা `nil` হয় তবে তা রেজাল্ট লিস্ট থেকে বাদ পড়ে যায়। আমরা এখন একটি রেঞ্জের উপর লুপ করে শুধুমাত্র জোড় সংখ্যা বের করব লিস্ট কম্প্রিহেনশান ও ফিল্টার দিয়ে। এর জন্য আমরা `is_even/1` ফাংশন ব্যবহার করব।

```elixir
import Integer
iex> for x <- 1..10, is_even(x), do: x
[2, 4, 6, 8, 10]
```

জেনারেটরের মত ফিল্টারও একাধিক ব্যবহার করা যায়। যদি আমরা আমাদের রেঞ্জকে বর্ধিত করি এবং শুধুমাত্র সেগুলিকে দেখতে চাই যারা জোড় এবং ৩ দ্বারা বিভাজ্য দুটোই তাহলে নিচের মত করে কোড সাজাব:

```elixir
import Integer
iex> for x <- 1..100,
...>   is_even(x),
...>   rem(x, 3) == 0, do: x
[6, 12, 18, 24, 30, 36, 42, 48, 54, 60, 66, 72, 78, 84, 90, 96]
```

## :into এর ব্যবহার

আমরা যদি লিস্ট ব্যতীত অন্য কিছু রিটার্ন করতে চাই তাহলে তা কিভাবে করব? উত্তর হল `:into`। `:into` এর মাধ্যমে আমরা যে কোন `Collectable` প্রোটোকল মেনে চলা স্ট্রাকচারে আমাদের জেনারেটর প্রদত্ত রেজাল্ট ধারণ করতে পারি।

`:into` দিয়ে ম্যাপ থেকে কীওয়ার্ড লিস্টে পরিণত করা যাক কিছু ডেটা:

```elixir
iex> for {k, v} <- [one: 1, two: 2, three: 3], into: %{}, do: {k, v}
%{one: 1, three: 3, two: 2}
```

যেহেতু বিটস্ট্রিংরাও এনুমেরেবল সেহেতু আমরা লিস্ট কম্প্রিহেনশান ও `:into` দিয়ে স্ট্রিং প্রস্তুত করতে পারি।

```elixir
iex> for c <- [72, 101, 108, 108, 111], into: "", do: <<c>>
"Hello"
```

শেষ! লিস্ট কম্প্রিহেনশান ব্যবহার করে বোধগম্য ও সংক্ষিপ্ত সিনট্যাক্স দিয়ে আমরা লুপিং ও ইটারেশান করতে পারি।
