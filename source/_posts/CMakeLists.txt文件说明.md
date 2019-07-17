---
title: CMakeLists.txt文件说明
tags: [Jni,CMake]
grammar_cjkRuby: true
date: 2019-07-17
categories: [Android]
---

Android Studio2.2之后，放弃以Android.mk文件，而采用CMakeLists.txt文件。Android.mk文件和CMakeLists.txt具有映射关系。

```cmake
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.


# 设置生成SO文件的目标位置。由于编译后SO文件会默认添加到工程引用里，这里会导致再添加一次，
# 从而出现“ More than one file was found with OS independent path 'lib/armeabi-v7a/libnative-lib.so'”这样的错误。
# 解决办法是，一种：不设置这一句；一种：在gradle里设置如下：
#    packagingOptions {
#        exclude 'lib/armeabi-v7a/xxx.so'
#    }

# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})




# 添加头文件依赖
include_directories(src/main/cpp/include)



# 添加源码依赖
add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/src/DownloadModule.cpp
             src/main/cpp/src/HttpPostModule.cpp
             src/main/cpp/src/Main.cpp
             src/main/cpp/src/JNIUtil.cpp
             src/main/cpp/native-lib.cpp )


#增加第三方so文件动态共享库依赖，${ANDROID_ABI}表示so文件的ABI类型的路径
add_library(curl SHARED IMPORTED)
set_target_properties(curl  PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/src/main/cpp/lib/${ANDROID_ABI}/libcurl.so)



# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

#添加系统预置库依赖
find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

# 关联依赖库
target_link_libraries( # Specifies the target library.
                       native-lib
                       curl

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
					   
```

[Demo工程](https://github.com/xhrong/blog/blob/master/source/attachments/curlTest2.rar)