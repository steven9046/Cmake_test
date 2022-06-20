## CMake 测试程序

**v1.0 2022.6.20**

测试 PUBLIC INTERFACE PRIVATE 参数
```
target_link_libraries(hello_world hello)
target_include_directories(hello_world PUBLIC hello)
```
相关链接: 
[知乎](https://zhuanlan.zhihu.com/p/82244559)
[CMake手册](https://cmake.org/cmake/help/latest/command/target_include_directories.html?highlight=target_include_directories)


实际上，这三个关键字指定的是目标文件依赖项的使用范围（scope）或者一种传递（propagate）。[官方说明](https://cmake.org/cmake/help/v3.15/manual/cmake-buildsystem.7.html#transitive-usage-requirements)

可执行文件依赖 libhello-world.so， libhello-world.so 依赖 libhello.so 和 libworld.so。

* 1.main.c 不使用 libhello.so 的任何功能，因此 libhello-world.so 不需要将其依赖—— libhello.so 传递给 main.c，hello-world/CMakeLists.txt 中使用 PRIVATE 关键字；
* 2.main.c 使用 libhello.so 的功能，但是libhello-world.so 不使用，hello-world/CMakeLists.txt 中使用 INTERFACE 关键字；
* 3.main.c 和 libhello-world.so 都使用 libhello.so 的功能，hello-world/CMakeLists.txt 中使用 PUBLIC 关键字；


**v2.0 2022.6.20**
* 1. 把 hello.cpp world.cpp 编程两个动态库，并直接在 hello_world 里引用这两个动态库
     模拟把开源库嵌入自己项目的动作
* 2. 把 hello_world 编译成动态库，并安装到指定位置，目的是之后别的包可以直接 find_package 找到这个包
     为了达成这个目的，要注意的地方比较多，这里也不是完全明白其中的道理
     * a. 首先include这里需要如下设置为变量，否则在find_package时会提示路径不对
        ```
        target_include_directories(hello_world PUBLIC 
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>)
        ```
     * b. 接下来要导出对象
        生成对象信息
        ```
        install(TARGETS hello_world
                EXPORT  hello_world
                DESTINATION lib
        )
        ```
        把生成的对象信息写入 .cmake 并安装到指定位置 (这里其实还会生成一个-noconfig.cmake,这个东西有什么用[请看](https://blog.csdn.net/zhanghm1995/article/details/105466372))
        ```
        install(EXPORT hello_world 
                FILE hello_world.cmake
                DESTINATION lib/cmake/hello_world
                )
        ```
     * c. 把用到的 .so 和 .h 都安装到指定目录(这一步要和下一步配合使用)
        ```
        # 把 .h 安装到指定位置
        install(FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/hello_world.h
        ${CMAKE_CURRENT_SOURCE_DIR}/hello.h
        ${CMAKE_CURRENT_SOURCE_DIR}/world.h
        DESTINATION include
        )

        # 把用到的 .so 安装到指定位置
        install(FILES
        ${HELLO_LIBS}
        ${WORLD_LIBS}
        DESTINATION lib
        )
        ```
        这里如果不安装 .so，find_package()的时候还是会去源码的路径找，如果源码路径改变了就找不到了
     * d. 生成 Config.cmake 并安装到指定位置
        ```
        # 根据 .in 生成 config (其实就是内容的复制)
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
        ```
        find_package()找包的时候是先找 XXXConfig.cmake，然后根据它的内容来找包
        Config的内容:
        ```
        @PACKAGE_INIT@ # 这个可以得到 PACKAGE_PREFIX_DIR，当然也可以自己设定
        include(CMakeFindDependencyMacro)
        # 这两个东西就是一个包的主要内容，需要在这里自己设定，只要能够把目录结构操作好，这里随便设置
        set(hello_world_INCLUDE_DIRS "${PACKAGE_PREFIX_DIR}/include") 
        set(hello_world_LIBRARIES "${PACKAGE_PREFIX_DIR}/lib") 
        # 这个必须包含进来，要不然就找不到 .cmake了
        include("${CMAKE_CURRENT_LIST_DIR}/hello_world.cmake")
        ```
* 3. 在另外的包里引用
     find_package 默认的搜索路径就是系统的安装位置，所以如果想在另外的项目里找到自定义包
     还需要在项目中设置 CMAKE_PREFIX_PATH 为我们包的安装位置
     ```
     get_filename_component(TOP_DIR "${CMAKE_CURRENT_LIST_DIR}" ABSOLUTE)
     set(CMAKE_PREFIX_PATH ${TOP_DIR}/install)
     ```
* 4. 总结下来就是能把路径弄明白了就可以解决大部分问题
     一些常用的宏
     ```
     CMAKE_CURRENT_LIST_DIR         当前的 CMakeLists.txt 所在目录      /home/ss/cmake_word_test/hello_world
     CMAKE_CURRENT_SOURCE_DIR       当前的 src 文件夹所在目录            /home/ss/cmake_word_test/hello_world
     CMAKE_CURRENT_BINARY_DIR       当前构建目录(build,二进制文件所在)    /home/ss/cmake_word_test/hello_world/build
     ```
     get_filename_component 命令，得到文件名
     ```
     #                                 这里的 ../ 可以返回上级目录      
     get_filename_component(TOP_DIR "${CMAKE_CURRENT_LIST_DIR}/../" ABSOLUTE)
     #                                          这里是文件名          得到它的路径   
     get_filename_component(_IMPORT_PREFIX "${CMAKE_CURRENT_LIST_FILE}" PATH)
     # /home/ss/cmake_word_test/install/hello_world/lib/cmake/hello_world/hello_world.cmake 就变为
     # /home/ss/cmake_word_test/install/hello_world/lib/cmake/hello_world
     # 在来一次，就变为
     get_filename_component(_IMPORT_PREFIX "${_IMPORT_PREFIX}" PATH)
     # /home/ss/cmake_word_test/install/hello_world/lib/cmake
     ```
* 5. 其他结果
     把 hello_world 编译并安装好了以后
     编译 main.cpp 的时候还是会去 hello_world 的源码里找 libhello.so
     这时把源码里的 .so 删掉就编译不过了
     但是运行时会去找 install 下的 .so