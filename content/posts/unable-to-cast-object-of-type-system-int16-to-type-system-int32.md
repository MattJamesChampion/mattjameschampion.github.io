+++
title = 'Unable to cast object of type `System.Int16` to type `System.Int32`'
date = '2025-06-23T08:00:00+01:00'
draft = false
aliases = ['/2025/06/23/unable-to-cast-object-of-type-system-int16-to-type-system-int32']
+++

I ran into this corker of an error message recently which initially really confused me:

```plain text
Unable to cast object of type 'System.Int16' to type 'System.Int32’
```

If you know about data types and sizes you’re probably thinking something similar to what I thought: why not? `Int16` is
_smaller_ than `Int32` and many languages will happily cast to a _larger_ version of a data type because there is no
risk of losing data/precision (unlike casting to a _smaller/less precise_ version which can change the underlying
value). Any value that can fit into an `Int16` can fit into an `Int32`!

{{< info "As a quick caveat, for all intents and purposes `Int16` is equivalent to short and `Int32` is equivalent to
int. I’ll be swapping between them where they each make the most sense." >}}

# The Problem

In my particular scenario I was trying to parse a `DataRow` with the (relatively lazy) approach of:

```csharp
var value = (int)inputRow["Value"];
```

Now, in my mind this was an easy (and previously very successful!) way to grab the value out in the type that I wanted,
and yet in this case it was falling over something terrible.

# The Reproduction

I tried a few approaches to figure out what was going wrong here. My first shot was to double check that you can
actually cast from an `Int16` (`short`) to an `Int32` (`int`):

```csharp
var inputValue = (short)16;
var castValue = (int)inputValue;
Console.WriteLine(castValue);
```

Aaaand... it works fine. Hmm.

Once it was clear that the act of casting from `short` to `int` is perfectly safe in and of itself, it was time to try
and find out what was different here.

As I mentioned, I wasn’t just dealing directly with value types, I was attempting to cast the result of
`inputRow["Value"]`, so a true attempt at reproducing the issue is to try the same with a `DataTable`:

```csharp
using System.Data;

var dataTable = new DataTable();
dataTable.Columns.Add(new DataColumn("shortCol", typeof(short)));

var exampleRow = dataTable.NewRow();
exampleRow["shortCol"] = 16;

var parsedShort = (int)exampleRow["shortCol"];
Console.WriteLine(parsedShort);
```

And what do you know, we get
`System.InvalidCastException: Unable to cast object of type 'System.Int16' to type 'System.Int32'.` again!

So what is the `DataTable` doing that we’re not handling properly?

The first question is "what does `DataTable` return if you don’t try to cast it?" and the answer to that is "an
`object`"! With this information we can take our original attempt at a minimal repro and get it down to only four lines:

```csharp
var shortValue = (short)16;
var shortValueAsObject = (object)shortValue;
var intValue = (int)shortValueAsObject;
Console.WriteLine(intValue);
```

We see the error again which is helpful, but what is actually going on here?

# The Explanation

The answer is... [boxing](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing)!
But not, like, 🥊, more like 📦.

Since we’re working on a boxed value the `(int)` statement before the expression is an unboxing operation. If we take a
look at the documentation, we see the following:
> For the unboxing of value types to succeed at run time, the item being unboxed must be a reference to an object that
> was previously created by boxing an instance of that value type.
> [Boxing and Unboxing (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing#unboxing)

Since an `Int16` isn’t the same as an `Int32`, the unboxing operation fails!

What we were doing before when we were changing a `short` value type to an `int` was actually
a [cast expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#cast-expression),
not an unboxing. Although they both use the same syntax and are broadly similar in what they do, they’re different
enough that their behaviour in this scenario diverges.

This is why you could take the minimal repro from before and change the third line as such

```csharp
var intValue = (int)(short)shortValueAsObject;
```

and it now works; `shortValueAsObject` is first unboxed back into a `short` value type (which is the same as how it was
boxed and therefore succeeds) and the resulting `short` is then _cast_ to an `int`.
The Solution

If you’re looking to fix the issue I probably wouldn’t recommend the chained boxing → casting approach; it’s
unnecessary! There are a couple of ways to fix the issue in a much easier and cleaner way:

## Unbox to the correct type

Simple enough, just change the type that you’re unboxing to! In the above example that means accurately reflecting the
fact that you’re storing and retrieving a `short` rather than trying to jam it into an `int` straight out of the box
(terrible pun _absolutely_ intended).

## Use `Convert`

If you’re adamant that you need an `int` (or whatever other type) even though it was originally boxed from a `short` you
can do so in a single step using `Convert.ToInt32` (or one of the other functions depending on your target type)! As the
name suggests, this function takes an `object` and attempts to convert it to an `int`, throwing an exception if it fails
to do so.

The one thing to note about this approach though is that `Convert.ToInt32` is quite flexible in ways you might not
expect. For example if you pass in certain types (like a `double` or `decimal`) you’ll have a successful conversion, but
may lose data. For example, the result of

```csharp
var intValue = Convert.ToInt32((object)12.8);
```

is that `intValue` now has the value `13` stored in it! Make sure that you _know_ that the type being boxed is one that
won’t lead to losing information.

# The Summary

And that’s it! This was a fun post to write; I originally came across this issue months ago but hadn’t gotten around to
writing it up, but when I did I realised that although I had indeed fixed the issue I hadn’t fully grasped what exactly
was causing it to fail! I knew about boxing/unboxing and casting separately but I’d never really considered that they
use the same syntax while having different behaviour in similar circumstances.