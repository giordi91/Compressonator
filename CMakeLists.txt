cmake_minimum_required(VERSION 3.13.0)

project(Compressonator)   

    #looking for  files
    file(GLOB_RECURSE SOURCE_FILES "Compressonator/Source/Compress.cpp" 
	"Compressonator/Source/Compressonator.cpp" 
	"Compressonator/Source/Codec/*.cpp" 
	"Compressonator/Source/Codec/*.c" 
	"Compressonator/Source/Codec/*.cxx"  
	"Compressonator/Source/common/HDR_Encode.cpp"
	 )
	
	file(GLOB_RECURSE CMP_HEADERS1 "Compressonator/Header/*.h")
	file(GLOB_RECURSE CMP_HEADERS2 "Compressonator/Source/*.h")

	if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    list(APPEND CMP_HEADERS1 ${CMP_HEADERS1} "${CMAKE_CURRENT_SOURCE_DIR}/Common/Lib/Ext/OpenEXR/ilmbase-2.2.0/Half/half.h")
    list(APPEND SOURCE_FILES ${SOURCE_FILES} "${CMAKE_CURRENT_SOURCE_DIR}/Common/Lib/Ext/OpenEXR/ilmbase-2.2.0/Half/half.cpp")
	endif()
	
	set (CMP_INCLUDE_DIRS "")
	foreach (_headerFile ${CMP_HEADERS1})
    get_filename_component(_dir ${_headerFile} PATH)
    list (APPEND CMP_INCLUDE_DIRS ${_dir})
	endforeach()

	foreach (_headerFile ${CMP_HEADERS2})
		get_filename_component(_dir ${_headerFile} PATH)
		list (APPEND CMP_INCLUDE_DIRS ${_dir})
	endforeach()

	#list(APPEND SOURCE_FILES ${CMP_INCLUDE_DIRS})
	
    list(REMOVE_DUPLICATES CMP_INCLUDE_DIRS)

	include_directories(${CMP_INCLUDE_DIRS})
	
	if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	add_definitions(-DDISABLE_TESTCODE)
	endif()

	if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	endif()
    list(APPEND CMP_HEADERS1 ${CMP_HEADERS1} "${CMAKE_CURRENT_SOURCE_DIR}/Common/Lib/Ext/OpenEXR/ilmbase-2.2.0/Half/*.h")
    list(APPEND CMP_SOURCES1 ${CMP_SOURCES1} "${CMAKE_CURRENT_SOURCE_DIR}/Common/Lib/Ext/OpenEXR/ilmbase-2.2.0/Half/*.cpp")

	##lets add specific windows build definitions
	#if (WIN32)
	add_compile_definitions(  WIN64  _LIB HALF_NO_STD
)
	#endif (WIN32)

	#making sure to add the common cpp flags, that are defined in the main cpp file
    set(COMPRESS_CXX_FLAGS "/std:c++14 /W3 /arch:AVX2  /MP")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMPRESS_CXX_FLAGS}")


	MACRO(SOURCE_GROUP_BY_FOLDER target)
	  SET(SOURCE_GROUP_DELIMITER "/")
	  SET(last_dir "")
	  SET(files "")
	  FOREACH(file ${SOURCE_FILES})
		file(RELATIVE_PATH relative_file "${PROJECT_SOURCE_DIR}/{target}" ${file})
		GET_FILENAME_COMPONENT(dir "${relative_file}" PATH)
		IF (NOT "${dir}" STREQUAL "${last_dir}")
		  IF (files)
			SOURCE_GROUP("${last_dir}" FILES ${files})
		  ENDIF (files)
		  SET(files "")
		ENDIF (NOT "${dir}" STREQUAL "${last_dir}")
		SET(files ${files} ${file})
		SET(last_dir "${dir}")
	  ENDFOREACH(file)
	  IF (files)
		SOURCE_GROUP("${last_dir}" FILES ${files})
	  ENDIF (files)
	ENDMACRO(SOURCE_GROUP_BY_FOLDER)

	SOURCE_GROUP_BY_FOLDER(SirEngineThe3rdLib)

    #adding the executable
    add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES} ${INCLUDES_FILES})

	target_include_directories(${PROJECT_NAME} PUBLIC ${HEADER_FILES})

	if(APPLE)
	elseif(UNIX)
	elseif(WIN32)
		#copying built lib to the output
		add_custom_command(
		TARGET ${PROJECT_NAME} 
		POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy
		  $<TARGET_FILE:${PROJECT_NAME}>
		  ${CMAKE_SOURCE_DIR}/../builtDependencies/windows/$<CONFIGURATION>/$<TARGET_FILE_NAME:${PROJECT_NAME}>)		
	endif()

	#copying the headers
	file(COPY "Compressonator/Header/CMP_Math" DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )
	file(COPY "Compressonator/Header/CMP_Mesh" DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )
	file(COPY "Compressonator/Header/Codec" DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )
	file(COPY "Compressonator/Header/GPU_Decode" DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )
	file(COPY "Compressonator/Header/Internal" DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )
	file(GLOB HEADERS_TO_COPY "Compressonator/Header/*.h")
	file(COPY ${HEADERS_TO_COPY} DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/Compressonator )

