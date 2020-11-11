##########################################################################
cmake_minimum_required(VERSION 2.8)

set(AVR_UPLOADTOOL avrdude)
set(AVR_PROGRAMMER usbasp)
set(AVR_UPLOADTOOL_PORT usb)

set(AVR_MCU atmega128a)
set(AVR_H_FUSE 0xd9)
set(AVR_L_FUSE 0xc3)
#set(AVR_E_FUSE 0xFF)
#set(AVR_LOCK_FUSE 0x3F)
set(CONFIG_LOAD_ADDR 0x0000) #--section-start=.text=<address>

set(MCU_SPEED "12000000UL")

##########################################################################
# set Toolchain
##########################################################################
#-DCMAKE_TOOLCHAIN_FILE=C:\development\cmake-avr\generic-gcc-avr.cmake
set(ENV{AVR_FIND_ROOT_PATH} "C:\\development\\Atmel\\Studio\\7.0\\toolchain\\avr8\\avr8-gnu-toolchain")
set(OLD_PATH $ENV{PATH})
list(APPEND OLD_PATH "$ENV{AVR_FIND_ROOT_PATH}\\bin")
list(APPEND OLD_PATH "$ENV{AVR_FIND_ROOT_PATH}\\avr\\bin")
set(ENV{PATH} "${OLD_PATH}")
message(STATUS "Environment AVR_FIND_ROOT_PATH: $ENV{AVR_FIND_ROOT_PATH}")
message(STATUS "Environment PATH: $ENV{PATH}")

set(CMAKE_TOOLCHAIN_FILE C:\\development\\cmake-avr\\generic-gcc-avr.cmake)
#set(CMAKE_VERBOSE_MAKEFILE ON)

project(EXAMPLE_CXX)

set(CMAKE_CXX_STANDARD 11)
##########################################################################
# status messages
##########################################################################
message(STATUS "Current uploadtool is: ${AVR_UPLOADTOOL}")
message(STATUS "Current programmer is: ${AVR_PROGRAMMER}")
message(STATUS "Current upload port is: ${AVR_UPLOADTOOL_PORT}")
message(STATUS "Current uploadtool options are: ${AVR_UPLOADTOOL_OPTIONS}")
message(STATUS "Current MCU is set to: ${AVR_MCU}")
message(STATUS "Current MCU_SPEED is set to: ${MCU_SPEED}")
message(STATUS "Current H_FUSE is set to: ${AVR_H_FUSE}")
message(STATUS "Current L_FUSE is set to: ${AVR_L_FUSE}")
message(STATUS "Current E_FUSE is set to: ${AVR_E_FUSE}")
message(STATUS "Current LOCK_FUSE is set to: ${AVR_LOCK_FUSE}")
message(STATUS "Current CONFIG_LOAD_ADDR is set to: ${CONFIG_LOAD_ADDR}")

##########################################################################
# set build type
##########################################################################
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

##########################################################################
# some cmake cross-compile necessities
##########################################################################

if(DEFINED ENV{AVR_FIND_ROOT_PATH})
  set(CMAKE_FIND_ROOT_PATH $ENV{AVR_FIND_ROOT_PATH})
else(DEFINED ENV{AVR_FIND_ROOT_PATH})
  if(EXISTS "/opt/local/avr")
    set(CMAKE_FIND_ROOT_PATH "/opt/local/avr")
  elseif(EXISTS "/usr/avr")
    set(CMAKE_FIND_ROOT_PATH "/usr/avr")
  elseif(EXISTS "/usr/lib/avr")
    set(CMAKE_FIND_ROOT_PATH "/usr/lib/avr")
  elseif(EXISTS "/usr/local/CrossPack-AVR")
    set(CMAKE_FIND_ROOT_PATH "/usr/local/CrossPack-AVR")
  else(EXISTS "/opt/local/avr")
    message(FATAL_ERROR "Please set AVR_FIND_ROOT_PATH in your environment.")
  endif(EXISTS "/opt/local/avr")
endif(DEFINED ENV{AVR_FIND_ROOT_PATH})
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
include_directories("$ENV{AVR_FIND_ROOT_PATH}/include")
include_directories("$ENV{AVR_FIND_ROOT_PATH}/avr/include")
#include_directories("$ENV{AVR_FIND_ROOT_PATH}/avr/include/avr")
link_directories($ENV{AVR_FIND_ROOT_PATH}/lib)
link_directories($ENV{AVR_FIND_ROOT_PATH}/avr/lib)

#DoingSoft Libs
include_directories(C:\\projects\\avr\\DoingSoftAvrLibs)

#FreeRTOS
#SET(FREE_RTOS_SOURCE_PATH "C:\\development\\FreeRTOSv10.4.1\\FreeRTOS\\Source")
#include_directories("${FREE_RTOS_SOURCE_PATH}/include")
#include_directories("${FREE_RTOS_SOURCE_PATH}/portable/ThirdParty/GCC/ATmega")
#FreeRTOS Demo
#SET(FREERTOS_DEMO_PATH "C:/development/FreeRTOSv10.4.1/FreeRTOS/Demo")
#include_directories("${FREERTOS_DEMO_PATH}/Common/include")

include_directories(${PROJECT_SOURCE_DIR})

avr_generate_fixed_targets()

##########################################################################
# building library and application in their subdirectories
##########################################################################
add_subdirectory(mycpplib)
add_subdirectory(cppapp)

#add_avr_executable(
#    ${PROJECT_NAME}
#)
