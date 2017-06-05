cmake_minimum_required(VERSION 3.1.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.18.66.tar.gz"
    SHA1 "26028d26e2a9e0496fd7c896fa04e05b309ab6fd"
)

project(onmt)

option(LIB_ONLY "Do not compile clients" OFF)
option(WITH_OPENMP "Use OpenMP if available" ON)
option(BUILD_SHARED_LIBS "Build shared libraries" ON)

set(CMAKE_CXX_STANDARD 11)

# Set Release build type by default to get sane performance.
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

message(STATUS "Build type: " ${CMAKE_BUILD_TYPE})

if(WITH_OPENMP)
  find_package(OpenMP)
  if(OPENMP_FOUND)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  else()
    message(WARNING "OpenMP not found: Compilation will not use OpenMP")
  endif()
endif()

if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Wall")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
endif()

####
# Installation (https://github.com/forexample/package-example) {

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }

hunter_add_package(Eigen)
find_package(Eigen3 3.3 CONFIG REQUIRED)

add_subdirectory(lib/TH)

hunter_add_package(OpenNMTTokenizer)
find_package(OpenNMTTokenizer CONFIG REQUIRED)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

if(NOT HUNTER_ENABLED)
  find_package(CUDA 6.5 QUIET)
endif()

set(SOURCES
  src/th/Env.cc
  src/th/Obj.cc
  src/th/Utils.cc
  src/Dictionary.cc
  src/PhraseTable.cc
  src/Profiler.cc
  src/ITranslator.cc
  src/TranslatorFactory.cc
  src/TranslationResult.cc
  src/Threads.cc
  )

if (CUDA_FOUND)
  add_definitions(-DWITH_CUDA)
  cuda_add_library(${PROJECT_NAME}
    ${SOURCES}
    src/cuda/Utils.cc
    src/cuda/Kernels.cu
    )
  cuda_add_cublas_to_target(${PROJECT_NAME})
else()
  add_library(${PROJECT_NAME} ${SOURCES})
endif()

target_link_libraries(${PROJECT_NAME} PUBLIC
  OpenNMTTokenizer::OpenNMTTokenizer
  TH)

if(ANDROID)
  if(ANDROID_STL MATCHES "gnustl")
    target_compile_definitions(${PROJECT_NAME} PUBLIC "ANDROID_GNUSTL_COMPAT")
  endif()
endif()

target_include_directories(
    ${PROJECT_NAME}
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/lib>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
)

if (NOT LIB_ONLY)
  add_subdirectory(cli)
endif()

install(
    TARGETS ${PROJECT_NAME}
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
  DIRECTORY include/onmt
  DESTINATION include
  FILES_MATCHING PATTERN "*.h*"
  )
