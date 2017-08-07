# Lua Coding Conventions 1.0

This document summarizes the high-level coding conventions for writing Lua code at Daedalic Entertainment. They are based on the de-facto standard Lua Style Guide:

* http://lua-users.org/wiki/LuaStyleGuide

The goal is to make it easier to work in similar teams inside and outside the company, as well as have client code blend in with other code of the Lua API. We are providing a complete summary here in order to allow people to understand the conventions at a glance, instead of having to open multiple documents. Our coding conventions are numbered, which makes it easier to refer to them in code reviews.


## 1. Modules

1.1. __DO__ use short and lowercase nouns as module names, with nothing between words.


## 2. Files

2.1. __DO__ use camelCase for folder names.

2.2. __DO__ use PascalCase for file names.

2.3. __DO__ add a header with a summary to each file. Use Doxygen syntax, and avoid redundant parts such as "This class..."

      --! \brief Schedules threads which are waiting for delay or event.

2.4. __DO__ list class members in the following order:

* meta function overrides (e.g. __index, __tostring)
* constants
* fields
* constructors
* public functions (see below)
* private functions (see below)

Separate these sections with section headers:

    -- ----------------------------------------------------------------------------
    -- Constructors
    -- ----------------------------------------------------------------------------

2.5. __DO NOT__ cover more than a single feature in each file.

2.6. __AVOID__ files with more than 1000 lines. Some Lua files might be used in Visionaire projects, in which case they'll be part the project file and must not exceed 65536 characters.


## 3. Classes

Although programming in Lua works well without classes, we've learned that grouping functionality in classes improved our code structure by a big deal.

3.1. __DO__ use PascalCase for class names (or at least metatables representing classes), e.g. BankAccount.

3.2. __DO__ uppercase the first letter of acronyms, only, e.g. XmlDocument.

3.3. __DO__ implement class inheritance through our `class` function (similar to `inheritsFrom` at http://lua-users.org/wiki/InheritanceTutorial).

3.4. __DO__ use the short syntax with colons (:) for accessing class members.

3.5. __DO__ define all fields of your class at the beginning of your class definition, even those that are not used immediately. This gives readers a better overview of the model of the class, and allows for easier documentation of these fields.

3.6. __AVOID__ classes with more than 40 methods.


## 4. Functions

4.1. __DO__ use PascalCase for names of functions that are designed to be part of the public interface of a class.

4.2. __DO__ use camelCase for names of functions that are not designed to be part of the public interface of a class.

4.3. __DO__ begin names of functions for accessing fields of a class with `Get` or `Set`. This implies that we consider them part of the public interface of the class.

4.4. __DO__ begin names of functions that are used as callbacks for events with `on`. This implies that we don't consider them part of the public interface of the class.

4.5. __CONSIDER__ using the [syntax shortcut for named functions](http://lua-users.org/wiki/FunctionsTutorial): 

      -- Right:
      function f(...)
      end

      -- Wrong:
      f = function (...)
      end

You may use the second variant of explicitly assigning it to a variable for special cases, e.g. using them as function parameters or table values.

4.6. __DO NOT__ add a space between function name and parameter list:

      -- Right:
      function f(...)
      end

      -- Wrong:
      function f (...)
      end

4.7. __DO__ Verify all mandatory function parameters using `assert` guards:

      assert(type(eventId) == "string", "Invalid parameter #2: expected string, got ".. type(eventId))

4.8. __DO__ put all optional function parameters last and provide reasonable default values:

      function CoroutineScheduler:Wait(delay, ...)
        delay = delay or self.MIN_WAIT_TIME

4.9. __DO__ define abstract methods by using our `abstract` function:

      function onClick() abstract() end

4.10. __AVOID__ defining functions in the global namespace. This is only allowed for core functions, such as `class()`, `enum()` or logging functions.

4.11. __AVOID__ functions with more than six parameters.

4.12. __CONSIDER__ using string constants instead of boolean function parameters.

      -- Right:
      MessageBox:Show("Nice Title", "Nice Text", MessageBox.MESSAGEBOX_BUTTONS_OK)

      -- Wrong: Meaning of third parameter is not immediately obvious.
      MessageBox:Show("Nice Title", "Nice Text", false)


## 5. Variables

5.1. __DO__ use camelCase for variable names.

5.2. __DO__ begin boolean variable names with a prefix that indicates its binary semantics (e.g. is, has):
    
    -- Right:
    if IsReady() then

    -- Wrong: Getter might more easily be confused with a function that actually makes the object ready.
    if Ready() then

5.3. __DO NOT__ use negative names for boolean variables.

    -- Right:
    if visible then

    -- Wrong: Double negation is hard to read.
    if not invisible then

5.4. __DO__ use [locals rather than globals](http://lua-users.org/wiki/ScopeTutorial) whenever possible. Access to locals is faster than globals, since globals require a table lookup at run-time, while locals exist as registers. 

      local x = 0
      local function count()
            x = x + 1
            print(x)
      end

5.5. __CONSIDER__ using a variable name consisting of only an underscore `_` as a placeholder when you want to ignore the variable.

In all other cases, avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget"). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious. Even for iterators, it is very helpful to use more descriptive names:

    for coroutine, thread in pairs(self.threads) do
      -- ...
    end

5.6. __DO__ test whether a variable is not `nil` before using it. Write the variable name rather than explicitly compare against `nil`. Lua treats `nil` and `false` as `false` (and all other values as `true`) in a conditional:

      local line = io.read()
      if line then  -- instead of line ~= nil
      ...
      end
      ...
      if not line then  -- instead of line == nil
      ...
      end

5.7. __AVOID__ using multiple assignments except for when explicitly swapping two simple values:

      -- Right:
      x, y = y, x

      -- Wrong: Assigned values are not immediately obvious.
      a, b = c, c - a


## 6. Constants

6.1. __DO__ use names given in ALL_CAPS for constants, with words separated by underscores.

6.2. __DO__ group related constants with tables made read-only through our custom `enum` function (similar to `readonlytable` at http://lua-users.org/wiki/ReadOnlyTables):

    Direction = enum {
      NORTH = 1,
      SOUTH = 2,
      WEST = 3,
      EAST = 4
    }


## 7. Indentation & Whitespaces

7.1. __DO__ use two spaces for indentation.

7.2. __DO__ surround binary operators with spaces.


## 8. Line Breaks

8.1. __CONSIDER__ keeping lines shorter than 100 characters.

8.2. __DO__ add two empty lines between two functions.


## 9. Parentheses

9.1. __DO__ use parentheses to group expressions:

      -- Right:
      if ((a and b) or c) then

      -- Wrong: Operator precedence is not immediately clear.
      if (a and b or c) then

9.2. __DO NOT__ use spaces after parentheses:
      
      -- Right:
      if ((a and b) or c) then

      -- Wrong:
      if ( ( a and b ) or c ) then


## 10. Control Flow

10.1. __DO NOT__ put `else` after jump statements:

      -- Right:
      if thisOrThat then
          return
      end
          
      somethingElse()


      -- Wrong: Causes unnecessary indentation of the whole else block.
      if thisOrThat then
          return
      else
          somethingElse()
      end


## 11. Error Handling

11.1. __DO__ use [`pcall`](http://www.lua.org/pil/8.4.html) if calling a function might result in an error.


## 12. Comments

12.1. __DO__ use a space after `--`. 

12.2. __DO__ place the comment on a separate line, not at the end of a line of code.

12.3. __DO NOT__ use square brackets [[ ]] for code comments, but for documentating files and members, only.


## 13. Additional Naming Conventions

13.1. __DO NOT__ use any swearing in symbol names, comments or log output.

13.2. __DO NOT__ use special characters such as hyphens (-) in symbol names.
