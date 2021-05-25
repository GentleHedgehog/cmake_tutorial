# --- Description ---

# this chapter describes an efficient way to produce versions

# --- 1. Project version ---

# in root CMakeLists.txt:
#   cmake_minimum_required(VERSION 3.0)
#   project(GooBar)
#   set(GooBar_VERSION 2.4.7)

# if indiv component needs to be extracted:
#   cmake_minimum_required(VERSION 3.0)
#   project(GooBar)
#   set(GooBar_VERSION_MAJOR 2)
#   set(GooBar_VERSION_MINOR 4)
#   set(GooBar_VERSION_PATCH 7)
#   set(GooBar_VERSION ${GooBar_VERSION_MAJOR}.${GooBar_VERSION_MINOR}.${GooBar_VERSION_PATCH})

# if we set version by project(foobar VERSION x.x.x[.x]) cmd, cmake populates next vars:
#   PROJECT_VERSION, projectName_VERSION
#   PROJECT_VERSION_MAJOR, projectName_VERSION_MAJOR
#   PROJECT_VERSION_MINOR, projectName_VERSION_MINOR
#   PROJECT_VERSION_PATCH, projectName_VERSION_PATCH
#   PROJECT_VERSION_TWEAK, projectName_VERSION_TWEAK
# projectName_* - to obtain in the current dir or below (project-specific, most unambiguous)
# PROJECT_* - to obtain after the most recent call to project() cmd
# from v3.12, for the top level project() call, cmake populates:
#   CMAKE_PROJECT_VERSION
#   CMAKE_PROJECT_VERSION_*
# CMAKE_PROJECT_* vars are equivalent to projectName_* (?)
# other useful vars (from v3.9, v3.12) for documentation,packaging:
#   PROJECT_NAME
#   PROJECT_DESCRIPTION
#   PROJECT_HOMEPAGE_URL
#   PROJECT_*


# --- 2. Source code access to version ---

# first method (force all project recompilation):
#   cmake_minimum_required(VERSION 3.0)
#   project(FooBar VERSION 2.4.7)
#   add_definitions(-DFOOBAR_VERSION=\"${FooBar_VERSION}\")
#   ...
#   void printVersion() {std::cout << FOOBAR_VERSION << std::endl;}

# second method (force part of project recompilation):
#   cmake_minimum_required(VERSION 3.0)
#   project(FooBar VERSION 2.4.7)
#   add_executable(foobar main.cpp src1.cpp)
#   get_source_file_property(defs src1.cpp COMPILE_DEFINITIONS)
#   list(APPEND defs "FOOBAR_VERSION=\"${FooBar_VERSION}\"")
#   set_source_files_properties(src1.cpp PROPERTIES COMPILE_DEFINITIONS ${defs})

# by configurable header, foobar_version.h.in:
#   std::string getFooBarVersion(){return "@FooBar_VERSION@";}
#   unsigned getFooBarVersionMajor(){return @FooBar_VERSION_MAJOR@;}
#   unsigned getFooBarVersionMinor(){return @FooBar_VERSION_MINOR@ +0;}
#   ...
#   cmake_minimum_required(VERSION 3.0)
#   project(FooBar VERSION 2.4.7)
#   configure_file(foobar_version.h.in, foobar_version.h @ONLY)
#   add_executable(foobar main.cpp)
#   target_include_directories(foobar PRIVATE "${CMAKE_CURRENT_BINARY_DIR}")

# to improve this method (add cpp and create static lib):
#   cmake_minimum_required(VERSION 3.0)
#   project(fooBar VERSION 2.4.7)
#   configure_file(foover.cpp.in foover.cpp @ONLY)
#   add_library(foover STATIC ${CMAKE_CURRENT_BINARY_DIR}/foover.cpp)
#   add_executable(foobar main.cpp)
#   target_link_libraries(foobar PRIVATE foover)
#   add_library(fooToolkit mylib.cpp)
#   target_link_libraries(fooToolkit PRIVATE foover)

# --- 3. Source control commits ---

# provide git hash:
#   std::string getHash() {return "@FooBar_GIT_HASH@";}
#   ...
#   find_package(Git REQUIRED) # look at ch 23
#   execute_process(
#       COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
#       RESULT_VARIABLE result
#       OUTPUT_VARIABLE FooBar_GIT_HASH
#       OUTPUT_STRIP_TRAILING_WHITESPACE
#       )
#   if(result)
#       message(FATAL_ERROR "Failed to get git hash: ${result}")
#   endif()
#   configure_file(foover.cpp.in foover.cpp @ONLY)

# commits history from last version change... look at the book

# --- Recommended practices ---
# major change - break of backward compatibility
# minor change - add new features
# patch - like fix some bugs (no new funcs)
# tweak - often omitted, less significant than patch
# avoid to set custom vars like *_VERSION[_*]
# consider to use a separate cmake file to define the version by include()
#   this provides the more transparent way to keep the version file in CVS
# consider using of a static lib to keep the version details (in approp.namespace)


