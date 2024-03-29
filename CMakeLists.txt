cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
PROJECT(Stepper_Motor LANGUAGES ASM C CXX)
set(CMAKE_C_COMPILER_WORKS 1)

#STM32 HAL Family must be set
SET(STM32_FAMILY L4)
#PREFIX must be set
SET(HAL_PREFIX stm32l4xx_)
set(MCU_FAMILY STM32L4)
set(MCU_LINE STM32L433xx)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMake")
set(STM32Cube_DIR "${CMAKE_CURRENT_SOURCE_DIR}/HalL4" CACHE INTERNAL "")
# SET(CMAKE_LINKER_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Linker" CACHE INTERNAL "")
FIND_PACKAGE(CMSIS REQUIRED)
FIND_PACKAGE(STM32L4HAL COMPONENTS uart crc cortex dma exti flash gpio i2c pwr rcc tim REQUIRED)  #required modules list need to specifiy
FIND_PACKAGE(APPLICATION COMPONENTS  Src) #required module folder name

ENABLE_LANGUAGE(ASM)

INCLUDE_DIRECTORIES(
    ${CMSIS_INCLUDE_DIRS}
    ${STM32HAL_INCLUDE_DIR}
    ${BSP_INC_DIR}
    ${APP_INC_DIR}
    )

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME}.elf ${APP_SOURCES} ${CMSIS_SOURCES} ${STM32HAL_SOURCES} ${BSP_SOURCES})
set_target_properties(${CMAKE_PROJECT_NAME}.elf PROPERTIES LINKER_LANGUAGE CXX)

FUNCTION(STM32_ADD_HEX_BIN_TARGETS TARGET)
    IF(EXECUTABLE_OUTPUT_PATH)
      SET(FILENAME "${EXECUTABLE_OUTPUT_PATH}/${TARGET}.elf")
    ELSE()
      SET(FILENAME "${TARGET}.elf")
    ENDIF()
    ADD_CUSTOM_TARGET(${TARGET}.hex ALL DEPENDS ${TARGET}.elf COMMAND ${CMAKE_OBJCOPY} -Oihex  ${FILENAME} ${CMAKE_CURRENT_SOURCE_DIR}/Executable/${TARGET}.hex)

ENDFUNCTION()

FUNCTION(STM32_ADD_DUMP_TARGET TARGET)
    IF(EXECUTABLE_OUTPUT_PATH)
      SET(FILENAME "${EXECUTABLE_OUTPUT_PATH}/${TARGET}.elf")
    ELSE()
      SET(FILENAME "${TARGET}.elf")
    ENDIF()
    ADD_CUSTOM_TARGET(${TARGET}.dump DEPENDS ${TARGET}.elf COMMAND ${CMAKE_OBJDUMP} -x -D -S -s ${FILENAME} | ${CMAKE_CPPFILT} > ${FILENAME}.dump)

ENDFUNCTION()

FUNCTION(STM32_PRINT_SIZE_OF_TARGETS TARGET)
    IF(EXECUTABLE_OUTPUT_PATH)
      SET(FILENAME "${EXECUTABLE_OUTPUT_PATH}/${TARGET}.elf")
    ELSE()
      SET(FILENAME "${TARGET}.elf")
    ENDIF()
    add_custom_command(TARGET ${TARGET}.elf POST_BUILD COMMAND ${CMAKE_SIZE} ${FILENAME})
ENDFUNCTION()

FUNCTION(STM32_REMOVE_EXTERNAL_MEM TARGET)
    IF(EXECUTABLE_OUTPUT_PATH)
      SET(FILENAME "${EXECUTABLE_OUTPUT_PATH}/${TARGET}.elf")
    ELSE()
      SET(FILENAME "${TARGET}.elf")
    ENDIF()
    add_custom_command(TARGET ${TARGET}.elf POST_BUILD COMMAND ${CMAKE_OBJCOPY} --remove-section=.ExtQSPIFlashSection ${FILENAME})

ENDFUNCTION()

STM32_ADD_HEX_BIN_TARGETS(${CMAKE_PROJECT_NAME})
STM32_PRINT_SIZE_OF_TARGETS(${CMAKE_PROJECT_NAME})
