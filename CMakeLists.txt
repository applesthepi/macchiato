cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(USE_FOLDERS ON)
set(CMAKE_BUILD_PARALLEL_LEVEL 12)
enable_language(C CXX ASM)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

project(ATP_NN_BASE)

#########################

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
	set(LINUX True)
	message(STATUS "Compiling For Linux")

	find_package(X11 REQUIRED)
	find_package(TBB REQUIRED)

	list(APPEND EXTRA_LIBRARIES ${X11_LIBRARIES})
	list(APPEND EXTRA_INCLUDES ${X11_INCLUDE_DIRS})

    add_compile_options(-msse2)
else()
	set(LINUX False)
endif()

#########################

file(GLOB_RECURSE MACCHIATO_SRC "include/*/*.cpp" "include/*.cpp")
add_library(MACCHIATO STATIC ${MACCHIATO_SRC})

if(MSVC)
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/include/lungo FILES ${ATP_NN_SRC})
	target_compile_options(MACCHIATO PRIVATE "/MP")
endif()

add_dependencies(
	MACCHIATO
	glfw glm freetype VulkanMemoryAllocator
)

target_include_directories(
	MACCHIATO PUBLIC SYSTEM
	${CMAKE_SOURCE_DIR}/dependencies/glfw/include
	${CMAKE_SOURCE_DIR}/dependencies/glm
	${CMAKE_SOURCE_DIR}/dependencies/freetype/include
	${CMAKE_SOURCE_DIR}/dependencies/vma/include
	${Vulkan_INCLUDE_DIRS})
)

if(LINUX)
	set(DEPENDENCIES_LINK imgui glfw glm freetype Vulkan::Vulkan VulkanMemoryAllocator X11) # Link X11 for linux
else()
	set(DEPENDENCIES_LINK imgui glfw glm freetype Vulkan::Vulkan VulkanMemoryAllocator imm32) # Link imm32 for windows
endif()

target_link_libraries(
	ATP_NN PUBLIC
	${DEPENDENCIES_LINK}
)