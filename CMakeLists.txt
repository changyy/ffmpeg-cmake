cmake_minimum_required(VERSION 2.8)
project (ffmpeg)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)


set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

set(TOOL_TARGET_NAME "build_ffmpeg")
set(TOOL_TARGET_BUILD_OUTPUT "BUILD_FFMPEG")
set(TOOL_TARGET_COMMENT "build service ffmpeg")
set(TOOL_TARGET_SOURCE ffmpeg-1.2.1.tar.bz2)
set(TOOL_TARGET_DIR ffmpeg-1.2.1)
#set(CONFIG_ADD_OPTIONS "--disable-yasm;--enable-cross-compile;--target-os=linux;--arch=arm6;--cross-prefix=${target_platform}-")
set(CONFIG_ADD_OPTIONS "")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://www.ffmpeg.org/releases/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND cd ${TOOL_TARGET_DIR} && ${target_platform_preconfigure_options} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-yasm --enable-cross-compile --target-os=linux --arch=arm6 --cross-prefix=${target_platform}- ${CONFIG_ADD_OPTIONS}
			COMMAND cd ${TOOL_TARGET_DIR} && make -j4 && make install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://www.ffmpeg.org/releases/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND cd ${TOOL_TARGET_DIR} && ${target_platform_preconfigure_options} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-yasm ${CONFIG_ADD_OPTIONS}
			COMMAND cd ${TOOL_TARGET_DIR} && make -j4 && make install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(${TOOL_TARGET_NAME} ALL DEPENDS ${SELF_DEPENDS} ${TOOL_TARGET_BUILD_OUTPUT} )
