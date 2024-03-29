############################################################################
# CMakeLists.txt
# Copyright (C) 2010-2021 Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#
############################################################################

# CMake 3.13 required for new Flexisip build process.
# See [CMP077](https://cmake.org/cmake/help/v3.13/policy/CMP0077.html).
cmake_minimum_required(VERSION 3.13)

project(mediastreamer2 VERSION 5.1.0 LANGUAGES C CXX)

if(WIN32)
    if( NOT CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
        add_compile_definitions(_WIN32_WINNT=0x0601 _ALLOW_KEYWORD_MACROS)	#_WIN32_WINNT_WIN7
    endif()
endif()

# See: https://cmake.org/cmake/help/v3.12/policy/CMP0072.html
if(POLICY CMP0072)
	cmake_policy(SET CMP0072 NEW)
endif()

set(MEDIASTREAMER_MAJOR_VERSION ${PROJECT_VERSION_MAJOR})
set(MEDIASTREAMER_MINOR_VERSION ${PROJECT_VERSION_MINOR})
set(MEDIASTREAMER_MICRO_VERSION ${PROJECT_VERSION_PATCH})
set(MEDIASTREAMER_VERSION ${PROJECT_VERSION})
set(MEDIASTREAMER_SO_VERSION "11") # incremented for mediastreamer2-4.4.0

string(REGEX MATCH "^(arm*|aarch64)" FIXED_POINT_PROCESSOR "${CMAKE_SYSTEM_PROCESSOR}")
if(FIXED_POINT_PROCESSOR)
	set(ENABLE_FIXED_POINT_DEFAULT_VALUE YES)
else()
	set(ENABLE_FIXED_POINT_DEFAULT_VALUE NO)
endif()
if(IOS)
	set(ENABLE_MACSND_DEFAULT_VALUE NO)
	set(ENABLE_MACAQSND_DEFAULT_VALUE YES)
else()
	set(ENABLE_MACSND_DEFAULT_VALUE YES)
	set(ENABLE_MACAQSND_DEFAULT_VALUE NO)
endif()
set(ENABLE_GLX_DEFAULT_VALUE NO)
set(ENABLE_X11_DEFAULT_VALUE NO)
if(UNIX)
	if(NOT APPLE)
		set(ENABLE_GLX_DEFAULT_VALUE YES)
		set(ENABLE_X11_DEFAULT_VALUE YES)
	endif()
endif()
if(UNIX AND NOT APPLE AND NOT ANDROID)
	set(LINUX_OR_BSD 1)
endif()

include(CMakeDependentOption)

option(ENABLE_SHARED "Build shared library." YES)
option(ENABLE_STATIC "Build static library." YES)
option(ENABLE_DEBUG_LOGS "Enable the display of traces showing the execution of the library." NO)
option(ENABLE_DOC "Enable documentation generation with Doxygen." YES)
option(ENABLE_FIXED_POINT "Turn on fixed point computations." ${ENABLE_FIXED_POINT_DEFAULT_VALUE})
option(ENABLE_NON_FREE_CODECS "Allow inclusion of non-free codecs in the build." NO)
option(ENABLE_PCAP "Enable PCAP support." NO)
option(ENABLE_STRICT "Build with strict compile options." YES)
option(ENABLE_RELATIVE_PREFIX "Find resources relatively to the installation directory." NO)
option(ENABLE_TOOLS "Turn on or off compilation of tools." YES)
option(ENABLE_UNIT_TESTS "Enable compilation of unit tests." YES)
option(ENABLE_PACKAGE_SOURCE "Create 'package_source' target for source archive making (CMake >= 3.11)" OFF)

option(ENABLE_SRTP "Build with the SRTP transport support." YES)
cmake_dependent_option(ENABLE_ZRTP "Build with ZRTP support." YES "ENABLE_SRTP" NO)

option(ENABLE_SOUND "Can be used to turn off all possible sound backends." YES)
cmake_dependent_option(ENABLE_ALSA "Enable ALSA support." YES "ENABLE_SOUND;LINUX_OR_BSD" NO)
cmake_dependent_option(ENABLE_ANDROIDSND "Enable Android sound support." NO "ENABLE_SOUND;ANDROID" NO)
cmake_dependent_option(ENABLE_ARTSC "Enable artsc (KDE<4) sound input/output." NO "ENABLE_SOUND;LINUX_OR_BSD" NO)
cmake_dependent_option(ENABLE_MACSND "Enable MAC OS X Audio Units sound support." ${ENABLE_MACSND_DEFAULT_VALUE} "ENABLE_SOUND;APPLE" NO)
cmake_dependent_option(ENABLE_MACAQSND "Enable native MAC OS X Audio Queue sound support." ${ENABLE_MACAQSND_DEFAULT_VALUE} "ENABLE_SOUND;APPLE" NO)
cmake_dependent_option(ENABLE_QSA "Enable QSA (QNX Sound Architecture) support." NO "ENABLE_SOUND" NO)
# OSS is mostly deprecated in 2016, so it should not be built by default (it won't work anyway). Let user defines it
# itself only if he REALLY needs it.
cmake_dependent_option(ENABLE_OSS "Enable OSS support." NO "ENABLE_SOUND;LINUX_OR_BSD" NO)
cmake_dependent_option(ENABLE_PORTAUDIO "Enable portaudio native support." NO "ENABLE_SOUND" NO)
cmake_dependent_option(ENABLE_PULSEAUDIO "Enable pulseaudio support." YES "ENABLE_SOUND" NO)
option(ENABLE_G726 "Build mediastreamer2 with the G726 codec." NO)
option(ENABLE_GSM "Build mediastreamer2 with the GSM codec." YES)
option(ENABLE_BV16 "Build mediastreamer2 with the BV16 codec." YES)
option(ENABLE_OPUS "Build mediastreamer2 with the OPUS codec." YES)
option(ENABLE_SPEEX_CODEC "Build mediastreamer2 with the SPEEX codec." YES)
option(ENABLE_SPEEX_DSP "Build mediastreamer2 with the SPEEX DSP support." YES)
option(ENABLE_G729 "Build mediastreamer2 with the G729 codec." YES)
option(ENABLE_G729B_CNG "Build mediastreamer2 with G729 annex B cng." NO)
cmake_dependent_option(ENABLE_RESAMPLE "Build mediastreamer2 with the resampling capability." YES "ENABLE_SPEEX_DSP" NO)

option(ENABLE_VIDEO "Build mediastreamer2 with video support." YES)
option(ENABLE_QT_GL "Enable Qt OpenGL rendering support." OFF)
cmake_dependent_option(ENABLE_FFMPEG "Build mediastreamer2 with ffmpeg video support." YES "ENABLE_VIDEO" NO)
cmake_dependent_option(ENABLE_GL "Enable generic OpenGL rendering support." YES "ENABLE_VIDEO" NO)
cmake_dependent_option(ENABLE_GLX "Enable X11+OpenGL rendering support (requires glx and glew)." ${ENABLE_GLX_DEFAULT_VALUE} "ENABLE_VIDEO;UNIX" NO)
cmake_dependent_option(ENABLE_SDL "Enable SDL support." NO "ENABLE_FFMPEG" NO)
cmake_dependent_option(ENABLE_V4L "Enable Video4Linux support." YES "ENABLE_VIDEO;LINUX_OR_BSD" NO)
cmake_dependent_option(ENABLE_X11 "Enable X11 support." ${ENABLE_X11_DEFAULT_VALUE} "ENABLE_VIDEO;UNIX" NO)
cmake_dependent_option(ENABLE_XV "Enable XV support." ${ENABLE_X11_DEFAULT_VALUE} "ENABLE_VIDEO;UNIX" NO)
cmake_dependent_option(ENABLE_THEORA "Enable theora video codec support." YES "ENABLE_VIDEO" NO)
cmake_dependent_option(ENABLE_VPX "Build mediastreamer2 with VPX codec." YES "ENABLE_VIDEO" NO)
cmake_dependent_option(ENABLE_LIBYUV "Build mediastreamer2 with LibYUV support." YES "ENABLE_VIDEO" NO)
option(ENABLE_MKV "Enable support of MKV files reading and writing." YES)
cmake_dependent_option(ENABLE_QNX "Build medistreamer2 with BB10 filters." NO "ENABLE_VIDEO" NO)
option(ENABLE_JPEG "Enable JPEG support" YES)
cmake_dependent_option(ENABLE_QRCODE "Enable QRCode support" YES "ENABLE_VIDEO" NO)

# Hidden non-cache options:
# * DISABLE_BC_PACKAGE_SEARCH: skip find_package() for every BC package (bctoolbox, ortp, etc.)
# * DISABLE_SRTP_SEARCH: skip find_package() for SRTP.


set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS NO)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(CheckIncludeFile)
include(CheckLibraryExists)
include(CMakePushCheckState)
include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

if(MSVC)
	if(MSVC_VERSION LESS 1800)
		set(MSVC_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include/MSVC")
		list(APPEND CMAKE_REQUIRED_INCLUDES "${MSVC_INCLUDE_DIR}")
	else()
		set(MSVC_INCLUDE_DIR "")
	endif()
endif()

check_include_file(sys/shm.h HAVE_SYS_SHM_H)
check_include_file(alloca.h HAVE_ALLOCA_H)
if(ENABLE_OSS)
	check_include_file(soundcard.h HAVE_SOUNDCARD_H)
	check_include_file(sys/soundcard.h HAVE_SYS_SOUNDCARD_H)
	check_include_file(machine/soundcard.h HAVE_MACHINE_SOUNDCARD_H)
	check_include_file(sys/audio.h HAVE_SYS_AUDIO_H)
	if(NOT (HAVE_SOUNDCARD_H OR HAVE_SYS_SOUNDCARD_H OR HAVE_MACHINE_SOUNDCARD_H OR HAVE_SYS_AUDIO_H))
		set(ENABLE_OSS OFF CACHE BOOL "Enable OSS support." FORCE)
	endif()
endif()

check_library_exists("dl" "dlopen" "" HAVE_DLOPEN)

include(TestBigEndian)
test_big_endian(WORDS_BIGENDIAN)

if(NOT DISABLE_BC_PACKAGE_SEARCH)
	find_package(ortp REQUIRED CONFIG)
endif()

if(ENABLE_PCAP)
	find_package(PCAP QUIET)
endif()
if(ENABLE_SRTP)
	if(NOT DISABLE_SRTP_SEARCH)
		find_package(SRTP "2" REQUIRED)
	else()
		set(SRTP_FOUND YES)
		set(SRTP_VERSION 2)
	endif()
endif()
if(ENABLE_ZRTP AND NOT DISABLE_BC_PACKAGE_SEARCH)
	find_package(bzrtp REQUIRED CONFIG)
endif()
if(ENABLE_G726)
	find_package(SpanDSP REQUIRED)
endif()
if(ENABLE_GSM)
	find_package(GSM)
	if(NOT GSM_FOUND)
		message(WARNING "Could not find GSM library, mediastreamer2 will be compiled without the GSM codec.")
		set(ENABLE_GSM OFF CACHE BOOL "Build mediastreamer2 with the GSM codec." FORCE)
	endif()
endif()
if(ENABLE_BV16)
	find_package(BV16)
	if(NOT BV16_FOUND)
		message(WARNING "Could not find BV16 library, mediastreamer2 will be compiled without the BV16 codec.")
		set(ENABLE_BV16 OFF CACHE BOOL "Build mediastreamer2 with the BV16 codec." FORCE)
	endif()
endif()
if(ENABLE_OPUS)
	find_package(Opus)
	if(NOT OPUS_FOUND)
		message(WARNING "Could not find opus library, mediastreamer2 will be compiled without the OPUS codec.")
		set(ENABLE_OPUS OFF CACHE BOOL "Build mediastreamer2 with the OPUS codec." FORCE)
	endif()
endif()
if(ENABLE_SPEEX_CODEC)
	find_package(Speex)
	if(NOT SPEEX_FOUND)
		message(WARNING "Could not find speex library, mediastreamer2 will be compiled without the SPEEX codec.")
		set(ENABLE_SPEEX_CODEC OFF CACHE BOOL "Build mediastreamer2 with the SPEEX codec." FORCE)
	endif()
endif()
if(ENABLE_SPEEX_DSP)
	find_package(SpeexDsp REQUIRED)
endif()
if(ENABLE_ALSA)
	find_package(ALSA)
	if(NOT ALSA_FOUND)
		set(ENABLE_ALSA OFF CACHE BOOL "Enable ALSA support." FORCE)
	endif()
endif()
if(ENABLE_ARTSC)
	find_package(Arts REQUIRED)
endif()
if(ENABLE_PORTAUDIO)
	find_package(PortAudio REQUIRED)
endif()
if(ENABLE_PULSEAUDIO)
	find_package(PulseAudio)
	if(NOT PULSEAUDIO_FOUND)
		set(ENABLE_PULSEAUDIO OFF CACHE BOOL "Enable pulseaudio support." FORCE)
	endif()
endif()
if(ENABLE_QSA)
	find_package(QSA REQUIRED)
	find_package(QnxAudioManager REQUIRED)
endif()
if(ENABLE_VIDEO)
	add_definitions(-DVIDEO_ENABLED)
endif()
if(ENABLE_FFMPEG)
	find_package(FFMpeg REQUIRED)
else()
	add_definitions(-DNO_FFMPEG)
endif()
if(ENABLE_V4L)
	find_package(V4L)
	if(NOT V4L_FOUND)
		message(FATAL_ERROR "Missing V4L. It is highly recommended to build with libv4l2 headers and library. Many camera will won't work or will crash your application if libv4l2 is not installed. If you know what you are doing, you can use -DENABLE_V4L=0 to disable this check.")
	endif()
endif()
if(ENABLE_X11)
	find_package(X11)
	if(NOT X11_FOUND)
		set(ENABLE_X11 OFF CACHE BOOL "Enable X11 support." FORCE)
	endif()
endif()
if(ENABLE_XV)
	find_package(Xv)
	if(XV_FOUND)
		set(HAVE_XV 1)
	else()
		set(HAVE_XV 0)
		set(ENABLE_XV OFF CACHE BOOL "Enable XV support." FORCE)
	endif()
else()
	set(HAVE_XV 0)
endif()
if(ENABLE_GL)
	if(WIN32)
		find_package(QtAngle)
		set(HAVE_GL 1)
		if(NOT QTANGLE_FOUND)#Not QtAngle found then we use internal files
			include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include/OpenGL")
		endif()
	else()
		find_package(OpenGL)
		if(OPENGL_FOUND)
			set(HAVE_GL 1)
			include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include/OpenGL")
		else()
			set(ENABLE_GL OFF CACHE BOOL "Enable generic OpenGL rendering support." FORCE)
		endif()
	endif()
endif()

if(ENABLE_VIDEO AND APPLE AND IOS)
	include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include/OpenGL")
endif()
if(ENABLE_GLX)
	find_package(GLX)
	if(GLX_FOUND)
		set(HAVE_GLX 1)
	else()
		set(ENABLE_GLX OFF CACHE BOOL "Enable X11+OpenGL rendering support (requires glx and glew)." FORCE)
	endif()
endif()
if(NOT APPLE AND NOT ANDROID AND OPENGL_FOUND OR GLX_FOUND)	
	set(GLEW_VERBOSE 1)
	include(FindGLEW)#use cmake find in place of find_package(Glew REQUIRED)
	if(NOT GLEW_FOUND)
		message(FATAL_ERROR "Glew is missing. Mediastreamer2 need it")
	endif()
	message(STATUS "GLEW_VERSION = ${GLEW_VERSION}")
	message(STATUS "GLEW_MAJOR_VERSION = ${GLEW_VERSION_MAJOR}")
	message(STATUS "GLEW_MINOR_VERSION = ${GLEW_VERSION_MINOR}")
	message(STATUS "GLEW_LIBRARIES = ${GLEW_LIBRARIES}")
	message(STATUS "GLEW_INCLUDE_DIRS = ${GLEW_INCLUDE_DIRS}")
	if( GLEW_VERSION_MAJOR LESS_EQUAL 1 AND GLEW_VERSION_MINOR LESS_EQUAL 11)
		set(MS2_USE_OLD_OPENGL_PROTOTYPE TRUE)#Before 1.11, Glew have few different headers from OpenGL Specs
	endif()
endif()
# TODO: Handle SDL option
if(ENABLE_THEORA)
	find_package(Theora)
	if(NOT THEORA_FOUND)
		message(WARNING "Could not find theora library, mediastreamer2 will be compiled without theora support.")
		set(ENABLE_THEORA OFF CACHE BOOL "Enable theora video codec support." FORCE)
	endif()
endif()
if(ENABLE_VPX)
	find_package(VPX)
	if(VPX_FOUND)
		set(HAVE_VPX 1)
	else()
		set(ENABLE_VPX OFF CACHE BOOL "Build mediastreamer2 with VPX codec." FORCE)
	endif()
endif()
if(ENABLE_MKV)
	find_package(BcMatroska2 QUIET)
	if(NOT BCMATROSKA2_FOUND)
		message(WARNING "Could not find BcMatroska2 library, mediastreamer2 will be compiled without MKV support.")
		set(ENABLE_MKV OFF CACHE BOOL "Enable support of MKV files reading and writing." FORCE)
	endif()
endif()

if(ENABLE_QNX)
	find_package(Screen)
	if(NOT SCREEN_FOUND)
		message(FATAL_ERROR "SCREEN NOT FOUND")
	endif()
	find_package(CamApi)
	if(NOT CAMAPI_FOUND)
		message(FATAL_ERROR "CAMAPI NOT FOUND")
	endif()
endif()

if(ENABLE_SOUND AND NOT(WIN32 OR ENABLE_ALSA OR ENABLE_ARTSC OR ENABLE_MACSND OR ENABLE_MACAQSND OR ENABLE_OSS OR ENABLE_PORTAUDIO OR ENABLE_PULSEAUDIO OR ENABLE_QSA OR ENABLE_ANDROIDSND))
	message(FATAL_ERROR "Could not find a support sound driver API. Use -DENABLE_SOUND=NO if you don't care about having sound.")
endif()

if(ENABLE_G729 OR ENABLE_G729B_CNG)
	find_package(Bcg729 1.0.1 CONFIG)
	if(NOT BCG729_FOUND)
		message(WARNING "Could not find bcg729, mediastreamer2 will be compiled without G729 codec and G729 AnnexB in RFC3389.")
		set(ENABLE_G729 OFF CACHE BOOL "Build mediastreamer2 with the G729 codec." FORCE)
		set(ENABLE_G729_CNG OFF CACHE BOOL "Build mediastreamer2 with G729 Annex B cng." FORCE)
	endif()
endif()

if(ENABLE_JPEG)
	find_package(TurboJpeg)
	if(NOT TURBOJPEG_FOUND)
		message(WARNING "Could not find libturbo-jpeg, mediastreamer2 will be compiled without LibJpeg-Turbo.")
		set(ENABLE_JPEG OFF CACHE BOOL "Enable JPEG support" FORCE)
	endif()
endif()
if(ENABLE_LIBYUV)
	find_package(LibYUV)
	if( NOT LIBYUV_FOUND)
		message(WARNING "Could not find libyuv, mediastreamer2 will be compiled without LibYUV support.")
		set(ENABLE_LIBYUV OFF CACHE BOOL "Enable LibYUV support" FORCE)
	endif()
endif()
if(ENABLE_QRCODE)
	find_package(Zxing)
	if(NOT ZXING_FOUND)
		message(WARNING "Could not find Zxing, mediastreamer2 will be compiled without Zxing(QRCode)")
		set(ENABLE_QRCODE OFF CACHE BOOL "Enable QRCode support" FORCE)
	else()
		add_definitions(-DQRCODE_ENABLED)
	endif()
endif()
if(ANDROID)
	find_library(ANDROID_CPU_FEATURES_LIBRARY NAMES cpufeatures)
	if (CMAKE_ANDROID_NDK_VERSION VERSION_LESS 19)
		find_library(ANDROID_SUPPORT_LIBRARY NAMES support)
	endif()
endif()


if(NOT DISABLE_BC_PACKAGE_SEARCH)
	find_package(bctoolbox 0.0.3 REQUIRED OPTIONAL_COMPONENTS tester CONFIG)
endif()

include_directories(
	include
	src/audiofilters
	src/utils
	src/voip
	${CMAKE_CURRENT_BINARY_DIR}
	${CMAKE_CURRENT_BINARY_DIR}/src
)

set(MEDIASTREAMER2_INCLUDE_DIRS )
set(LINK_LIBS bctoolbox ortp)
if(WIN32)
	if(NOT CMAKE_SYSTEM_NAME STREQUAL "WindowsPhone" AND NOT CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
		list(APPEND LINK_LIBS ole32 oleaut32 uuid Winmm Ws2_32)
		if(ENABLE_VIDEO)
			list(APPEND LINK_LIBS gdi32 strmiids user32 vfw32)
		endif()
	endif()
	list(APPEND LINK_LIBS ws2_32)# symbols for in6addr_any
endif()
if(LIBM)
	list(APPEND LINK_LIBS m)
endif()
if(ANDROID_CPU_FEATURES_LIBRARY)
	list(APPEND LINK_LIBS ${ANDROID_CPU_FEATURES_LIBRARY})
endif()
if(ANDROID_SUPPORT_LIBRARY)
	list(APPEND LINK_LIBS ${ANDROID_SUPPORT_LIBRARY})
endif()
if(ANDROID AND ENABLE_VIDEO)
	list(APPEND LINK_LIBS EGL GLESv2 android)
endif()
if(HAVE_DLOPEN)
	list(APPEND LINK_LIBS dl)
endif()
if(ALSA_FOUND)
	list(APPEND LINK_LIBS asound)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${ALSA_INCLUDE_DIRS})
	set(__ALSA_ENABLED__ 1)
endif()
if(ARTS_FOUND)
	list(APPEND LINK_LIBS arts)
  	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${ARTS_INCLUDE_DIRS})
	set(__ARTS_ENABLED__ 1)
endif()
if(PORTAUDIO_FOUND)
	list(APPEND LINK_LIBS portaudio)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${PORTAUDIO_INCLUDE_DIRS})
	set(__PORTAUDIO_ENABLED__ 1)
endif()
if(PULSEAUDIO_FOUND)
	list(APPEND LINK_LIBS pulse)
  	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${PULSEAUDIO_INCLUDE_DIRS})
  	set(__PULSEAUDIO_ENABLED__ 1)
endif()
if(QSA_FOUND)
	list(APPEND LINK_LIBS ${QSA_LIBRARIES})
	list(APPEND LINK_LIBS ${QNXAUDIOMANAGER_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${QSA_INCLUDE_DIRS})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${QNXAUDIOMANAGER_INCLUDE_DIRS})
	set(__QSA_ENABLED__ 1)
	set(__QNX__ 1)
endif()
if(FFMPEG_FOUND)
	list(APPEND LINK_LIBS ${FFMPEG_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${FFMPEG_INCLUDE_DIRS})
	if(MSVC)
		add_definitions(-Dinline=__inline)
	endif()
endif()
if(V4L_FOUND)
	list(APPEND LINK_LIBS ${V4L_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${V4L_INCLUDE_DIRS})
endif()
if(X11_FOUND)
	list(APPEND LINK_LIBS X11)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${X11_INCLUDE_DIRS})
endif()
if(XV_FOUND)
	list(APPEND LINK_LIBS Xv)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${XV_INCLUDE_DIRS})
endif()
if(GLX_FOUND)
	list(APPEND LINK_LIBS GL)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${GLX_INCLUDE_DIRS})
endif()
if(GLEW_FOUND)
	list(APPEND LINK_LIBS GLEW)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${GLEW_INCLUDE_DIRS})
endif()
if(THEORA_FOUND)
	list(APPEND LINK_LIBS theora)
endif()
if(VPX_FOUND)
	list(APPEND LINK_LIBS ${VPX_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${VPX_INCLUDE_DIRS})
endif()
if (SCREEN_FOUND)
	list(APPEND LINK_LIBS screen)
 	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${SCREEN_INCLUDE_DIRS})
endif()
if (CAMAPI_FOUND)
	list(APPEND LINK_LIBS camapi)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${CAMAPI_INCLUDE_DIRS})
endif()
if(TURBOJPEG_FOUND)
	list(APPEND LINK_LIBS ${TURBOJPEG_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${TURBOJPEG_INCLUDE_DIRS})
	set(HAVE_TURBO_JPEG 1)
endif()
if( LIBYUV_FOUND)
	list(APPEND LINK_LIBS ${LIBYUV_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${LIBYUV_INCLUDE_DIRS})
endif()
if(ZXING_FOUND)
	list(APPEND LINK_LIBS ${ZXING_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${ZXING_INCLUDE_DIRS})
endif()
if(BCG729_FOUND)
	if(ENABLE_G729_CNG)
		add_definitions("-DHAVE_G729B=1")
	endif()
	list(APPEND LINK_LIBS ${BCG729_LIBRARIES})
endif()
if(GSM_FOUND)
	list(APPEND LINK_LIBS ${GSM_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${GSM_INCLUDE_DIRS})
endif()
if(BZRTP_FOUND)
	list(APPEND LINK_LIBS bzrtp)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${BZRTP_INCLUDE_DIRS})
	set(HAVE_ZRTP 1)
endif()

if(SRTP_FOUND)
	list(APPEND LINK_LIBS "SRTP")
	set(HAVE_SRTP 1)
endif()
if(SPANDSP_FOUND)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${SPANDSP_INCLUDE_DIRS})
	list(APPEND LINK_LIBS ${SPANDSP_LIBRARIES})
endif()
if(OPUS_FOUND)
	list(APPEND LINK_LIBS ${OPUS_LIBRARIES})
 	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${OPUS_INCLUDE_DIRS})
endif()
if(SPEEX_FOUND)
	list(APPEND LINK_LIBS ${SPEEX_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${SPEEX_INCLUDE_DIRS})
endif()
if(SPEEXDSP_FOUND)
	list(APPEND LINK_LIBS  ${SPEEXDSP_LIBRARIES})
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${SPEEXDSP_INCLUDE_DIRS})
	set(HAVE_SPEEXDSP 1)
endif()
if(BV16_FOUND)
	list(APPEND LINK_LIBS ${BV16_LIBRARIES})
endif()
if(BCMATROSKA2_FOUND)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${BCMATROSKA2_INCLUDE_DIRS})
	list(APPEND LINK_LIBS ${BCMATROSKA2_LIBRARIES})
	set(HAVE_MATROSKA 1)
endif()
if(PCAP_FOUND)
  	list(APPEND LINK_LIBS pcap)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${PCAP_INCLUDE_DIRS})
	set(HAVE_PCAP 1)
endif()
if(QTANGLE_FOUND)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${QTANGLE_INCLUDE_DIRS})
	list(APPEND LINK_LIBS ${QTANGLE_LIBRARIES})
endif()
if(OPENGL_FOUND)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${OPENGL_INCLUDE_DIR})
endif()
if(MSVC_INCLUDE_DIR)
	list(APPEND MEDIASTREAMER2_INCLUDE_DIRS ${MSVC_INCLUDE_DIR})
endif()
if(ENABLE_ANDROIDSND)
	include_directories(
		src/android
		src/android/media
	)
endif()
if(ENABLE_DEBUG_LOGS)
	add_definitions(-DDEBUG)
endif()
if(ENABLE_FIXED_POINT)
	set(MS_FIXED_POINT 1)
endif()
if(ENABLE_NON_FREE_CODECS)
	set(HAVE_NON_FREE_CODECS 1)
else()
	set(HAVE_NON_FREE_CODECS 0)
endif()
if(ENABLE_MACSND)
	set(__MACSND_ENABLED__ 1)
endif()
if(ENABLE_MACAQSND)
	set(__MAC_AQ_ENABLED__ 1)
endif()

if(ENABLE_QT_GL)#Only add headers and lib to Qt plugins
	set(QT5_PACKAGES Core Gui Quick QmlModels Qml Network)

# Cmake/Qt hackshit : find_package read CMAKE_PREFIX_PATH and QT5_DIR but cannot find any Qt5Config.cmake even if there are inside them.
	if(LINPHONESDK_QT_DIR)
		set(Qt5_base "${LINPHONESDK_QT_DIR}/lib/cmake/")
		set(Qt5_DIR "${Qt5_base}/Qt5")
		set(ENV{Qt5_DIR} "${Qt5_base}/Qt5")#Use env and local too because it could not work without them
		foreach (package ${QT5_PACKAGES})
			set(Qt5${package}_DIR "${Qt5_base}/Qt5${package}")
			set(ENV{Qt5${package}_DIR} "${Qt5_base}/Qt5${package}")
		endforeach ()
	endif()
# end of hackshit

	find_package(Qt5 COMPONENTS ${QT5_PACKAGES} QUIET)
	foreach (package ${QT5_PACKAGES})
		if(Qt5${package}_FOUND)
			get_target_property(Qt${package}_location Qt5::${package} LOCATION)
			string(REPLACE "Qt${package}.framework/Qt${package}" "" Qt${package}_ROOT_LOCATION ${Qt${package}_location})
			list(APPEND MEDIASTREAMER2_INCLUDE_DIRS "${Qt5${package}_INCLUDE_DIRS}")
			list(APPEND QT_LINK_LIBS ${Qt5${package}_LIBRARIES})
		endif()
	endforeach ()
endif()
set(LINK_FLAGS )
if(APPLE)
	list(APPEND LINK_FLAGS "-framework CoreFoundation" "-framework AudioToolbox" "-framework CoreAudio")
	if(IOS) # Necessary for Mediastreamer's tools and testers
		list(APPEND LINK_FLAGS "-framework Foundation" "-framework QuartzCore" "-framework OpenGLES" "-framework UIKit" "-framework AVFoundation")
		if(ENABLE_VIDEO)
			list(APPEND LINK_FLAGS "-framework CoreGraphics" "-framework CoreMedia" "-framework CoreVideo" "-framework VideoToolbox")
		endif()
	else()
		list(APPEND LINK_FLAGS "-framework AppKit")
		if(ENABLE_MACSND)
			list(APPEND LINK_FLAGS "-framework Carbon" "-framework AudioUnit")
		endif()
		if(ENABLE_VIDEO)
			list(APPEND LINK_FLAGS "-framework Cocoa" "-framework OpenGL" "-framework QuartzCore" "-framework AVFoundation" "-framework VideoToolbox" "-framework CoreMedia")
		endif()
		add_definitions("-DTARGET_OS_MAC=1")
	endif()
elseif(WIN32 AND CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")#Fix error on 'vccorlib_lib_should_be_specified_before_msvcrt_lib_to_linker''
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        list(APPEND LINK_FLAGS "/nodefaultlib:vccorlibd /nodefaultlib:msvcrtd vccorlibd.lib msvcrtd.lib")
    else()
        list(APPEND LINK_FLAGS "/nodefaultlib:vccorlib /nodefaultlib:msvcrt vccorlib.lib msvcrt.lib")
    endif()
endif()
string(REPLACE ";" " " LINK_FLAGS_STR "${LINK_FLAGS}")

set(MEDIASTREAMER2_CPPFLAGS ${ORTP_CPPFLAGS} ${BZRTP_CPPFLAGS} ${BCTOOLBOX_CPPFLAGS} ${BCG729_CPPFLAGS})
if(ENABLE_STATIC)
	list(APPEND MEDIASTREAMER2_CPPFLAGS "-DMS2_STATIC")
endif()
if(MEDIASTREAMER2_CPPFLAGS)
	list(REMOVE_DUPLICATES MEDIASTREAMER2_CPPFLAGS)
	add_definitions(${MEDIASTREAMER2_CPPFLAGS})
endif()

set(STRICT_OPTIONS_CPP )
set(STRICT_OPTIONS_C )
set(STRICT_OPTIONS_OBJC "-Wno-error")
if(MSVC)
	list(APPEND STRICT_OPTIONS_CPP "/wd4142") # Disable "benign redefinition of type" warnings.
	list(APPEND STRICT_OPTIONS_CPP "/wd4996") # Disable POSIX functions warnings from bctbx.
	if(ENABLE_STRICT)
		list(APPEND STRICT_OPTIONS_CPP "/WX")
	endif()
else()
	list(APPEND STRICT_OPTIONS_CPP "-Wall" "-Wuninitialized")
	list(APPEND STRICT_OPTIONS_C "-Wstrict-prototypes" "-Wno-error=strict-prototypes")

	if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
		list(APPEND STRICT_OPTIONS_CXX "-x c++")
		if(${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER_EQUAL "10")
			list(APPEND STRICT_OPTIONS_CXX "-Wno-deprecated-copy") # introduced by clang 10
		endif()
	elseif(ENABLE_QT_GL AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND ${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER "9")
		list(APPEND STRICT_OPTIONS_CPP "-Wno-error=deprecated-copy") # introduced by gcc 9
	endif()

	if(CMAKE_C_COMPILER_ID MATCHES "Clang")
		list(APPEND STRICT_OPTIONS_CPP "-Qunused-arguments" "-Wno-unused-function" "-Wno-array-bounds")
	elseif(CMAKE_C_COMPILER_ID STREQUAL "GNU")
		if(${CMAKE_C_COMPILER_VERSION} VERSION_GREATER "7")
		    list(APPEND STRICT_OPTIONS_CPP "-Wno-format-truncation")
		endif()
		if(${CMAKE_C_COMPILER_VERSION} VERSION_GREATER "8")
			list(APPEND STRICT_OPTIONS_CPP "-Wno-error=stringop-truncation" "-Wno-cast-function-type")
		endif()
	endif()
	if(APPLE)
		list(APPEND STRICT_OPTIONS_CPP "-Wno-error=unknown-warning-option" "-Qunused-arguments" "-Wno-tautological-compare" "-Wno-unused-function" "-Wno-array-bounds")
	endif()
	if(APPLE OR WIN32)
		list(APPEND STRICT_OPTIONS_CPP "-Wno-shorten-64-to-32")
	endif()
	if(ENABLE_STRICT)
		list(APPEND STRICT_OPTIONS_CPP "-Werror" "-Wextra" "-Wno-unused-parameter" "-fno-strict-aliasing" "-Wno-missing-field-initializers" "-Wno-error=deprecated-declarations")
	endif()
endif()
if(STRICT_OPTIONS_CPP)
	list(REMOVE_DUPLICATES STRICT_OPTIONS_CPP)
endif()
if(STRICT_OPTIONS_C)
	list(REMOVE_DUPLICATES STRICT_OPTIONS_C)
endif()
if(STRICT_OPTIONS_OBJC)
	list(REMOVE_DUPLICATES STRICT_OPTIONS_OBJC)
endif()

set(EXPORT_TARGETS_NAME "Mediastreamer2")

# Set log domain for the entire project.
add_definitions(
	-DBCTBX_LOG_DOMAIN="mediastreamer"
)

#Setup framework structure and variables for ms2 plugins
set(MS2_FRAMEWORK_VERSION "A")

if(APPLE AND NOT IOS)
	set(MS2_PLUGINS_LOCATION "Frameworks/mediastreamer2.framework/Versions/${MS2_FRAMEWORK_VERSION}/Libraries")

	install(DIRECTORY DESTINATION "${CMAKE_INSTALL_PREFIX}/${MS2_PLUGINS_LOCATION}")

	if (NOT EXISTS "${CMAKE_INSTALL_PREFIX}/Frameworks/mediastreamer2.framework/Libraries")
	     install(CODE "execute_process(COMMAND sh -c \"ln -sf Versions/Current/Libraries ${CMAKE_INSTALL_PREFIX}/Frameworks/mediastreamer2.framework/Libraries\")")
	endif()
else()
	set(MS2_PLUGINS_LOCATION "${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins")
endif()

if(ENABLE_RELATIVE_PREFIX)
	if(CMAKE_SYSTEM_NAME STREQUAL "WindowsPhone" OR CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
		set(PACKAGE_PLUGINS_DIR ".")
		set(PACKAGE_DATA_DIR ".")
	else()
		set(PACKAGE_PLUGINS_DIR "./${MS2_PLUGINS_LOCATION}")
		set(PACKAGE_DATA_DIR "./${CMAKE_INSTALL_DATADIR}")
	endif()
else()
	set(PACKAGE_PLUGINS_DIR "${CMAKE_INSTALL_PREFIX}/${MS2_PLUGINS_LOCATION}")
	set(PACKAGE_DATA_DIR "${CMAKE_INSTALL_DATADIR}")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/mediastreamer-config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/mediastreamer-config.h)
set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/mediastreamer-config.h PROPERTIES GENERATED ON)
add_definitions("-DHAVE_CONFIG_H")

if(ENABLE_DOC)
	add_subdirectory(help)
endif()
add_subdirectory(include)
add_subdirectory(src)
if(ENABLE_UNIT_TESTS)
	add_subdirectory(tester)
endif()
if(ENABLE_TOOLS AND NOT ANDROID)
	add_subdirectory(tools)
endif()

include(CMakePackageConfigHelpers)
set(CONFIG_PACKAGE_LOCATION "${CMAKE_INSTALL_DATADIR}/Mediastreamer2/cmake")

write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/Mediastreamer2ConfigVersion.cmake"
	VERSION ${MEDIASTREAMER_VERSION}
	COMPATIBILITY AnyNewerVersion
)
if(LINPHONE_BUILDER_GROUP_EXTERNAL_SOURCE_PATH_BUILDERS)
	export(EXPORT ${EXPORT_TARGETS_NAME}Targets
		FILE "${CMAKE_CURRENT_BINARY_DIR}/Mediastreamer2Targets.cmake"
	)
endif()
configure_package_config_file(cmake/Mediastreamer2Config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/Mediastreamer2Config.cmake"
	INSTALL_DESTINATION ${CONFIG_PACKAGE_LOCATION}
	NO_SET_AND_CHECK_MACRO
)

install(EXPORT ${EXPORT_TARGETS_NAME}Targets
	FILE Mediastreamer2Targets.cmake
	DESTINATION ${CONFIG_PACKAGE_LOCATION}
)
install(FILES
	"${CMAKE_CURRENT_BINARY_DIR}/Mediastreamer2Config.cmake"
	"${CMAKE_CURRENT_BINARY_DIR}/Mediastreamer2ConfigVersion.cmake"
	DESTINATION ${CONFIG_PACKAGE_LOCATION}
)

if (ENABLE_PACKAGE_SOURCE)
	add_subdirectory(build)
endif()
