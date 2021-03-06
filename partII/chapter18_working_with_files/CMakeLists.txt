# --- Description ---

# cmake provides a variety of features to work with files:
#   - constructing paths to files
#   - obtaining a list of files from a directory
#   - generation of a file from different sources
#   - calc hash, checksum of a file
#   - reading of files


# --- 1. Manipulating paths ---

# get_filename_component(outVar input component [BASE_DIR dir] [CACHE])
# component can be:
#   - DIRECTORY - extract dir (PATH is synonym from old versions before 2.8.12)
#   - NAME - file name with extension
#   - NAME_WE - only file name (before first ".")
#   - EXT - extension (after first ".")
#   for BASE_DIR:
#       - ABSOLUTE - compute absolute path of input without resolving symbolic links
#       - REALPATH - compute absolute path of input with resolving symbolic links
# relative paths in input can be absolute or:
#   - relative to current source dir or CMAKE_CURRENT_SOURCE_DIR
#   - relative to dir, if there is BASE_DIR
# CACHE - result var is stored as a cache var (not often required)

set(input /some/path/foo.bar.txt)
get_filename_component(out ${input} DIRECTORY)
message(${out})
get_filename_component(out ${input} PATH)
message(${out})
get_filename_component(out ${input} NAME)
message(${out})
get_filename_component(out ${input} NAME_WE)
message(${out})
get_filename_component(out ${input} EXT)
message(${out})

set(input some/relative/path/foo.bar.txt)
get_filename_component(out ${input} ABSOLUTE BASE_DIR "C:/")
message(${out})
get_filename_component(out ${input} REALPATH BASE_DIR "C:/Temp" )
message(${out})


# get_filename_component(progVar input PROGRAM [PROGRAM_ARGS argVar] [CACHE])
# this form - to extract parts of command line with arguments
# progVar - result with a full path to exe in command line (from PATH env)
# argVar - sorted list of args



# file(RELATIVE_PATH outVar relativeToDir input)
# converts an absolute path to the realtive
set(basePath /base)
set(fooBarPath /base/foo/bar)
set(otherPath /other/place)

file(RELATIVE_PATH fooBar ${basePath} ${fooBarPath})
message(${fooBar})
file(RELATIVE_PATH other ${basePath} ${otherPath})
message(${other})


# file(TO_NATIVE_PATH input out)
# file(TO_CMAKE_PATH input out)
file(TO_NATIVE_PATH "/C/temp" out)
message(${out}) # /C/temp ??
file(TO_CMAKE_PATH "C:\\Temp" out)
message(${out})


# --- 2. Copying files ---
message(--- Copying files ---)

# cmake has special functions to do it

# -- copying at CONFIGURATION time:

# configure_file(source dest [COPYONLY | @ONLY] [ESCAPE_QUOTES])
# it works at configure time, and rerun if source becomes out of date
# all vars with formatting ${var} or @var@ will be replaced in the out file
# source - file (abs / rel to cur src dir)
# dest - file name / path (abs / rel to cur src dir) (cmake will attempt to create the path)
# ESCAPE_QUOTES - causes any substituted quotes to be preceded with a backslash
#   example: ("a \"b\" c" -> a \"b\" c)
# @ONLY - causes replacement only for vars with format @var@
# COPYONLY - copy without any substitutions


# next func provides the same functionality:
# string(CONFIGURE input out [@ONLY] [ESCAPE_QUOTES])

# another alternative:
# file(<COPY|INSTALL> fileOrDir1 [fileOrDir2...]
#     DESTINATION dir
#     [NO_SOURCE_PERMISSIONS |
#         USE_SOURCE_PERMISSIONS |
#         [FILE_PERMISSIONS permissions...]
#         [DIRECTORY_PERMISSIONS permissions...]]
#     [FILES_MATCHING]
#     [PATTERN pattern | REGEX regex]
#     [EXCLUDE]
#     [PERMISSIONS permissions...]
#     [...]
#     )
# COPY
#   - preserves original permissions (can be overrided)
# INSTALL
#   - doesnt preserve original permissions (can be overrided)
#   - print messages
# both preserve timestamps, do copy only if there isnt a file with a such timestamp
# FILES_MATCHING, example:
#   REGEX .*_private\\.h EXCLUDE
#   PATTERN *.h
#   PATTERN *.sh
# PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE (can be used after PATTERN or REGEX for subset of files)


# file(COPY base/srcDir DESTINATION destDir) # --> destDir/srcDir (copy folder)
# file(COPY base/srcDir/ DESTINATION destDir) # --> destDir (copy the content of folder)



# -- copying at BUILD time:

# cmake -E copy file1 [file2...] dest
# it will update files in dest always (regardless of timestamp),
#   or use copy_if_different if timestamps should not be updated
# dest:
#   - is a dir, or a file name (if src has only one file)
#   - if it is a dir (abs/rel) - cmake try to create it (if src has only one file)

# add_custom_target(copyOne
#     COMMAND ${CMAKE_COMMAND} -E copy a.txt output/textfiles/a.txt
# )
# add_custom_target(copyTwo
#     COMMAND ${CMAKE_COMMAND} -E make_directory output/textfiles
#     COMMAND ${CMAKE_COMMAND} -E copy a.txt b.txt output/textfiles
# )

# to copy dirs, use copy_directory instead (it copies src dirs content only and creates dest path always)



# --- 3. Reading and writing files directly ---

# WRITING:
#   set(msg "hello world")
#   file(WRITE hello.txt msg)
#   file(APPEND hello.txt " from CMake")

# to add text with new lines:
#   file(WRITE multi.txt "First line
#       Second line
#       ")
# or use bracket syntax from ch 5.1: [[ ... ]] or [=[ .. ]=] (from cmake v3.0)

# write file whose contents depend on the build type:
#   file(GENERATE # all args support gen expr
#       OUTPUT outFile
#       INPUT inFile | CONTENT content # only one variant accepted
#       [CONDITION expression] # to skip build type for which expression evaluates to 0
#       )
# will replace the target file if only contents differ (like file(COPY...))
# works at gen phase (we cannot use out files at the configure phase, only at the build phase)
# before cmake v3.10 rel paths are not accurately specified (usually relative to cmake working dir),
#   but after v3.10 - OUTPUT ~ relative to cur bin dir, INPUT ~ relative to cur src dir
# example:
#   # gen unique files for all but Release:
#   file(GENERATE
#       OUTPUT
#       ${CMAKE_CURRENT_BINARY_DIR}/outFile-$<CONFIG>.txt
#       INPUT
#       ${CMAKE_CURRENT_SOURCE_DIR}/input.txt.in # any gen expr will be evaluated inside input file
#       CONDITION $<NOT:$<CONFIG:Release>>
#       )
#   # embedded content, bracket syntax doesnt prevent the use of gne expr:
#   file(GENERATE
#       OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/details-$<CONFIG>.txt
#       CONTENT [[
#       Built as "$<CONFIG>" for platform "$<PLATFORM_ID>"
#       ]])



# READING:
#   file(READ fileName outVar [OFFSET offset] [LIMIT byteCount] [HEX])
#   OFFSET - to read by the offset
#   LIMIT - to read only byteCount bytes
#   HEX - convert data to the hex repr (for binary files)

# to read lines as a list:
#   file(STRINGS fileName outVar
#       [LENGTH_MAXIMUM maxBytesPerLine]
#       [LENGTH_MINIMUM minBytesPerLine]
#       [LIMIT_INPUT maxReadBytes]
#       [LIMIT_OUTPUT maxStoredBytes]
#       [LIMIT_COUNT maxStoredLines]
#       [REGEX regex]
#       )

# read by regexp:
#   file(STRINGS myStory.txt versionLines REGEX "(PKG|MODULE)_VERSION")
#   will read lines that contain either PKG_VERSION or MODULE_VERSION
#   can be combined with LIMIT_COUNT - to obtain just the first match
# example to find version:
#   set(regex "^ *FOO_VERSION *= *([^ ]+) *$")
#   file(STRINGS config.txt fooVersion REGEX "${regex}")
#   string(REGEX REPLACE "${regex}" "\\1" fooVersion "${fooVersion}")
#   result: if config.txt contains FOO_VERSION = 2.3.5, then fooVersion = 2.3.5


# --- 4. File system manipulation ---

# configure time:
#   file(RENAME src dst) # all paths must exist
#   file(REMOVE files..)
#   file(REMOVE_RECURSE filesOrDirs...)
#   file(MAKE_DIRECTORY dirs..)

# build time:
#   cmake -E rename src dst
#   cmake -E remove [-f] files.. # -f mimics Unix rm -f cmd
#   cmake -E remove_directory dir
#   cmake -E make_directory dirs...


# lists dir contents:
#   file(GLOB outVar
#       [LIST_DIRECTORIES true|false]
#       [RELATIVE path]
#       [CONFIGURE_DEPENDS] # from v3.12
#       expressions... # wildcards - simplified regexps
#       )
#   file(GLOB_RECURSE outVar # is not fast as Unix find cmd
#       [LIST_DIRECTORIES true|false] # to include dir names
#       [RELATIVE path] # to convert abs paths to relative (relative to path)
#       [FOLLOW_SYMLINKS] # allows to descend into symlinks, otherwise them will be entries
#       [CONFIGURE_DEPENDS] # from v3.12
#       expressions... # wildcards - simplified regexps
#       )
# wildcards: *.txt, foo?.txt, bar[0-9].txt, /images/*.png (search in images dir for recurse ver)
# expr ex: ${base}/*/*.png - to find png images in any dirs below the base point

# !!! attention !!!
# DO NOT use file(GLOB..) cmds to find sources and other files that act as input to the build
#  if files are added or removed, CMake is not auto re-run!
#  and using of CONFIGURE_DEPENDS doesnt guarantee the re-run behaviour (depend on generators)
#  look at ch 28.5.1 (Target Sources)


# --- 5. Downloading and Uploading ---

# purpose - as part of install, packaging, test, download bootstrap, sensitive, big files..

#   file(DOWNLOAD url fileName [options...])
#   file(UPLOAD fileName url [options...])
# filename - relative to cur bin dir
# common options:
#   - LOG outVar - save log to outVar
#   - SHOW_PROGRESS - enable verbose log messages
#   - TIMEOUT seconds - abort operation after total time
#   - INACTIVITY_TIMEOUT seconds - abort operation if incativity period is more than time
# download options:
#   - EXPECTED_HASH ALGO=value - can be MD5 (MD5=...) or SHA1
#   - TLS_VERIFY value - bool val to enable the server certificate checking
#   ...




# --- Recommended practices ---
# use configure_file() for tailoring: *.h.in (src) -> *.h (bin)
# to copy several files use file(COPY|INSTALL..), prefer INSTALL ver for installation only
# specify abs paths for file(GENERATE..) to guarantee the proper working
# dont use hardcoding of authentication info (passwords, etc..) for file(DOWNLOAD..)

