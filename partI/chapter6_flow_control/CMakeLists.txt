# description:

if(expression1)
    #commands
elseif(expression2)
    #commands
else()
    #commands
endif()

# unquoted constant treated as true: 1(nonzero number), ON, YES, TRUE, Y (case insensitive)
# unquoted constant treated as false:  0, OFF, NO, FALSE, N, IGNORE, NOTFOUND, an empty string or a string that ends in -NOTFOUND

# unquoted constats: if (YES) or if (${myVar})

# cmake doc: if (<variable|string>) -> unquoted name of a var OR quoted string
# variable: its content is compared with false constants, if no match, interpreted as true (undefined var ~ empty string)
# quoted string: evaluates to false (v3.1 and newer) (without special policy), before 3.1 - transform to variable (substitution)

# Logic operators: AND, OR, NOT (with parentheses)

# Comparison of two operands (vars and [quoted] values):
# Numeric: LESS, GREATER, EQUAL, LESS_EQUAL*, GREATER_EQUAL*
# String: STRLESS, STRGREATER, STREQUAL, STRLESS_EQUAL*, STRGREATER_EQUAL*
# Version: VERSION_LESS, VERSION_GREATER, VERSION_EQUAL, VERSION_LESS_EQUAL*, VERSION_GREATER_EQUAL*
# * last two operators - available from 3.7

#if (2 GREATER 1) # ok
#if ("23" EQUAL 23) #ok
#set(val 42)
#if (${val} EQUAL 42) #ok
#if ("${val}" EQUAL 42) #ok
#if ("23a" EQUAL 23) # error!

#if(1.2.3 VERSION_GREATER 1.2) #ok

# strings are compared lexicographically (bear in mind about variable substitution)

# regex:
#set(who Fred)
#set(who Barney)
if("Hi from ${who}" MATCHES "Hi from (Fred|Barney).*")
    message("${CMAKE_MATCH_1} says hello")
endif()
# CMAKE_MATCH_0 - entire matched string, (n ~ group to match)

# file system operators (they dont perform var/string substitution):
# if (EXISTS pathToFileOrDir)
# if (IS_DIRECTORY pathToDir)
# if (IS_SYMLINK fileName)
# if (IS_ABSOLUTE path)

# if (file1 IS_NEWER_THAN file2)
# true: if files' timestamp is the same OR one of them is missing
# and all paths must be absolute

# EXISTENCE tests:

#if (DEFINED name) # true if name exists (can be ENV{name})
#if (COMMAND name) # true if command exists
#if (POLICY name) # true if policy (CMPxxxx) is known to cmake
#if (TARGET name) # true if target was defined (by add_executable, add_library or add_custom_target) (in any directory relative to this point)
#if (TEST name) # true if test was defined (by add_test())
# if (value IN_LIST listVar) # true if value is in listVar

# -- COMMON EXAMPLES --
# use if(MSVC) instead of just if(WIN32), if you want to compile with msvc on Windows
# to use xcode generator on apple: if (CMAKE_GENERATOR STREQUAL "Xcode").. (instead of just if(APPLE))

# enable part of the build:
#   option(BUILD_MYLIB "enable building the myLib target")
#   if(BUILD_MYLIB)
#       add_library(myLib src1.cpp src2.cpp)
#   endif()

# --- LOOPING ---

# -- FOREACH --
message("-- FOREACH --")

# basic form of foreach:
foreach(loopVar arg1 arg2)
    message("basic Iteration for: ${loopVar}")
endforeach()

# general form of foreach:
set(list1 A B)
set(list2)
set(foo WillNotBeShown)
foreach(loopVar IN LISTS list1 list2 ITEMS foo bar)
    message("Iteration for: ${loopVar}")
endforeach()

# C-like iteration:
#foreach(loopVar RANGE start stop [step]) # from start to stop (inclusive) with step (prefer this version!)
#foreach(loopVar RANGE value) # from 0 to value (inclusive)

foreach(loopVar RANGE 0 3 1)
    message("C-like iteration: ${loopVar}")
endforeach()

# -- WHILE --
message("-- WHILE --")

set(condition TRUE)
set(counter 0)

while(condition)
    message("while loop: counter = ${counter}")
    math(EXPR counter "${counter}+1")
    if (counter EQUAL 2)
        set(condition FALSE)
    endif()
endwhile()

# -- --
message("-- loop interrupting --  (like in C)")

foreach(outerVar IN ITEMS a b c)
    unset(s)
    foreach(innerVar IN ITEMS 1 2 3)
        #stop inner loop once string s gets long
        string(APPEND s "${outerVar}${innerVar}") # it uses string-var
        string(LENGTH ${s} length) # it uses string
        message("s = ${s} len = ${length}")
        if (length GREATER 5)
            break() # (1)
        endif()

        #do more processing if outer var is "b"
        if (outerVar STREQUAL "b")
            continue() # (2)
        endif()

        message("Processing ${outerVar}-${innerVar}")

    endforeach()

    message("Accumulated list: ${s}")
endforeach()

# recommended practices:
# minimize opportunities for strings to be interpreted as vars (substitution)
# avoid unary expression with quotes (prefer string comparison)
# use cmake 3.1 or newer
# store CMAKE_MATCH_<n> results asap (if(xxx MATCHES regex)), it will be overwritten by next regexp operation
# for foreach() always specify start and stop values, prefere IN LISTS or IN ITEMS forms
