
cmake_minimum_required(VERSION 3.31.6)

project(fancy_llama LANGUAGES C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT DEFINED LLAMA_CPP_DIR)
    set(LLAMA_CPP_DIR "${CMAKE_CURRENT_LIST_DIR}/src/llama.cpp-Android")
endif()

if(NOT EXISTS "${LLAMA_CPP_DIR}/CMakeLists.txt")
    message(FATAL_ERROR
        "llama.cpp-Android not found at ${LLAMA_CPP_DIR}. The local engine checkout is required."
    )
endif()

# CPU-only build: disable GPU and Qualcomm Hexagon backends.
set(BUILD_SHARED_LIBS ON CACHE BOOL "" FORCE)
set(LLAMA_BUILD_COMMON ON CACHE BOOL "" FORCE)
set(LLAMA_OPENSSL OFF CACHE BOOL "" FORCE)

if(ANDROID_ABI STREQUAL "arm64-v8a")
    set(GGML_SYSTEM_ARCH "ARM" CACHE STRING "" FORCE)
endif()

set(GGML_BACKEND_DL ON CACHE BOOL "" FORCE)
set(GGML_CPU_ALL_VARIANTS ON CACHE BOOL "" FORCE)
set(GGML_CPU_KLEIDIAI ON CACHE BOOL "" FORCE)
set(GGML_NATIVE OFF CACHE BOOL "" FORCE)
set(GGML_OPENMP ON CACHE BOOL "" FORCE)

set(GGML_OPENCL OFF CACHE BOOL "" FORCE)
set(GGML_HEXAGON OFF CACHE BOOL "" FORCE)

set(LLAMA_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/llama")

add_subdirectory(
    "${LLAMA_CPP_DIR}"
    "${LLAMA_BINARY_DIR}"
    EXCLUDE_FROM_ALL
)

add_library(fancy_llama SHARED llama_jni.cpp)

target_include_directories(fancy_llama PRIVATE
    "${LLAMA_CPP_DIR}"
    "${LLAMA_CPP_DIR}/common"
    "${LLAMA_CPP_DIR}/include"
    "${LLAMA_CPP_DIR}/ggml/include"
)

target_link_libraries(fancy_llama PRIVATE
    llama
    llama-common
    android
    dl
    log
)

target_compile_options(fancy_llama PRIVATE
    -fvisibility=hidden
)
