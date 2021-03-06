cmake_minimum_required (VERSION 3.5)
include(GNUInstallDirs)
project (LiSpScPaRo)
message ("Generating build procedure for ${PROJECT_NAME} ...")

if(NOT DEPS_DIR)
   set(DEPS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/..)
endif ()
add_subdirectory(${DEPS_DIR}/FFJSON ${DEPS_DIR}/FFJSON/build)
if (NOT TARGET logger)
   add_subdirectory(${DEPS_DIR}/logger ${DEPS_DIR}/logger/build)
endif ()
if (NOT TARGET ferrybase)
   add_subdirectory(${DEPS_DIR}/ferrybase ${DEPS_DIR}/ferrybase/build)
endif ()

#set (Boost_DEBUG 1)
SET(Boost_USE_STATIC_LIBS ON)
SET(Boost_USE_MULTITHREADED ON)
SET(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost 1.58.0 REQUIRED COMPONENTS system serialization thread exception date_time regex)

IF (DEFINED _DEBUG)
ADD_DEFINITIONS(-D_DEBUG=${_DEBUG})
ENDIF(DEFINED _DEBUG)
set (VERSION_MAJOR 1)
set (VERSION_MINOR 0)
set (Umbrella "ferryfair")
configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
)

message(${CMAKE_SYSTEM_NAME})
# macOS specific code
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(macOS ON)
  set(ARCHITECTURE "Darwin")
endif ()

# Linux specific code
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set(Linux ON)
  set(ARCHITECTURE "Linux")
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

# Windows specific code
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  set(Windows ON)
  set(ARCHITECTURE "Win")
endif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
set(ARCHITECTURE "${ARCHITECTURE}${CMAKE_SYSTEM_PROCESSOR}")

file(GLOB HEADERS RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" *.h)
file(GLOB SOURCES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" *.cpp)
list(FILTER HEADERS EXCLUDE REGEX "^\\..+")
list(FILTER SOURCES EXCLUDE REGEX "^\\..+")
list (APPEND HEADERS "config.h.in")
set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (GCC_COVERAGE_LINK_FLAGS "")

if (Linux)
  set (GCC_COVERAGE_LINK_FLAGS "${GCC_COVERAGE_LINK_FLAGS} -fPIC -Wl,-unresolved-symbols=ignore-in-shared-libs -static")
  set (BUILD_SHARED_LIBRARIES OFF)
endif (Linux)
if (Windows)
  set (BUILD_SHARED_LIBRARIES OFF)
endif (Windows)
set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
set (CMAKE_CXX_LINKER_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}")
set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${GCC_COVERAGE_LINK_FLAGS}")
include_directories(${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_INSTALL_FULL_INCLUDEDIR}/${Umbrella}
)
link_directories("${CMAKE_INSTALL_FULL_LIBDIR}")
message("Configurations")
foreach(CONFIGURATION IN LISTS CMAKE_CONFIGURATION_TYPES)
  message(${CONFIGURATION})
endforeach()
link_directories(
   ${CMAKE_INSTALL_FULL_LIBDIR}
   ${Boost_LIBRARY_DIRS}
   "C:/Program Files/boost_1_65_1/lib64-msvc-14.1"
)
if(NOT LINKAGE)
   set(LINKAGE "STATIC")
   message("Building static by default")
else(NOT LINKAGE)
   string(TOUPPER "${LINKAGE}" LINKAGE)
endif()

if(CMAKE_CONFIGURATION_TYPES)
   message("Linking corresponding to the Configuration Name")
   link_directories("${DEPS_DIR}/ferrybase/build/$(ConfigurationName)")
   link_directories("${DEPS_DIR}/logger/build/$(ConfigurationName)")
   link_directories("${DEPS_DIR}/FFJSON/build/$(ConfigurationName)")
else(CMAKE_CONFIGURATION_TYPES)
   link_directories("${DEPS_DIR}/ferrybase/build")
   link_directories("${DEPS_DIR}/logger/build")
   link_directories("${DEPS_DIR}/FFJSON/build")
endif()
message("ARCHITECTURE: ${ARCHITECTURE}/${CMAKE_BUILD_TYPE}")
message("${Boost_LIBRARY_DIRS}${Boost_INCLUDE_DIRS}")
include_directories(${Boost_INCLUDE_DIRS} "${DEPS_DIR}" "${DEPS_DIR}/FFJSON" "${DEPS_DIR}/logger")
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} README.md)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} FFJSON logger ferrybase)
#set_target_properties(${PROJECT_NAME}
#    PROPERTIES
#    VERSION ${VERSION_MAJOR}.${VERSION_MINOR}
#    SOVERSION ${VERSION_MAJOR}
#)
install(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES "${PROJECT_SOURCE_DIR}/config.json"
   DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}
   RENAME ${PROJECT_NAME}.json
)
install(FILES "${PROJECT_SOURCE_DIR}/init.conf"
   DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}/init
   RENAME ${PROJECT_NAME}.init
)
install(FILES "${PROJECT_SOURCE_DIR}/init.override"
   DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}/init
   RENAME ${PROJECT_NAME}.override
)
