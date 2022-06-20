cmake_minimum_required(VERSION 3.15)
project(test)

add_subdirectory(hello_world)

add_executable(test main.cpp)
target_link_libraries(test PRIVATE hello_world)
target_include_directories(test PRIVATE hello_world)
