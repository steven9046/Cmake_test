cmake_minimum_required(VERSION 3.15)
# project(hello-world)

add_subdirectory(hello)
add_subdirectory(world)

add_library(hello_world SHARED hello_world.cpp)

# 连接到库
target_link_libraries(hello_world hello)
target_include_directories(hello_world PUBLIC hello)

target_link_libraries(hello_world world)
target_include_directories(hello_world PUBLIC world)

# add_executable(hello_world_test  hello_world_test)
# target_link_libraries(hello_world_test hello)
# target_include_directories(hello_world_test PRIVATE hello)

# target_link_libraries(hello_world world)
