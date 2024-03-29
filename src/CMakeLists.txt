############################################################################
# CMakeLists.txt
# Copyright (C) 2014  Belledonne Communications, Grenoble France
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

find_library(LIBM NAMES m)

find_program(AWK_PROGRAM awk CMAKE_FIND_ROOT_PATH_BOTH)
if(NOT AWK_PROGRAM)
	message(FATAL_ERROR "You need the awk program to build mediastreamer2.")
endif()

set(BASE_SOURCE_FILES_C
	base/eventqueue.c
	base/mscommon.c
	base/msfactory.c
	base/msfilter.c
	base/msqueue.c
	base/mssndcard.c
	base/msticker.c
	base/msvideopresets.c
	base/mswebcam.c
	base/mtu.c
	base/msasync.c
	otherfilters/itc.c
	otherfilters/join.c
	otherfilters/tee.c
	otherfilters/void.c
)
if(ANDROID)
	list(APPEND BASE_SOURCE_FILES_C utils/msjava.c)
endif()

bc_git_version(ms2 ${PROJECT_VERSION})

add_compile_definitions(
	_TRUE_TIME
	MS2_EXPORTS
	MS2_INTERNAL
	MS2_FILTERS
)
if(DISABLE_SRTP_SEARCH)
	# Set -DSRTP_INTERNAL in order to include srtp.h instead of srtp2/srtp.h
	add_compile_definitions(SRTP_INTERNAL)
endif()

set(VOIP_SOURCE_FILES_C
	audiofilters/alaw.c
	audiofilters/audiomixer.c
	audiofilters/chanadapt.c
	audiofilters/devices.c
	audiofilters/dtmfgen.c
	audiofilters/equalizer.c
	audiofilters/flowcontrol.c
	audiofilters/g711.c
	audiofilters/g711.h
	audiofilters/genericplc.h
	audiofilters/genericplc.c
	audiofilters/msgenericplc.c
	audiofilters/l16.c
	audiofilters/msfileplayer.c
	audiofilters/msfilerec.c
	audiofilters/asyncrw.c
	audiofilters/asyncrw.h
	audiofilters/msg722.c
	audiofilters/msvaddtx.c
	audiofilters/msvolume.c
	audiofilters/tonedetector.c
	audiofilters/ulaw.c
	audiofilters/waveheader.h
	crypto/dtls_srtp.c
	crypto/ms_srtp.c
	crypto/zrtp.c
	otherfilters/msrtp.c
	utils/_kiss_fft_guts.h
	utils/audiodiff.c
	utils/box-plot.c
	utils/dsptools.c
	utils/g722.h
	utils/g722_decode.c
	utils/g722_encode.c
	utils/kiss_fft.c
	utils/kiss_fft.h
	utils/kiss_fftr.c
	utils/kiss_fftr.h
	utils/pcap_sender.c
	utils/pcap_sender.h
	utils/stream_regulator.c
	utils/srtp_prefix.h
	voip/audioconference.c
	voip/audiostream.c
	voip/bandwidthcontroller.c
	voip/bitratecontrol.c
	voip/bitratedriver.c
	voip/ice.c
	voip/mediastream.c
	voip/msiframerequestslimiter.c
	voip/msmediaplayer.c
	voip/msmediarecorder.c
	voip/msvoip.c
	voip/private.h
	voip/qosanalyzer.c
	voip/qosanalyzer.h
	voip/qualityindicator.c
	otherfilters/rfc4103_source.c
	otherfilters/rfc4103_sink.c
	otherfilters/msudp.c
	voip/rfc4103_textstream.c
	voip/ringstream.c
	voip/stun.c
	voip/offeranswer.c
	voip/private.h
)
set(VOIP_SOURCE_FILES_CXX
	utils/filter-wrapper/decoding-filter-wrapper.cpp
	utils/filter-wrapper/encoding-filter-wrapper.cpp
	voip/audiostreamvolumes.cpp
	voip/turn_tcp.cpp
)
set(VOIP_SOURCE_FILES_OBJC )
set(VOIP_SOURCE_FILES_ASM )

if (APPLE)
        list(APPEND VOIP_SOURCE_FILES_OBJC utils/apple_utils.h utils/apple_utils.m)
endif()

if(ENABLE_ALSA)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/alsa.c)
endif()
if(ANDROID)
	list(APPEND VOIP_SOURCE_FILES_CXX voip/msvoip_jni.cc)
endif()
if(ENABLE_ANDROIDSND)
	list(APPEND VOIP_SOURCE_FILES_CXX
		android/androidsound_utils.cpp
		android/androidsound.cpp
		android/androidsound_depr.cpp
		android/androidsound_opensles.cpp
		android/AudioRecord.cpp
		android/AudioSystem.cpp
		android/AudioTrack.cpp
		android/hardware_echo_canceller.cpp
		android/loader.cpp
		android/String8.cpp
		#audiofilters/aac-eld-android.cpp
		#Not working on Android, to be fix
	)
endif()
if(ENABLE_ARTSC)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/arts.c)
endif()
if(ENABLE_MACSND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/macsnd.c)
endif()
if(ENABLE_MACAQSND)
	list(APPEND VOIP_SOURCE_FILES_OBJC audiofilters/aqsnd.m)
endif()
if(ENABLE_OSS)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/oss.c)
endif()
if(ENABLE_PORTAUDIO)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/pasnd.c)
endif()
if(ENABLE_PULSEAUDIO)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/pulseaudio.c)
endif()
if(ENABLE_QSA)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/qsa.c)
endif()
if(APPLE)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/aac-eld.c)
endif()
if(IOS)
	list(APPEND VOIP_SOURCE_FILES_OBJC audiofilters/msiounit.m)
endif()

if(ENABLE_G726)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/g726.c)
endif()
if(BCG729_FOUND AND ENABLE_G729)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/g729.c)
endif()
if(GSM_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/gsm.c)
endif()
if(BV16_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/bv16.c)
endif()
if(OPUS_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/msopus.c)
endif()
if(ENABLE_RESAMPLE)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/msresample.c)
endif()
if(SPEEX_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/msspeex.c)
endif()
if(SPEEXDSP_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C audiofilters/speexec.c)
endif()

if(ENABLE_VIDEO)
	list(APPEND VOIP_SOURCE_FILES_C
		utils/bits_rw.c
		videofilters/extdisplay.c
		videofilters/msanalysedisplay.c
		videofilters/mire.c
		videofilters/nowebcam.c
		videofilters/pixconv.c
		videofilters/sizeconv.c
		videofilters/videoswitcher.c
		voip/layouts.c
		voip/layouts.h
		voip/msvideo_neon.c
		voip/msvideo_neon.h
		voip/msvideo.c
		voip/msvideoqualitycontroller.c
		voip/nowebcam.h
		voip/rfc2429.h
		voip/video_preset_high_fps.c
		voip/videostarter.c
		voip/videostream.c
		voip/videoconference.c
		voip/vp8rtpfmt.c
		voip/vp8rtpfmt.h
	)
	if(FFMPEG_FOUND AND NOT TURBOJPEG_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			utils/jpgloader-ffmpeg.c
		)
	elseif(TURBOJPEG_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			utils/jpgloader-turbojpeg.c
		)
		list(APPEND VOIP_SOURCE_FILES_CXX
			videofilters/msturbojpeg.cpp	#MJPEG using turbojpeg
		)
	else()
		list(APPEND VOIP_SOURCE_FILES_C
			utils/jpgloader-stub.c
		)
	endif()
	if(ENABLE_QT_GL)
		set(QT_VOIP_SOURCE_FILES_CXX videofilters/msqogl.cpp )
	endif()
	if(WIN32)
		list(APPEND VOIP_SOURCE_FILES_CXX
			videofilters/msmfoundationcap.cpp	#Media-Foundation
			videofilters/msmfoundationcap.h
		)
	    if(NOT CMAKE_SYSTEM_NAME STREQUAL "WindowsPhone" AND NOT CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
			add_definitions(-DHAVE_DIRECTSHOW)
			if(NOT(ENABLE_MICROSOFT_STORE_APP))
				list(APPEND VOIP_SOURCE_FILES_C videofilters/drawdib-display.c)
			endif()
			list(APPEND VOIP_SOURCE_FILES_CXX
				videofilters/msdscap.cc				#DirectShow
			)
		elseif(CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
			set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /ZW /EHsc /await")
	    endif()
	endif()
	if(APPLE)
		if(IOS)
			list(APPEND VOIP_SOURCE_FILES_C
				utils/opengl_functions.c
				utils/opengl_functions.h
				utils/opengles_display.c
				utils/opengles_display.h
				utils/shader_util.c
				utils/shader_util.h
				voip/scaler.c
			)
			list(APPEND VOIP_SOURCE_FILES_OBJC
				videofilters/iosdisplay.mm
				videofilters/ioscapture.m
				voip/ioshardware.m
			)
		else()
			list(APPEND VOIP_SOURCE_FILES_OBJC
				videofilters/msosxdisplay.m
				videofilters/avfound-capture.m
			)

			list(APPEND VOIP_SOURCE_FILES_C
				utils/opengl_functions.c
				utils/opengl_functions.h
				utils/opengles_display.c
				utils/opengles_display.h
				utils/shader_util.c
				utils/shader_util.h
			)
		endif()
	endif()
	if(ANDROID)
		list(APPEND VOIP_SOURCE_FILES_C
			android/android-display.c
			android/android-opengl-display.c
			android/android-texture-display.c
			utils/opengl_functions.c
			utils/opengl_functions.h
			utils/opengles_display.c
			utils/opengles_display.h
			utils/shader_util.c
			utils/shader_util.h
			voip/scaler.c
		)
		if(CMAKE_SYSTEM_PROCESSOR STREQUAL "armv7-a")
			list(APPEND VOIP_SOURCE_FILES_ASM
				voip/scaler_arm.S
			)
		endif()
		list(APPEND VOIP_SOURCE_FILES_CXX
			android/android_mediacodec.cpp
			android/android-display-bad.cpp
			android/androidvideo.cpp
		)
	endif()
	if (ENABLE_QNX)
		list(APPEND VOIP_SOURCE_FILES_CXX
			videofilters/bb10_display.cpp
			videofilters/bb10_capture.cpp
		)
	endif()
	if(FFMPEG_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			utils/ffmpeg-priv.c
			utils/ffmpeg-priv.h
			videofilters/videodec.c
			videofilters/videoenc.c
		)
		list(APPEND VOIP_SOURCE_FILES_CXX
			videofilters/h264dec.cpp
		)
		if(NOT TURBOJPEG_FOUND)
			list(APPEND VOIP_SOURCE_FILES_C
				videofilters/ffmpegjpegwriter.c
			)
		endif()
	endif()
	if(TURBOJPEG_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			videofilters/turbojpegwriter.c
		)
		if(TURBOJPEG_USE_CONST_BUFFERS)
			set_source_files_properties(videofilters/turbojpegwriter.c PROPERTIES COMPILE_DEFINITIONS "TURBOJPEG_USE_CONST_BUFFERS")
		endif()
	endif()
	if(ZXING_FOUND)
		list(APPEND VOIP_SOURCE_FILES_CXX
			videofilters/zxing_qrcode.cpp
			utils/qrcode_image.cpp
		)
	endif()
	if(WIN32)
		list(APPEND VOIP_SOURCE_FILES_CXX
			utils/opengl_functions.c
			utils/opengles_display.c
			utils/shader_util.c
			videofilters/generic_opengl_display.c
		)
	elseif(OPENGL_FOUND OR QTANGLE_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			utils/opengl_functions.c
			utils/opengles_display.c
			utils/shader_util.c
			videofilters/generic_opengl_display.c
		)
	endif()
	if(GLX_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			utils/opengl_functions.c
			utils/opengles_display.c
			utils/shader_util.c
			videofilters/glxvideo.c
		)
	endif()
	if(SDL_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C videofilters/videoout.c)
	endif()
	if(THEORA_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C videofilters/theora.c)
	endif()
	if(V4L_FOUND)
		if(HAVE_LINUX_VIDEODEV_H)
			list(APPEND VOIP_SOURCE_FILES_C videofilters/msv4l.c)
		endif()
		if(HAVE_LINUX_VIDEODEV2_H)
			list(APPEND VOIP_SOURCE_FILES_C videofilters/msv4l2.c)
		endif()
	endif()
	if(VPX_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C
			videofilters/vp8.c
		)
	endif()
	if(X11_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C utils/x11_helper.c)
	endif()
	if(XV_FOUND)
		list(APPEND VOIP_SOURCE_FILES_C videofilters/x11video.c)
	endif()
endif()
if(BCMATROSKA2_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C
		voip/rfc2429.h
	)
	list(APPEND VOIP_SOURCE_FILES_CXX
		utils/mkv_reader.cpp
		videofilters/mkv.cpp
	)
endif()

if(PCAP_FOUND)
	list(APPEND VOIP_SOURCE_FILES_C otherfilters/mspcapfileplayer.c)
endif()

if(ENABLE_VIDEO)
	add_subdirectory(voip/h26x)
endif()

set(VOIP_SOURCE_FILES_ALL ${VOIP_SOURCE_FILES_C} ${VOIP_SOURCE_FILES_CXX} ${VOIP_SOURCE_FILES_OBJC} ${VOIP_SOURCE_FILES_ASM})
set(SOURCE_FILES_ALL ${BASE_SOURCE_FILES_C} ${VOIP_SOURCE_FILES_ALL})

add_custom_target(ms2-basedescs-header
	COMMAND ${CMAKE_COMMAND} -DAWK_PROGRAM=${AWK_PROGRAM} -DAWK_SCRIPTS_DIR="${CMAKE_CURRENT_SOURCE_DIR}/../"
		-DINPUT_DIR=${CMAKE_CURRENT_SOURCE_DIR} -DOUTPUT_DIR=${CMAKE_CURRENT_BINARY_DIR}
		-DTYPE=base -DSOURCE_FILES="${BASE_SOURCE_FILES_C}"
		-P "${CMAKE_CURRENT_SOURCE_DIR}/generate_descs_header.cmake"
	BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/basedescs.h")

add_custom_target(ms2-voipdescs-header
	COMMAND ${CMAKE_COMMAND} -DAWK_PROGRAM=${AWK_PROGRAM} -DAWK_SCRIPTS_DIR="${CMAKE_CURRENT_SOURCE_DIR}/../"
		-DINPUT_DIR=${CMAKE_CURRENT_SOURCE_DIR} -DOUTPUT_DIR=${CMAKE_CURRENT_BINARY_DIR}
		-DTYPE=voip -DSOURCE_FILES="${VOIP_SOURCE_FILES_ALL}"
		-P "${CMAKE_CURRENT_SOURCE_DIR}/generate_descs_header.cmake"
	BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/voipdescs.h"
)


if(APPLE OR ANDROID OR OPENGL_FOUND OR GLX_FOUND OR QTANGLE_FOUND OR WIN32)
	find_package(PythonInterp 3 REQUIRED)
	find_program(SED_PROGRAM sed CMAKE_FIND_ROOT_PATH_BOTH)
	if(NOT SED_PROGRAM)
		message(FATAL_ERROR "You need the sed program to build mediastreamer2.")
	endif()
	add_custom_target(ms2-yuv2rgb-fs-header
		COMMAND ${CMAKE_COMMAND} -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE} -DSED_PROGRAM=${SED_PROGRAM}
			-DINPUT_DIR=${CMAKE_CURRENT_SOURCE_DIR} -DOUTPUT_DIR=${CMAKE_CURRENT_BINARY_DIR}
			-DSOURCE_FILE="yuv2rgb.fs"
			-P "${CMAKE_CURRENT_SOURCE_DIR}/generate_yuv2rgb_header.cmake"
		BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/yuv2rgb.fs.h"
	)
	add_custom_target(ms2-yuv2rgb-vs-header
		COMMAND ${CMAKE_COMMAND} -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE} -DSED_PROGRAM=${SED_PROGRAM}
			-DINPUT_DIR=${CMAKE_CURRENT_SOURCE_DIR} -DOUTPUT_DIR=${CMAKE_CURRENT_BINARY_DIR}
			-DSOURCE_FILE="yuv2rgb.vs"
			-P "${CMAKE_CURRENT_SOURCE_DIR}/generate_yuv2rgb_header.cmake"
		BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/yuv2rgb.vs.h"
	)
endif()

bc_apply_compile_flags(BASE_SOURCE_FILES_C STRICT_OPTIONS_CPP STRICT_OPTIONS_C)
bc_apply_compile_flags(VOIP_SOURCE_FILES_C STRICT_OPTIONS_CPP STRICT_OPTIONS_C)
bc_apply_compile_flags(VOIP_SOURCE_FILES_OBJC STRICT_OPTIONS_CPP STRICT_OPTIONS_OBJC)
if(VOIP_SOURCE_FILES_CXX)
	set_source_files_properties(${VOIP_SOURCE_FILES_CXX} PROPERTIES LANGUAGE CXX)
	bc_apply_compile_flags(VOIP_SOURCE_FILES_CXX STRICT_OPTIONS_CPP STRICT_OPTIONS_CXX)
endif()
if(QT_VOIP_SOURCE_FILES_CXX)
	set_source_files_properties(${QT_VOIP_SOURCE_FILES_CXX} PROPERTIES LANGUAGE CXX)
	bc_apply_compile_flags(QT_VOIP_SOURCE_FILES_CXX STRICT_OPTIONS_CPP STRICT_OPTIONS_CXX)
endif()

if(ENABLE_VIDEO AND ANDROID AND CMAKE_SYSTEM_PROCESSOR STREQUAL "armv7-a")
	set(NEON_SOURCE_FILES
		voip/msvideo_neon.c
		voip/scaler.c
	)
endif()

if(VOIP_SOURCE_FILES_ASM)
	get_directory_property(DIRECTORY_PROPERTY_COMPILE_OPTIONS "COMPILE_OPTIONS")
	string(REPLACE ";" " " DIRECTORY_PROPERTY_COMPILE_OPTIONS "${DIRECTORY_PROPERTY_COMPILE_OPTIONS}")
	set(CMAKE_ASM_TARGET_FLAG "")
	if(CMAKE_C_COMPILER_TARGET)
		set(CMAKE_ASM_TARGET_FLAG "--target=${CMAKE_C_COMPILER_TARGET}")
	endif()
	set(CMAKE_ASM_COMPILE_OBJECT "<CMAKE_ASM_COMPILER> ${CMAKE_ASM_TARGET_FLAG} ${CMAKE_CPP_FLAGS} ${CMAKE_C_FLAGS} ${DIRECTORY_PROPERTY_COMPILE_OPTIONS} ${ARCH_OPTIONS} -o <OBJECT> -c <SOURCE>")
	enable_language(ASM)
endif()

set(MS2_DEPENDENCIES ms2-git-version ms2-basedescs-header ms2-voipdescs-header)
if(APPLE OR ANDROID OR OPENGL_FOUND OR GLX_FOUND OR QTANGLE_FOUND OR WIN32)
	list(APPEND MS2_DEPENDENCIES ms2-yuv2rgb-fs-header ms2-yuv2rgb-vs-header)
endif()


string(REPLACE ";" " " LINK_FLAGS_STR "${LINK_FLAGS}")
if(ENABLE_STATIC)
	add_library(mediastreamer-static STATIC ${MEDIASTREAMER2_HEADER_FILES} ${SOURCE_FILES_ALL})
	set_target_properties(mediastreamer-static PROPERTIES OUTPUT_NAME mediastreamer)
	if(LINK_FLAGS_STR)
		set_target_properties(mediastreamer-static PROPERTIES LINK_FLAGS "${LINK_FLAGS_STR}")
	endif()
	add_dependencies(mediastreamer-static ${MS2_DEPENDENCIES})
	target_include_directories(mediastreamer-static PRIVATE ${MEDIASTREAMER2_INCLUDE_DIRS})
	target_link_libraries(mediastreamer-static PRIVATE ${LINK_LIBS})

	install(TARGETS mediastreamer-static EXPORT ${EXPORT_TARGETS_NAME}Targets
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
endif()
if(ENABLE_SHARED)
	add_library(mediastreamer SHARED ${MEDIASTREAMER2_HEADER_FILES} ${SOURCE_FILES_ALL})
	set_target_properties(mediastreamer PROPERTIES LINKER_LANGUAGE CXX)
	if(LINK_FLAGS_STR)
		set_target_properties(mediastreamer PROPERTIES LINK_FLAGS "${LINK_FLAGS_STR}")
	endif()
	add_dependencies(mediastreamer ${MS2_DEPENDENCIES})
	if(APPLE)
		if(IOS)
			set(MIN_OS ${LINPHONE_IOS_DEPLOYMENT_TARGET})
		else()
			set(MIN_OS ${CMAKE_OSX_DEPLOYMENT_TARGET})
		endif()
		set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/build/osx/")
		# According to Apple documentation (https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPFrameworks/Tasks/IncludingFrameworks.html),
		# framework header files must be included in application code using the #include directive using the followin patern: #include <Framework_name/Header_filename.h>,
		# so name the framework mediastreamer2 instead of mediastreamer.
		set_target_properties(mediastreamer PROPERTIES OUTPUT_NAME mediastreamer2)
		set_target_properties(mediastreamer PROPERTIES
			FRAMEWORK TRUE
			FRAMEWORK_VERSION ${MS2_FRAMEWORK_VERSION}
			MACOSX_FRAMEWORK_IDENTIFIER org.linphone.mediastreamer2
  			MACOSX_FRAMEWORK_INFO_PLIST Info.plist.in
			PUBLIC_HEADER "${MEDIASTREAMER2_HEADER_FILES}"
		)
		if(ENABLE_QT_GL AND QT5_PACKAGES)
			foreach (package ${QT5_PACKAGES})
				if(Qt5${package}_FOUND)
					add_custom_command(TARGET mediastreamer POST_BUILD COMMAND install_name_tool -change "${Qt${package}_ROOT_LOCATION}Qt${package}.framework/Versions/5/Qt${package}" "@rpath/Qt${package}.framework/Versions/5/Qt${package}" $<TARGET_FILE:mediastreamer>)
				endif()
			endforeach()
		endif()
	endif()
	if(ANDROID)
		if(ENABLE_VPX)
			set_target_properties(mediastreamer PROPERTIES LINK_FLAGS "-Wl,--exclude-libs,libvpx.a")
		endif()
	else()
		# Do not version shared library on Android
		set_target_properties(mediastreamer PROPERTIES SOVERSION ${MEDIASTREAMER_SO_VERSION})
        endif()
		target_include_directories(mediastreamer INTERFACE
			$<INSTALL_INTERFACE:include/>
			$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include/>
			PRIVATE ${MEDIASTREAMER2_INCLUDE_DIRS}
		)
        target_link_libraries(mediastreamer PRIVATE ${LINK_LIBS})
	if(IOS)
		target_link_libraries(mediastreamer INTERFACE "-framework Foundation" "-framework AVFoundation")
	endif()
	if(MSVC)
		if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
			install(FILES $<TARGET_PDB_FILE:mediastreamer>
				DESTINATION ${CMAKE_INSTALL_BINDIR}
				PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
			)
		endif()
	endif()
	if(ENABLE_LIBYUV)
		if( CMAKE_COMPILER_IS_GNUCC AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5.3)
			message(STATUS "Mediastreamer2 : Build with position independent code")
			set_property(TARGET mediastreamer PROPERTY POSITION_INDEPENDENT_CODE ON)
		endif()
	endif()
	install(TARGETS mediastreamer EXPORT ${EXPORT_TARGETS_NAME}Targets
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		FRAMEWORK DESTINATION Frameworks
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
    
endif()

#Qt Plugin
if(ENABLE_QT_GL)
	if(ENABLE_STATIC)
		add_library(msqogl-static STATIC ${QT_VOIP_SOURCE_FILES_CXX})
		set_target_properties(msqogl-static PROPERTIES OUTPUT_NAME msqogl)
		set_target_properties(msqogl-static PROPERTIES LINKER_LANGUAGE CXX)
		install(TARGETS msqogl-static
			ARCHIVE DESTINATION "${MS2_PLUGINS_LOCATION}"
			PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
		)
	endif()
	if(ENABLE_SHARED)
		if(NOT IOS)
			add_library(msqogl MODULE ${QT_VOIP_SOURCE_FILES_CXX})
		else()
			add_library(msqogl SHARED ${QT_VOIP_SOURCE_FILES_CXX})
		endif()
		add_dependencies(msqogl mediastreamer)
		target_link_libraries(msqogl PRIVATE mediastreamer bctoolbox ortp)
		set_target_properties(msqogl PROPERTIES LINKER_LANGUAGE CXX)
		
		target_include_directories(msqogl INTERFACE
			$<INSTALL_INTERFACE:include/>
			$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include/>
			PRIVATE ${QT_MEDIASTREAMER2_INCLUDE_DIRS} ${MEDIASTREAMER2_INCLUDE_DIRS}
		)
		target_link_libraries(msqogl PRIVATE ${QT_LINK_LIBS})
		
		if(APPLE)
			if(IOS)
				set(MIN_OS ${LINPHONE_IOS_DEPLOYMENT_TARGET})
				set_target_properties(msqogl 	PROPERTIES
								FRAMEWORK TRUE
								MACOSX_FRAMEWORK_IDENTIFIER org.linphone.msqogl
								MACOSX_FRAMEWORK_INFO_PLIST msqogl.Info.plist.in
				)
	
			else()
				set(MIN_OS ${CMAKE_OSX_DEPLOYMENT_TARGET})
			endif()
	
			set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/build/osx/")
			foreach (package ${QT5_PACKAGES})
				if(Qt5${package}_FOUND)
					add_custom_command(TARGET msqogl POST_BUILD COMMAND install_name_tool -change "${Qt${package}_ROOT_LOCATION}Qt${package}.framework/Versions/5/Qt${package}" "@rpath/Qt${package}.framework/Versions/5/Qt${package}" $<TARGET_FILE:msqogl>)
				endif()
			endforeach()
		endif()
		if(MSVC)
			if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
				install(FILES $<TARGET_PDB_FILE:msqogl>
					DESTINATION ${CMAKE_INSTALL_BINDIR}
					PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
				)
			endif()
			if(NOT IOS)
				set_target_properties(msqogl PROPERTIES PREFIX "lib")
			endif()
		endif()
		install(TARGETS msqogl
				RUNTIME DESTINATION ${MS2_PLUGINS_LOCATION}
				LIBRARY DESTINATION ${MS2_PLUGINS_LOCATION}
				ARCHIVE DESTINATION ${MS2_PLUGINS_LOCATION}
				FRAMEWORK DESTINATION Frameworks
				PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
		)
	endif()
	
endif()


if(FFMPEG_FOUND AND NOT WIN32)
	set(NO_DEPRECATED_DECLARATIONS_SOURCE_FILES
		videofilters/videoenc.c
		videofilters/videodec.c
		videofilters/h264dec.cpp
	)
	if(NOT TURBOJPEG_FOUND)
		set(NO_DEPRECATED_DECLARATIONS_SOURCE_FILES
			${NO_DEPRECATED_DECLARATIONS_SOURCE_FILES}
			videofilters/ffmpegjpegwriter.c
			videofilters/nowebcam.c
		)
	endif()
	foreach(file ${NO_DEPRECATED_DECLARATIONS_SOURCE_FILES})
		get_source_file_property(NO_DEPRECATED_DECLARATIONS_COMPILE_FLAGS ${file} COMPILE_FLAGS)
		set_source_files_properties(${file} PROPERTIES COMPILE_FLAGS "${NO_DEPRECATED_DECLARATIONS_COMPILE_FLAGS} -Wno-error=deprecated-declarations")
	endforeach()
endif()

if(ENABLE_VIDEO)
  	install(FILES voip/nowebcamCIF.jpg
		  DESTINATION ${CMAKE_INSTALL_DATADIR}/images
		  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
	)
endif()
