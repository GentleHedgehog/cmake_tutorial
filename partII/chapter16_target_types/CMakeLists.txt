# description:
# besides the ordinary targets, there are abstract targets (to hide and abstract complexities of platform diffs):
# - abstract targets as a reference to the other targets
#   (to collect and bear transitive props and deps without own binaries)
# - an abstract collection of object files (rather than static or shared lib)
# - custom targets (to execute arbitrary commands, define custom build rules)


# --- Executables ---

# add_executable(target [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] src1 [src2 ...])
#   it is GLOBAL by default (is visible everywhere in a project)

# add_executable(target IMPORTED [GLOBAL])
#   IMPORTED - to create target from an existing executable
#       - benefit: cmake can replace this target name to file path (for tests or custom builds)
#           so we need to set some props: IMPORTED_LOCATION or IMPORTED_LOCATION_<CONFIG>
#       - cannot be installed
#       - it is GLOBAL optionally (if no - directory scope and below)

# add_executable(alias ALIAS target)
#   to alias real targets (not the other aliases!)
#   can alias imported targets from v3.11 (but only the GLOBAL targets) (read-only ref)

# --- Libraries ---

# Object and other type of libs:
# add_library(target [STATIC | SHARED | MODULE | OBJECT] [EXCLUDE_FROM_ALL] src1 [src2...])
#       - can be linked (by target_link_libraries) only from v3.12
#       - using format: $<TARGET_OBJECTS:objlib>
#       - cannot provide transitive deps (before v3.12? when there was no linking for them)
#           (only a manual carry of search paths, defines, etc)
#       - from v3.12 object libs propagate transitively (like an ordinary lib),
#            but we need link to them directly only!!! (no transitive linking)
#       - static lib is easier to use than object lib



# Imported lib:
#   add_library(target (STATIC | SHARED | MODULE | OBJECT | UNKNOWN) IMPORTED [GLOBAL])
#       - ref to existing lib
#       - is often used in packaging or by Find module
#       - OBJECT type can be specified from v3.9
#       - UNKNOWN if lib has unknown type (cmake simply use the full path, no handling of DLL import libs for Win)
#       - location must be specified (IMPORTED_LOCATION[_<CONFIG>]) (except OBJECT case)
#           for Win: IMPORTED_LOCATION for dll and IMPORTED_IMPLIB should hold assoc.import lib (*.lib)
#       - srcs must be specified (IMPORTED_OBJECTS) for OBJECT case
#       - other props can be found in docs for IMPORTED_... props (but them are fairly uncommon)
#       - GLOBAL option works as in add_executable()

# examples:

#   add_library(myWinLib SHARED IMPORTED)
#   set_target_properties(myWinLib PROPERTIES
#           IMPORTED_LOCATION /some/path/bin/foo.dll
#           IMPORTED_IMPLIB /some/path/lib/foo.lib
#       )

#   add_library(mysteryLib UNKNOWN IMPORTED)
#   set_target_properties(mysteryLib PROPERTIES
#           IMPORTED_LOCATION ${FOO_LIB}
#       )

#   add_library(myWinLib OBJECT IMPORTED)
#   set_target_properties(myWinLib PROPERTIES
#           IMPORTED_OBJECTS /some/path/obj1.obj # *.o for most other platforms
#           IMPORTED_IMPLIB /some/path/obj2.obj
#       )
#   add_executable(myExe $<TARGET_OBJECTS:myWinLib>) # what is $<TARGET_SOURCES..> in the tutorial???



# Interface libs:
#  add_library(target INTERFACE [IMPORTED [GLOBAL]])
#       - are used to carry deps without physical libs (header-only lib for example)
#       - IMPORTED - usually arise when lib is exported or installed for use outside of the project
#       - summary table (except INTERFACE, all combinations have meaning for installing (ch25)):
#   keyword                     visibility      imported location   set interface props     installable
#   INTERFACE                   global          prohibited          any method              yes
#   IMPORTED                    local           required            restricted*             no
#   IMPORTED GLOBAL             global          required            restricted*             no
#   INTERFACE IMPORTED          local           prohibited          restricted*             no
#   INTERFACE IMPORTED GLOBAL   global          prohibited          restricted*             no
# * - using target_...() to set INTERFACE_... props - only from version v3.11
#       (set_property or set_target_property - in any version)


# examples:

#   add_library(myHeaderOnlyToolkit INTERFACE)
#   target_include_directories(myHeaderOnlyToolkit INTERFACE /some/path/include )
#   target_compile_definitions(myHeaderOnlyToolkit INTERFACE COOL_FEATURE=1 $<$<COMPILE_FEATURES:cxx_std_11>:HAVE_CXX11>)
#   add_executable(myApp ...)
#   target_link_libraries(myApp PRIVATE myHeaderOnlyToolkit)

# to work with library set (useful example to hide subtleties of library using):
#   add_library(algo_fast ...)
#   add_library(algo_accurate ...)
#   add_library(algo_beta ...)
#   add_library(algo_all INTERFACE)
#   target_link_libraries(algo_all INTERFACE
#       algo_fast
#       algo_accurate
#       $<$<BOOL:${ENABLE_ALGO_BETA}>:algo_beta>
#       )
#   add_executable(myApp ..)
#   target_link_libraries(myApp PRIVATE algo_all)



# Alias libs:
#  add_library(aliasName ALIAS otherTarget)
#   - read only ref to other targets
#   - cannot be installed, cannot alias aliases
#   - for global imported targets - only from v3.11

# example:

# common pattern for packaged or installed libs:
#   add_library(myRealThings SHARED src1.cpp ..)
#   add_library(otherThings STATIC srcA.cpp ..)
#   # these names can be used inside and outside the project:
#   add_library(BagOfBeans::myRealThings ALIAS myRealThings)
#   add_library(BagOfBeans::otherThings ALIAS otherThings)
#   ... in other project:
#   find_package(BagOfBeans REQUIRED) # see ch23
#   add_executable(eatLunch main.cpp ...)
#   target_link_libraries(eatLunch PRIVATE BagOfBeans::myRealThings)
#   if we want to incoporate BagOfBeans project, we just replace find_package with add_subdirectory(BagOfBeans)

#   Cmake treats namespace name (before ::) as a target name,
#   and it is useful when we make a mistake in a target name. If we point out 'bart' instead of 'bar':
#       target_link_libraries(main PRIVATE bart) # no error at config step (delayed for generation step)
#       target_link_libraries(main PRIVATE foo::bart) # error at config step



# --- Promoting Imported Targets ---
# Main intended use of imported targets (module/package) is to be visible in the directory scopy (locally),
# so that we can pull them in different parts of consuming project with different settings without any interferences.
# But sometimes we need to promote it to the global scope
# (to ensure the same version of package is used throughout the whole project).
# example:
#   add_library(builtElsewhere STATIC IMPORTED)
#   set_target_properties(builtElsewhere PROPERTIES
#           IMPORTED_LOCATION /path/to/libSomething.a
#       )
#   set_target_properties(builtElsewhere PROPERTIES
#       IMPORTED_GLOBAL TRUE
#   )
#  - we cannot cancel this settings (roll back to local visibility)
#  - target must be defined in the same scope (with this command)  (include(), find_package() satisfy this criterion)



# --- Recommended practices ---
# - prefer imported libs to variable with path to the same thing
# - prefer static libs to object libs
# - add namespace:: prefix to each non-private target
# - use constructions like 'add_library(algo_all INTERFACE)' to alias old names (during refactoring for ex.)
