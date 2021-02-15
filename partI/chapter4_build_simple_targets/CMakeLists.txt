# description:

# add_executable(tutorialExe main.cpp) # several commands can produce diffrent target names
# [WIN32] - windows gui app: WinMain(), link to /SUBSYSTEM:WINDOWS
# [EXCLUDE_FROM_ALL] - by default cmake build 'all' target,
#and we can exclude this target from 'all' (for example: for a rarely needed developer tool)

#add_library(targetName [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 [source2 ..])
# STATIC - targetName.lib (win), targetName.a (unix)
# SHARED - targetName.dll (.so)
# MODULE - like plugin (.dll on win), loaded at runtime
# if no type: cmake -DBUILD_SHARED_LIBS=YES /path/to/source, or set(BUILD_SHARED_LIBS YES) before add_library


#linking libs:
# * PRIVATE - dependency of lib A from lib B (internal dependency, A can be used without B)
# PUBLIC - A use B in its interface (A cannot be used without B, for ex: func in A has arg type from B) (used by default)
# INTERFACE - A doesnt require B internally, only in its interface
#   target_link_libraries(targetName
#    <PRIVATE|PUBLIC|INTERFACE> item1 [...]
#    ...
#    )
# * !!! targetName must have been defined by add_executable or add_library
#in the same directory from which target_link_libraries is being called
# * can be specified as items:
# full path to lib file; lib name like foo (becomes -lfoo or foo.lib) (often system libs); link flag (prefer for PRIVATE items)
# * each item can be preceded by (but it is deprecated): debug, optimized or general (depends from build type)
# oprimized ~ is not a debug build, general ~ for all build configs (default)

# do not use project name as target name! name target according to what the target does
# do not use prefix 'lib' for libs! it can be added by system
# prefere not to use SHARED|STATIC for libs (use BUILD_SHARED_LIBS)
# always use lib item specifiers: PRIVATE, PUBLIC or INTERFACE
