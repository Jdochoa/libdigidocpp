cmake_minimum_required(VERSION 3.10)
project(libdigidocpp VERSION 3.14.5)
if(POLICY CMP0074)
    cmake_policy(SET CMP0074 NEW)
endif()
if(POLICY CMP0078)
    cmake_policy(SET CMP0078 NEW)
endif()
if(POLICY CMP0086)
    cmake_policy(SET CMP0086 OLD)
endif()
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")

include(VersionInfo)
include(GNUInstallDirs)

set_env( TSL_URL "https://ec.europa.eu/tools/lotl/eu-lotl.xml" CACHE STRING "TSL trust list primary URL" )
set_env( TSL_CERTS
    "${CMAKE_SOURCE_DIR}/etc/tl-mp1.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp2.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp3.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp4.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp5.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp6.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp7.crt"
    "${CMAKE_SOURCE_DIR}/etc/tl-mp8.crt"
    CACHE FILEPATH "TSL trust list signing certificates" )
set_env( TSA_URL "http://dd-at.ria.ee/tsa" CACHE STRING "Default TSA URL" )
set_env( SIVA_URL "https://siva.eesti.ee/V3/validate" CACHE STRING "Default SiVa validation service URL" )
set( LIBDIGIDOC_LINKED YES CACHE BOOL "Link with libdigidoc or load runtime with dlopen" )
set( BUILD_TOOLS YES CACHE BOOL "Build digidoc-tool" )
set( BUILD_SHARED_LIBS YES CACHE BOOL "Build library as SHARED or STATIC" )
set( SIGNCERT "" CACHE STRING "Common name of certificate to used sign binaries, empty skip signing" )
set( CROSSSIGNCERT "" CACHE STRING "Common name of certificate to used cross sign binaries, empty skip signing" )
if( APPLE )
    set( FRAMEWORK YES CACHE BOOL "Build library as Mac OS X Framework" )
    set( USE_KEYCHAIN YES CACHE BOOL "Use keychain to store PKCS12 access certificate" )
endif()

find_package(Doxygen)
find_package(LibDigiDoc)
find_package(OpenSSL 1.0.2 REQUIRED)
find_package(PKCS11)
find_package(PoDoFo)
find_package(Threads)
find_package(XercesC REQUIRED)
find_package(XmlSecurityC REQUIRED)
find_package(XSD 4.0 REQUIRED)
find_package(ZLIB REQUIRED)
find_package(MiniZip QUIET)
if(UNIX AND NOT APPLE)
    find_package(PkgConfig)
    pkg_check_modules(MINIZIP minizip IMPORTED_TARGET)
endif()
find_package(SWIG)
find_package(JNI)

if( DOXYGEN_FOUND )
    configure_file( ${CMAKE_SOURCE_DIR}/etc/Doxyfile.in Doxyfile @ONLY )
    add_custom_target( docs ALL
        ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
    install( DIRECTORY ${CMAKE_BINARY_DIR}/doc/ DESTINATION ${CMAKE_INSTALL_DOCDIR} )
    install( DIRECTORY doc/ DESTINATION ${CMAKE_INSTALL_DOCDIR} )
endif()

if( WIN32 )
    add_definitions( -DUNICODE -D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS -DWIN32_LEAN_AND_MEAN )
endif()

enable_testing()
add_subdirectory(src)
add_subdirectory(test)
add_subdirectory(examples)
