# --- Description ---

# toolchain - set of files, scripts, compilers, linkers, etc.
# cmake checks the toolchain after project() / enable_language() and caches this info

# If you want to change the toolchain -> build dir should be completely cleared!

# Cmake assumes using of only a single toolchain,
# else look at ch 27.1 External project and ch 28.1 Superbuild Structure.

# --- 1. Toolchain files ---

# file consists of set() commands to discribe target platform, paths, flags, etc.:
# cmake -DCMAKE_TOOLCHAIN_FILE=myToolchain.cmake path/to/source
#   relative path: from build dir, or from src dir.
#   this file must be specified before the first build of the build dir
#   this file is read by project() cmd each time (so dont put any project info into it)
#   it can influence the behaviour of find_...() (but put this burden on the project file)


# --- 2. Defining the target system ---

# CMAKE_SYSTEM_NAME - target system (opposed to CMAKE_HOST_SYSTEM_NAME - building system)
# typical: Linux, Windows, QNX, Android, Darwin, Generic (bare metal)
# if it is set -> CMAKE_CROSSCOMPILING = true
# if it is not set -> copy from CMAKE_HOST_SYSTEM_NAME

# CMAKE_SYSTEM_PROCESSOR
# if it is not set -> copy from CMAKE_HOST_SYSTEM_PROCESSOR
# (but it is advisable to set if the arch is doesnt match)
# is typically not set for Android (ch 21.6.3)

# CMAKE_SYSTEM_VERSION
# diff for diffrent systems

# --- 3. Tool selection ---

# CMAKE_<LANG>_COMPILER
# a path to or a name of the compiler
# if it is not set, a compiler can be obtained by evn var: CC, CXX, etc.

# cmake populates:
# CMAKE_<LANG>_COMPILER_ID - Intel, GNU, ...
# CMAKE_<LANG>_COMPILER_VERSION - major.minor.patch.tweak

# use gen expr (without CMAKE_ prefix):
#   add_library(foo ...)
#   # Conditionally add -fcode-hoisting option using variables
#   if(CXX_COMPILER_ID STREQUAL GNU AND
#      NOT CXX_COMPILER_VERSION VERSION_LESS 7)
#       target_compile_options(foo PRIVATE -fcode-hoisting)
#   endif()
#   # Same thing using generator expressions instead
#   target_compile_options(foo PRIVATE
#       $<$<AND:$<CXX_COMPILER_ID:GNU>,
#               $<VERSION_GREATER_EQUAL:$<CXX_COMPILER_VERSION>,7>>:-fcode-hoisting>
#   )


# flag vars: look at ch 14.3
# developers can add default flags by same vars with *_INIT suffix
# (useful only before the first cmake run)
# a common mistake - set vars without _INIT suffix
# useful for modern compilers (appeared after a release of cmake v3.7)
# for Clang/QNX -> use CMAKE_<LANG>_COMPILER_TARGET to specify the target platform/arch

# if compiler doesnt contain all stuff inside:
# use CMAKE_<LANG>_COMPILER_EXTERNAL_TOOLCHAIN
# to set path where are the additional stuff: linker, archiver...


# --- 4. System Roots ---
# System root or sysroot - the mounted target file system with a set of necessary files
# var -> CMAKE_SYSROOT (look at ch 23.1.2)

# CMAKE_SYSROOT_COMPILE and CMAKE_SYSROOT_LINK -
# use them in the complex scenarios (diff root for compile/link) (from v3.9)

# CMAKE_STAGING_PREFIX - staging point (see 25.1.2 Base Install Location)

# --- 5. Compiler checks ---
# CMAKE_TRY_COMPILE_PLATFORM_VARIABLES - set vars that will be passed to test builds
# use list(APPEND..)

# CMAKE_TRY_COMPILE_TARGET_TYPE - set to STATIC_LIBRARY,
# if you dont want to run result after try_compile() (from v3.6, before - CMakeForceCompiler).

#  CMAKE_CROSSCOMPILING_EMULATOR  - to emulate the target system in the cross compile case
# ch 24.6 Cross...

# --- 6. Examples ---

# GCC:
#   set(CMAKE_SYSTEM_NAME      Linux)
#   set(CMAKE_SYSTEM_PROCESSOR i686)
#   set(CMAKE_C_COMPILER   gcc)
#   set(CMAKE_CXX_COMPILER g++)
#   set(CMAKE_C_FLAGS_INIT   -m32)
#   set(CMAKE_CXX_FLAGS_INIT -m32)
#   set(CMAKE_EXE_LINKER_FLAGS_INIT    -m32)
#   set(CMAKE_SHARED_LINKER_FLAGS_INIT -m32)
#   set(CMAKE_MODULE_LINKER_FLAGS_INIT -m32)
# to confirm the settings:
# math(EXPR bitness "${CMAKE_SIZEOF_VOID_P} * 8")
# message("${bitness}-bit build")

# Android, VS, RaspberryPi - see in the book


# --- Recommended practices ---
# put info only for the toolchain
# dont assume a specific context for the toolchain file
# avoid using the deprecated CMakeForceCompiler
# use vars with _INIT suffix for flags
# dont rely on CMAKE_CROSSCOMPILING var (it can be true, but the platform is the same)
# for finding - see ch 23
