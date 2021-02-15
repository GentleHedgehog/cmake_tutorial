# -- add_subdirectory() --
message("-- add_subdirectory() --")
# add_subdirectory(sourceDir [ binaryDir ] [ EXCLUDE_FROM_ALL ])
# sourceDir - path to dir (absolute or relative to current source directory) (usually contains CMakeLists.txt itself)
# binaryDir - path to dir (aboslute or relative to current bin dir) (if not specified -> cmake create bin dir with sourceDir name)
# if sourceDir - abs path outside cur dir, binaryDir must be specified
# EXCLUDE_FROM_ALL - doesnt always act as expected (for some cmake versions or generators)

# read only vars after processing each CMakeLists.txt:
# CMAKE_SOURCE_DIR - top-most dir of source tree
message("CMAKE_SOURCE_DIR = ${CMAKE_SOURCE_DIR}")
# CMAKE_BINARY_DIR - top-most dir of bin tree
message("CMAKE_BINARY_DIR = ${CMAKE_BINARY_DIR}")
# CMAKE_CURRENT_SOURCE_DIR - dir of current CMakeLists.txt
message("CMAKE_CURRENT_SOURCE_DIR = ${CMAKE_CURRENT_SOURCE_DIR}")
# CMAKE_CURRENT_BINARY_DIR - dir of current build
message("CMAKE_CURRENT_BINARY_DIR = ${CMAKE_CURRENT_BINARY_DIR}")


# -- Scope --
message("-- Scope --")
# subdirectory - child scope (parent scope cannot see child scope vars)
# if child scope work with var with name that exists in parent scope - this is local changes (parent var is left unchanged)

set(varWithParentScope "foo2") # create local scope var (doesnt change parent scope)
set(varWithChildScope "bar") # create local scope var (doesnt pollute parent scope)
set(varWithParentScopeToChangeInSubdir "changed in subdir" PARENT_SCOPE) # change parent scope var only!
message("varWithParentScope changed in subdir = ${varWithParentScope}")
message("varWithChildScope changed in subdir = ${varWithChildScope}")
message("varWithParentScopeToChangeInSubdir changed in subdir = ${varWithParentScopeToChangeInSubdir}") # in local scope - previous value

# more appropriate variant of parent scope changing:
#   set(localVar bar)
#   set(myVar ${localVar} PARENT_SCOPE)

# policies and properties have similar behavior (relative to scope)


# -- include --
message("-- include() --")

# first form:
# include(fileName [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
# analog to add_subdirectory():
# - expects fileName instead of CMakeLists.txt
# - extension of fileName is *.cmake (may be anything)
# - does not introduce a new variable scope
# - introduce policy scope (NO_POLICY_SCOPE - to prevent it)
# - CMAKE_CURRENT_..(source/bin dir) vars do not change
# - OPTIONAL - no error if file doesnt exist
# - RESULT_VARIABLE - set full name of the included file, or NOTFOUND is it failed

# second form for modules (ditto, all but the first rule are accepted):
# include(module [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])

# for facilitating purposes (it works for any file being processing by CMake):
# CMAKE_CURRENT_LIST_DIR - analog of CMAKE_CURRENT_SOURCE_DIR
# CMAKE_CURRENT_LIST_FILE - current file name
# CMAKE_CURRENT_LIST_LINE - current line (for debug)

# include() allows to include a file that was already used in project independently


# -- ending processing early --
message("-- ending processing early --")

# return() - ends processing of the current file (but cannot return a value to the caller)
# include_guard(arg) - like #pragma once (begins from v3.10)
# - no arg - variable scope is assumed, analog: if (DEFINED var) return() endif() set(var 1) # variable scope of a parent file
#   i.e. this file cannot be included twice in the parent file scope
# - GLOBAL - if file has been processed anywhere (variable scope is ignored)
# - DIRECTORY - if file has been processed only withing the current dir and below (can be included within another dir more than once)


# -- Recommended practices --
# prefer add_subdirectory() in simple projects
# use include() only if it makes sense
# prefer CMAKE_CURRENT_LIST_DIR over CMAKE_CURRENT_SOURCE_DIR (to facilitate a transfer from add_subdirectory() to include())
# prefer to use include_guard() without arguments
