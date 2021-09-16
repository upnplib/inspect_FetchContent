# Copyright (C) 2021 GPL 3 and higher by Ingo Höft,  <Ingo@Hoeft-online.de>
# Redistribution only with this Copyright remark. Last modified: 2021-09-16

cmake_minimum_required(VERSION 3.18)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(INSPECT_FETCH_CONTENT)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "MinSizeRel" CACHE STRING "Choose the type of build, options are: Debug, Release, RelWithDebInfo, or MinSizeRel." FORCE)
    message(STATUS "No build type specified, defaulting to MinSizeRel.")
endif()

message(CHECK_START "Testing FetchContent")

include(FetchContent)

FetchContent_Declare(
    FetchContent_test
    GIT_REPOSITORY  https://github.com/upnplib/hello.git
    GIT_TAG         origin/main
    GIT_SHALLOW     ON
#    GIT_CONFIG      advice.detachedHead=false
)
#FetchContent_MakeAvailable(FetchContent_test)

# Check if population has already been performed
FetchContent_GetProperties(FetchContent_test)
# Sets variables <lcName>_POPULATED, <lcName>_SOURCE_DIR, <lcName>_BINARY_DIR if available.
string(TOLOWER "FetchContent_test" lcName)

if(NOT ${lcName}_POPULATED)
    # Fetch the content using previously declared details
    FetchContent_Populate(FetchContent_test)

    # Output with cmake option --log-level=DEBUG
    message(DEBUG "  DEBUG: lcName_SOURCE_DIR is: ${${lcName}_SOURCE_DIR}")
    message(DEBUG "  DEBUG: lcName_BINARY_DIR is: ${${lcName}_BINARY_DIR}")

    # Bring the populated content into the build
#    add_subdirectory(${${lcName}_SOURCE_DIR} ${${lcName}_BINARY_DIR})

    message(STATUS "Configure dependent project")
    execute_process(COMMAND cmake -S ${${lcName}_SOURCE_DIR} -B ${${lcName}_BINARY_DIR} -G "NMake Makefiles" -D CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -D BUILD_TESTING=${BUILD_TESTING}
                    WORKING_DIRECTORY ${${lcName}_SOURCE_DIR}
                    RESULT_VARIABLE RETURN_CODE
                    ERROR_VARIABLE UPNP_ERROR_MESSAGE
                    COMMAND_ECHO STDOUT)
    if(NOT ${RETURN_CODE} EQUAL 0)
        # This will stop the installation
        message(FATAL_ERROR "${RETURN_CODE}: ${UPNP_ERROR_MESSAGE}")
    endif()

    message(STATUS "Build dependent project")
    execute_process(COMMAND cmake --build  ${${lcName}_BINARY_DIR} --config ${CMAKE_BUILD_TYPE} #--target install
                    WORKING_DIRECTORY ${${lcName}_SOURCE_DIR}
                    RESULT_VARIABLE RETURN_CODE
                    ERROR_VARIABLE UPNP_ERROR_MESSAGE
                    COMMAND_ECHO STDOUT)
    if(NOT ${RETURN_CODE} EQUAL 0)
        # This will stop the installation
        message(FATAL_ERROR "${RETURN_CODE}: ${UPNP_ERROR_MESSAGE}")
    endif()
endif()

message(CHECK_PASS "done")
