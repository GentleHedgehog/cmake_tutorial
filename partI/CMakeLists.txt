# description:

message("------- chapter 4 - build simple targets ----------")
add_subdirectory(chapter4_build_simple_targets)

message("------- chapter 5 - variables ----------")
add_subdirectory(chapter5_variables)

message("------- chapter 6 - flow control ----------")
add_subdirectory(chapter6_flow_control)

message("------- chapter 7 - using subdirs ----------")

set(varWithParentScopeToChangeInSubdir "change me")
set(varWithParentScope "foo")
set(varWithChildScope "")
add_subdirectory(chapter7_using_subdirs)
message("varWithParentScope = ${varWithParentScope}")
message("varWithChildScope = ${varWithChildScope}")
message("varWithParentScopeToChangeInSubdir = ${varWithParentScopeToChangeInSubdir}")


message("------- chapter 8 - funcs and macros ----------")
add_subdirectory(chapter8_funcs_and_macros)

message("------- chapter 9 - Properties ----------")
add_subdirectory(chapter9_properties)

message("------- chapter10 - generator expression ----------")
add_subdirectory(chapter10_generator_expression)


message("------- chapter11 - modules ----------")
add_subdirectory(chapter11_modules)

message("------- chapter12 - policies ----------")
add_subdirectory(chapter12_policies)



