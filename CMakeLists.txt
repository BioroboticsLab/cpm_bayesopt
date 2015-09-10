#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME cpm_bayesopt)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# BayesOpt
#------------------------------------------------------------------------------

set(BAYESOPT_URL "https://bitbucket.org/nebw/bayesopt/get/4ffbb6f38f46.zip")
set(BAYESOPT_ZIP "${CMAKE_BINARY_DIR}/3rdParty/bayesopt.zip")
set(BAYESOPT_UNPACKED "${CMAKE_BINARY_DIR}/3rdParty/nebw-bayesopt-4ffbb6f38f46")

set(REPO_SOURCE_DIR "${CMAKE_BINARY_DIR}/3rdParty/bayesopt")
set(REPO_BUILD_DIR "${CMAKE_BINARY_DIR}/3rdParty/bayesopt_build")
if(NOT EXISTS ${BAYESOPT_ZIP} OR NOT EXISTS ${REPO_SOURCE_DIR})
    message("Downloading BayesOpt 4ffbb6f")
    file(DOWNLOAD ${BAYESOPT_URL} ${BAYESOPT_ZIP} SHOW_PROGRESS
        EXPECTED_HASH SHA1=1171b4781f3774c1ef12b804975ce93f632472e2)
    execute_process(
        COMMAND ${CMAKE_COMMAND} -E tar -xf ${BAYESOPT_ZIP}
        WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/3rdParty/"
        RESULT_VARIABLE error_code)
    if(error_code)
        message(FATAL_ERROR "Failed to unpack BayesOpt")
    endif()
    file(RENAME ${BAYESOPT_UNPACKED} ${REPO_SOURCE_DIR})
endif()

set(BAYESOPT_BUILD_EXAMPLES OFF CACHE BOOL "")
set(BAYESOPT_MATLAB_COMPATIBLE OFF CACHE BOOL "")
set(NLOPT_BUILD ON CACHE BOOL "")
set(NLOPT_BUILD_SHARED OFF CACHE BOOL "")

add_subdirectory("${REPO_SOURCE_DIR}" "${REPO_BUILD_DIR}")

CPM_ExportAdditionalLibraryTarget("bayesopt")
