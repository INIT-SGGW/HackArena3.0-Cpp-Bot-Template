cmake_minimum_required(VERSION 3.24)

project(hackarena3_cpp_template LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(HACKARENA3_ARTIFACTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/artifacts")
set(HACKARENA3_SDK_ROOT "" CACHE PATH "Path to the unzipped hackarena3 C++ SDK package root")
set(HACKARENA3_EFFECTIVE_SDK_ROOT "${HACKARENA3_SDK_ROOT}")

if(NOT HACKARENA3_EFFECTIVE_SDK_ROOT)
    file(GLOB HACKARENA3_SDK_ARCHIVES "${HACKARENA3_ARTIFACTS_DIR}/hackarena3-cpp-sdk-*.zip")
    list(LENGTH HACKARENA3_SDK_ARCHIVES HACKARENA3_SDK_ARCHIVE_COUNT)

    if(HACKARENA3_SDK_ARCHIVE_COUNT GREATER 1)
        message(FATAL_ERROR
            "Multiple SDK zip archives found in ${HACKARENA3_ARTIFACTS_DIR}. "
            "Keep one archive there or set HACKARENA3_SDK_ROOT explicitly."
        )
    endif()

    if(HACKARENA3_SDK_ARCHIVE_COUNT EQUAL 1)
        list(GET HACKARENA3_SDK_ARCHIVES 0 HACKARENA3_SDK_ARCHIVE)
        get_filename_component(HACKARENA3_SDK_ARCHIVE_FILENAME "${HACKARENA3_SDK_ARCHIVE}" NAME)
        string(REGEX REPLACE "\\.zip$" "" HACKARENA3_SDK_ARCHIVE_STEM "${HACKARENA3_SDK_ARCHIVE_FILENAME}")
        set(HACKARENA3_EXTRACTED_SDK_ROOT
            "${CMAKE_BINARY_DIR}/_sdk/${HACKARENA3_SDK_ARCHIVE_STEM}"
        )

        if(NOT EXISTS "${HACKARENA3_EXTRACTED_SDK_ROOT}/lib/cmake/hackarena3/hackarena3Config.cmake")
            file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/_sdk")
            file(ARCHIVE_EXTRACT
                INPUT "${HACKARENA3_SDK_ARCHIVE}"
                DESTINATION "${CMAKE_BINARY_DIR}/_sdk"
            )
        endif()

        set(HACKARENA3_EFFECTIVE_SDK_ROOT "${HACKARENA3_EXTRACTED_SDK_ROOT}")
    endif()
endif()

if(HACKARENA3_EFFECTIVE_SDK_ROOT)
    set(HACKARENA3_PACKAGE_CONFIG_DIR "${HACKARENA3_EFFECTIVE_SDK_ROOT}/lib/cmake/hackarena3")
endif()

if(HACKARENA3_EFFECTIVE_SDK_ROOT)
    if(NOT EXISTS "${HACKARENA3_PACKAGE_CONFIG_DIR}/hackarena3Config.cmake")
        message(FATAL_ERROR
            "Detected SDK root '${HACKARENA3_EFFECTIVE_SDK_ROOT}', but "
            "'${HACKARENA3_PACKAGE_CONFIG_DIR}/hackarena3Config.cmake' does not exist."
        )
    endif()
    include("${HACKARENA3_PACKAGE_CONFIG_DIR}/hackarena3Config.cmake")
else()
    find_package(hackarena3 CONFIG REQUIRED)
endif()

add_executable(bot src/main.cpp)
target_link_libraries(bot PRIVATE hackarena3::hackarena3)

if(COMMAND hackarena3_copy_runtime_dlls)
    hackarena3_copy_runtime_dlls(bot)
endif()

if(MSVC)
    target_compile_options(bot PRIVATE /W4 /permissive-)
else()
    target_compile_options(bot PRIVATE -Wall -Wextra -Wpedantic)
endif()
