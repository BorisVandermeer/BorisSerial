cmake_minimum_required(VERSION 3.21.3)
project(BorisSerial)

# Top Level Check
if( BorisSerial_IS_TOP_LEVEL)
    SET(IS_TOP_DIR YES)
    SET(ENABLE_TEST YES)
else()
    SET(IS_TOP_DIR NO)
    SET(ENABLE_TEST NO)
endif()

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
else()

endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(BORIS_SERIAL_LIB ${serial_SRCS})
if(APPLE)
    target_link_libraries(BORIS_SERIAL_LIB ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(BORIS_SERIAL_LIB rt pthread)
else()
    target_link_libraries(BORIS_SERIAL_LIB setupapi)
endif()
target_include_directories(BORIS_SERIAL_LIB PUBLIC ${BORIS_SERIAL_INCLUDE_DIRS})

SET(BORIS_SERIAL_LIBRARIES BORIS_SERIAL_LIB)
SET(BORIS_SERIAL_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include)

if(IS_TOP_DIR)
    SET(BORIS_SERIAL_VERSION BorisSerial_VERSION )
    SET(BORIS_SERIAL_FOUND YES )
    SET(BORIS_SERIAL_LIBRARIES ${BORIS_SERIAL_LIBRARIES})
    SET(BORIS_SERIAL_INCLUDE_DIRS ${BORIS_SERIAL_INCLUDE_DIRS})
else()
    SET(BORIS_SERIAL_VERSION BorisSerial_VERSION PARENT_SCOPE)
    SET(BORIS_SERIAL_FOUND YES PARENT_SCOPE)
    SET(BORIS_SERIAL_LIBRARIES ${BORIS_SERIAL_LIBRARIES} PARENT_SCOPE)
    SET(BORIS_SERIAL_INCLUDE_DIRS ${BORIS_SERIAL_INCLUDE_DIRS} PARENT_SCOPE)
endif()

## Include headers
# include_directories(include)

## Install executable
# install(TARGETS ${PROJECT_NAME}
#     ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
# )

## Install headers
# install(FILES include/serial/serial.h include/serial/v8stdint.h
#   DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

## Tests
if(ENABLE_TEST)
#     add_subdirectory(tests)
    # Uncomment for example
    include_directories(include)
    add_executable(serial_example examples/serial_example.cc)
    target_link_libraries(serial_example ${BORIS_SERIAL_LIBRARIES})
endif()
