# Copyright (C) 2017 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##

# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

#include the native part of JUI Helper
add_subdirectory (${JUI_HELPER_PATH}
  "${CMAKE_CURRENT_SOURCE_DIR}/libs/${ANDROID_ABI}/libjuihelper")

#include the native part of NDKHelper
add_subdirectory (${NDK_HELPER_PATH}
  "${CMAKE_CURRENT_SOURCE_DIR}/libs/${ANDROID_ABI}/libndkhelper")

#include the GPG C++ SDK
add_library(gpg_sdk STATIC IMPORTED)
set_target_properties(gpg_sdk PROPERTIES IMPORTED_LOCATION
    ${GPG_SDK_PATH}/lib/c++/${ANDROID_ABI}/libgpg.a)

# build native_app_glue as a static lib
add_library(native_app_glue STATIC
    ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c)

# build cpufeatures as a static lib
add_library(cpufeatures STATIC
             ${ANDROID_NDK}/sources/android/cpufeatures/cpu-features.c)

# Export ANativeActivity_onCreate(),
# Refer to: https://github.com/android-ndk/ndk/issues/381.
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             ButtonClickerNativeActivity

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/ButtonClickerNativeActivity.cpp
             src/main/cpp/ButtonClickerNativeActivity_Engine.cpp
             ${TEAPOT_RENDERER_PATH}/TeapotRenderer.cpp
           )

target_include_directories(ButtonClickerNativeActivity PRIVATE
      ${ANDROID_NDK}/sources/android/native_app_glue
      ${ANDROID_NDK}/sources/android/cpufeatures
      ${GPG_SDK_PATH}/include
      ${juihelper_SOURCE_DIR}/src/main/cpp
      ${ndkhelper_SOURCE_DIR}/src/main/cpp
      ${TEAPOT_RENDERER_PATH}
)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries(ButtonClickerNativeActivity
      gpg_sdk
      native_app_glue
      cpufeatures
      juihelper
      ndkhelper
      log
      android
      EGL
      GLESv2
      z
)
