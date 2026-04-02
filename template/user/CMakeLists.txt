cmake_minimum_required(VERSION 3.24)

project(hackarena3_cpp_template LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(MSVC)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
endif()

set(HACKARENA3_ARTIFACTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../artifacts")
set(HACKARENA3_SDK_ROOT "" CACHE PATH "Path to the extracted hackarena3 C++ SDK package root")
set(HACKARENA3_EFFECTIVE_SDK_ROOT "${HACKARENA3_SDK_ROOT}")

if(NOT HACKARENA3_EFFECTIVE_SDK_ROOT)
    if(WIN32)
        file(GLOB HACKARENA3_SDK_ARCHIVES "${HACKARENA3_ARTIFACTS_DIR}/hackarena3-cpp-sdk-*.zip")
    else()
        file(GLOB HACKARENA3_SDK_ARCHIVES "${HACKARENA3_ARTIFACTS_DIR}/hackarena3-cpp-sdk-*.tar.gz")
    endif()
    
    list(LENGTH HACKARENA3_SDK_ARCHIVES HACKARENA3_SDK_ARCHIVE_COUNT)

    if(HACKARENA3_SDK_ARCHIVE_COUNT GREATER 1)
        message(FATAL_ERROR
            "Multiple SDK archives found in ${HACKARENA3_ARTIFACTS_DIR}. "
            "Keep one archive there or set HACKARENA3_SDK_ROOT explicitly."
        )
    endif()

    if(HACKARENA3_SDK_ARCHIVE_COUNT EQUAL 1)
        list(GET HACKARENA3_SDK_ARCHIVES 0 HACKARENA3_SDK_ARCHIVE)
        get_filename_component(HACKARENA3_SDK_ARCHIVE_FILENAME "${HACKARENA3_SDK_ARCHIVE}" NAME)
        
        string(REGEX REPLACE "\\.(tar\\.gz|zip)$" "" HACKARENA3_SDK_ARCHIVE_STEM "${HACKARENA3_SDK_ARCHIVE_FILENAME}")
        
        set(HACKARENA3_EXTRACTED_SDK_ROOT "${CMAKE_BINARY_DIR}/_sdk/${HACKARENA3_SDK_ARCHIVE_STEM}")

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
    set(hackarena3_DIR "${HACKARENA3_EFFECTIVE_SDK_ROOT}/lib/cmake/hackarena3")
endif()

find_package(hackarena3 CONFIG REQUIRED)
add_executable(bot src/main.cpp)
target_link_libraries(bot PRIVATE hackarena3::hackarena3)

if(WIN32)
    add_custom_command(TARGET bot POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_RUNTIME_DLLS:bot>
        $<TARGET_FILE_DIR:bot>
        COMMAND_EXPAND_LISTS
        COMMENT "Copying hackarena3.dll to the executable directory..."
    )
endif()

if(MSVC)
    target_compile_options(bot PRIVATE /W4 /permissive-)
else()
    target_compile_options(bot PRIVATE -Wall -Wextra -Wpedantic)
endif()