# CMake整理


在CMake中依赖第三方(或者其他module编译出来的)so文件的方法
``` cmake
CMake.txt
add_library( )： .c或者.cpp文件要假如里面；
include_directories( ) ：第三库使用到的头文件；
依赖第三方库
每次只能写一个：
add_library(curl STATIC IMPORTED)
set_target_properties(curl
  PROPERTIES IMPORTED_LOCATION
  ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libcurl.a)
STATIC：表示静态的.a的库。
SHARED：表示.so的库。
${CMAKE_SOURCE_DIR}：表示CMake.txt的当前文件夹路径。
${ANDROID_ABI}：编译时会自动根据CPU架构去选择相应的库。
依赖NDK中的库
find_library( jnigraphics-lib
jnigraphics )
target_link_libraries( StackBlur
${log-lib}
${m-lib}
${jnigraphics-lib} )
最后附上Cmake.txt：
cmake_minimum_required(VERSION 3.4.1)

add_library(native-lib
             SHARED
             src/main/cpp/native-lib.cpp
             src/main/cpp/JniUtils.cpp
             src/main/cpp/web_task.cpp )

include_directories( src/main/cpp/include/jsoncpp
                      src/main/cpp/include/curl
                     )

add_library(curl STATIC IMPORTED)
set_target_properties(curl
  PROPERTIES IMPORTED_LOCATION
  ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libcurl.a)

add_library(jsoncpp STATIC IMPORTED)
set_target_properties(jsoncpp
  PROPERTIES IMPORTED_LOCATION
  ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libjsoncpp.a)
```

```  cmake
# 不能向下面格式写，会出现 Error:error: '../../../../src/main/libs/libjsoncpp.a', needed by '../obj/armeabi/libnative-lib.so', missing and no known rule to make it

# add_library(curl STATIC IMPORTED)
# set_target_properties(curl
#   PROPERTIES IMPORTED_LOCATION
#   ${CMAKE_SOURCE_DIR}/src/main/libs/libcurl.a)

# add_library(jsoncpp STATIC IMPORTED)
# set_target_properties(jsoncpp
#   PROPERTIES IMPORTED_LOCATION
#   ${CMAKE_SOURCE_DIR}/src/main/libs/libjsoncpp.a)

find_library( # Sets the name of the path variable.
              log-lib
              log )

target_link_libraries( # Specifies the target library.
                       native-lib

                       jsoncpp
                       curl
                       ${log-lib} )
```
下面是另一个例子

``` cmake
SET(SOURCE_FILES
  Gassociation.cpp
  ConfigurationHandler.cpp
  )
  INCLUDE_DIRECTORIES（
  /usr/local/include
  ${PROJECT_SOURCE_DIR}/utility
  ${PROJECT_SOURCE_DIR}/association
  )
  LINK_LIBRARIES(
  /usr/local/lib
  ${PROJECT_SOURCE_DIR}/lib
  )
  ADD_EXECUTABLE(server ${SOURCE_FILES})
  TARGET_LINK_LIBRARIES(server
  utility
  ）
  SET_TARGET_PROPERTIES(server PROPERTIES #表示生成的执行文件所在路径
  RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin")

```
参考文献：

https://blog.csdn.net/ningjingsun/article/details/52960355
https://www.jianshu.com/p/5f29fd671750
https://blog.csdn.net/wfei101/article/details/77150234
