# Copyright (c) 2014 The Chromium Embedded Framework Authors. All rights
# reserved. Use of this source code is governed by a BSD-style license that
# can be found in the LICENSE file.

#
# Source files.
#
project(test_package)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

option(NO_BROWSER_WORKING_TEST "If enabled the application will not try to actually display something" ON)

if(NO_BROWSER_WORKING_TEST)
add_definitions(-DNO_BROWSER_WORKING_TEST)
message(STATUS "Browser working test disabled.")
else()
message(STATUS "Browser working test enabled.")
endif()

include(cmake/cef_macros.cmake) # Same as shipped with cef but modified so we can include them directly (removed 2 lines)
include(cmake/cef_variables.cmake)
include_directories(${CONAN_INCLUDE_DIRS})

# cefsimple sources.
set(CEFSIMPLE_SRCS
  simple_app.cc
  simple_app.h
  simple_handler.cc
  simple_handler.h
  )
set(CEFSIMPLE_SRCS_LINUX
  cefsimple_linux.cc
  simple_handler_linux.cc
  )
set(CEFSIMPLE_SRCS_MACOSX
  cefsimple_mac.mm
  simple_handler_mac.mm
  )
set(CEFSIMPLE_SRCS_WINDOWS
  cefsimple_win.cc
  resource.h
  simple_handler_win.cc
  )
APPEND_PLATFORM_SOURCES(CEFSIMPLE_SRCS)
source_group(cefsimple FILES ${CEFSIMPLE_SRCS})

set(CEFSIMPLE_SRCS
  ${CEFSIMPLE_SRCS}
  ${CEFSIMPLE_RES_SRCS}
  )

# cefsimple helper sources.
set(CEFSIMPLE_HELPER_SRCS_MACOSX
  process_helper_mac.cc
  )
APPEND_PLATFORM_SOURCES(CEFSIMPLE_HELPER_SRCS)
source_group(cefsimple FILES ${CEFSIMPLE_HELPER_SRCS})

#
# Shared configuration.
#

# Target executable names.
set(CEF_TARGET "cefsimple")
if(OS_MACOSX)
  set(CEF_HELPER_TARGET "cefsimple Helper")
endif()


#
# Linux configuration.
#

if(OS_LINUX)
  # Executable target.
  if(NOT NO_BROWSER_WORKING_TEST)
  find_package(X11 REQUIRED)
  include_directories(${X11_INCLUDE_DIR})
  endif()
  add_executable(${CEF_TARGET} ${CEFSIMPLE_SRCS})
  target_link_libraries(${CEF_TARGET} ${CONAN_LIBS})
  if(NOT NO_BROWSER_WORKING_TEST)
    target_link_libraries(${CEF_TARGET} ${X11_LIBRARIES})
  endif()

  # Set rpath so that libraries can be placed next to the executable.
  set_target_properties(${CEF_TARGET} PROPERTIES INSTALL_RPATH "$ORIGIN")
  set_target_properties(${CEF_TARGET} PROPERTIES BUILD_WITH_INSTALL_RPATH TRUE)
endif()


#
# Mac OS X configuration.
#

if(OS_MACOSX)
  # Output paths for the app bundles.
  set(CEF_APP "${CEF_TARGET_OUT_DIR}/${CEF_TARGET}.app")
  set(CEF_HELPER_APP "${CEF_TARGET_OUT_DIR}/${CEF_HELPER_TARGET}.app")

  # Variable referenced from Info.plist files.
  set(PRODUCT_NAME "${CEF_TARGET}")

  # Helper executable target.
  add_executable(${CEF_HELPER_TARGET} ${CEFSIMPLE_HELPER_SRCS})
  target_link_libraries(${CEF_HELPER_TARGET} ${CONAN_LIBS})

  # Main executable target.
  add_executable(${CEF_TARGET} ${CEFSIMPLE_RESOURCES_SRCS} ${CEFSIMPLE_SRCS})
  target_link_libraries(${CEF_TARGET} ${CONAN_LIBS})
endif()


#
# Windows configuration.
#

if(OS_WINDOWS)
  # Executable target.
  add_executable(${CEF_TARGET} WIN32 ${CEFSIMPLE_SRCS})
  target_link_libraries(${CEF_TARGET} ${CONAN_LIBS})
endif()
