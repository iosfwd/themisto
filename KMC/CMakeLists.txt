cmake_minimum_required (VERSION 3.1)
project(KMC)

## Override flags (raduls won't build with e.g. -march=native)
## but store them for use with other targets
set(CMAKE_CXX_FLAGS_PASSED ${CMAKE_CXX_FLAGS})
if(CMAKE_BUILD_TYPE MATCHES Release)
  set(CMAKE_CXX_FLAGS_PASSED "${CMAKE_CXX_FLAGS_PASSED} ${CMAKE_CXX_FLAGS_RELEASE}")
elseif(CMAKE_BUILD_TYPE MATCHES Debug)
  set(CMAKE_CXX_FLAGS_PASSED "${CMAKE_CXX_FLAGS_PASSED} ${CMAKE_CXX_FLAGS_RELEASE}")
elseif(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release) ## Build Release by default
endif()
set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-sign-compare")
set(CMAKE_CXX_FLAGS_DEBUG " -g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
string(REPLACE " " ";" CMAKE_CXX_FLAGS_PASSED "${CMAKE_CXX_FLAGS_PASSED}")

## Require c++14
set (CMAKE_CXX_STANDARD 14)
set (CMAKE_CXX_STANDARD_REQUIRED ON)

file(GLOB KMER_COUNTER_SOURCES
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/kmer_counter.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/mmer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/mem_disk_file.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/rev_byte.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/bkb_writer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/cpu_info.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/bkb_reader.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/fastq_reader.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/timer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/develop.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/kb_completer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/kb_storer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/kmer.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/splitter.cpp
${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/kb_collector.cpp)

add_library(raduls_sse2 ${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/raduls_sse2.cpp)
target_compile_options(raduls_sse2 PRIVATE "-msse2")
add_library(raduls_sse41 ${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/raduls_sse41.cpp)
target_compile_options(raduls_sse41 PRIVATE "-msse4.1")
add_library(raduls_avx ${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/raduls_avx.cpp)
target_compile_options(raduls_avx PRIVATE "-mavx")
add_library(raduls_avx2 ${CMAKE_CURRENT_SOURCE_DIR}/kmer_counter/raduls_avx2.cpp)
target_compile_options(raduls_avx2 PRIVATE "-mavx2")

file(GLOB KMC_API_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/kmc_api/*.cpp)
add_library(kmcapi ${KMC_API_SOURCES})
target_compile_options(kmcapi PRIVATE "${CMAKE_CXX_FLAGS_PASSED}")

add_library(kmc ${KMER_COUNTER_SOURCES})
target_compile_options(kmc PRIVATE "${CMAKE_CXX_FLAGS_PASSED}")

target_link_libraries(kmc kmcapi raduls_sse2 raduls_sse41 raduls_avx raduls_avx2 ${BZIP2} ${ZLIB})
