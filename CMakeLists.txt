cmake_minimum_required(VERSION 3.5) # cmake should bahave the same as version 3.5 (major.minor[.patch[.tweak]])

# project - checks compiler and linker for workability
# [VERSION major[.minor[.patch[.tweak]]]], LANGUAGES C CXX Java ... NONE (by default C and CXX):
project(CMakeTutorial VERSION 1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(tutorialExe main.cpp)

message("------- Part I - Fundamentals ----------")
add_subdirectory(partI)

message("------- Part II - Builds In Depth ----------")
add_subdirectory(partII)

message("------- Part III - Bigger Picture ----------")
add_subdirectory(partIII)


message("\n\n")
