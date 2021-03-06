cmake_minimum_required(VERSION 3.0)
project(gravity VERSION 1.0 LANGUAGES C)

set(CMAKE_C_STANDARD 99)

SET(COMPILER_DIR src/compiler/)
SET(RUNTIME_DIR src/runtime/)
SET(SHARED_DIR src/shared/)
SET(UTILS_DIR src/utils/)
SET(OPT_DIR src/optionals/)

SET(GRAVITY_SRC src/cli/gravity.c)

AUX_SOURCE_DIRECTORY(${COMPILER_DIR} COMPILER_FILES)
AUX_SOURCE_DIRECTORY(${RUNTIME_DIR} RUNTIME_FILES)
AUX_SOURCE_DIRECTORY(${SHARED_DIR} SHARED_FILES)
AUX_SOURCE_DIRECTORY(${UTILS_DIR} UTILS_FILES)
AUX_SOURCE_DIRECTORY(${OPT_DIR} OPT_FILES)

set(GRAVITY_INCLUDE_DIR ${COMPILER_DIR} ${RUNTIME_DIR} ${SHARED_DIR} ${UTILS_DIR} ${OPT_DIR})

SET(SRC_FILES ${COMPILER_FILES} ${RUNTIME_FILES} ${SHARED_FILES} ${UTILS_FILES} ${OPT_FILES})

set(GRAVITY_DEPENDENT_LIBS "")
set(GRAVITY_PRIVATE_DEFINITIONS "")
set(GRAVITY_PRIVATE_COMPILE_OPTIONS "")

set(GRAVITY_INSTALL_RUNTIME_PATH "/usr/local/bin")  # Gravity executable install path
set(GRAVITY_INSTALL_LIB_PATH "lib")                 # Gravity shared library install path
set(GRAVITY_INSTALL_LIB_STATIC_PATH "lib")          # Gravity static library install path

# ----------------------------------------------------------------
if(MSVC)

    # for path functions
    list(APPEND GRAVITY_DEPENDENT_LIBS "shlwapi")

    # supress _CRT_SECURE_NO_WARNINGS for MSVC builds
    list(APPEND GRAVITY_PRIVATE_DEFINITIONS "_CRT_SECURE_NO_WARNINGS")

    # MSVC does not like static inlining
    list(APPEND GRAVITY_PRIVATE_DEFINITIONS "inline=")

    # warning C4068: unknown pragma
    list(APPEND GRAVITY_PRIVATE_COMPILE_OPTIONS "/wd4068")

    # make Windows installs local
    set(GRAVITY_INSTALL_RUNTIME_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin")
    set(GRAVITY_INSTALL_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin/lib")
    set(GRAVITY_INSTALL_LIB_STATIC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin/lib")

elseif(MINGW)

    # for path functions
    list(APPEND GRAVITY_DEPENDENT_LIBS "shlwapi")

    # make Windows installs local
    set(GRAVITY_INSTALL_RUNTIME_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin")
    set(GRAVITY_INSTALL_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin/lib")
    set(GRAVITY_INSTALL_LIB_STATIC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/bin/lib")

elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin|NetBSD|BSD|DragonFly|Linux")

    # for math functions
    list(APPEND GRAVITY_DEPENDENT_LIBS "m")

    if (${CMAKE_SYSTEM_NAME} MATCHES "NetBSD|BSD|DragonFly")
        list(APPEND GRAVITY_PRIVATE_DEFINITIONS "_WITH_GETLINE")
    endif()

endif()

# ----------------------------------------------------------------
add_library(gravityapi SHARED ${SRC_FILES})
add_library(gravityapi_s STATIC ${SRC_FILES})

# ----------------------------------------------------------------
set(GRAVITY_TARGETS gravityapi gravityapi_s)

foreach(target ${GRAVITY_TARGETS})

    target_link_libraries(${target} PRIVATE ${GRAVITY_DEPENDENT_LIBS})
    target_compile_definitions(${target} PRIVATE ${GRAVITY_PRIVATE_DEFINITIONS})
    target_compile_options(${target} PRIVATE ${GRAVITY_PRIVATE_COMPILE_OPTIONS})
    target_include_directories(${target} PUBLIC ${GRAVITY_INCLUDE_DIR})

endforeach()

# ----------------------------------------------------------------

add_executable(gravity ${GRAVITY_SRC})
target_link_libraries(gravity gravityapi_s)
target_include_directories(gravity PUBLIC ${GRAVITY_INCLUDE_DIR})

# ----------------------------------------------------------------

install(TARGETS gravity ${GRAVITY_TARGETS}
        RUNTIME DESTINATION ${GRAVITY_INSTALL_RUNTIME_PATH}
        LIBRARY DESTINATION ${GRAVITY_INSTALL_LIB_PATH}
        ARCHIVE DESTINATION ${GRAVITY_INSTALL_LIB_STATIC_PATH})
