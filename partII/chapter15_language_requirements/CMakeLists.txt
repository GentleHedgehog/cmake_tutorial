# description:
# how to select flags for C/C++ standard version
# many compilers were added in version v3.2 (intel in v3.6)

# --- Set directly for target ---

# <LANG>_STANDARD - can be C/CXX/CUDA, define standard for a target
#   default value is taken from CMAKE_<LANG>_STANDARD
#   before v3.8: C - 90/99/11, CXX - 98/11/14
#   from v3.8: add version 17
#   from v3.12: add version 20
#   it is a minimum required standard (cmake can choose more recent version)

# <LANG>_STANDARD_REQUIRED - is OFF by default, it means - decay to a smaller version without error
#   default value is taken from CMAKE_<LANG>_STANDARD_REQUIRED

# <LANG>_EXTENSIONS - to choose some extentions (standard library version for ex.)
#   default value is taken from CMAKE_<LANG>_EXTENSIONS
#   to enable extensions we must also set <LANG>_STANDARD


# we should set all settings explicitly:
# require c++11 and disable extensions for all targets:
#   set(CMAKE_CXX_STANDARD 11)
#   set(CMAKE_CXX_STANDARD_REQUIRED ON)
#   set(CMAKE_CXX_EXTENSIONS OFF)

# use c++14 if available and enable extensions for all targets:
#   set(CMAKE_CXX_STANDARD 14)
#   set(CMAKE_CXX_STANDARD_REQUIRED OFF)
#   set(CMAKE_CXX_EXTENSIONS ON)

# build target with C99, no extentions
#   set_target_properties(foo PROPERTIES
#       C_STANDARD 99
#       C_STANDARD_REQUIRED ON
#       C_EXTENSIONS OFF
#   )

# --- Set by feature requirements ---

#   target_compile_features(target
#       <PRIVATE|PUBLIC|INTERFACE> feature1 [feature2 ...]
#       [<PRIVATE|PUBLIC|INTERFACE> feature1 [feature2 ...]]
#       ...
#       )
# it populates COMPILE_FEATURES and INTERFACE_COMPILE_FEATURES
# each feature must be supported by underlying compiler (if no - error)
# known features lists:
#   - CMAKE_<LANG>_KNOWN_FEATURES - all known for the language
#   - CMAKE_<LANG>_COMPILE_FEATURES - only supported by the compiler

# from v3.8:
# language meta-features - relative to the standard rather than specific compiler:
# <lang>_std_<value>
# for example, target and anything that links against it has C++14 support enabled:
#   target_compile_features(target PUBLIC cxx_std_14)
# cmake will choose appropriate compiler flags itself


# if there are CXX_STANDARD 11 and cxx_std_14 together,
# cmake will enforce the stronger standard requirements (it select C++14 in the case above)


# --- Optional language features ---

# cmake allow to make fine-grained tune (for ex. to tune using of some cxx keywords):
#   add_library(foo ...)
#   # Make override a feature requirement only if available:
#   target_compile_features(foo PUBLIC
#       $<$<COMPILE_FEATURES:cxx_override>:cxx_override>
#   )
#   # define the foo_OVERRIDE symbol so it provides the override keyword if available or empty otherwise:
#   target_compile_definitions(foo PUBLIC
#       $<$<COMPILE_FEATURES:cxx_override>:-Dfoo_OVERRIDE=override>
#       $<$<NOT:$<COMPILE_FEATURES:cxx_override>>:-Dfoo_OVERRIDE>
#   )
#   class MyClass : public Base
#   {
#   ... void func() foo_OVERRIDE; // will be compilable regardless of 'override' supporting
#   }

# facilitating func exists for this purposes:
#   include(WriteCompilerDetectionHeader)
#   write_compiler_detection_header(
#       FILE fileName
#       PREFIX prefix
#       COMPILERS compiler1 [...]
#       FEATURES feature1 [...]
#   )
# It write out a header file with name 'fileName': macros for listed features for each compiler
# Each feature:
#   - prefix_COMPILER_UPPERCASEFEATURE - has value 1 or 0 - supported or not
#   - prefix_UPPERCASEFEATURE - provides more appropriate implementation for feature


# example:
#   include(WriteCompilerDetectionHeader)
#   write_compiler_detection_header(
#       FILE foo_compiler_detection.h
#       PREFIX foo
#       COMPILERS GNU Clang MSVC Intel
#       FEATURES cxx_override
#                cxx_final
#                cxx_nullptr
#                cxx_rvalue_references
#   )
# result:
#   class MyClass : public Base
#   {
#       public:
#   #if foo_COMPILER_CXX_RVALUE_REFERENCES
#       MyClass(MyClass&& c)
#   #endif
#       void f1() foo_OVERRIDE;
#       void f2(int* p = foo_NULLPTR);
#   }
# now we can set CMAKE_CXX_STANDARD_REQUIRED = OFF
# foo_compiler_detection.h will be written to binary dir, so:
# target_include_directories(foo PUBLIC "${CMAKE_CURRENT_BINARY_DIR}")

# WriteCompilerDetectionHeader can be helpful for large projects (require many compilers or old version compilers)



# --- Recommended practices ---
# - use features instead of direct settings (but it can lead to collisions between targets)
# - use 3 direct settings together and after project() cmd
# - use meta-features
# - use WriteCompilerDetectionHeader only as transition path when updating an older code base to use newer features,
#   think about a balance of complexity and readability
