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

2.3. _Each file starts with a header containing the file summary. Use Doxygen syntax, and avoid redundant parts such as "This class..."_

      --! \brief Schedules threads which are waiting for delay or event.

2.4. _Each file with a class definition should list their members in the following order:_

* meta function overrides (e.g. __index, __tostring)
* constants
* fields
* constructors
* public functions (see below)
* private functions (see below)

_Separate these sections with section headers:_

    -- ----------------------------------------------------------------------------
    -- Constructors
    -- ----------------------------------------------------------------------------

2.5. _In general, do not use any swearing in symbol names, comments or log output._

2.6. _Do not use special characters such as hyphens (-) in symbol names._

## 3. Classes

3.1. Class names (or at least metatables representing classes) are mixed case (BankAccount).

3.2. Acronyms (e.g. XML) only uppercase the first letter (XmlDocument). 

3.3. _Use the short syntax with colons (:) for accessing class members._

3.4. _Define all fields of your class at the beginning of your class definition, even those that are not used immediately. This gives readers a better overview of the model of the class, and allows for easier documentation of these fields._

3.5. _Avoid classes with more than 40 methods._

## 4. Functions

4.1. _Function names are camelCase. If the function is designed to be part of the public interface of a class, its name is PascalCase._

4.2. In general, use the [syntax shortcut for named functions](http://lua-users.org/wiki/FunctionsTutorial): 

      -- Right:
      function f(...)
      end

      -- Wrong:
      f = function (...)
      end

_You may use the second variant of explicitly assigning it to a variable for special cases, e.g. using them as function parameters or table values._

4.3. _Don't add a space between function name and parameter list:_

      -- Right:
      function f(...)
      end

      -- Wrong:
      function f (...)
      end

4.4. _Functions for accessing fields of a class have their name begin with Get or Set._

4.5. _Verify all mandatry function parameters using `assert` guards:_

      assert(type(eventId) == "string", "Invalid parameter #2: expected string, got ".. type(eventId))

4.6. _Put all optional function parameters last and provide reasonable default values:_

      function CoroutineScheduler:Wait(delay, ...)
        delay = delay or self.MIN_WAIT_TIME

4.7. _Define abstract methods by providing a default implementation that raises an error._

4.8. _Generally, do not define functions in the global namespace._

4.9. _Avoid functions with more than six parameters._

4.10. _Consider using string constants instead of boolean function parameters._

      -- Hard to read.
      MessageBox:Show("Nice Title", "Nice Text", false)

      -- Easy to read.
      MessageBox:Show("Nice Title", "Nice Text", MessageBox.MESSAGEBOX_BUTTONS_OK)

## 5. Variables

5.1. Variable names are lowercase and use underscores.

5.2. Use [locals rather than globals](http://lua-users.org/wiki/ScopeTutorial) whenever possible. Access to locals is faster than globals, since globals require a table lookup at run-time, while locals exist as registers. 

      local x = 0
      local function count()
            x = x + 1
            print(x)
      end


5.3. The variable consisting of only an underscore _ is commonly used as a placeholder when you want to ignore the variable.

_In all other cases, avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget"). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious. Even for iterators, it is very helpful to use more descriptive names:_

    for coroutine, thread in pairs(self.threads) do
      -- ...
    end

5.4. _Don't add more than 65535 keys to a table, as this will break some Lua runtimes we're using._

5.5. _Begin boolean variable names with a prefix that indicates its binary semantics (e.g. is, has):_
    
    -- Right.
    if (isReady)

    -- Wrong: Might more easily be confused with a function that actually makes the object ready.
    if (ready)

5.6. _Don't use negative names for boolean variables._

    -- Right.
    if visible then

    -- Wrong.
    if not invisible then

## 6. Constants

6.1. Constants are given in ALL_CAPS, with words separated by underscores.

6.2. _Group related constants with tables made read-only through our custom `enum` function (similar to `readonlytable` at http://lua-users.org/wiki/ReadOnlyTables):_

    Direction = enum {
      NORTH = 1,
      SOUTH = 2,
      WEST = 3,
      EAST = 4
    }

## 7. Indentation & Whitespaces

7.1. Indenting uses two spaces.

7.2. _Surround binary operators with spaces._

## 8. Line Breaks

8.1. _Keep lines shorter than 100 characters._

8.2. _Add two empty lines between two functions._

## 9. Parentheses

9.1. _Use parentheses to group expressions:_

      // Wrong
      if (a and b or c) then

      // Correct
      if ((a and b) or c) then

9.2. _Don't use spaces after parentheses:_

      // Wrong
      if ( ( a and b ) or c ) then

      // Correct
      if ((a and b) or c) then


## 10. Control Flow

10.1. _Do not put `else` after jump statements:_

      -- Wrong
      if thisOrThat then
          return
      else
          somethingElse()
      end

      -- Correct
      if thisOrThat then
          return
      end
          
      somethingElse()

## 11. Comments

11.1. Use a space after `--`. 

11.2. _Place the comment on a separate line, not at the end of a line of code._

11.3. _Don't use square brackets [[ ]] for comments._

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

12.2. _Implement class inheritance through our `class` function (similar to `inheritsFrom` at http://lua-users.org/wiki/InheritanceTutorial)._

12.3. _Use multiple assignments only when explicitly swapping two simple values:_

      -- Okay.
      x, y = y, x

      -- Not okay.
      a, b = c, c - a


## 13. Error Handling

13.1. _Use [`pcall`](http://www.lua.org/pil/8.4.html) if calling a function might result in an error._

# Questions

* enum keyword = http://lua-users.org/wiki/ReadOnlyTables ?
* savegame compatibility
  * tag savegame variable names
  * after first release, always test new variables before using them
* inheritance rules
* multiple assignments per line: self._now, self._frameDelta = now, now - self._now
* for loops: for i = #self._schedule, 1, -1 do
* usage examples
* topics added by Thorben and Daniel

# ToDo

* Check LuaLint: http://lua-users.org/wiki/DetectingUndefinedVariables
* Create Atom Settings File (2 spaces, CRLF)
* Add custom assertion functions, e.g.


      function assertType(value, parameterIndex, expectedType)
        assert(type(value) == expectedType, "Assertion failed: Invalid parameter #" .. parameterIndex .. ": expected " .. expectedType .. ", but was " .. type(value))
      end
