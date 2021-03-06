cmake_minimum_required(VERSION 3.9) # CMP0069 NEW
include(GNUInstallDirs)

message(STATUS "cmake version ${CMAKE_VERSION}")
if (NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type selected, default to Release")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

project(simdjson
  DESCRIPTION "Parsing gigabytes of JSON per second"
  LANGUAGES CXX C
)

# LTO seems to create all sorts of fun problems. Let us
# disable temporarily.
#include(CheckIPOSupported)
#check_ipo_supported(RESULT ltoresult)
#if(ltoresult)
#  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
#endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_MACOSX_RPATH OFF)
set(CMAKE_THREAD_PREFER_PTHREAD ON)
set(THREADS_PREFER_PTHREAD_FLAG ON)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 3)
set(PROJECT_VERSION_PATCH 1)
set(SIMDJSON_LIB_VERSION "0.3.1" CACHE STRING "simdjson library version")
set(SIMDJSON_LIB_SOVERSION "1" CACHE STRING "simdjson library soversion")

if(MSVC)
  option(SIMDJSON_BUILD_STATIC "Build a static library" ON) # turning it on disables the production of a dynamic library
  option(SIMDJSON_COMPETITION "Compile competitive benchmarks" OFF)
  else()
  option(SIMDJSON_BUILD_STATIC "Build a static library" OFF) # turning it on disables the production of a dynamic library
  option(SIMDJSON_COMPETITION "Compile competitive benchmarks" ON)
endif()
option(SIMDJSON_GOOGLE_BENCHMARKS "compile the Google Benchmark benchmarks" OFF)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/tools/cmake")

#
# Create the top level simdjson library (must be done at this level to use both src/ and include/
# directories)
#
add_subdirectory(windows)
add_subdirectory(include)
add_subdirectory(src)

#
# Compile tools / tests / benchmarks
#
enable_testing()

add_library(test-data INTERFACE)
target_compile_definitions(test-data INTERFACE SIMDJSON_TEST_DATA_DIR="${CMAKE_CURRENT_SOURCE_DIR}/jsonchecker/")
target_compile_definitions(test-data INTERFACE SIMDJSON_BENCHMARK_DATA_DIR="${CMAKE_CURRENT_SOURCE_DIR}/jsonexamples/")

add_subdirectory(dependencies)
add_subdirectory(tools)
add_subdirectory(tests)
add_subdirectory(examples)
add_subdirectory(benchmark)

# for fuzzing, read the comments in the fuzz/CMakeLists.txt file
option(ENABLE_FUZZING "enable building the fuzzers" ON)
if(ENABLE_FUZZING)
  add_subdirectory(fuzz)
endif()

if(${CMAKE_C_COMPILER_ID} MATCHES "Intel") # icc / icpc
  # prevent shared libraries from depending on Intel provided libraries
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -static-intel")
endif()

set(CPACK_PACKAGE_VENDOR "Daniel Lemire")
set(CPACK_PACKAGE_CONTACT "lemire@gmail.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Parsing gigabytes of JSON per second")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

set(CPACK_RPM_PACKAGE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

include(CPack)


