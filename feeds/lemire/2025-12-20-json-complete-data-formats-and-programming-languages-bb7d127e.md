---
title: JSON-complete data formats and programming languages
url: https://lemire.me/blog/2025/12/20/json-complete-data-format-and-programming-languages/
published: "2025-12-20T21:24:33Z"
feed: lemire
guid: https://lemire.me/blog/?p=22382
---

# JSON-complete data formats and programming languages

![](https://lemire.me/blog/wp-content/uploads/2025/12/Capture-decran-le-2025-12-20-a-16.24.01-150x150.png)

Much of the data on the Internet is shared using a simple format called JSON. JSON is made of two composite types (arrays and key-value maps) and a small number of primitive types (64-bit floating-point numbers, strings, null, Booleans). That JSON became ubiquitous despite its simplicity is telling.

```
{
 "name": "Nova Starlight",
 "age": 28,
 "powers": ["telekinesis", "flight","energy blasts"]
}
```

Interestingly, JSON matches closely the data structures provided by default in the popular language Go. Go gives you arrays/slices and maps… in addition to the standard primitive types. It is a bit more than C which does not provide maps by default. But it is significantly simpler than Java, C++, C#, and many other programming languages where the standard library covers much of the data structures found in textbooks.

There is at least one obvious data structure that is missing in JSON, and in Go, the set. Because objects are supposed to have no duplicate keys, you can implement a set of strings by assigning keys to an arbitrary value like true.

```
{"element1": true, "element2": true}
```

But I believe that it is a somewhat unusual pattern. Most times, when we mean to represent a set of objects, an array suffices. We just need to handle the duplicates somehow.

There have been many attempts at adding more concepts to JSON, more complexity, but none of them have achieved much traction. I believe that it reflects the fact that JSON is good enough as a data format.

I refer to any format that allows you to represent JSON data, such as YAML, as a JSON-complete data format. If it is at least equivalent to JSON, it is rich enough for most problems.

Similarly, I suggest that new programming languages should aim to be JSON-complete: they should provide a map with key-value pairs, arrays, and basic primitive types. In this light, the C and the Pascal programming languages are not JSON-complete.
