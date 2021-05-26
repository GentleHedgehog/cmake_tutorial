# --- Description ---

# cmake abstracts away differences of using libs between OS

# --- 1. Build Basics ---
# add_library(...)
# SHARED libs is connected to the other targets, but MODULEs is not.
# Unix:
#   SHARED or STATIC will have lib prefix in Unix, but MODULE - might not.
# On Windows:
#   STATIC - *.lib files, SHARED - *.dll for runtime and *.lib for importing at build time.
#   Cmake has the ability to convert GNU import libs *.dll.a
#   to *.lib (Visual Studio, must be installed) import file (MinGW, MSYS).
#   This ability is enabled by GNUtoMS target property for shared libs
#   (default from CMAKE_GNUtoMS, at add_library()).


# --- 2. Linking Static libs ---
# if A lib is a PRIVATE deps for B lib, A will be treated as a PUBLIC deps (only for linking)
# (due to the fact that A will be inserted to the linker cmd line).
# Cmake can recognize cyclic dependencies of static libs,
# if there is a relationship specification by target_link_libraries().
# if A depends from B, and B depends from A, cmake produces: link_cmd ... A B A B,
# this behaviour is controlled by property LINK_INTERFACE_MULTIPLICITY.
# Object libs can be used to solve such problems,
# because the object files ordering has no influence as static libs ordering has.

# --- 3. Shared Lib Versioning ---
# If a shared lib is used for the other projects, it should follow the system conventions.
# On Linux:
# libmy.so.2.4.3
# libmy.so.2 --> libmy.so.2.4.3
# libmy.so --> libmy.so.2
# Cmake automates the creation of the full set of files and symlinks if ver details are provided,
# these target properties are VERSION (major.minor.patch - current ver)
# and SOVERSION (major - compatibility ver):
#   add_library(mystuff SHARED src.cpp..)
#   set_target_properties(mystuff PROPERTIES VERSION 2.4.3 SOVERSION 2)
# any target that deps on mystuff will contain info: libmystuff.2.dylib to look for it in runtime.
# On Windows:
# Cmake takes the major.minor part from VERSION and writes into the dll as the DLL image version.
# SOVERSION is not used.


# --- 4. Interface compatibility ---
# Optional using of a some lib can be enforced by interface compatibility property.
# All libs should declare the property, and cmake will check it's equality between them:
#   add_library(networking net.cpp)
#   set_target_properties(networking PROPERTIES
#       COMPATIBLE_INTERFACE_BOOL SLL_SUPPORT
#       INTERFACE_SLL_SUPPORT YES
#       )
#   add_library(util util.cpp)
#   set_target_properties(util PROPERTIES
#       COMPATIBLE_INTERFACE_BOOL SLL_SUPPORT
#       INTERFACE_SLL_SUPPORT YES
#       )
#   add_executable(myApp main.cpp)
#   target_link_libraries(myApp PRIVATE networking util)
#   target_compile_definitions(myApp PRIVATE $<$<BOOL:$<TARGET_PROPERTY:SSL_SUPPORT>>:HAVE_SSL>)
# cmake automatically populates myApp with the prop SSL_SUPPORT. To set necessary requirement:
#   set_target_properties(myApp PROPERTIES SSL_SUPPORT YES)

# to require that the libs must have the same SSL implementation:
#   add_library(networking net.cpp)
#   set_target_properties(networking PROPERTIES
#       COMPATIBLE_INTERFACE_STRING SLL_IMPL
#       INTERFACE_SLL_IMPL OpenSSL
#       )
#   add_library(util util.cpp)
#   set_target_properties(util PROPERTIES
#       COMPATIBLE_INTERFACE_STRING SLL_IMPL
#       INTERFACE_SLL_IMPL OpenSSL
#       )
#   add_executable(myApp main.cpp)
#   target_link_libraries(myApp PRIVATE networking util)
#   target_compile_definitions(myApp PRIVATE SLL_IMPL=$<TARGET_PROPERTY:SLL_IMPL>)
# To set necessary requirement: myApp could have defined its SSL_IMPL property
#   set_target_properties(myApp PROPERTIES SLL_IMPL OpenSSL)

# to set diapasons, myApp will get the min protocol ver and max buffers count:
#   add_library(bigAndFast strategy1.cpp)
#   set_target_properties(bigAndFast PROPERTIES
#       COMPATIBLE_INTERFACE_NUMBER_MIN PROTOCOL_VER
#       COMPATIBLE_INTERFACE_NUMBER_MAX TMP_BUFFERS
#       INTERFACE_PROTOCOL_VER 3
#       INTERFACE_TMP_BUFFERS 200
#       )
#   add_library(smallAndSlow strategy2.cpp)
#   set_target_properties(smallAndSlow PROPERTIES
#       COMPATIBLE_INTERFACE_NUMBER_MIN PROTOCOL_VER
#       COMPATIBLE_INTERFACE_NUMBER_MAX TMP_BUFFERS
#       INTERFACE_PROTOCOL_VER 2
#       INTERFACE_TMP_BUFFERS 15
#       )
#   add_executable(myApp myapp.cpp)
#   target_link_libraries(myApp PRIVATE bigAndFast smallAndSlow)
#   target_compile_definitions(myApp PRIVATE
#       MIN_API=$<TARGET_PROPERTY:PROTOCOL_VER>
#       TMP_BUFFERS=$<TARGET_PROPERTY:TMP_BUFFERS>)
# Min and max properties must have different names! for ex: PROTOCOL_VER_MIN and PROTOCOL_VER_MAX

# Libraries only need to define interface compatibilities that are relevant to them.

# use case:
# myApp -> libNet  (INTERFACE_FOO ON)
#       -> libUtil (INTERFACE_FOO ON) -> libCalc (INTERFACE_FOO OFF)
# if libNet and libUtil are PRIVATE deps for myApp -> only they must have the consistent property.
# libCalc property is not considered.
# libUtil and libCalc are not direct link deps of a common target,
# so they are not required to have compatible props.

# another use case:
# same as previous but libCalc is a PUBLIC link dep for libUtil,
# so libCalc becomes a direct link dep of myApp and it does participate in compatibility


# --- 5. Symbol visibility ---
# All symbols from the static lib can be used from outside.
# But the shared lib exposes only a subset of the symbols.
# But different compiles have different policies:
#   VisualStudio - hide all symbols except of marked as exported
#   GCC/Clang - expose all symbols except of marked as not exported (?)

# Cmake settins for a shared lib target:
#   C_VISIBILITY_PRESET
#   CXX_VISIBILITY_PRESET
# values:
#   hidden - to hide all symbols
#   default, protected, internal - less likely to be useful
# to hide inline symbols:
#   VISIBILITY_INLINES_HIDDEN
# default sources for these props:
#   CMAKE_CXX_VISIBILITY_PRESET
#   CMAKE_C_VISIBILITY_PRESET
#   CMAKE_VISIBILITY_INLINES_HIDDEN
# if you want to expose all symbols on all platforms (set to true):
#   [CMAKE_]WINDOWS_EXPORT_ALL_SYMBOLS


# to provide export keywords, there is the func:
#   include(GenerateExportHeader)
#   generate_export_header(
#   target # myTool -> MYTOOL_EXPORT define in mytools_export.h
#   [BASE_NAME baseName] # foobar -> FOOBAR_EXPORT define in foobar_export.h
#   [EXPORT_FILE_NAME exportFileName] # ex: defines in export_myTools.h
#   [EXPORT_MACRO_NAME exportMacroName] # API_MYTOOLS -> API_MYTOOLS define in export_myTools.h
#   [NO_EXPORT_MACRO_NAME noExportMacroName] # to prevent export: REALLY_PRIVATE define in mytools_export.h
#   [DEPRECATED_MACRO_NAME deprecatedMacroName] # to create macros for deprecated symbols (or <BaseName>_DEPRECATED)
#   [DEFINE_NO_DEPRECATED] # provides <BaseName>_NO_DEPRECATED
#   [NO_DEPRECATED_MACRO_NAME noDeprecatedMacroName] # to override DEFINE_NO_DEPRECATED
#   [STATIC_DEFINE staticDefine] # define to do nothing if static lib (or <BaseName>_STATIC_DEFINE)
#   [PREFIX_NAME prefix] # for each preproc definition
#   [CUSTOM_CONTENT_FROM_VARIABLE var] # to inject the custom content into generated header (comments for ex)
#   )


# --- 6. Mixing Static And Shared libs ---
# Successful build with static libs is very likely
# (due to they are set to the linker cmd in spite of PUBLIC/PRIVATE linking).

# use case:
# myApp -> libUtil  -> libCalc
#       -> libCalc
# if libs are static - OK.
# if libUtil is shared - ODR can be violated (code duplication),
# myApp and libUtil can have different objects of the class (Singleton for ex) from libCalc.
# if such symbol is not exported in libUtil - linker will not complain!
# Resume: static lib must be used only in one exe or dynamic lib, or transform it to the shared lib!

# generate_export_header will not produce export keywords if you define <myShared>_EXPORTS
# for the static lib.
# (The key to making this work is to ensure the static
# library has a compile definition for the name of the shared library target with _EXPORTS appended,
# which is how the generated header detects whether the code is being built as part of the shared
# library or not )

# A static lib must be compiled with PIC (on many x64 Unix systems) to be part of the dynamic lib:
#   add_library(myShared SHARED shared.cpp)
#   add_library(myStatic STATIC static.cpp)
#   target_link_libraries(myShared PRIVATE myStatic)
#   set_target_properties(myStatic PROPERTIES
#       POSITION_INDEPENDENT_CODE ON # method 1 (specific for target)
#   )
#   set(CMAKE_POSITION_INDEPENDENT_CODE ON) # method 2 (more global)
#   add_library(myOtherStatic STATIC other.cpp)
#   target_link_libraries(myShared PRIVATE myOtherStatic)


# --- Recommended practices ---
# use MODULE - for plugins.
# use SHARED - to build with, to hide sensitive impl for API, to deliver as part of a release package.
# use STATIC - if target uses smth from lib directly, do not rely on indirect link deps,
#   express the right type: PRIVATE, PUBLIC, INTERFACE.
# do not make all deps as PUBLIC
# begin versioning from 0.1.0 ASAP
# use interface compatibilities to detect consistency (but take care about complexity)
# think about symbol visibility asap
# try to avoid using the mix of dyn/shared libs in the same project (see ch 28.5.1)


