# description:
# build type ~ build configuration ~ build scheme
# high level control of the build (compiler/linker settings)

# --- basics ---
# types:
# - Debug - no optimization, full debug info
# - Release - full optimization, no debug info, for final production
# - RelWithDebInfo - compromise (some optimization / some debug info), note - it disables assertions by default!
# - MinSizeRel - optimization for size only, no debug info (for embedded developement)

# --- single configuration generators --- (SCG)
# some gens (Makefiles, Ninja) support only a single build type per dir
# we should set cache var CMAKE_BUILD_TYPE:
#   cmake -G Ninja -DCMAKE_BUILD_TYPE:STRING=Debug ../source
#   cmake --build .
# or we can create different build folders and switch: build_Debug, build_Release, etc..
# !work at CONFIGURE time!

# --- multiple configuration generators --- (MCG)
# some gens (Xcode, Visual Studio) support multiple build types per dir
# they ignore CMAKE_BUILD_TYPE:
#   cmake -G Xcode ../source
#   cmake --build . --config Debug
# !work at BUILD time!

# --- Common Errors ---
# - use carefully the logic with CMAKE_BUILD_TYPE in CMakeLists.txt (it can be useless for multi config gens)
#       use $<CONFIG:...> expression instead
# - to script build for all type of gens, use a combined method:
#       mkdir build
#       cd build
#       cmake -G Ninja -DCMAKE_BUILD_TYPE=Release ../source
#       cmake --build . --config Release
#           user can change Ninja to another name without problems
# - if CMAKE_BUILD_TYPE is not set -> build type is empty -> compiler/linker will be used with a default set of flags
#       (usually with debug flags, but there is no guarantee)


# --- Custom Build Types ---

# developer can extend or reduce a set of build types
# current build type can be set by:
#   - CMAKE_BUILD_TYPE cache var for SCG (only)
#   - drop-down list entries for MCG

# CMAKE_CONFIGURATION_TYPES cache var is populated during a project() command execution
# content of this var can be edited
# before v3.11 adding custom build types is not safe
# before v3.9 this var was used to determine MCG
# from v3.9 there is a new global property -> GENERATOR_IS_MULTI_CONFIG

# how to add custom type:
#   cmake_minimum_required(3.11)
#   project(Foo)
#   if(CMAKE_CONFIGURATION_TYPES)
#       if(NOT "Profile" IN_LIST CMAKE_CONFIGURATION_TYPES)
#           list(APPEND CMAKE_CONFIGURATION_TYPES Profile)
#       endif()
#   endif()
#   # then set Profile-specific flag variables (if not already set)...

# how to for SCG:
#   set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release Profile)
#   this command influences only on GUI representation (user can edit or use another type)

# default value for CMAKE_BUILD_TYPE is empty

# universal method to edit such vars:
#   cmake_minimum_required(3.11)
#   project(Foo)
#   if(CMAKE_CONFIGURATION_TYPES)
#       if(NOT "Profile" IN_LIST CMAKE_CONFIGURATION_TYPES)
#           list(APPEND CMAKE_CONFIGURATION_TYPES Profile)
#       endif()
#   else()
#       set(allowableBuildTypes Debug Release Profile)
#       set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${allowableBuildTypes}")
#       if (NOT CMAKE_BUILD_TYPE)
#           set(CMAKE_BUILD_TYPE Debug CACHE STRING "" FORCE)
#       elseif(NOT CMAKE_BUILD_TYPE IN_LIST allowableBuildTypes)
#           message(FATAL_ERROR "Invalid build type: ${CMAKE_BUILD_TYPE}")
#       endif()
#   endif()
#   # then set Profile-specific flag variables (if not already set)...

# vars to select build flags:
#   CMAKE_<LANG>_FLAGS_<CONFIG>
#   CMAKE_<TARGETTYPE>_LINKER_FLAGS_<CONFIG>
#      they are used to add extra flags (main flags are added by same vars without the '_<CONFIG>' suffix)
#   example for profiling with debug info and most optimization:
#       set(CMAKE_C_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#       set(CMAKE_CXX_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#       set(CMAKE_EXE_LINKER_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#       set(CMAKE_SHARED_LINKER_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#       set(CMAKE_STATIC_LINKER_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#       set(CMAKE_MODULE_LINKER_FLAGS_PROFILE "-p -g -O2" CACHE STRING "")
#   instead of "-p -g -O2" we can insert: "${CMAKE_<TARGETTYPE>_[LINKER_]FLAGS_RELWITHDEBINFO} -p"

# another var: CMAKE_<CONFIG>_POSTFIX
# it can be used to set postfix for multiple build types in the same dir:
# set(CMAKE_PROFILE_POSTFIX _profile)

# items in target_link_libraries() can be prefixed whith 'debug' or 'optimized' keywords
# to link for debug or non-debug builds
# debug build types are listed in DEBUG_CONFIGURATIONS global property:
#   set_property(GLOBAL PROPERTY APPEND DEBUG_CONFIGURATIONS StrictChecker)
#   this command update this prop for a custom build type 'StrictChecker'


# --- Recommended practices ---
# dont assume xCG type (SCG or MCG)
# use multiple dirs for SCG
# for SCG: consider set CMAKE_BUILD_TYPE to a better default val if it empty, but prefer generator expression
# for MCG: change CMAKE_CONFIGURATION_TYPES as regular var (not cache), if it is not empty
# from v3.9 use GENERATOR_IS_MULTI_CONFIG global prop to query gen type
# instead of using the LOCATION target prop, use $<TARGET_FILE:...> to get path (works for all gen types)

