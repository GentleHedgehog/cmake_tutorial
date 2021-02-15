# basics:

# funcs:
# - introduce a new scope with args
# - args are converted to vars (with all vars settings)

#function(name [arg1 [arg2 [..]]])
    # body
#endfunction()

# macros:
# - paste their body into the point of the call
# - args are substituted as simple strings (interpreted as string in if statement)

#macro(name [arg1 [arg2 [..]]])
    # body
#endfunction()

function(print_me) # case insensitive name
    message("hello from func")
    message("all done")
endfunction()

print_me()

message("-- func/macro args --")

function(func arg)
    if (DEFINED arg)
        message("func arg is defined")
    else()
        message("func arg is NOT defined")
    endif()
endfunction()

macro(macr arg)
    if (DEFINED arg)
        message("macro arg is defined")
    else()
        message("macro arg is NOT defined")
    endif()
endmacro()

func(foobar) # var is defined
macr(foobar) # var is NOT defined

# arg is case sensitive label
function(func myArg)
    message("myArg = ${myArg}") # act like a var
endfunction()

macro(macr myArg)
    message("myArg = ${myArg}") # act like a string substitution
endmacro()

func(foobar) # myArg = foobar
macr(foobar) # myArg = foobar

# special vars:
message("-- special vars --")

function(func arg0 arg1)
    message("total vars ARGC = ${ARGC}")
    message("list of all vars ARGV = ${ARGV}")
    message("list of unnamed vars ARGN = ${ARGN}")
    # arg can be referenced by ARG#, where #: 1, 2, 3..
endfunction()

func(foo bar another_arg)

# prefer function to use ARGN (macro can get it from outside scope):
# foreach(arg IN LISTS ARGN) - for funcs, or
# foreach(arg IN LISTS ${ARGN}) - for macros


# -- Keyword args
message("-- keyword args --")

# include(CMakeParseArguments) # only for Cmake v3.4 and earlier
# cmake_parse_arguments(prefix, noValueKeywords, singleValueKeywords, multiValueKeywords, argsToParse)
# - argsToParse: list of args, processed with keywords (typically - ${ARGN})
# - noValueKeywords: standalone keywords like boolean switches (true if specified, false - if not)
# - singleValueKeywords: requires single arg
# - multiValueKeywords: requires zero or more args
# - prefix - prepended to found keywords: PREFIX_FOUND_KEYWORD
# keywods should be like: UPPERCASE_KEYWORD
# this func fills keyword vars with keywords with prefixes

function(func)
    set(prefix ARG)
    set(noValues ENABLE_NET COOL_STUFF)
    set(singleValues TARGET)
    set(multiValues SOURCES IMAGES)

    cmake_parse_arguments(${prefix} "${noValues}" "${singleValues}" "${multiValues}" ${ARGN})

    message("option summary:")
    foreach(arg IN LISTS noValues)
        if(${${prefix}_${arg}})
            message("  ${arg} enabled")
        else()
            message("  ${arg} disabled")
        endif()
    endforeach()

    foreach(arg IN LISTS singleValues multiValues)
        message("  ${arg} = ${${prefix}_${arg}}")
    endforeach()
endfunction()

func(SOURCES foo.cpp bar.cpp TARGET myApp ENABLE_NET)
func(COOL_STUFF TARGET dummy IMAGES here.png there.png)


# -- Scope
message("-- Scope --")

# func DO NOT introduce a new policy scope
# to return value from func - use PARENT_SCOPE (macro can use a usual set() without PARENT_SCOPE):

function(func resVar1 resVar2)
    set(${resVar1} "first res" PARENT_SCOPE)
    set(${resVar2} "second res" PARENT_SCOPE)
endfunction()

func(myVar otherVar)
message("myVar: ${myVar}")
message("otherVar: ${otherVar}")

# return() - stop scope processing and returns (stop the func or stop an outer scope for the macro)


# -- Overriding commands
message("-- Overriding commands --")
# new func/macro overrides previous one with the same name
# undocumented feature - previous one can be accessed using the same name with an underscore prepended:
# nameOfFunc -> _nameOfFunc
# but this is not reliable solution to use this feature (and it works only for the last override)

# -- Recommended practices --
# prefer funcs
# use macros only where the execution withing the scope of the caller makes sense
# avoid calling return() in macro
# use cmake_parse_arguments() (unless it is trivial func)
# create a special directory to keep all *.cmake files
# DO NOT call funcs with a name that starts with a single underscore
# DO NOT override buil-in cmake commands

