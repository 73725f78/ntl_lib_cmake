cmake_minimum_required(VERSION 3.18)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

enable_testing()

find_path(GMP_INCLUDE_DIR NAMES gmp.h)
find_library(GMP_LIBRARY NAMES gmp)

if(NOT GMP_INCLUDE_DIR OR NOT GMP_LIBRARY)
    message(FATAL_ERROR "GMP library or headers not found! Please install libgmp-dev.")
endif()

file(GLOB NTL_SOURCES "src/*.cpp")

list(FILTER NTL_SOURCES EXCLUDE REGEX "src/Check.*\\.cpp")
list(FILTER NTL_SOURCES EXCLUDE REGEX "src/GetTime[2-5]\\.cpp")   # 3. Keep ONLY GetTime1.cpp and remove the others (2, 3, 4, 5)

add_library(ntl_lib STATIC ${NTL_SOURCES})

target_link_libraries(ntl_lib PUBLIC ${GMP_LIBRARY})
target_compile_definitions(ntl_lib PRIVATE _GNU_SOURCE)
target_include_directories(ntl_lib PUBLIC ${GMP_INCLUDE_DIR})
target_compile_definitions(ntl_lib PUBLIC NTL_GMP_LIP)

target_compile_options(ntl_lib PRIVATE 
    -pthread 
    -march=native 
    -maes 
    -mpclmul
    -msse2
    -mavx2
)

# 1. Find all test source files
file(GLOB NTL_TEST_SOURCES "src/*Test.cpp")
foreach(test_source ${NTL_TEST_SOURCES})                      # 2. Create an executable for each one
    get_filename_component(test_name ${test_source} NAME_WE)  # Get the filename without extension (e.g., "ZZTest")
    add_executable(${test_name} ${test_source})               # Create the executable
    target_link_libraries(${test_name} PRIVATE ntl_lib)       # Link it to your NTL library and GMP
    add_test(NAME ${test_name} COMMAND ${test_name})		  # Register it with CTest
endforeach()
