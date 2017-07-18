# Lua Coding Conventions

__This page is work-in-progress. Please do not start adhering to these guidelines yet, as they may change very shortly and frequently.__

This document summarizes the high-level coding conventions for writing Lua code at Daedalic Entertainment. They are based on the de-facto Lua Style Guide:

* http://lua-users.org/wiki/LuaStyleGuide

The goal is to make it easier to work in similar teams inside and outside the company, as well as have client code blend in with other code of the Lua API. We are providing a complete summary here in order to allow people to understand the conventions at a glance, instead of having to open multiple documents. Our coding conventions are numbered, which makes it easier to refer to them in code reviews.

Additions are _highlighted in italic_. These have been made where the official coding standard doesn't explictily include a rule, but we felt that further clarification is required.

Exceptions are __highlighted in bold__. You'll always find the justification of the exception right next to the rule.

In case we've missed recent changes to the Lua Style Guide, or you can spot any other issue, please [create a pull request](https://help.github.com/articles/creating-a-pull-request/).


## 1. Modules

1.1. Module names are nouns with names that are short and lowercase, with nothing between words.

## 2. Files

2.1. _Files should not be longer than 1000 lines. Some Lua files might be used in Visionaire projects, in which case they'll be part the project file and must not exceed 65536 characters._

2.2. _Each file should contain a single feature._

## 3. Classes

3.1. Class names (or at least metatables representing classes) are mixed case (BankAccount).

3.2. Acronyms (e.g. XML) only uppercase the first letter (XmlDocument). 

## 4. Functions

4.1. Function names are lowercase and use underscores.

4.2. Use the [syntax shortcut for named functions](http://lua-users.org/wiki/FunctionsTutorial): 

      -- Right:
      function f(...)
      end

      -- Wrong:
      f = function (...)
      end

## 5. Variables

5.1. Variable names are lowercase and use underscores.

5.2. Use [locals rather than globals](http://lua-users.org/wiki/ScopeTutorial) whenever possible. Access to locals is faster than globals, since globals require a table lookup at run-time, while locals exist as registers. 

      local x = 0
      local function count()
            x = x + 1
            print(x)
      end


5.3. _Avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget"). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious._

## 6. Constants

6.1. Constants are given in ALL_CAPS, with words separated by underscores.

## 7. Indentation & Whitespaces

7.1. Indenting uses two spaces.

7.2. _Surround binary operators with spaces._

## 8. Line Breaks

8.1. _Keep lines shorter than 100 characters._

## 9. Parentheses

9.1. Use parentheses to group expressions:

      // Wrong
      if (a && b || c)

      // Correct
      if ((a && b) || c)

## 10. Control Flow

10.1. _Do not put `else` after jump statements:_

      -- Wrong
      if (thisOrThat) then
          return
      else
          somethingElse()
      end

      -- Correct
      if (thisOrThat) then
          return
      end
          
      somethingElse()

## 11. Comments

11.1. Use a space after `--`. 

11.2. _Place the comment on a separate line, not at the end of a line of code._

11.3. _Write comments as full sentences._

## 12. Language Features

12.1. Test whether a variable is not `nil` before using it. Write the variable name rather than explicitly compare against `nil`. Lua treats `nil` and `false` as `false` (and all other values as `true`) in a conditional:

      local line = io.read()
      if line then  -- instead of line ~= nil
      ...
      end
      ...
      if not line then  -- instead of line == nil
      ...
      end

# Questions

* LuaLint?
* Modules? Classes?
* File Structure?
* Coroutines? http://lua-users.org/wiki/CoroutinesTutorial
* Tables? http://lua-users.org/wiki/ReadOnlyTables
* Error Handling:
** Use error and pcall? https://www.lua.org/pil/8.4.html 
** Strict Lua? http://lua-users.org/wiki/DetectingUndefinedVariables 
