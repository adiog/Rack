cmake_minimum_required(VERSION 3.0)

project(rack)

set(CMAKE_CXX_STANDARD 14)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dep/nanovg)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dep/osdialog)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dep/pffft)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dep/rtaudio)

execute_process(
                 COMMAND
                     git rev-parse --short HEAD
                 RESULT_VARIABLE
                     SHORT_HASH_RESULT
                 OUTPUT_VARIABLE
                     SHORT_HASH
                OUTPUT_STRIP_TRAILING_WHITESPACE)
add_definitions(-DVERSION=1.dev.${SHORT_HASH})

if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()

if(LINUX)
    add_definitions(-DARCH_LIN)
endif()
if (APPLE)
    add_definitions(-DARCH_MAC)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,-pagezero_size,10000 -Wl,-image_base,100000000")

    execute_process(
            COMMAND
                brew --prefix
            RESULT_VARIABLE
                BREW_PREFIX_RESULT
            OUTPUT_VARIABLE
                BREW_PREFIX
            OUTPUT_STRIP_TRAILING_WHITESPACE)
    include_directories(${BREW_PREFIX}/include)
    link_directories(${BREW_PREFIX}/lib)
endif()
if(WIN32)
    add_definitions(-DARCH_WIN)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/nanosvg/src)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/nanovg/src)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/osdialog)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/oui-blendish)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/rtaudio/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/rtaudio)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/pffft)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dep/stb)

file(GLOB_RECURSE sources src/**.cpp)
file(GLOB_RECURSE headers include/**.hpp)

add_executable(rack ${sources} ${headers})

target_link_libraries(rack
        GLEW
        glfw
        jansson
        curl
        zip
        z
        pffft
        nanovg
        osdialog
        speexdsp
        rtmidi
        rtaudio
)

if(LINUX)
    target_link_libraries(rack GL dl)
endif()

if(APPLE)
    target_link_libraries(rack "-framework Cocoa" "-framework OpenGL" "-framework IOKit" "-framework CoreVideo" "-framework CoreAudio" "-framework CoreMIDI")
endif()

ADD_CUSTOM_TARGET(res ALL COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_CURRENT_SOURCE_DIR}/res ${CMAKE_CURRENT_BINARY_DIR}/res)