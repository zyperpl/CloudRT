CMAKE_MINIMUM_REQUIRED(VERSION 3.19)
ENABLE_LANGUAGE(CUDA)

PROJECT("cloudrt")
SET(NAME "cloudrt")

FIND_PROGRAM(CCACHE ccache)
IF(CCACHE)
  SET(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE})
ELSE()
  message("ccache cannot be found")
  ENDIF()

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE "Debug")
ENDIF()

# submodules
SET(JSON_BuildTests OFF CACHE INTERNAL "")
ADD_SUBDIRECTORY(src/external/json)

INCLUDE_DIRECTORIES("src/" 
                    "src/external" 
                    "src/external/glm" 
                    "src/external/imgui" 
                    "src/external/imgui/backends"
                    "src/external/stb")

ADD_DEFINITIONS(-DNAME="${NAME}")

SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -ggdb -fno-omit-frame-pointer -Wall -Wextra -Wpedantic -DDEBUG -ggdb -std=c++20")
SET(CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_MINSIZEREL} -O3 -s -DNDEBUG")
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -DNDEBUG")
SET(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} --diag-suppress 20012,20014")
ADD_COMPILE_OPTIONS(-fPIC -Wall -Wextra)

FILE(GLOB SOURCES "src/*.cu" "src/*.cpp" "src/external/imgui/backends/imgui_impl_glfw.cpp" "src/external/imgui/backends/imgui_impl_opengl3.cpp" "src/external/imgui/*.cpp")
ADD_EXECUTABLE(${NAME} "${SOURCES}")
TARGET_LINK_LIBRARIES(${NAME} PRIVATE stdc++ pthread GLEW GL glfw nlohmann_json::nlohmann_json curl)

ADD_CUSTOM_TARGET(run
  COMMAND build/${NAME}
  DEPENDS ${NAME}
  WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
)
SET_PROPERTY(TARGET ${NAME} PROPERTY CUDA_STANDARD 20)
SET_PROPERTY(TARGET ${NAME} PROPERTY CUDA_ARCHITECTURES OFF)
SET_PROPERTY(TARGET ${NAME} PROPERTY POSITION_INDEPENDENT_CODE ON)
SET_PROPERTY(TARGET ${NAME} PROPERTY CUDA_SEPARABLE_COMPILATION ON)
