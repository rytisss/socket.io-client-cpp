cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
PROJECT(sioclient)

option(BUILD_SHARED_LIBS "Build the shared library" OFF)
option(Boost_USE_STATIC_LIBS "Use Boost static version" ON)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(MAJOR 1)
set(MINOR 6)
set(PATCH 0)

if(NOT CMAKE_BUILD_TYPE )
MESSAGE(STATUS "not define build type, set to release" )
set(CMAKE_BUILD_TYPE Release )
elseif(NOT (${CMAKE_BUILD_TYPE} STREQUAL "Release" OR ${CMAKE_BUILD_TYPE} STREQUAL "Debug" ))
MESSAGE(SEND_ERROR "CMAKE_BUILD_TYPE must be either Release or Debug")
return()
endif()

set(BOOST_VER "1.69.0" CACHE STRING "boost version" )

set(Boost_USE_MULTITHREADED ON) 
set(Boost_USE_STATIC_RUNTIME OFF) 
find_package(Boost ${BOOST_VER} REQUIRED COMPONENTS system date_time random) 

aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src ALL_SRC)
aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src/internal ALL_SRC)
file(GLOB ALL_HEADERS ${CMAKE_CURRENT_LIST_DIR}/src/*.h )
set(SIO_INCLUDEDIR ${CMAKE_CURRENT_LIST_DIR})

add_library(sioclient ${ALL_SRC})
target_include_directories(sioclient PRIVATE ${Boost_INCLUDE_DIRS} 
    ${CMAKE_CURRENT_LIST_DIR}/src 
    ${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp 
    ${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include
	/usr/local/ssl/include
    #${OPENSSL_INCLUDE_DIR}#RYTIS
)

set_property(TARGET sioclient PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient PROPERTY CXX_STANDARD_REQUIRED ON)
set_property(TARGET sioclient PROPERTY POSITION_INDEPENDENT_CODE ON)

#target_link_libraries(sioclient PRIVATE ${Boost_LIBRARIES})#RYTIS
target_link_libraries(sioclient PRIVATE ${Boost_LIBRARIES}
IF (WIN32)
  ${CMAKE_CURRENT_LIST_DIR}/lib/openssl-1.0.2l-vs2017/lib64/libeay32MT.lib
  ${CMAKE_CURRENT_LIST_DIR}/lib/openssl-1.0.2l-vs2017/lib64/ssleay32MT.lib
ELSE()
  /usr/local/ssl/lib/libcrypto.a
  /usr/local/ssl/lib/libssl.a
ENDIF()
	  )
target_compile_definitions(sioclient PRIVATE -DSIO_TLS) #RYTIS
if(BUILD_SHARED_LIBS)
set_target_properties(sioclient
	PROPERTIES
		SOVERSION ${MAJOR}
		VERSION ${MAJOR}.${MINOR}.${PATCH}
	)
endif()
list(APPEND TARGET_LIBRARIES sioclient)

#find_package(OpenSSL)
add_definitions(-DSIO_TLS)
#if(OPENSSL_FOUND)
add_library(sioclient_tls ${ALL_SRC})
target_include_directories(sioclient_tls PRIVATE ${Boost_INCLUDE_DIRS} 
    ${CMAKE_CURRENT_LIST_DIR}/src 
    ${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp 
    ${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include
    /usr/local/ssl/include
)

set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD_REQUIRED ON)
set_property(TARGET sioclient_tls PROPERTY POSITION_INDEPENDENT_CODE ON)
target_link_libraries(sioclient_tls PRIVATE ${Boost_LIBRARIES}
	/usr/local/ssl/lib/libcrypto.a
	/usr/local/ssl/lib/libssl.a )
target_compile_definitions(sioclient_tls PRIVATE -DSIO_TLS)
if(BUILD_SHARED_LIBS)
set_target_properties(sioclient_tls
	PROPERTIES
		SOVERSION ${MAJOR}
		VERSION ${MAJOR}.${MINOR}.${PATCH}
	)
endif()
list(APPEND TARGET_LIBRARIES sioclient_tls)

#endif()

install(FILES ${ALL_HEADERS} 
    DESTINATION "${CMAKE_CURRENT_LIST_DIR}/build/include"
)

install(TARGETS ${TARGET_LIBRARIES}
    DESTINATION "${CMAKE_CURRENT_LIST_DIR}/build/lib/${CMAKE_BUILD_TYPE}"
)

install(FILES ${Boost_LIBRARIES} 
    DESTINATION "${CMAKE_CURRENT_LIST_DIR}/build/lib/${CMAKE_BUILD_TYPE}"
)
##########################################################