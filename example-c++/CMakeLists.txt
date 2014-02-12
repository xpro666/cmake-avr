##########################################################################
# "THE ANY BEVERAGE-WARE LICENSE" (Revision 42 - based on beer-ware
# license):
# <dev@layer128.net> wrote this file. As long as you retain this notice
# you can do whatever you want with this stuff. If we meet some day, and
# you think this stuff is worth it, you can buy me a be(ve)er(age) in 
# return. (I don't like beer much.)
# 
# Matthias Kleemann
##########################################################################

cmake_minimum_required(VERSION 2.8)

project(AVR-CPP-EXAMPLE)

##########################################################################
# tools to beused for programming the AVR
##########################################################################
set(AVR_UPLOADTOOL avrdude)
set(AVR_PROGRAMMER avrispmkII)
set(AVR_UPLOADTOOL_PORT usb)

##########################################################################
# AVR and fuses needs to be set
##########################################################################
set(AVR_MCU atmega8)
set(AVR_H_FUSE 0xd9)
set(AVR_L_FUSE 0xc3)

##########################################################################
# status messages
##########################################################################
message(STATUS "Current uploadtool is: ${AVR_UPLOADTOOL}")
message(STATUS "Current programmer is: ${AVR_PROGRAMMER}")
message(STATUS "Current upload port is: ${AVR_UPLOADTOOL_PORT}")
message(STATUS "Current uploadtool options are: ${AVR_UPLOADTOOL_OPTIONS}")
message(STATUS "Current MCU is set to: ${AVR_MCU}")
message(STATUS "Current H_FUSE is set to: ${AVR_H_FUSE}")
message(STATUS "Current L_FUSE is set to: ${AVR_L_FUSE}")

##########################################################################
# set build type
##########################################################################
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

##########################################################################
# needs to be defined for AVR toolchain
##########################################################################
set(MCU_SPEED "4000000UL")

##########################################################################
# some cmake cross-compile necessities
##########################################################################
if(CMAKE_HOST_SYSTEM MATCHES "Windows")
    if(DEFINED ENV{AVR_FIND_ROOT_PATH})
        set(CMAKE_FIND_ROOT_PATH $ENV{AVR_FIND_ROOT_PATH})
    else(DEFINED ENV{AVR_FIND_ROOT_PATH})
        message(FATAL_ERROR "Please set AVR_FIND_ROOT_PATH in your environment.")
    endif(DEFINED ENV{AVR_FIND_ROOT_PATH})
else(CMAKE_HOST_SYSTEM MATCHES "Windows")
    set(CMAKE_FIND_ROOT_PATH "/usr/avr")
endif(CMAKE_HOST_SYSTEM MATCHES "Windows")
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
# not added automatically, since CMAKE_SYSTEM_NAME is "generic"
set(CMAKE_SYSTEM_INCLUDE_PATH "${CMAKE_FIND_ROOT_PATH}/include")
set(CMAKE_SYSTEM_LIBRARY_PATH "${CMAKE_FIND_ROOT_PATH}/lib")

##########################################################################
# status messages for generating
##########################################################################
message(STATUS "Set CMAKE_FIND_ROOT_PATH to ${CMAKE_FIND_ROOT_PATH}")
message(STATUS "Set CMAKE_SYSTEM_INCLUDE_PATH to ${CMAKE_SYSTEM_INCLUDE_PATH}")
message(STATUS "Set CMAKE_SYSTEM_LIBRARY_PATH to ${CMAKE_SYSTEM_LIBRARY_PATH}")

##########################################################################
# set compiler options for build types
##########################################################################
if(CMAKE_BUILD_TYPE MATCHES Release)
   set(CMAKE_C_FLAGS_RELEASE "-Os")
   set(CMAKE_CXX_FLAGS_RELEASE "-Os")
endif(CMAKE_BUILD_TYPE MATCHES Release)

if(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
   set(CMAKE_C_FLAGS_RELWITHDEBINFO "-Os -save-temps -g -gdwarf-3 -gstrict-dwarf")
   set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-Os -save-temps -g -gdwarf-3 -gstrict-dwarf")
endif(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)

if(CMAKE_BUILD_TYPE MATCHES Debug)
   set(CMAKE_C_FLAGS_DEBUG "-O0 -save-temps -g -gdwarf-3 -gstrict-dwarf")
   set(CMAKE_CXX_FLAGS_DEBUG "-O0 -save-temps -g -gdwarf-3 -gstrict-dwarf")
endif(CMAKE_BUILD_TYPE MATCHES Debug)

##########################################################################
# compiler options for all build types
##########################################################################
add_definitions("-DF_CPU=${MCU_SPEED}")
add_definitions("-fpack-struct")
add_definitions("-fshort-enums")
add_definitions("-Wall")
add_definitions("-Werror")
add_definitions("-pedantic")
add_definitions("-pedantic-errors")
add_definitions("-funsigned-char")
add_definitions("-funsigned-bitfields")
add_definitions("-ffunction-sections")
add_definitions("-c")

##########################################################################
# include search paths
##########################################################################
include_directories(${AVR-CPP-EXAMPLE_SOURCE_DIR}/mycpplib)

##########################################################################
# building library and application in their subdirectories
##########################################################################
add_subdirectory(mycpplib)
add_subdirectory(cppapp)

##################################################################################
# find doxygen
##################################################################################
find_package(Doxygen)

##################################################################################
# create docu target, if package was found
##################################################################################
if(DOXYGEN_FOUND)
    # configuration input and output
    set(DOXYGEN_CONF_IN ${AVR-CPP-EXAMPLE_SOURCE_DIR}/doxygen.conf)
    set(DOXYGEN_CONF_OUT doxygen.conf)

    # set to override variable within configuration
    set(DOXYGEN_INPUT_PATH "\"${CMAKE_CURRENT_SOURCE_DIR}\"")
    set(DOXYGEN_OUTPUT_PATH "\"${CMAKE_CURRENT_BINARY_DIR}\"")
    set(DOXYGEN_IMAGE_PATH "\"${CMAKE_CURRENT_SOURCE_DIR}\"")
    set(DOXYGEN_DOTFILE_PATH "\"${CMAKE_CURRENT_SOURCE_DIR}\"")

    # cleanup properties from html output
    set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
                    "${CMAKE_CURRENT_BINARY_DIR}/html"
    )

    # copy configuration file and replace input path
    configure_file( ${DOXYGEN_CONF_IN}
                    ${DOXYGEN_CONF_OUT}
                    @ONLY
    )

    # add documentation target
    add_custom_target(
        docu
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_CONF_OUT}
        COMMENT "Create HTML documentation."
    )
else(DOXYGEN_FOUND)
    message(WARNING "Doxygen not found. Documentation target not created.")
endif(DOXYGEN_FOUND)

