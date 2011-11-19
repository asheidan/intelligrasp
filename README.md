# Intelligrasp

Yes, that is a play on Intellisense.

This is a try to create a general implementation of code completion for use in
any editor/debugger.

You probably shouldn't use this, for anything, except amusement...


## Why?

I haven't been able to find a "good" system that does this, or even tries.

Everything that I've found (but I haven't been looking that hard) only does
this for a few languages. Why not make a general enough architecture that
allows this to work for "any" and all languages with minimal effort (at least
on my part as I probably won't implement any parsers).

This should already be part of both Emacs and Vim (I wrote those in
alphabetical order, don't read anything else into the order), but it isn't. Not
on the level that we all have come to expect when using "proper" IDEs (XCode,
Eclipse, NetBeans, Visual Studio).

Let's make this thing so we all can enjoy our favourite editors once more.


## Design idea

_This should probably be moved to a wiki or somewhere where it's easier to
discuss the design. Use Github's possibility comment commits/code for now._

It should by plugin-based.

Or rather, the parsing of the source should be done by a plugin. That way it
will be (at least I hope so) easier to write a new parser för a language that
hasn't got support yet instead of trying to write the entire thing from the
ground up. This might make it possible to leverage work done previously as well
(CLang-complete?).

It should be easy (even trivial) to implement support for a new editor. I'm not
sure if this means that there should be a plugin for each editor or if the
output should be simple enough that it's easy to write a script or similar for
your editor of choice.

Creating a tree of scopes. Every part of the code belongs to one of the scopes.

I'll try to explain this with an example. Look at the code below.


	#include <stdio.h>

	int a = 42;

	int something() {
		return a + 1;
	}

	int main(int argc, char** argv) {
		int b;

		b = something();

		return a;
	}


This would be represented by the scope tree below.


* __root / global:__  
  _int_ a  
  _int_ something()  
  _int_ main(_int_ argc, _char**_ argv)
	* __something()__
	* __main()__  
	  _int_ b


This is quite a simplistic example but I hope you get the idea. When using
structs or programming in C++ the scopes quickly becomes more complicated. In
OO-languages scopes for the object before the cursor (if there is one) should
be taken into account.

When doing a lookup the following algorithm is used.

1. Locate the correct scope (or scopes) for the cursor's place in the file
2. Create a list of possible completions based on what's available in the
   current scope and parent scopes of the current one.
3. It's probably a good idea to sort the list based on what's "wanted" by the
   editor, ie. what is or returns an int.

### Handling plugins

This depends on the language of the implementation, but as I lean towards using
C++ for the implementation (reasonably fast and not many strange external
dependencies) using _dlopen()_ to load a shared library which exposes some sort
of factory or parser-function seems like the way to go.

### Parser plugin

This plugin should take a file and/or filecontent as input and produce a scope
tree. I still haven't thought of a good way to indicate included external
files. One idea is to represent each file by a named scope and then use this
name to find references to other files/scopes.

### Scopes

* Name
	- function : The name is the name of the function
	- file     : If this is a "top-level" scope the name represent the name of
	             the file
	- class/struct
* Parent(s)
	- function : Enclosing (function's) scope
	- file     : Null?
	- class/struct : Inherited classes (public/protected/private), unioned structs

How to handle typedefs/inheritance? A tree representing the inherited/aliased types?


## Feedback

Ooh, yes. I'm not stupid enough to believe that I've already thought of
everything. I'm quite sure that there are a few quite large things that I've
missed. Now is your chance, point them out to me before I realise it myself and
corrects my mistakes myself.
