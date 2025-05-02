cmake_minimum_required(VERSION 3.10)

project(APCpp)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

add_library(APCpp SHARED Archipelago.cpp)
set_target_properties(APCpp PROPERTIES PUBLIC_HEADER Archipelago.h)

add_library(APCpp-static STATIC Archipelago.cpp)
set_target_properties(APCpp-static PROPERTIES PUBLIC_HEADER Archipelago.h)

set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(ENABLE_TESTING OFF CACHE BOOL "" FORCE)
set(ENABLE_PROGRAMS OFF CACHE BOOL "" FORCE)

# Enable TLS. On Windows, build mbedTLS
set(USE_TLS ON CACHE BOOL "Enable TLS support" FORCE)
if (WIN32 AND USE_TLS)
    execute_process(COMMAND ${CMAKE_COMMAND} -E remove -f mbedtls-3.4.0)
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xzf ${CMAKE_CURRENT_SOURCE_DIR}/mbedtls-3.4.0.tar.gz)
    set(USE_STATIC_MBEDTLS_LIBRARY ON CACHE BOOL "" FORCE)
    set(MBEDTLS_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/mbedtls-3.4.0/include)
    set(MBEDTLS_VERSION_GREATER_THAN_3 ${MBEDTLS_INCLUDE_DIRS})
    set(MBEDTLS_LIBRARY mbedtls)
    set(MBEDX509_LIBRARY mbedx509)
    set(MBEDCRYPTO_LIBRARY mbedcrypto)
    add_subdirectory(${CMAKE_BINARY_DIR}/mbedtls-3.4.0 mbedtls_bin)
endif (WIN32 AND USE_TLS)

if (NOT MINGW)
    find_package(ZLIB)
endif (NOT MINGW)
if (MINGW OR NOT ZLIB_FOUND)
    set(ZLIB_BUILD_SHARED OFF)
    include(FetchContent)
    FetchContent_Declare(
        zlib
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib
        OVERRIDE_FIND_PACKAGE
    )
    FetchContent_MakeAvailable(zlib)
    add_library(ZLIB::ZLIB ALIAS zlibstatic)
endif (MINGW OR NOT ZLIB_FOUND)

set(IXWEBSOCKET_INSTALL OFF CACHE BOOL "" FORCE)
add_subdirectory(IXWebSocket)
include_directories(IXWebSocket)
target_include_directories(APCpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip)
target_include_directories(APCpp-static PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip)
target_sources(APCpp PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip/unzip.c
    ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip/ioapi.c)
target_sources(APCpp-static PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip/unzip.c
    ${CMAKE_CURRENT_SOURCE_DIR}/lib/zlib/contrib/minizip/ioapi.c)

target_link_libraries(APCpp ixwebsocket ZLIB::ZLIB)
target_link_libraries(APCpp-static ixwebsocket ZLIB::ZLIB)

# Always use bundled JsonCpp
set(JSONCPP_WITH_TESTS OFF CACHE BOOL "" FORCE)
set(JSONCPP_WITH_POST_BUILD_UNITTEST OFF CACHE BOOL "" FORCE)
add_subdirectory(jsoncpp)
set_property(TARGET jsoncpp_static PROPERTY POSITION_INDEPENDENT_CODE ON)
target_include_directories(APCpp PRIVATE jsoncpp/include)
target_include_directories(APCpp-static PRIVATE jsoncpp/include)
target_link_libraries(APCpp jsoncpp_static)
target_link_libraries(APCpp-static jsoncpp_static)

if (WIN32)
    target_link_libraries(APCpp ws2_32)
endif(WIN32)

install(TARGETS APCpp PUBLIC_HEADER)
install(TARGETS APCpp-static PUBLIC_HEADER)
