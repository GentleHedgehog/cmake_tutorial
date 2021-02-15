# description:

# CMake working steps:
# 1. configure step - reads CMakeLists.txt (and files it pulls in) and execute all commands in it
# 2. generation step - creates build tool's project files

# step 2 is important for complex generators, they can have multi configuration setup (Release, Debug..)
# Choise of the setup will be selected after configure step (1), and generator expressions can help us



# --- simple boolean logic ---

# syntax: $<...>

# $<1:...> # result - ...
# $<0:...> # result - empty string (ignored)
# $<BOOL:...> # works like casting to boolean

# provide simple way to include content:
# $<AND:expr[,expr..]> # consider wrapping this expr in a $<BOOL:...> to force more tolerant logic
# $<OR:expr[,expr..]>
# $<NOT:expr>
# expressions may be nested in any part of another expression

# since v3.8:
# $<IF:expr,val1,val0> # ~ $<expr:val1>$<$<NOT:expr>:val0>

# !!! dont use true or false directly, use - $<BOOL:true>, or 1/0

# ex:
# $<IF: $<BOOL:${foo}>, yes, no> # result depends on ${foo}

# other comparisons:
# $<STREQUAL: str1, str2>
# $<EQUAL: num1, num2>
# $<VERSION_EQUAL: ver1, ver2>
# $<VERSION_GREATER: ver1, ver2>
# $<VERSION_LESS: ver1, ver2>

# test the build type:
# $<CONFIG:arg>
# target_link_libraries(myApp PRIVATE $<IF:$<CONFIG:Debug>,lib1,lib2>)

# see documentation for other expressions



# --- Target details ---

# $<TARGET_PROPERTY: target, property> # prop of named target
# $<TARGET_PROPERTY: property> # prop of target on which genexpr is being used

# more direct and useful expr for targets:
# TARGET_FILE - absolute path to target bin file (includes all prefixes and extensions)
# TARGET_FILE_NAME - same without the path
# TARGET_FILE_DIR - same without the file name

# provides some library-specific expressions: TARGET_LINKER_FILE, TARGET_SONAME_FILE, TARGET_PDB_FILE...

# Object library - just a collection of object files (without a library file creation)
# add_library(objlib OBJECT src1.cpp src2.cpp) # define object library
# add_executable(app1 app1.cpp $<TARGET_OBJECTS:objlib>) # share objlib
# add_executable(app2 app2.cpp $<TARGET_OBJECTS:objlib>) # share objlib
# this method helps to avoid a recompilation, build time cost of creating a static lib, run time cost of symbol resolution for dll

# --- General information ---

# $<CONFIG> - build type, use it in preference to the CMAKE_BUILD_TYPE (this var is not used in multi config projects, Xcode, VisualStudio)
# $<PLATFORM_ID> - identifies the platform (closely related to CMAKE_SYSTEM_NAME)
# $<C[XX]_COMPILER_VERION> :
#   $<$<VERSION_LESS:$<CXX_COMPILER_VERSION>,4.2.0>:OLD_COMPILER> - produces string OLD_COMPILER if it's version less than 4.2.0
# $<LOWER_CASE:...>, $<UPPER_CASE:...> - useful for string comparisons



# --- Recommended practices ---
# keep the balance when use genexpr
# use it when there is no simple alternative in CMake
# use it when you need to rely on the build configuration
