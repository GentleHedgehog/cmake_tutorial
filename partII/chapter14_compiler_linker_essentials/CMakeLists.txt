# description:
# there are many features of build controling, from global settings (coarse settings of a build)
# to more focused features that are targeted to tune each target independently

# --- Target properties ---

# -- compiler flags --

# There are lists for compiler flags ('non-Interface props', 'build requirements'):
#   - INCLUDE_DIRECTORIES - list of absolute paths for headers,
#       when target is created, this prop is taken from the directory property of the same name
#   - COMPILE_DEFINITIONS - definitions look like: VAR=VALUE
#       when target is created, it is empty (it will be combined at the end with the directory property with the same name)
#   - COMPILE_OPTIONS - any compiler flags that are neither header search paths nor symbol definitions
#       when target is created, this prop is taken from the directory property of the same name
#       deprecated analog -> COMPILE_FLAGS, but it is a string and requires escape seq. for spaces

# There are similar lists with prefix: INTERFACE_*. They are 'Interface props', or 'usage requirements'.
# The aim of them is to apply to any other target, which links directly to it.
# They are not initialized from directory properties, and start out empty.

# These target properties support generator expressions.
# It is useful for COMPILE_OPTIONS: to set some flag conditionally.
# Or to obtain path to target and use it as a part of an include directory.

# To work on the source file granularity, we can use source properties (why the names differ from the target counterparts?):
#   COMPILE_DEFINITIONS - support gen expr from v3.8
#   COMPILE_FLAGS - support gen expr from v3.11
#   COMPILE_OPTIONS (arose in v3.11)  - support gen expr from v3.11
# Xcode doesnt support these props

# -- linker flags --

# - LINK_LIBRARIES - list of libs to link directly (empty after init, support gen expr)
#   can contain:
#       path to a lib (usually an absolute path)
#       just a lib name (without prefixes(lib..)/suffixes(.a, .so, .dll))
#       target name (will be converted to a path with appropriate prefix/suffix) - preferred method
# - LINK_FLAGS - list of flags for executables, shared or module libs (not for static libs)
#   for general linker flags, no gen expr, init empty
# - STATIC_LIBRARY_FLAGS - counterpart to LING_FLAGS, for librarian or archiver tools

# Only INTERFACE_LINK_LIBRARIES exists for linker flags (==LINK_INTERFACE_LIBRARIES, deprecated since 2.8.12, CMP0022)
# Due to the lack of gen exp for other props, we can use:
# - LINK_FLAGS_<CONFIG>
# - STATIC_LIBRARY_FLAGS_<CONFIG>

# -- target property commands --

# more precise description for:

#   target_link_libraries(target
#       <PRIVATE|PUBLIC|INTERFACE> item1 [item2..]
#       )
# - PRIVATE - affects only target itself (only non-Interface props: LINK_LIBRARIES, LINK_FLAGS, STATIC_LIBRARY_FLAGS)
# - INTERFACE - complement to PRIVATE - only having an effect on targets that link to targetName
#   (only Interface prop: INTERFACE_LINK_LIBRARIES)
# - PUBLIC - combining the effects of PRIVATE and INTERFACE


#   target_include_directories(target [BEFORE] [SYSTEM]
#       <PRIVATE|PUBLIC|INTERFACE> dir1 [dir2..]
#       )
# This cmd adds paths to INCLUDE_DIRECTORIES (if PRIVATE) and INTERFACE_INCLUDE_DIRECTORIES (if INTERFACE),
# if PUBLIC -> adds to both
#   BEFORE - to prepend instead of append
#   SYSTEM - if paths point to system dirs; it suppresses some warnings, makes some compiler-specific magic
#       Dont use SYSTEM if system headers are included in the project itself
#       Imported targets's INTERFACE_INCLUDE_DIRECTORIES prop will be treated as SYSTEM by default,
#       to suppress, set the consuming targets's NO_SYSTEM_FROM_IMPORTED prop to true (ch16)
# Another advantage -  target_include_directories will convert relative
#   paths to absolute where needed (relative to cur src dir)
# It can use gen expr to set diff paths for installing and building:
#   $<BUILD_INTERFACE:...>
#   $<INSTALL_INTERFACE:...> - relative path are used (ralative to the base install location rather than src dir)


#   target_compile_definitions(target
#       <PRIVATE|PUBLIC|INTERFACE> item1 [item2...]
#       )
# each item can be: VAR or VAR=VALUE
# PRIVATE - populate COMPILE_DEFINITIONS
# INTERFACE - populate INTERFACE_COMPILE_DEFINITIONS
# PUBLIC - populate both
# it support gen expr (but there is no need in it)


#   target_compile_options(target [BEFORE]
#       <PRIVATE|PUBLIC|INTERFACE> item1 [item2...]
#       )
# PRIVATE - populate COMPILE_OPTIONS
# INTERFACE - populate INTERFACE_COMPILE_OPTIONS
# PUBLIC - populate both
# it support gen expr (but there is no need in it)


# --- Directory properties and commands ---

# Prefer to uset target_* (since v3.0) instead of set directory props directly, but we should at least be aware of them:

#   include_directories([AFTER | BEFORE] [SYSTEM] dir1 [dir2..])
# adds search paths to targets in the cur dir and below
#   [AFTER | BEFORE] - preprend or append (CMAKE_INCLUDE_DIRECTORIES_BEFORE=ON can change the default appending)
#   SYSTEM - ditto as in target_include_directories
# relative/absolute paths can be used (gen expr too)
# has two main effects:
#   1. paths are added to INCLUDE_DIRECTORIES dir prop for cur CMakeLists.txt
#   2. any target in cur dir scope will also have the paths in its INCLUDE_DIRECTORIES target prop,
#      even if target was created before the call to include_directories. This applies stricly only to
#      targets created within the current CMakeLists.txt file or other files via include(), but not to
#      any targets created in parent or child dir scopes
# to cope with this surprise: dont use it or use it before: any target creation, any include() or add_subdirectory()


#   add_definitions(-DSomeSymbol /DFoo=Value ...)
#   remove_definitions(-DSomeSymbol /DFoo=Value ...)
# changes COMPILE_DEFINITIONS prop of directory(!) (cmake discards -D or /D symbols, so they have no influence)
# targets are affected as in include_directories() case,
# targets created in child dir scope will only be affected if created after the call
# if cmake cannot recognize an item as a define it tries to add this item to COMPILE_OPTION (prefer to avoid this behavior)
# gen expr can be used only for values after '=' symbol

# due to troubles with add_definition, there is add_compile_definition() (from v3.12):
# add_compile_definitions(SomeSymbol Foo=Value ...)
# there is no prefixes (-D or /D), gen expr can be used freely without restrictions
# still affects all targets as add_definitions()


# add_compile_options(opt1 [opt2 ...])
# adds options to COMPILE_OPTIONS prop of directory
# each target in this scope will be inherit this opts in its COMPILE_OPTION target property
# targets created before the call - isn't affected
# support gen expressions


# link_libraries(item1 [item2...] [[debug | optimized | general] item] ...)
# for link libs to other targets (affect all targets after the call in this scope)
# item - can be target, lib name, full path, linker flag
# use of debug/optimized/general(by default) is discouraged, use gen expr instead

# link_directories(dir1 [dir2...])
# cmake adds these dirs to a linker command line (it must search libs on its own)
# relative path (from this source dir)
# there is no convenient way to remove this paths
# avoid to use it


# --- Compiler and linker variables ---

# To set custom features users can use variables instead of props
# These variables are usually cache vars (but can be ordinal var)
# This is no longer (from v3.0) a good model to use these vars

# compiler flags vars:

# CMAKE_<LANG>_FLAGS - for all build types
# CMAKE_<LANG>_FLAGS_<CONFIG> - for config
#   lang - C, CXX, Fortran, Swift ...
#   config - DEBUG, RELEASE, RELWITHDEBINFO, MINSIZEREL

# project() will create them at the first launch
# for c++:
#   CMAKE_CXX_FLAGS: (no flags)
#   CMAKE_CXX_FLAGS_DEBUG: -g -O0
#   CMAKE_CXX_FLAGS_RELEASE: -O3 -DNDEBUG
#   CMAKE_CXX_FLAGS_RELWITHDEBINFO: -O2 -g -DNDEBUG
#   CMAKE_CXX_FLAGS_MINSIZEREL: -Os -DNDEBUG

# linker flags vars:

# CMAKE_<TARGETTYPE>_LINKER_FLAGS
# CMAKE_<TARGETTYPE>_LINKER_FLAGS_<CONFIG>
# TARGETTYPE: EXE, SHARED, STATIC, MODULE

# common errors:
# 1. using lists instead of strings, use appending for several items:
#   set(CMAKE_CXX_FLAGS -Wall -Werror) # error - list
#   set(CMAKE_CXX_FLAGS "-Wall -Werror") # ok - string
#   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror") # ok - appending
#   string(APPEND CMAKE_CXX_FLAGS " -Wall -Werror") # ok - appending

# 2. distinguish between cache and non-cache:
#   - set(CMAKE_CXX_FLAGS "-Wall.." CACHE STRING "C++ flags") # has effect if it is not already in cache
#   - set(CMAKE_CXX_FLAGS "-Wall.." CACHE STRING "C++ flags" FORCE) # overwrites cache
#   - set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall.." CACHE STRING "C++ flags" FORCE) # force + append = disaster
# force + append -> causes flag string growing in each call (makes duplicates)
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall..") # preserves the cache, append new flags safely

# 3. Prefer appending over replacing flags - avoid replacing or do it near cmake_minimum_required()

# 4. Only the last save to CMAKE_CXX_FLAGS in CMakeLists.txt file matters
# notice, that include() can insert files that change such variables and influence on current scope


# --- Recommended practices ---
# Prefer target_* commands to describe relations and modify compiler/linker behavior
# Avoid dir props, if you use directory props - use it asap
# Querying target props can be useful to investigate unexpected compiler/linker flags
# Prefer to avoid modifying CMAKE_..._FLAGS (may be useful only for toolchains, ch21)
# Prefer start from PRIVATE in target_..() to reduce link time (PUBLIC - only if very needed)
# INTERFACE - for imported or interface lib targets,
#   all target_* (except target_link_libraries) can use a target from the other parts of project,
#   but single condition - if they already defined, thus compiler interface deps can be done from any part of project
#   and linker interface deps - only in the same dir.

