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

