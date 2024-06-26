# FreeRDP: A Remote Desktop Protocol Implementation
# FreeRDP X11 Client
#
# Copyright 2012 Marc-Andre Moreau <marcandre.moreau@gmail.com>
# Copyright 2013 Corey Clayton <can.of.tuna@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(MODULE_NAME "xfreerdp-client")
set(MODULE_PREFIX "FREERDP_CLIENT_X11_CONTROL")

include(FindDocBookXSL)
include_directories(${X11_INCLUDE_DIRS})
include_directories(${OPENSSL_INCLUDE_DIR})

set(${MODULE_PREFIX}_SRCS
	xf_utils.h
	xf_utils.c
	xf_gdi.c
	xf_gdi.h
	xf_gfx.c
	xf_gfx.h
	xf_rail.c
	xf_rail.h
	xf_input.c
	xf_input.h
	xf_event.c
	xf_event.h
	xf_floatbar.c
	xf_floatbar.h
	xf_input.c
	xf_input.h
	xf_channels.c
	xf_channels.h
	xf_cliprdr.c
        xf_cliprdr.h
	xf_monitor.c
	xf_monitor.h
	xf_disp.c
	xf_disp.h
	xf_graphics.c
	xf_graphics.h
	xf_keyboard.c
	xf_keyboard.h
	xf_video.c
	xf_video.h
	xf_window.c
	xf_window.h
	xf_client.c
	xf_client.h)

if (CHANNEL_TSMF_CLIENT)
	set(${MODULE_PREFIX}_SRCS ${${MODULE_PREFIX}_SRCS}
	xf_tsmf.c
	xf_tsmf.h)
endif()

if(WITH_CLIENT_INTERFACE)
	if(CLIENT_INTERFACE_SHARED)
		add_library(${MODULE_NAME} SHARED ${${MODULE_PREFIX}_SRCS})
		if (WITH_LIBRARY_VERSIONING)
			set_target_properties(${MODULE_NAME} PROPERTIES VERSION ${FREERDP_VERSION} SOVERSION ${FREERDP_API_VERSION})
		endif()
	else()
		add_library(${MODULE_NAME} ${${MODULE_PREFIX}_SRCS})
	endif()

else()
	set(${MODULE_PREFIX}_SRCS ${${MODULE_PREFIX}_SRCS} cli/xfreerdp.c xfreerdp.h)
	add_executable(${MODULE_NAME} ${${MODULE_PREFIX}_SRCS})
	set_target_properties(${MODULE_NAME} PROPERTIES OUTPUT_NAME "xfreerdp")
	include_directories(..)
endif()

set(${MODULE_PREFIX}_LIBS
	${X11_LIBRARIES})

if(WITH_MANPAGES)
	find_program(XSLTPROC_EXECUTABLE NAMES xsltproc REQUIRED)
	if (NOT DOCBOOKXSL_FOUND)
		message(FATAL_ERROR "docbook xsl not found but required for manpage generation")
	endif()

	# We need the variable ${MAN_TODAY} to contain the current date in ISO
	# format to replace it in the configure_file step.
	include(today)

	TODAY(MAN_TODAY)

	configure_file(xfreerdp.1.xml.in xfreerdp.1.xml @ONLY IMMEDIATE)

	# Compile the helper tool with default compiler settings.
	# We need the include paths though.
	get_property(dirs DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
	set(GENERATE_INCLUDES "")
	foreach(dir ${dirs})
		set(GENERATE_INCLUDES ${GENERATE_INCLUDES} -I${dir})
	endforeach(dir)

	add_custom_command(OUTPUT xfreerdp.1
				COMMAND ${CMAKE_C_COMPILER} ${GENERATE_INCLUDES}
					${CMAKE_CURRENT_SOURCE_DIR}/generate_argument_docbook.c
					-o ${CMAKE_CURRENT_BINARY_DIR}/generate_argument_docbook
				COMMAND ${CMAKE_CURRENT_BINARY_DIR}/generate_argument_docbook
				COMMAND ${CMAKE_COMMAND} -E copy
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-channels.1.xml ${CMAKE_CURRENT_BINARY_DIR}
				COMMAND ${CMAKE_COMMAND} -E copy
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-examples.1.xml ${CMAKE_CURRENT_BINARY_DIR}
				COMMAND ${CMAKE_COMMAND} -E copy
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-envvar.1.xml ${CMAKE_CURRENT_BINARY_DIR}
				COMMAND ${XSLTPROC_EXECUTABLE} ${DOCBOOKXSL_DIR}/manpages/docbook.xsl xfreerdp.1.xml
				WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
				DEPENDS
					${CMAKE_CURRENT_BINARY_DIR}/xfreerdp.1.xml
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-examples.1.xml
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-channels.1.xml
					${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp-envvar.1.xml)

	add_custom_target(xfreerdp.manpage ALL
		DEPENDS xfreerdp.1)

	install_freerdp_man(${CMAKE_CURRENT_BINARY_DIR}/xfreerdp.1 1)
endif(WITH_MANPAGES)

set(XSHM_FEATURE_TYPE "REQUIRED")
set(XSHM_FEATURE_PURPOSE "X11 shared memory")
set(XSHM_FEATURE_DESCRIPTION "X11 shared memory extension")

set(XINERAMA_FEATURE_TYPE "RECOMMENDED")
set(XINERAMA_FEATURE_PURPOSE "multi-monitor")
set(XINERAMA_FEATURE_DESCRIPTION "X11 multi-monitor extension")

set(XEXT_FEATURE_TYPE "RECOMMENDED")
set(XEXT_FEATURE_PURPOSE "X11 extension")
set(XEXT_FEATURE_DESCRIPTION "X11 core extensions")

set(XCURSOR_FEATURE_TYPE "RECOMMENDED")
set(XCURSOR_FEATURE_PURPOSE "cursor")
set(XCURSOR_FEATURE_DESCRIPTION "X11 cursor extension")

set(XV_FEATURE_TYPE "RECOMMENDED")
set(XV_FEATURE_PURPOSE "video")
set(XV_FEATURE_DESCRIPTION "X11 video extension")

set(XI_FEATURE_TYPE "RECOMMENDED")
set(XI_FEATURE_PURPOSE "input")
set(XI_FEATURE_DESCRIPTION "X11 input extension")

set(XRENDER_FEATURE_TYPE "RECOMMENDED")
set(XRENDER_FEATURE_PURPOSE "rendering")
set(XRENDER_FEATURE_DESCRIPTION "X11 render extension")

set(XRANDR_FEATURE_TYPE "RECOMMENDED")
set(XRANDR_FEATURE_PURPOSE "tracking output configuration")
set(XRANDR_FEATURE_DESCRIPTION "X11 randr extension")

set(XFIXES_FEATURE_TYPE "RECOMMENDED")
set(XFIXES_FEATURE_PURPOSE "X11 xfixes extension")
set(XFIXES_FEATURE_DESCRIPTION "Useful additions to the X11 core protocol")

find_feature(XShm ${XSHM_FEATURE_TYPE} ${XSHM_FEATURE_PURPOSE} ${XSHM_FEATURE_DESCRIPTION})
find_feature(Xinerama ${XINERAMA_FEATURE_TYPE} ${XINERAMA_FEATURE_PURPOSE} ${XINERAMA_FEATURE_DESCRIPTION})
find_feature(Xext ${XEXT_FEATURE_TYPE} ${XEXT_FEATURE_PURPOSE} ${XEXT_FEATURE_DESCRIPTION})
find_feature(Xcursor ${XCURSOR_FEATURE_TYPE} ${XCURSOR_FEATURE_PURPOSE} ${XCURSOR_FEATURE_DESCRIPTION})
find_feature(Xv ${XV_FEATURE_TYPE} ${XV_FEATURE_PURPOSE} ${XV_FEATURE_DESCRIPTION})
find_feature(Xi ${XI_FEATURE_TYPE} ${XI_FEATURE_PURPOSE} ${XI_FEATURE_DESCRIPTION})
find_feature(Xrender ${XRENDER_FEATURE_TYPE} ${XRENDER_FEATURE_PURPOSE} ${XRENDER_FEATURE_DESCRIPTION})
find_feature(XRandR ${XRANDR_FEATURE_TYPE} ${XRANDR_FEATURE_PURPOSE} ${XRANDR_FEATURE_DESCRIPTION})
find_feature(Xfixes ${XFIXES_FEATURE_TYPE} ${XFIXES_FEATURE_PURPOSE} ${XFIXES_FEATURE_DESCRIPTION})

if(WITH_XINERAMA)
	add_definitions(-DWITH_XINERAMA)
	include_directories(${XINERAMA_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XINERAMA_LIBRARIES})
endif()

if(WITH_XEXT)
	add_definitions(-DWITH_XEXT)
	include_directories(${XEXT_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XEXT_LIBRARIES})
endif()

if(WITH_XCURSOR)
	add_definitions(-DWITH_XCURSOR)
	include_directories(${XCURSOR_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XCURSOR_LIBRARIES})
endif()

if(WITH_XV)
	add_definitions(-DWITH_XV)
	include_directories(${XV_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XV_LIBRARIES})
endif()

if(WITH_XI)
	add_definitions(-DWITH_XI)
	include_directories(${XI_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XI_LIBRARIES})
endif()

if(WITH_XRENDER)
	add_definitions(-DWITH_XRENDER)
	include_directories(${XRENDER_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XRENDER_LIBRARIES})
endif()

if(WITH_XRANDR)
	add_definitions(-DWITH_XRANDR)
	include_directories(${XRandR_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XRandR_LIBRARIES})
endif()

if(WITH_XFIXES)
	add_definitions(-DWITH_XFIXES)
	include_directories(${XFIXES_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XFIXES_LIBRARIES})
endif()

include_directories(${PROJECT_SOURCE_DIR}/resources)

set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} freerdp-client freerdp m)
if (NOT APPLE)
	list(APPEND ${MODULE_PREFIX}_LIBS rt)
endif()
target_link_libraries(${MODULE_NAME} PRIVATE ${${MODULE_PREFIX}_LIBS})

if(WITH_IPP)
    target_link_libraries(${MODULE_NAME} PRIVATE ${IPP_LIBRARY_LIST})
endif()

if(WITH_CLIENT_INTERFACE)
	install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT libraries)
	add_subdirectory(cli)
else()
	install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT client)
endif()

set_property(TARGET ${MODULE_NAME} PROPERTY FOLDER "Client/X11")

