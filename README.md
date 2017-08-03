# Lua Coding Conventions

__This page is work-in-progress. Please do not start adhering to these guidelines yet, as they may change very shortly and frequently.__

This document summarizes the high-level coding conventions for writing Lua code at Daedalic Entertainment. They are based on the de-facto Lua Style Guide:

* http://lua-users.org/wiki/LuaStyleGuide

The goal is to make it easier to work in similar teams inside and outside the company, as well as have client code blend in with other code of the Lua API. We are providing a complete summary here in order to allow people to understand the conventions at a glance, instead of having to open multiple documents. Our coding conventions are numbered, which makes it easier to refer to them in code reviews.


## 1. Modules

1.1. Module names are nouns with names that are short and lowercase, with nothing between words.

## 2. Files

2.1. Files should not be longer than 1000 lines. Some Lua files might be used in Visionaire projects, in which case they'll be part the project file and must not exceed 65536 characters.

2.2. Each file should contain a single feature.

2.3. Each file starts with a header containing the file summary. Use Doxygen syntax, and avoid redundant parts such as "This class..."

      --! \brief Schedules threads which are waiting for delay or event.

2.4. Each file with a class definition should list their members in the following order:

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

2.5. In general, do not use any swearing in symbol names, comments or log output.

2.6. Do not use special characters such as hyphens (-) in symbol names.

2.7. Use PascalCase for file names.

2.8. Use camelCase for folder names.

## 3. Classes

3.1. Class names (or at least metatables representing classes) are mixed case (BankAccount).

3.2. Acronyms (e.g. XML) only uppercase the first letter (XmlDocument). 

3.3. Use the short syntax with colons (:) for accessing class members.

3.4. Define all fields of your class at the beginning of your class definition, even those that are not used immediately. This gives readers a better overview of the model of the class, and allows for easier documentation of these fields.

3.5. Avoid classes with more than 40 methods.

## 4. Functions

4.1. Function names are camelCase. If the function is designed to be part of the public interface of a class, its name is PascalCase.

4.2. In general, use the [syntax shortcut for named functions](http://lua-users.org/wiki/FunctionsTutorial): 

      -- Right:
      function f(...)
      end

      -- Wrong:
      f = function (...)
      end

You may use the second variant of explicitly assigning it to a variable for special cases, e.g. using them as function parameters or table values.

4.3. Don't add a space between function name and parameter list:

      -- Right:
      function f(...)
      end

      -- Wrong:
      function f (...)
      end

4.4. Names of functions for accessing fields of a class begin with `Get` or `Set`. This implies that we consider them part of the public interface of the class.

4.5. Names of functions that are used as callbacks for events begin with `on`. This implies that we don't consider them part of the public interface of the class.

4.6. Verify all mandatory function parameters using `assert` guards:

      assert(type(eventId) == "string", "Invalid parameter #2: expected string, got ".. type(eventId))

4.7. Put all optional function parameters last and provide reasonable default values:

      function CoroutineScheduler:Wait(delay, ...)
        delay = delay or self.MIN_WAIT_TIME

4.8. Define abstract methods by providing a default implementation that raises an error.

4.9. Generally, do not define functions in the global namespace.

4.10. Avoid functions with more than six parameters.

4.11. Consider using string constants instead of boolean function parameters.

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

In all other cases, avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget"). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious. Even for iterators, it is very helpful to use more descriptive names:

    for coroutine, thread in pairs(self.threads) do
      -- ...
    end

5.4. Begin boolean variable names with a prefix that indicates its binary semantics (e.g. is, has):
    
    -- Right:
    if IsReady() then

    -- Wrong: Getter might more easily be confused with a function that actually makes the object ready.
    if Ready() then

5.5. Don't use negative names for boolean variables.

    -- Right.
    if visible then

    -- Wrong.
    if not invisible then

## 6. Constants

6.1. Constants are given in ALL_CAPS, with words separated by underscores.

6.2. Group related constants with tables made read-only through our custom `enum` function (similar to `readonlytable` at http://lua-users.org/wiki/ReadOnlyTables):

    Direction = enum {
      NORTH = 1,
      SOUTH = 2,
      WEST = 3,
      EAST = 4
    }

## 7. Indentation & Whitespaces

7.1. Indenting uses two spaces.

7.2. Surround binary operators with spaces.

## 8. Line Breaks

8.1. Keep lines shorter than 100 characters.

8.2. Add two empty lines between two functions.

## 9. Parentheses

9.1. Use parentheses to group expressions:

      // Wrong
      if (a and b or c) then

      // Correct
      if ((a and b) or c) then

9.2. Don't use spaces after parentheses:

      // Wrong
      if ( ( a and b ) or c ) then

      // Correct
      if ((a and b) or c) then


## 10. Control Flow

10.1. Do not put `else` after jump statements:

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

11.2. Place the comment on a separate line, not at the end of a line of code.

11.3. Don't use square brackets [[ ]] for comments.

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

12.2. Implement class inheritance through our `class` function (similar to `inheritsFrom` at http://lua-users.org/wiki/InheritanceTutorial).

12.3. Use multiple assignments only when explicitly swapping two simple values:

      -- Okay.
      x, y = y, x

      -- Not okay.
      a, b = c, c - a


## 13. Error Handling

13.1. Use [`pcall`](http://www.lua.org/pil/8.4.html) if calling a function might result in an error.

# ToDo

* Remove italic and bold highlights
* Do, Don't, Consider, Avoid
* Reorder and regroup rules
* Spell Check
* Remove WIP notice