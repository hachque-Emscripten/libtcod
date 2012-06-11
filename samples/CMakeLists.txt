IF(MSVC)
	# msvc wants to "deprecate" standard C. Turn those warnings off.
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
ENDIF()

IF(MSVC)
	SET(UPX ${DEPENDENCY_DIR}/upx/win/upx.exe)
	SET(PROGRAM_ENDING .exe)
ELSEIF(MINGW)
	SET(UPX ${DEPENDENCY_DIR}/upx/win/upx.exe)
	SET(PROGRAM_ENDING .exe)
ELSEIF(APPLE)
	SET(PROGRAM_ENDING )
ELSEIF(UNIX)
	SET(UPX upx)
	SET(PROGRAM_ENDING )
ENDIF()

SET(LIBTCOD_SAMPLE_C_SRC samples_c.c)
SET(LIBTCOD_SAMPLE_CPP_SRC samples_cpp.cpp)
SET(LIBTCOD_HP_TOOL_SRC hmtool/operation.cpp hmtool/hmtool.cpp)

add_executable(samples_c ${LIBTCOD_SAMPLE_C_SRC})
if(DEFINED EMSCRIPTEN_ACTIVE)
	target_link_libraries(samples_c ${LIBTCOD_NAME} $ENV{EMSCRIPTEN_LIBPNG} $ENV{EMSCRIPTEN_ZLIB}) 
	set_target_properties(samples_c PROPERTIES SUFFIX ".html")
else(DEFINED EMSCRIPTEN_ACTIVE)
	target_link_libraries(samples_c ${LIBTCOD_NAME})
endif(DEFINED EMSCRIPTEN_ACTIVE)

add_executable(samples_cpp ${LIBTCOD_SAMPLE_CPP_SRC})
if(DEFINED EMSCRIPTEN_ACTIVE)
	target_link_libraries(samples_cpp ${LIBTCOD_NAME} ${LIBTCOD_CPP_NAME} $ENV{EMSCRIPTEN_LIBPNG} $ENV{EMSCRIPTEN_ZLIB}) 
	set_target_properties(samples_cpp PROPERTIES SUFFIX ".html")
else(DEFINED EMSCRIPTEN_ACTIVE)
	target_link_libraries(samples_cpp ${LIBTCOD_NAME} ${LIBTCOD_CPP_NAME})  #${LIBTCOD_CPP_NAME} is empty on non-linux platforms
endif(DEFINED EMSCRIPTEN_ACTIVE)

if(NOT DEFINED EMSCRIPTEN_ACTIVE)
	add_executable(hmtool.bin ${LIBTCOD_HP_TOOL_SRC})
	if(DEFINED EMSCRIPTEN_ACTIVE)
		target_link_libraries(hmtool.bin ${LIBTCOD_NAME} ${LIBTCOD_GUI_NAME} $ENV{EMSCRIPTEN_LIBPNG} $ENV{EMSCRIPTEN_ZLIB}) 
		set_target_properties(hmtool.bin PROPERTIES SUFFIX ".html")
	else(DEFINED EMSCRIPTEN_ACTIVE)
		target_link_libraries(hmtool.bin ${LIBTCOD_NAME} ${LIBTCOD_GUI_NAME}) 
	endif(DEFINED EMSCRIPTEN_ACTIVE)
endif(NOT DEFINED EMSCRIPTEN_ACTIVE)

IF(UNIX)
	SET(CMAKE_EXE_LINKER_FLAGS "-Wl,-rpath,. ${CMAKE_EXE_LINKER_FLAGS}")
ENDIF()

set_property(TARGET samples_c PROPERTY DEBUG_POSTFIX "_debug")
set_property(TARGET samples_cpp PROPERTY DEBUG_POSTFIX "_debug")
if(NOT DEFINED EMSCRIPTEN_ACTIVE)
	set_property(TARGET hmtool.bin PROPERTY DEBUG_POSTFIX "_debug")
endif(NOT DEFINED EMSCRIPTEN_ACTIVE)

IF(CMAKE_BUILD_TYPE MATCHES "Release")
	IF(NOT APPLE)
		ADD_CUSTOM_COMMAND(
	  	 TARGET samples_c
	  	 POST_BUILD
	   	 COMMAND ${UPX} samples_c${PROGRAM_ENDING}
		    VERBATIM
		)

		ADD_CUSTOM_COMMAND(
		    TARGET samples_cpp
		    POST_BUILD
		    COMMAND ${UPX} samples_cpp${PROGRAM_ENDING}
		    VERBATIM
		)
	
		ADD_CUSTOM_COMMAND(
   		    TARGET hmtool.bin
		    POST_BUILD
		    COMMAND ${UPX} hmtool.bin${PROGRAM_ENDING}
		    VERBATIM
		)
	ENDIF()
ENDIF()

if(NOT DEFINED EMSCRIPTEN_ACTIVE)
	INSTALL(TARGETS samples_c samples_cpp hmtool.bin
		RUNTIME DESTINATION ${INSTALL_DIR}
		LIBRARY DESTINATION ${INSTALL_DIR}
	)
endif(NOT DEFINED EMSCRIPTEN_ACTIVE)
