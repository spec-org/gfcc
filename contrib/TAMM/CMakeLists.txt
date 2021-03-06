cmake_minimum_required(VERSION 3.18.0)

include(FetchContent)

project(TAMM VERSION 0.0.0 LANGUAGES C CXX Fortran)
include(${PROJECT_SOURCE_DIR}/cmake/check_user_env.cmake)

find_package( CMakeBuild QUIET )

if( NOT ${CMakeBuild_FOUND} )
  if (NOT CMSB_TAG)
    set(CMSB_TAG dc7840bca1c879f09c193a392696fa7246856f22)
    if (ENABLE_DEV_MODE)
      set(CMSB_TAG master)
    endif()
  endif()

  FetchContent_Declare(
    CMakeBuild
    GIT_REPOSITORY https://github.com/NWChemEx-Project/CMakeBuild.git
    GIT_TAG ${CMSB_TAG}
  )

  FetchContent_MakeAvailable( CMakeBuild )
  set(CMSB_MACROS ${CMakeBuild_SOURCE_DIR}/cmake/macros)

  FetchContent_MakeAvailable( linalg-cmake-modules )
  set(CMSB_LAM_PATH ${linalg-cmake-modules_SOURCE_DIR})
endif()

set(CMSB_PROJECTS TAMM)

set(TAMM_DEPENDENCIES NJSON MSGSL DOCTEST Eigen3 LibInt2 HPTT HDF5) # GlobalArrays BLAS LAPACK

if(USE_CUDA OR USE_HIP) 
   list(APPEND TAMM_DEPENDENCIES TALSH)
endif()

if(DEFINED LINALG_VENDOR)
    if(NOT ${LINALG_VENDOR} STREQUAL "BLIS")
        list(APPEND TAMM_DEPENDENCIES BLIS)
    endif()
endif()

if(USE_GA_AT)
    if(USE_DPCPP)
        list(APPEND TAMM_DEPENDENCIES IntelSYCL)
    endif()
    list(APPEND TAMM_DEPENDENCIES GlobalArrays)
    if(USE_SCALAPACK)
        list(APPEND TAMM_DEPENDENCIES ScaLAPACK LAPACK BLAS)
    else()
        list(APPEND TAMM_DEPENDENCIES LAPACK BLAS)
    endif()
else()
    list(APPEND TAMM_DEPENDENCIES GlobalArrays)
endif()

set(BUILD_METHODS OFF)
set(TAMM_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/tamm)
set(TAMM_TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/tests/)


if(NOT CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE Release)
endif()

set(CMSB_BASISSET_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../basis)
build_cmsb_module(${CMAKE_CURRENT_LIST_DIR})

