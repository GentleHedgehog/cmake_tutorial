# description:
# introduce methods to do custom operations at the configure step: post-processing, copying, etc..



# --- custom targets ---

# custom target performs a sequence of commands at build time
#   add_custom_target(target [ALL]
#       [cmd1 [args1 ..]]
#       [COMMAND cmd2 [args2 ..]]
#       [DEPENDS depends1...]
#       [BYPRODUCTS files...]
#       [WORKING_DIRECTORY dir]
#       [COMMENT comment]
#       [VERBATIM]
#       [USES_TERMINAL]
#       [SOURCES src1 [src2...]]
#       )
# - target is always considered out of date
# - ALL - causes the 'all' target to depend from this target
# - COMMAND - prefer to begin with this word before each command (even before the first cmd)
#   commands are executed in their order (but dont assume that they are executed within one shell environment)
#   command is an anything action on the target platform:
#       - run script,
#       - run another target of this project (as part of the build)
#         cmake substitute path, but it wont do this for arguments, use TARGET_FILE:
#               add_executable(hasher hasher.cpp)
#               add_library(myLib api.cpp)
#               add_custom_target(createHash COMMAND hasher $<TARGET_FILE:myLib>)
#         cmake creates the dependency (ex: createHash depends on hasher, genexpr creates dependency for myLib too)
#         dependency can be added manually by add_dependencies()  #
# - DEPENDS - dependency from a file can be added by this keyword (use absolute path only!)
#       it is very useful when there is a dependency from a file created by another custom target
# - VERBATIM - escaping only at the step of parsing CMakeLists.txt (to reduce diffs in using diff shells)
# - WORKING_DIRECTORY - absoule or relative path (relative to ${CMAKE_CURRENT_BINARY_DIR})
# - BYPRODUCTS - lists by-product files (as part of cmd running) (cmake marks them as GENERATED for different generator tools)
#       cosider using of add_custom_command() too
# - COMMENT - cmake prints a comment before a command execution (some generators dont support it)
# - USES_TERMINAL - give the access to terminal for a command (from v3.2) (example: for interactive input)
# - SOURCES - files that are loosely related to target (to show in the IDE project window, for example)



# --- Add build steps to an existing target ---

# custom commands can be used to specify additional steps to existing target
#   add_custom_command(TARGET targetName buildStage
#                      COMMAND cmd1 [args1...]
#                      [COMMAND cmd2 [args2...]]
#                      [WORKING_DIRECTORY dir]
#                      [BYPRODUCTS files...]
#                      [COMMENT comment]
#                      [VERBATIM]
#                      [USES_TERMINAL]
#                     )
# - targetName - exe, lib or custom target
# - buildStage:
#       - PRE_BUILD - run before any other rules (only from VisualStudio 7)
#       - PRE_LINK - run after compilation but before the linking step (static lib - before archiver tool, custom target - not supported)
#       - POST_BUILD - run after all rules (prefer it)

# we can call add_custom_command() several times to append multiple sets of custom commands
#   add_executable(myExe main.cpp)
#   add_custom_command(TARGET myExe POST_BUILD
#       COMMAND script1 $<TARGET_FILE:myExe>
#   )
#   #additional cmd which will run after the above from a diff dir
#   add_custom_command(TARGET myExe POST_BUILD
#       COMMAND writeHash $<TARGET_FILE:myExe>
#       BYPRODUCTS ${CMAKE_BINARY_DIR}/verify/myExe.md5
#       WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/verify
#   )



# --- Commands that generate files ---

# to create files without any target:
#   add_custom_command(OUTPUT output1 [output2...]
#       COMMAND cmd1 [args1...]
#       [COMMAND cmd2 [args2...]]
#       [WORKING_DIRECTORY dir]
#       [BYPRODUCTS files...]
#       [COMMENT comment]
#       [VERBATIM]
#       [USES_TERMINAL]
#       [APPEND]
#       [DEPENDS depends1...]
#       [MAIN_DEPENDENCY depend]
#       [IMPLICIT_DEPENDS <lang1> dep1 [<lang2> dep2...]]
#       [DEPFILE depfile]
#       )
# - OUTPUT - out files' names (abs or rel(to current bin dir) paths)
#   if some target (in the same dir scope) depends from these files, cmake creates dependency automatically:
#       add_executable(myExe main.cpp)
#       # out file with rel path (in build dir):
#       add_custom_command(OUTPUT myExe.md5 COMMAND writeHash $<TARGET_FILE:myExe>)
#       # hash will be computed only when we build 'computeHash' target (not when we build myExe)
#       add_custom_target(computeHash DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/myExe.md5)
# - DEPENDS - deps from files and targets(!) (use abs paths for files too)
#   point out files here that are needed by custom command (common error - omit file deps for custom command)
# - MAIN_DEPENDENCY - main dependency src file (for command)
# - IMPLICIT_DEPENDS (for Makefiles gen) and DEPFILE (for Ninja) - gen-specific options (try to avoid these options)
# - APPEND - requirements: first out file must be the same in subsequent calls,
#               only COMMAND/DEPENDS/COMMENT/WORKING_DIRECTORY can be used in subsequent calls

#   example of generating src file:
#       add_executable(generator gen.cpp)
#       add_custom_command(OUTPUT onTheFly.cpp COMMAND generator) # will be executed ONLY if onTheFly.cpp does NOT exist!!! or use DEPENDS
#       add_executable(myExe ${CMAKE_CURRENT_BINARY_DIR}/onTheFly.cpp)

#   example of an error when main deps is used:
#       # custom command becomes a replacement for original.cpp compilation rule:
#       add_custom_command(OUTPUT transformed.cpp
#               COMMAND transform
#                       ${CMAKE_CURRENT_SOURCE_DIR}/original.cpp
#                       transformed.cpp
#               # change to DEPENDS to fix error (but there will not default compilation rule for src):
#               MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/original.cpp
#           )
#       # produces linker error (build of original.cpp doesnt produce an object file):
#       add_executable(original original.cpp)
#       add_executable(transformed transformed.cpp)



# --- Configure time tasks ---






# --- Recommended practices ---
