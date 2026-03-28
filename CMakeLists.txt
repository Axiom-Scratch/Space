cmake_minimum_required(VERSION 3.20)

project(procedural_planet LANGUAGES C CXX)

# options
function(axiom_option NAME DEFAULT DESCRIPTION)
    option(${NAME} "${DESCRIPTION}" ${DEFAULT})
endfunction()

function(axiom_define_toggle TARGET NAME)
    if(${NAME})
        target_compile_definitions(${TARGET} PRIVATE ${NAME}=1)
    else()
        target_compile_definitions(${TARGET} PRIVATE ${NAME}=0)
    endif()
endfunction()

axiom_option(AXIOM_ENABLE_ASSERTS ON "Enable engine assertions")
axiom_option(AXIOM_ENABLE_LOGGING ON "Enable engine logging")

axiom_option(AXIOM_ENABLE_ASAN ON "AddressSanitizer")
axiom_option(AXIOM_ENABLE_UBSAN ON "UBSan")
axiom_option(AXIOM_ENABLE_TSAN OFF "TSan")

axiom_option(AXIOM_ENABLE_TRACY ON "Tracy profiler")
axiom_option(AXIOM_ENABLE_LTO OFF "Link-time optimization")
axiom_option(AXIOM_WARNINGS_AS_ERRORS ON "Treat warnings as errors")

# standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# compiler
if(MSVC)
    set(AXIOM_COMPILER MSVC)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(AXIOM_COMPILER GCC)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(AXIOM_COMPILER CLANG)
else()
    message(FATAL_ERROR "Unsupported compiler")
endif()

# warnings
include(cmake/compiler_warnings.cmake)

# deps
add_subdirectory(external/glfw)

add_subdirectory(external/glad/cmake)
glad_add_library(glad_core REPRODUCIBLE API gl:core=4.6)

set(ASSIMP_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(ASSIMP_BUILD_TOOLS OFF CACHE BOOL "" FORCE)
add_subdirectory(external/assimp)

add_subdirectory(external/spdlog)

if(AXIOM_ENABLE_TRACY)
    add_subdirectory(external/tracy)
endif()

# engine
add_subdirectory(Engine)

# config
function(axiom_configure_target TARGET)

    set_project_warnings(${TARGET})

    if(MSVC)
        target_compile_options(${TARGET} PRIVATE /Zi)
    else()
        target_compile_options(${TARGET} PRIVATE -g)
    endif()

    if(NOT MSVC)
        target_compile_options(${TARGET} PRIVATE -fno-omit-frame-pointer)
    endif()

    target_compile_options(${TARGET} PRIVATE
        $<$<CONFIG:Debug>:-O0>
        $<$<CONFIG:Release>:-O2>
        $<$<CONFIG:RelWithDebInfo>:-O2>
    )

    if(AXIOM_WARNINGS_AS_ERRORS)
        if(MSVC)
            target_compile_options(${TARGET} PRIVATE /WX)
        else()
            target_compile_options(${TARGET} PRIVATE -Werror)
        endif()
    endif()

    if(AXIOM_ENABLE_ASAN OR AXIOM_ENABLE_UBSAN)

        if(MSVC)
            message(FATAL_ERROR "Sanitizers not supported on MSVC")
        endif()

        set(SAN_FLAGS "")

        if(AXIOM_ENABLE_ASAN)
            list(APPEND SAN_FLAGS address)
        endif()

        if(AXIOM_ENABLE_UBSAN)
            list(APPEND SAN_FLAGS undefined)
        endif()

        list(JOIN SAN_FLAGS "," SAN_FLAGS_STR)

        target_compile_options(${TARGET} PRIVATE
            $<$<CONFIG:Debug>:-fsanitize=${SAN_FLAGS_STR}>
        )
        target_link_options(${TARGET} PRIVATE
            $<$<CONFIG:Debug>:-fsanitize=${SAN_FLAGS_STR}>
        )
    endif()

    if(AXIOM_ENABLE_LTO)
        if(MSVC)
            target_compile_options(${TARGET} PRIVATE /GL)
            target_link_options(${TARGET} PRIVATE /LTCG)
        else()
            target_compile_options(${TARGET} PRIVATE -flto)
            target_link_options(${TARGET} PRIVATE -flto)
        endif()
    endif()

    axiom_define_toggle(${TARGET} AXIOM_ENABLE_ASSERTS)
    axiom_define_toggle(${TARGET} AXIOM_ENABLE_LOGGING)

    if(AXIOM_ENABLE_TRACY)
        target_compile_definitions(${TARGET} PRIVATE TRACY_ENABLE)

        target_sources(${TARGET} PRIVATE
            ${CMAKE_SOURCE_DIR}/external/tracy/public/TracyClient.cpp
        )

        if(TARGET Tracy::TracyClient)
            target_link_libraries(${TARGET} PRIVATE Tracy::TracyClient)
        endif()
    endif()

endfunction()

axiom_configure_target(Engine)

# compile_commands
if(CMAKE_EXPORT_COMPILE_COMMANDS)
    file(REMOVE ${CMAKE_SOURCE_DIR}/compile_commands.json)
    execute_process(
        COMMAND ${CMAKE_COMMAND} -E create_symlink
                ${CMAKE_BINARY_DIR}/compile_commands.json
                ${CMAKE_SOURCE_DIR}/compile_commands.json
    )
endif()

# info
message(STATUS "========== Axiom Build ==========")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "System: ${CMAKE_SYSTEM_NAME}")
message(STATUS "C++: ${CMAKE_CXX_STANDARD}")
message(STATUS "ASAN: ${AXIOM_ENABLE_ASAN}")
message(STATUS "UBSAN: ${AXIOM_ENABLE_UBSAN}")
message(STATUS "TRACY: ${AXIOM_ENABLE_TRACY}")
message(STATUS "================================")
