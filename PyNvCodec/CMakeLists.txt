#
# Copyright 2019 NVIDIA Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

cmake_minimum_required(VERSION 3.16)

project(PyNvCodec)
add_subdirectory(TC)

#Add src & inc directories;
set (src_dir ${CMAKE_CURRENT_SOURCE_DIR}/src)
set (inc_dir ${CMAKE_CURRENT_SOURCE_DIR}/inc)
add_subdirectory(${src_dir})
add_subdirectory(${inc_dir})

#Do version stuff;
set (PYNVCODEC_VERSION_MAJOR 1)
set (PYNVCODEC_VERSION_MINOR 0)

configure_file(
	"${inc_dir}/Version.hpp.in"
	"${inc_dir}/Version.hpp"
)

#Add TC & TC_CORE includes;
include_directories(${TC_CORE_INC_PATH})
include_directories(${TC_INC_PATH})
include_directories(${VIDEO_CODEC_SDK_INCLUDE_DIR})

#Set up python bindings;
set(GENERATE_PYTHON_BINDINGS FALSE CACHE BOOL "Generate VPF Python bindings")
if(GENERATE_PYTHON_BINDINGS)
	find_package (Python COMPONENTS Interpreter Development)
	if(Python_FOUND)

		MESSAGE("${Python_SITEARCH}")
		SET( SITEDEST ${Python_SITEARCH}/vpf )
		#Adding pybind11 stuff;
		FIND_PACKAGE( pybind11 REQUIRED )

		#Adding path to Python & pybind11 headers;
		include_directories(${PYBIND11_INCLUDE_DIRS})
		include_directories(${PYTHON_INCLUDE_DIR})

		#Add target;
		pybind11_add_module(PyNvCodec SHARED ${PYNVCODEC_SOURCES} ${PYNVCODEC_HEADERS})

		#Link libs;
		target_link_libraries(PyNvCodec PUBLIC TC_CORE TC ${PYTHON_LIBRARIES} ${NVCUVID_LIBRARY} ${NVENCODE_LIBRARY})
	endif(Python_FOUND)
endif(GENERATE_PYTHON_BINDINGS)


#Promote variables to parent scope;
set (TC_CORE_INC_PATH         ${TC_CORE_INC_PATH}         PARENT_SCOPE)
set (TC_INC_PATH              ${TC_INC_PATH}              PARENT_SCOPE)

set (AVUTIL_INCLUDE_DIR       ${AVUTIL_INCLUDE_DIR}       PARENT_SCOPE)
set (AVCODEC_INCLUDE_DIR      ${AVCODEC_INCLUDE_DIR}      PARENT_SCOPE)
set (AVFORMAT_INCLUDE_DIR     ${AVFORMAT_INCLUDE_DIR}     PARENT_SCOPE)

set (FFMPEG_DIR               ${FFMPEG_DIR}               PARENT_SCOPE)
set (FFMPEG_INCLUDE_DIR       ${FFMPEG_INCLUDE_DIR}       PARENT_SCOPE)
set (FFMPEG_LIB_DIR           ${FFMPEG_LIB_DIR}           PARENT_SCOPE)
set (AVUTIL_LIBRARY           ${AVUTIL_LIBRARY}           PARENT_SCOPE)
set (AVCODEC_LIBRARY          ${AVCODEC_LIBRARY}          PARENT_SCOPE)
set (AVFORMAT_LIBRARY         ${AVFORMAT_LIBRARY}         PARENT_SCOPE)
set (SWRESAMPLE_LIBRARY       ${SWRESAMPLE_LIBRARY}       PARENT_SCOPE)

set (GENERATE_PYTHON_BINDINGS ${GENERATE_PYTHON_BINDINGS} PARENT_SCOPE)

if(GENERATE_PYTHON_BINDINGS)
	set( PERMS OWNER_EXECUTE OWNER_WRITE OWNER_READ )
	install(FILES $<TARGET_FILE:PyNvCodec> PERMISSIONS ${PERMS} RENAME vpf.so DESTINATION ${SITEDEST} )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/__init__.py			DESTINATION ${SITEDEST} )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleDecode.py			DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleEncode.py			DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleTranscode.py		DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleFrameUpload.py		DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleColorConversion.py	DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleSufraceDownload.py	DESTINATION share/vpf )
	install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/samples/SampleTranscodeOneToN.py	DESTINATION share/vpf )
endif(GENERATE_PYTHON_BINDINGS)
