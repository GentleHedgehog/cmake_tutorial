# description:
# - a property is always attached to an entity
# - it has a common prefix: CMAKE_*
# - all properties are well definded and documented by CMake



# -- general property commands --:

# set_property(entitySpecific [APPEND] [APPEND_STRING] PROPERTY propName [value1 [value2 [...]]])
# entitySpecific:
# - GLOBAL                  # build itself
# - DIRECTORY [dir]         # if no dir - current source dir
# - TARGET [target1 [..]]
# - SOURCE [src1 [..]]
# - INSTALL  [file1 [..]]
# - TEST [test1 [..]]
# - CACHE [var1 [..]]
# APPEND & APPEND_STRING:
# - neither specified - the given value replace any previous value
# - APPEND - appends new value to the old one (as a list) (a;b append c;d = a;b;c;d)
# - APPEND_STRING - concatenates values (as a string) (if an old value was a list: a;b + c;d = a;bc;d)

# get_property(resultVar entitySpecific PROPERTY propName [DEFINED | SET | BRIEF_DOCS | FULL_DOCS])
# entitySpecific:
# - GLOBAL
# - DIRECTORY [dir]
# - TARGET target
# - SOURCE src
# - INSTALL  file
# - TEST test
# - CACHE var
# - VARIABLE -  specifying the current scop, and property of interest is the variable named by propName (?)
#   this case is relatively uncommon (common case is to use ${} syntax)
# optional keywords:
# - DEFINED - return true if a property has been defined
# - SET - true if a property was set (it can be defined but not set, and vice versa)
# - BRIEF_DOCS - retrieves the brief documentation string (NOTFOUND if no docs)
# - FULL_DOCS - retrieves the full documentation  (NOTFOUND if no docs)

# rarely used command (to populate info for entity):
# define_property(entityType
#                 PROPERTY propName [INHERITED] # INHERITED - will chain-up to the parent scope (not for VARIABLE and CACHE)
#                 BRIEF_DOCS briefDoc [moreBriefDocs...]
#                 FULL_DOCS fullDoc [moreFullDocs...])
# This command does not set the property’s value, only its documentation and whether
# or not it inherits its value from elsewhere if it has not been set.
# The entityType must be one of: GLOBAL, DIRECTORY, TARGET, SOURCE, TEST, VARIABLE or CACHED_VARIABLE



# --- global properties ---
# for providing some degree of build-level information

# get_cmake_property(resultVar property) # to retrieve the value of any global property
# property can be the name of any global prop or pseudo property (can be retrieved only by this cmd):
# - VARIABLES - return list of all regular (non-cache) vars
# - CACHE_VARIABLES - ret list of all cache vars
# - COMMANDS - list of all defined commands/funcs/macros/
# - MACROS - list of all macros only
# - COMPONENTS - list of all components defined by install() commands



# --- directory props ---
# override global props and set default prop's vals for targets

# the dedicated commands:

# set_directory_properties(PROPERTIES prop1 val1 [..])
# applies only for the current dir

# get_directory_property(resultVar [DIRECTORY dir] property)
# get_directory_property(reusltVar [DIRECTORY dir] DEFINITION varName) # only for debug (retrieve var from another dir)
# dir - must be encountered by CMake



# --- Target properties ---
# define all target traits

# the dedicated commands:

# set_target_properties(target1 [..]
#                       PROPERTIES
#                       propName1 val1 [..])
# get_target_property(resultVar target propName)

# family of target_..() commands - very helps to work with targets (defines properties and propagation info for the other linked targets)



# --- Source properties ---
# define file-based props (how to compile, link, etc.)

# the dedicated commands:

# set_source_file_properties(file1 [..]
#                            PROPERTIES
#                            propName1 val1 [..])
# get_source_file_property(resultVar sourceFile propName)

# these props are only visible to targets in the same dir scope !!!
# use carefully! Some generators (Unix Makefiles, for ex.) propagate a source prop of the file to the whole target!



# --- Cache var properties ---
# are used to configurate how cache vars are handled in the GUI and the ccmake tools

# use the prop name TYPE (in get_property()) to obtain it
# set ADVANCED prop ~ mark_as_advanced()
# set HELPSTRING prop for the tooltip
# set STRINGS prop for the combobox



# -- Other props ---
# set_tests_properties(test1 [..]
#                       PROPERTIES
#                       propName1 val1 [..])
# get_test_property(resultVar test propName)


# --- Recommended Practices ---
# use dedicated commands (instead of set_property()) if you dont need to append values
# use target_..() commands (instead of manipulating the associated target props directly)
# use source props carefully (or use alternatives: ch19.2 - "source code access to version details")
