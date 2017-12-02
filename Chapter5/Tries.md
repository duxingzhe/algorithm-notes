### Tires

Specifically, the methods that we consider in this section achieve the following performance characteristics in typical applications, even for huge tables:

* Search hits take time proportional to the length of the search key.
* Search misses involve examining only a few characters.

This API differs from the symbol-table API introduced in Chapter 3 in the following aspects:

* We replace the generic type Key with the concrete type String.
* We add three new methods, longestPrefixOf(), keysWithPrefix() and keysThatMatch().

We retain the basic conventions of our symbol-table implementations in Chapter 3 (no duplicate or null keys and no null values).

The following descriptions of the three new methods use the keys she sells sea shells by the sea shore to give examples:

* longestPrefixOf() takes a string as argument and returns the longest key in the symbol table that is a prefix of that string. For the keys above, longestPrefixOf ("shell") is she and longestPrefixOf("shellsort") is shells.
* keysWithPrefix() takes a string as argument and returns all the keys in the symbol table having that string as prefix. For the keys above, keysWithPrefix("she") is she and shells, and keysWithPrefix ("se") is sells and sea.
* keysThatMatch() takes a string as argument and returns all the keys in the symbol table that match that string, in the sense that a period (.) in the argument string matches any character. For the keys above, keysThatMatch (".he") returns she and the, and keysThatMatch("s..") returns she and sea.

