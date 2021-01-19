cmake_minimum_required(VERSION 3.5) # cmake should bahave the same as version 3.5 (major.minor[.patch[.tweak]])

project(ch1 VERSION 1.0 LANGUAGES CXX) # [VERSION major[.minor[.patch[.tweak]]]], LANGUAGES C CXX Java ... NONE (by default C and CXX)
# project - check compiler and linker for workability



set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------- chapter 4 ----------

add_executable(ch1 main.cpp) # several commands can produce diffrent target names
# [WIN32] - windows gui app: WinMain(), link to /SUBSYSTEM:WINDOWS
# [EXCLUDE_FROM_ALL] - by default cmake build 'all' target,
#and we can exclude this target from 'all' (for example: for a rarely needed developer tool)

#add_library(targetName [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 [source2 ..])
# STATIC - targetName.lib (win), targetName.a (unix)
# SHARED - targetName.dll (.so)
# MODULE - like plugin (.dll on win), loaded at runtime
# if no type: cmake -DBUILD_SHARED_LIBS=YES /path/to/source, or set(BUILD_SHARED_LIBS YES) before add_library


#linking libs:
# * PRIVATE - dependency of lib A from lib B (internal dependency, A can be used without B)
# PUBLIC - A use B in its interface (A cannot be used without B, for ex: func in A has arg type from B) (used by default)
# INTERFACE - A doesnt require B internally, only in its interface
#   target_link_libraries(targetName
#    <PRIVATE|PUBLIC|INTERFACE> item1 [...]
#    ...
#    )
# * !!! targetName must have been defined by add_executable or add_library
#in the same directory from which target_link_libraries is being called
# * can be specified as items:
# full path to lib file; lib name like foo (becomes -lfoo or foo.lib) (often system libs); link flag (prefer for PRIVATE items)
# * each item can be preceded by (but it is deprecated): debug, optimized or general (depends from build type)
# oprimized ~ is not a debug build, general ~ for all build configs (default)

# do not use project name as target name! name target according to what the target does
# do not use prefix 'lib' for libs! it can be added by system
# prefere not to use SHARED|STATIC for libs (use BUILD_SHARED_LIBS)
# always use lib item specifiers: PRIVATE, PUBLIC or INTERFACE


# ------- chapter 5 ----------

message(------- chapter 5 ----------)

#set(varName value... [PARENT_SCOPE])
# varName - case sensitive, has a scope for read/write
# variable - interpret as string, that is quoted if contains spaces (if not quoted - space is interpreted as ';')
# set(var a b c) -> var = "a;b;c"
# set(var a;b;c) -> var = "a;b;c"
# set(var "a b c") -> var = "a b c"
set(ch5Var1 a b c)
set(ch5Var2 a;b;c)
set(ch5Var3 "a b c")
message(${ch5Var1} - ${ch5Var2} - ${ch5Var3})
# undefined var produces an empty string
set(foo ab)               # foo   = "ab"
set(bar ${foo}cd)         # bar   = "abcd"
set(baz ${foo} cd)        # baz   = "ab;cd"
set(myVar ba)             # myVar = "ba"
set(big "${${myVar}r}ef") # big   = "${bar}ef" = "abcdef"
set(${foo} xyz)           # ab    = "xyz"
set(bar ${notSetVar})     # bar   = ""
message(foo = ${foo})
message(ab = ${ab})
message(bar = ${bar})
message(baz = ${baz})
message(big = ${big})

set(multiLine "First line ${myVar}
Second line with a \"quoted\" word")
message(multiLine = ${multiLine})

#useful for Unix shell scripts:
set(multiLine1 [[
    First line
    Second line
]])
message(multiLine1 = ${multiLine1})

set(multiLine2 [=[
    First line [[ bla ]]
    Second line
]=])
message(multiLine2 = ${multiLine2})

#unset vars:
set(myVar)
unset(myVar)

# env vars (wont be visible at build time, only at cmake working time):
#set(ENV{PATH} "$ENV{PATH}:/opt/myDir")
#message($ENV{PATH})

# CACHE VARS, read as usual vars, but write as:
# set(varName value... CACHE type "docstring" [FORCE])  # docstring can be empty
# FORCE - allow to rewrite cache var
# type and docstring(as tooltip) - for GUI purposes
# types:
# BOOL - ON/OFF, TRUE/FALSE, 1/0:
#   special func: option(optVar helpString [initialValue]), initial == OFF by default, FORCE by default
#   equal to: set(optVar initialValue CACHE BOOL helpString)
# FILEPATH - path string (-> open file dialog for user)
# PATH - ditto (-> open dir dialog)
# STRING - arbitrary string
# INTERNAL - not available for the user (for example vars for internal caching)

# normal vars have PRECEDENCE over cache vars!

# set cache value in cmd:
# cmake -D myVar:type=someValue ~ set(myVar someValue CACHE type FORCE)
# we can omit type (is similar to INTERNAL), but shouldnt: set(varWithoutType .. [FILE]PATH ..) -> may produce absolute path instead of relative
# use -D for each var
# if var with spaces:  cmake -D "myVar:type=some value"

# delete cache vars in cmd (try to avoid wildcards: *, ?):
# cmake -U 'help*' -U foo ..


# -- GUI --

# cmake-gui and ccmake(text-only environments, ssh for example)
# configure - build in memory stage (repeat it stage before there are no changes (no red lines))
# generate - build projects files inside build dir
# in cmd this two stages are gathered to the one stage: cmake ...

# combobox feature:
set(trafficLight Green CACHE STRING "Status")
set_property(CACHE trafficLight PROPERTY STRINGS Red Orange Green) # can be rewritten in the next 'set' commands

# advanced vars (viewed in advanced mode in gui):
#mark_as_advanced([CLEAR|FORCE] var1 [var2...])
# CLEAR - to unmark

# -- Debugging Vars and Diagnostics --

# message([mode] msg1 ...)
set(myVar HiThere)
message("The value of myVar = ${myVar}") # print out of order, without hyphens
#message(STATUS "status message") # print in order, with hyphens
#message(WARNING "warning highlighted in red")
#message(AUTHOR_WARNING "author warn message, if -Wdev on cmake cmd")
#message(SEND_ERROR "send error - continue configure, but generation will not be called")
#message(FATAL_ERROR "fatal error - stop immediately")
#message(DEPRECATION "equal to: warn - if CMAKE_WARN_DEPRECATED=TRUE, error - if CMAKE_ERROR_DEPRECATED=TRUE")

# to watch all accesses to var:
# variable_watch(myVar [command])
# variable_watch(myVar) # common use (without command)

# -- string handling --

# string(FIND inputString subString outVar [REVERSE])
# find first substring index (from 0 index) (if REVERSE - last substring index), return -1 if not found
set(longStr abcdefabcdef)
set(shortBit def)
string(FIND ${longStr} ${shortBit} fwdIndex)
string(FIND ${longStr} ${shortBit} revIndex REVERSE)
message("fwdIndex = ${fwdIndex}, revIndex = ${revIndex}")

#string(REPLACE matchString replaceWith outVar input [input..])
# will replace each occurence of matchString with replaceWith for input (many inputs will be concatenated)
string(REPLACE abc def myOutVar abcdef)
message("myOutVar = ${myOutVar}")

# string(REGEX MATCH regex outVar input [input..]) # find first match
# string(REGEX MATCHALL regex outVar input [input..]) # find all matches, save as list
# string(REGEX REPLACE regex replaceWith outVar input [input..]) # return input with all matches replaced
set(longStr abcdefabcdef)
string(REGEX MATCHALL "[ace]" matchVar ${longStr})
string(REGEX REPLACE "([de])" "X\\1Y" replVar ${longStr})
message("matchVar = ${matchVar}, replVar = ${replVar}")

#string(SUBSTRING input index length outVar)
#string(LENGTH input outVar)
#string(TOLOWER input outVar)
#string(TOUPPER input outVar)
#string(STRIP input outVar) # strip whitespaces

# -- Lists --
set(myList a b c)
list(LENGTH myList len) # myList - must be a variable
message("len of ${myList} = ${len}")
list(GET myList 2 1 letters) # (GET listVar index [index..] outVar)
message("letters = ${letters}")

list(APPEND myList d e f)
message("appended myList = ${myList}")

list(INSERT myList 2 X Y Z) # insert to position 2 (index)
message("insert to index 2 myList = ${myList}")

list(FIND myList b foundIndex)
message("foundIndex for b = ${foundIndex}")

# list(REMOVE_ITEM myList value [value..]) # if no such value - no error
# list(REMOVE_AT myList index [index..]) # if no such index - error
# list(REMOVE_DUPLICATES myList)

# list(REVERSE myList)
# list(SORT myList) # alphabetical

# index can be negative - last index = -1, then -2...

# -- MATH --
# math(EXPR outVar mathExpr)
# outVar = result, mathExpr - can contain C code operators: + - * / % | & ^ ~ << >> * / %
set(x 3)
set(y 7)
math(EXPR z "(${x} + ${y}) / 2")
message("z = ${z}")

# -- recommended practices --
# use optional cache vars to enable/disable optional parts of project
# prefere cache to env vars (env vars are not the reliable solution)
# start vars with common prefix (may be closely associated with project name) to group them easily
# do NOT use the same names for usual and cache vars
# try to use mode for message command (except debug case, temporarily)
# occasionally look at the predefined variables in cmake documentation (WIN32, UNIX, etc.)


message("\n\n")
