message("------- chapter 7 - using subdirs ----------")

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
