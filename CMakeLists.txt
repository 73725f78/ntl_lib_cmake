cmake_minimum_required(VERSION 3.18)

project(NTL_Project LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

enable_testing()

# FreeBSD-specific: Add /usr/local to search paths if not already there
if(${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
    list(APPEND CMAKE_PREFIX_PATH "/usr/local")
endif()

# 1. Find Threads correctly
find_package(Threads REQUIRED)

# 2. Find GMP
find_path(GMP_INCLUDE_DIR NAMES gmp.h)
find_library(GMP_LIBRARY NAMES gmp)

if(NOT GMP_INCLUDE_DIR OR NOT GMP_LIBRARY)
    message(FATAL_ERROR "GMP library or headers not found! On FreeBSD, run: pkg install gmp")
endif()

file(GLOB NTL_SOURCES "src/*.cpp")
list(FILTER NTL_SOURCES EXCLUDE REGEX "src/Check.*\\.cpp")
list(FILTER NTL_SOURCES EXCLUDE REGEX "src/GetTime[2-5]\\.cpp")

add_library(ntl_lib STATIC ${NTL_SOURCES})

# Use the variables found by find_package/path/library
target_include_directories(ntl_lib PUBLIC 
    ${GMP_INCLUDE_DIR}
    "${CMAKE_CURRENT_SOURCE_DIR}/include"  # Path to the directory containing the NTL/ folder
)

target_link_libraries(ntl_lib PUBLIC ${GMP_LIBRARY} Threads::Threads)

# Definitions
target_compile_definitions(ntl_lib PRIVATE _GNU_SOURCE)
target_compile_definitions(ntl_lib PUBLIC NTL_GMP_LIP)

# Hardware-specific flags
target_compile_options(ntl_lib PRIVATE 
    -march=native 
    -maes 
    -mpclmul
    -msse2
    -mavx2
)

# Test generation
file(GLOB NTL_TEST_SOURCES "src/*Test.cpp")
foreach(test_source ${NTL_TEST_SOURCES})
    get_filename_component(test_name ${test_source} NAME_WE)
    add_executable(${test_name} ${test_source})
    target_link_libraries(${test_name} PRIVATE ntl_lib)
    add_test(NAME ${test_name} COMMAND ${test_name})
endforeach()
