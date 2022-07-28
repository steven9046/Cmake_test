cmake_minimum_required(VERSION 3.15)
project(hello_world)

# 设置安装路径 /home/ss/orbbec_camera_driver/install
get_filename_component(TOP_DIR "${CMAKE_CURRENT_LIST_DIR}/../" ABSOLUTE)
set(CMAKE_INSTALL_PREFIX ${TOP_DIR}/install/hello_world)

set(HELLO_LIBS "${CMAKE_CURRENT_SOURCE_DIR}/libhello.so")
set(WORLD_LIBS "${CMAKE_CURRENT_SOURCE_DIR}/libworld.so")

add_library(hello_world SHARED hello_world.cpp)

# 连接到库
target_link_libraries(hello_world PUBLIC ${HELLO_LIBS})
target_link_libraries(hello_world PUBLIC ${WORLD_LIBS})

target_include_directories(hello_world PUBLIC
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>)

# add_executable(hello_world_test  hello_world_test.cpp)
# target_link_libraries(hello_world_test hello_world)
# target_link_libraries(hello_world_test PUBLIC ${WORLD_LIBS})
# target_include_directories(hello_world_test PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
# target_link_libraries(hello_world world)

# 导出对象
install(TARGETS hello_world
        EXPORT  hello_world
        DESTINATION lib
        )

install(EXPORT hello_world 
        FILE hello_world.cmake
        DESTINATION lib/cmake/hello_world
        )

# 把 .h 也安装到指定位置
install(FILES
${CMAKE_CURRENT_SOURCE_DIR}/hello_world.h
${CMAKE_CURRENT_SOURCE_DIR}/hello.h
${CMAKE_CURRENT_SOURCE_DIR}/world.h
DESTINATION include
)


# 根据 .in 生成 .cmake 并安装到指定位置
include(CMakePackageConfigHelpers)

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/hello_worldConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/hello_worldConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/hello_world"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

# 把 config.cmake 也安装到指定位置
install(FILES
${CMAKE_CURRENT_BINARY_DIR}/hello_worldConfig.cmake
DESTINATION lib/cmake/hello_world
)

# 把 用到的.so 也安装到指定位置
install(FILES
${HELLO_LIBS}
${WORLD_LIBS}
DESTINATION lib
)
