cmake_minimum_required(VERSION 3.21)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
else()
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
endif()
set(CMAKE_CXX_VISIBILITY_PRESET hidden)

project(DiscordRichPresense VERSION 1.0.0)

set(DISCORD_RPC_SRC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/discord-rpc/src)

add_library(discord-rpc STATIC
    ${DISCORD_RPC_SRC}/discord_rpc.cpp
    ${DISCORD_RPC_SRC}/rpc_connection.cpp
    ${DISCORD_RPC_SRC}/serialization.cpp
    $<$<PLATFORM_ID:Windows>:${DISCORD_RPC_SRC}/connection_win.cpp>
    $<$<OR:$<PLATFORM_ID:Darwin>,$<PLATFORM_ID:iOS>>:${DISCORD_RPC_SRC}/connection_unix.cpp>
    $<$<PLATFORM_ID:Android>:${DISCORD_RPC_SRC}/connection_unix.cpp>
)

target_include_directories(discord-rpc PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/vendor/discord-rpc/include
    ${CMAKE_CURRENT_SOURCE_DIR}/vendor/discord-rpc/src
    ${CMAKE_CURRENT_SOURCE_DIR}/vendor/discord-rpc/thirdparty/rapidjson/include
)

target_compile_definitions(discord-rpc PRIVATE DISCORD_DISABLE_IO_THREAD)

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS src/*.cpp)
add_library(${PROJECT_NAME} SHARED ${SOURCES})

if (NOT DEFINED ENV{GEODE_SDK})
    message(FATAL_ERROR "Unable to find Geode SDK! Please define GEODE_SDK environment variable to point to Geode")
else()
    message(STATUS "Found Geode: $ENV{GEODE_SDK}")
endif()

add_subdirectory($ENV{GEODE_SDK} ${CMAKE_CURRENT_BINARY_DIR}/geode)

target_link_libraries(${PROJECT_NAME}
    geode-sdk
    discord-rpc
)

if (WIN32)
    target_link_libraries(${PROJECT_NAME} ws2_32)
endif()

setup_geode_mod(${PROJECT_NAME})