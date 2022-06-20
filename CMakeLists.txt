cmake_minimum_required(VERSION 3.15)
project(test)

get_filename_component(TOP_DIR "${CMAKE_CURRENT_LIST_DIR}" ABSOLUTE)
set(CMAKE_PREFIX_PATH ${TOP_DIR}/install)
message(STATUS "CMAKE_PREFIX_PATH: " ${CMAKE_PREFIX_PATH})

find_package(hello_world)

add_executable(test main.cpp)
target_link_libraries(test PUBLIC hello_world)
target_include_directories(test PUBLIC hello_world)
