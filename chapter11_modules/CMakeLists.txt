# description:

# two ways to use modules:

# 1. include module directly:
# include(module [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
# - module - base name, without full path or file extension (case sensitive)
#       cmake will look for the module in a well-defined set of locations
#       CMAKE_MODULE_PATH - list of dirs for looking (if empty or not found - look in a cmake internal directory)
#       Useful pattern - create 'cmake' folder near the beginning of the top level CMakeLists.txt (to keep all user modules)
#           then add this dir to the CMAKE_MODULE_PATH list:
#           list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
#           include(MyModule)
#       system modules will be searched in the cmake internal dir at first! (before CMAKE_MODULE_PATH)

# 2. second way - find_package(PackageName)
#   cmake will search for a file called FindPackageName.cmake
#   richer than include() - bring additional information: imported targets, location of relevant files and so on..


# -- some useful system cmake modules --

# CMakePrintHelpers - helps to print props and vars (for debugging in general)
# cmake_print_properties(
#           [TARGETS target1 [target2...]]
#           [SOURCES src1 [src2...]]
#           [DIRECTORIES dir1 [dir2...]]
#           [TESTS tst1 [tst2...]]
#           [CACHE_ENTRIES var1 [var2...]]
#           PROPERTIES prop1 [prop2...]
#        )
# it combines get_property() with message():
#   add_executable(myApp main.c)
#   add_library(myLib STATIC src.cpp)
#   include(CMakePrintHelpers)
#   cmake_print_properties(TARGETS myApp myLib PROPERTIES TYPE)
# output:
#   Properties for TARGET myApp:
#     myApp.TYPE = "EXECUTABLE"
#   Properties for TARGET myLib:
#     myLib.TYPE = "STATIC_LIBRARY"


set(foo "My variable")
unset(bar)

include(CMakePrintHelpers)
cmake_print_variables(foo bar CMAKE_VERSION)

# check endianness (use the cache to check once):

include(TestBigEndian)
test_big_endian(isBigEndian)
message("Is target system big endian: ${isBigEndian}")

# check existence and support:

# CheckC[XX]SourceCompiles
# check_c[xx]_source_compiles(code resultVar [FAIL_REGEX regex])
#   compiles code and save compilation&link result to resultVar
#   if output of compile/link matches regex - the check will be deemed to have failed

include(CheckCSourceCompiles)
check_c_source_compiles("
    int main(int argc, char* argv[])
    {
        int myVar;
        return 0;
    }" noWarnUnused FAIL_REGEX "[Ww]arn")
if(noWarnUnused)
    message("Unused variables do not generate warnings by default")
endif()

# before compilation these vars can be set (for try_compile() inside):
# specified as string:
# - CMAKE_REQUIRED_FLAGS - single string with additional flags (with spaces),
#       main flags are in CMAKE_<LANG>_FLAGS and CMAKE_<LANG>_FLAGS_<CONFIG>
# specified as lists:
# - CMAKE_REQUIRED_DEFINITIONS: -DFOO or -DFOO=bar
# - CMAKE_REQUIRED_INCLUDES: paths to dirs with headers
# - CMAKE_REQUIRED_LIBRARIES: list of libs (dont prefix names with any -l(or similar) option, just the lib name)
# - CMAKE_REQUIRED_QUIET: if present - no status message


# check a result of the binary execution (compiled from the source):
# include(CheckC[XX]SourceRuns)
# check_c[xx]_source_runs(code rusultVar) # 0 exit code ~ success

# it macro uses try_run()
# for cross-compilation - there are a lot of subtleties with try_run() (simulation, predetermined result, etc..)


# include(CheckC[XX]CompilerFlag)
# check_c[xx]_compiler_flag(flag resultVar) # resultVar is true, if a flag can be accepted by a compiler (see ch14.3)
#   this macro updates CMAKE_REQUIRED_DEFINITIONS in a call to
#   check_c[xx]_source_compiles and look for this flag in output warnings (presence in the warning ~ false result)


# include(CheckSymbolExists)
# check_symbol_exists(sprintf stdio.h[other headers..] HAVE_SPRINTF)
# includes headers to a trivial source and checks for symbols: pre-processor symbol, function, variable
# if symbol is in library - set the CMAKE_REQUIRED_LIBRARIES var


# to isolate different checks (state of the CMAKE_REQUIRED_* vars set):
# include(CMakePushCheckState)
# cmake_push_check_state([RESET]) -> to virtual stack (optionally reset the set, from v3.10)
# cmake_pop_check_state() -> from virtual stack
# cmake_reset_check_state() -> clear values in the set


# --- Other modules ---
# - FindCUDA, FindJava, etc..

# CTest, CPack - see ch24, ch26


# --- Recommended practices ---
# prefer CMAKE_MODULE_PATH to hardcoded paths
