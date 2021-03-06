cmake_minimum_required(VERSION 2.8.7)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../" ${CMAKE_MODULE_PATH})
set(EXTERNAL_PROJECT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/TheExternals)
set(EXTERNAL_PROJECT_FILE_PREFIX "TheExternal_")
include(ExternalProject)
include(Artichoke)

project(MarkAsSuperBuild-Test NONE)

option(${PROJECT_NAME}_SUPERBUILD "Build ${PROJECT_NAME} and the projects it depends" ON)
mark_as_advanced(${PROJECT_NAME}_SUPERBUILD)

# Set variables needed for '*_CMAKE_ARGS' case
if(NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_CFG_INTDIR MyConfig)
  set(CMAKE_CONFIGURATION_TYPES ${CMAKE_CFG_INTDIR})
  set(EXPECTED_CMAKE_CFG_INTDIR ${CMAKE_CFG_INTDIR})
else()
  set(EXPECTED_CMAKE_CFG_INTDIR Release)
endif()

if(${PROJECT_NAME}_SUPERBUILD)
  option(FOO_OPTION "Option mark_as_superbuild" ON)
  mark_as_superbuild(FOO_OPTION)

  set(FOO_VAR "FooVar")
  mark_as_superbuild(FOO_VAR)

  set(FOO_LIST Item1 Item2)
  mark_as_superbuild(FOO_LIST)

  set(FOO_LIST_FOR_CMAKE_ARGS Item1 ${CMAKE_CFG_INTDIR})
  mark_as_superbuild(FOO_LIST_FOR_CMAKE_ARGS)

  set(FOO_LIST_FOR_CMAKE_ARGS_WITH_CMAKE_CMD_IGNORED Item1 ${CMAKE_CFG_INTDIR})
  mark_as_superbuild(FOO_LIST_FOR_CMAKE_ARGS_WITH_CMAKE_CMD_IGNORED CMAKE_CMD)

  set(FOO_VAR_WITH_LABELS "FooVarWithLabels")
  mark_as_superbuild(VARS FOO_VAR_WITH_LABELS LABELS "LABEL_A" "LABEL_B")

  set(OTHER_FOO_VAR_WITH_LABELS "OtherFooVarWithLabels")
  mark_as_superbuild(VARS OTHER_FOO_VAR_WITH_LABELS LABELS "LABEL_A" "LABEL_B")

  set(FOO_VAR_FOR_CMAKE_ARGS_WITH_LABELS "FooVarWithLabels-${CMAKE_CFG_INTDIR}")
  mark_as_superbuild(VARS FOO_VAR_FOR_CMAKE_ARGS_WITH_LABELS LABELS "LABEL_A" "LABEL_B")

  set(BAR_VAR_WITH_LABELS "BarVarWithLabels")
  mark_as_superbuild(VARS BAR_VAR_WITH_LABELS LABELS "LABEL_B")

  foreach(lib LibA LibB LibE)
    set(${lib}_VAR ${lib})
    mark_as_superbuild(VARS ${lib}_VAR PROJECTS ${lib})
  endforeach()

  set(LibA_VAR_WITH_LABELS "LibAWithLabels")
  mark_as_superbuild(VARS LibA_VAR_WITH_LABELS PROJECTS LibA LABELS "LABEL_A")

  set(MULTIPROJECT_VAR "MultiProjectVar")
  mark_as_superbuild(VARS MULTIPROJECT_VAR PROJECTS ${PROJECT_NAME} LibA LibB)

  set(MULTIPROJECT_VAR_WITH_LABELS "MultiProjectVarWithLabels")
  mark_as_superbuild(VARS MULTIPROJECT_VAR_WITH_LABELS PROJECTS ${PROJECT_NAME} LibA LibB LABELS "MULTIPROJECT")

  set(ALL_PROJECTS_VAR "AllProjectsVar")
  mark_as_superbuild(VARS ALL_PROJECTS_VAR ALL_PROJECTS)

  # XXX Add 'ALL_PROJECTS' test with labels

  set(WITHCOLON_VAR "c:/path/to/something" CACHE PATH "Variable with colon")
  mark_as_superbuild(WITHCOLON_VAR)

  # Since LibC depends on LibD, LibD will also be expected to be found on the system
  set(${CMAKE_PROJECT_NAME}_USE_SYSTEM_LibC 1)
endif()

set(${PROJECT_NAME}_DEPENDS LibB LibC LibE)

ExternalProject_Include_Dependencies(${PROJECT_NAME} PROJECT_VAR PROJECT_NAME)

if(${PROJECT_NAME}_SUPERBUILD)
  ExternalProject_Add(${PROJECT_NAME}
    ${${PROJECT_NAME}_EP_ARGS}
    DOWNLOAD_COMMAND ""
    CMAKE_CACHE_ARGS
      -D${PROJECT_NAME}_SUPERBUILD:BOOL=OFF
      -DCMAKE_CFG_INTDIR_FILE:FILEPATH=${CMAKE_CFG_INTDIR_FILE}
    SOURCE_DIR ${${PROJECT_NAME}_SOURCE_DIR}
    BINARY_DIR ${${PROJECT_NAME}_BINARY_DIR}/${PROJECT_NAME}-build
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    DEPENDS
      ${${PROJECT_NAME}_DEPENDS}
    )
  return()
endif()

include(${CMAKE_CURRENT_SOURCE_DIR}/../ArtichokeTestUtility.cmake)

check_variable(FOO_OPTION "ON")
check_variable(FOO_VAR "FooVar")
check_variable(FOO_LIST "Item1;Item2")
check_variable(FOO_LIST_FOR_CMAKE_ARGS "Item1;${EXPECTED_CMAKE_CFG_INTDIR}")
check_variable(FOO_LIST_FOR_CMAKE_ARGS_WITH_CMAKE_CMD_IGNORED "Item1;${EXPECTED_CMAKE_CFG_INTDIR}")
check_variable(FOO_VAR_WITH_LABELS "FooVarWithLabels")
check_variable(OTHER_FOO_VAR_WITH_LABELS "OtherFooVarWithLabels")
check_variable(FOO_VAR_FOR_CMAKE_ARGS_WITH_LABELS "FooVarWithLabels-${EXPECTED_CMAKE_CFG_INTDIR}")
check_variable(BAR_VAR_WITH_LABELS "BarVarWithLabels")

check_variable(MarkAsSuperBuild-Test_EP_LABEL_LABEL_A "FOO_VAR_WITH_LABELS;OTHER_FOO_VAR_WITH_LABELS;FOO_VAR_FOR_CMAKE_ARGS_WITH_LABELS")
check_variable(MarkAsSuperBuild-Test_EP_LABEL_LABEL_B "FOO_VAR_WITH_LABELS;OTHER_FOO_VAR_WITH_LABELS;FOO_VAR_FOR_CMAKE_ARGS_WITH_LABELS;BAR_VAR_WITH_LABELS")

check_variable(MarkAsSuperBuild-Test_EP_LABEL_FIND_PACKAGE "LibA_DIR;LibB_DIR;LibF_DIR;LibG_DIR;LibE_DIR")

get_filename_component(expected_LibA_DIR "${CMAKE_CURRENT_BINARY_DIR}/../LibA-build" REALPATH)
check_variable(LibA_DIR ${expected_LibA_DIR})

get_filename_component(expected_LibB_DIR "${CMAKE_CURRENT_BINARY_DIR}/../LibB-build" REALPATH)
check_variable(LibB_DIR ${expected_LibB_DIR})

check_variable(${CMAKE_PROJECT_NAME}_USE_SYSTEM_LibA "")
check_variable(${CMAKE_PROJECT_NAME}_USE_SYSTEM_LibB "")
check_variable(${CMAKE_PROJECT_NAME}_USE_SYSTEM_LibC 1)
check_variable(${CMAKE_PROJECT_NAME}_USE_SYSTEM_LibD 1)

check_variable(MULTIPROJECT_VAR "MultiProjectVar")

check_variable(MarkAsSuperBuild-Test_EP_LABEL_MULTIPROJECT "MULTIPROJECT_VAR_WITH_LABELS")
check_variable(MULTIPROJECT_VAR_WITH_LABELS "MultiProjectVarWithLabels")

check_variable(ALL_PROJECTS_VAR "AllProjectsVar")

check_variable(WITHCOLON_VAR "c:/path/to/something")
