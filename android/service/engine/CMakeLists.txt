set(engine OpenCVEngine)
set(JNI_LIB_NAME ${engine} ${engine}_jni)

unset(__android_project_chain CACHE)
add_android_project(opencv_engine "${CMAKE_CURRENT_SOURCE_DIR}" SDK_TARGET 8 ${ANDROID_SDK_TARGET} IGNORE_JAVA ON)

set(ANDROID_PLATFORM_VERSION_CODE "0")

if(ARMEABI_V7A)
    set(ANDROID_PLATFORM_VERSION_CODE "2")
  elseif(ARMEABI_V6)
    set(ANDROID_PLATFORM_VERSION_CODE "1")
  elseif(ARMEABI)
    set(ANDROID_PLATFORM_VERSION_CODE "1")
  elseif(X86)
    set(ANDROID_PLATFORM_VERSION_CODE "3")
  elseif(MIPS)
    set(ANDROID_PLATFORM_VERSION_CODE "4")
  else()
    message(WARNING "Can not automatically determine the value for ANDROID_PLATFORM_VERSION_CODE")
  endif()

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${ANDROID_MANIFEST_FILE}" "${OpenCV_BINARY_DIR}/android/service/engine/.build/${ANDROID_MANIFEST_FILE}"  @ONLY)

link_directories("${ANDROID_SOURCE_TREE}/out/target/product/generic/system/lib" "${ANDROID_SOURCE_TREE}/out/target/product/${ANDROID_PRODUCT}/system/lib" "${ANDROID_SOURCE_TREE}/bin/${ANDROID_ARCH_NAME}")

# -D__SUPPORT_ARMEABI_FEATURES key is also available
add_definitions(-DPLATFORM_ANDROID -D__SUPPORT_ARMEABI_V7A_FEATURES -D__SUPPORT_TEGRA3 -D__SUPPORT_MIPS)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti -fno-exceptions")

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-allow-shlib-undefined")
file(GLOB engine_files "jni/BinderComponent/*.cpp" "jni/BinderComponent/*.h" "jni/include/*.h")

include_directories(jni/BinderComponent jni/include)
include_directories(SYSTEM "${ANDROID_SOURCE_TREE}/frameworks/base/include" "${ANDROID_SOURCE_TREE}/system/core/include")
add_library(${engine} SHARED ${engine_files})
target_link_libraries(${engine} z binder log utils)

set_target_properties(${engine} PROPERTIES
    OUTPUT_NAME ${engine}
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/.build/libs/${ANDROID_NDK_ABI_NAME}"
    )

get_target_property(engine_lib_location ${engine} LOCATION)
add_custom_command(TARGET ${engine} POST_BUILD COMMAND ${CMAKE_STRIP} --strip-unneeded "${engine_lib_location}")

file(GLOB engine_jni_files "jni/JNIWrapper/*.cpp" "jni/JNIWrapper/*.h" "jni/include/*.h")
list(APPEND engine_jni_files jni/NativeService/CommonPackageManager.cpp jni/NativeService/PackageInfo.cpp)

include_directories(jni/include jni/JNIWrapper jni/NativeService jni/BinderComponent "${ANDROID_SOURCE_TREE}/frameworks/base/include" "${ANDROID_SOURCE_TREE}/system/core/include" "${ANDROID_SOURCE_TREE}/frameworks/base/core/jni")
add_library(${engine}_jni SHARED ${engine_jni_files})
target_link_libraries(${engine}_jni z binder log utils android_runtime ${engine})

set_target_properties(${engine}_jni PROPERTIES
    OUTPUT_NAME ${engine}_jni
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/.build/libs/${ANDROID_NDK_ABI_NAME}"
    )

get_target_property(engine_lib_location ${engine}_jni LOCATION)
add_custom_command(TARGET ${engine}_jni POST_BUILD COMMAND ${CMAKE_STRIP} --strip-unneeded "${engine_lib_location}")
