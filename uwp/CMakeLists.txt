cmake_minimum_required(VERSION 3.14)

# Setup UWP platform
set(CMAKE_SYSTEM_NAME WindowsStore)
set(CMAKE_SYSTEM_VERSION 10.0.19041.0)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type (Debug, Release, RelWithDebInfo, MinSizeRel)" FORCE)
endif()

# Define project
project(starship-uwp LANGUAGES CXX)
set_directory_properties(PROPERTIES VS_STARTUP_PROJECT ${PROJECT_NAME})

# Config for main project build 
set(STARSHIP_DIR "" CACHE PATH "")
if(NOT STARSHIP_DIR)
  message(FATAL_ERROR "STARSHIP_DIR must be set to a valid path pointing to the static lib build")
endif()

add_executable(${PROJECT_NAME} WIN32
	main.cpp
)

set(BinDeps
	${STARSHIP_DIR}/${CMAKE_BUILD_TYPE}/starship.dll

	#${STARSHIP_DIR}/sf64.o2r
	${STARSHIP_DIR}/../starship.o2r

	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/dxil.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/glfw3.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/libgallium_wgl.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/libuwp.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/opengl32.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/SDL2.dll
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/bin/z-1.dll

	Package.appxmanifest
)
source_group("BinDeps" FILES ${BinDeps})
set_source_files_properties(${BinDeps} PROPERTIES
	VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "."
)

set(PkgAssets
	Assets/LockScreenLogo.scale-200.png
	Assets/Square44x44Logo.targetsize-24_altform-unplated.png
	Assets/Square44x44Logo.scale-200.png
	Assets/Square150x150Logo.scale-200.png
	Assets/StoreLogo.png
)
source_group("PkgAssets" FILES ${PkgAssets})
set_source_files_properties(${PkgAssets} PROPERTIES
	#VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "Assets"
)

# Setup data deployment
file(GLOB_RECURSE DATA_FILES
	RELATIVE ${CMAKE_SOURCE_DIR}/..
	${CMAKE_SOURCE_DIR}/../assets/*
)

foreach(ASSET ${DATA_FILES})
	set(FULL_PATH "${CMAKE_SOURCE_DIR}/../${ASSET}")

	get_filename_component(DEST_DIR ${ASSET} DIRECTORY)

	set_source_files_properties("${FULL_PATH}" PROPERTIES
		VS_COPY_TO_OUT_DIR Always
		VS_DEPLOYMENT_CONTENT TRUE
		VS_DEPLOYMENT_LOCATION "${DEST_DIR}" # preserves folder structure
	)

	target_sources(${PROJECT_NAME} PRIVATE "${FULL_PATH}")
	source_group("BinData" FILES "${FULL_PATH}")
endforeach()

target_sources(${PROJECT_NAME} PRIVATE
	${BinDeps}
	${PkgAssets}
)

target_include_directories(${PROJECT_NAME} PRIVATE
 	${STARSHIP_DIR}/_deps/uwplibs-src/x64/include
)

set(BinLibs
	${STARSHIP_DIR}/${CMAKE_BUILD_TYPE}/starship.lib

	${STARSHIP_DIR}/_deps/uwplibs-src/x64/lib/libuwp.lib
	${STARSHIP_DIR}/_deps/uwplibs-src/x64/lib/SDL2.lib

	OneCore.lib
	WindowsApp.lib
)
target_link_libraries(${PROJECT_NAME} PRIVATE ${BinLibs})
set_target_properties(${PROJECT_NAME} PROPERTIES VS_WINDOWS_TARGET_PLATFORM_MIN_VERSION 10.0.19041.0)
