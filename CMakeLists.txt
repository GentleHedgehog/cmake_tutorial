cmake_minimum_required(VERSION 3.5) # cmake should bahave the same as version 3.5 (major.minor[.patch[.tweak]])

project(ch1 VERSION 1.0 LANGUAGES CXX) # [VERSION major[.minor[.patch[.tweak]]]], LANGUAGES C CXX Java ... NONE (by default C and CXX)
# project - check compiler and linker for workability



set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(ch1 main.cpp) # several commands can produce diffrent target names
