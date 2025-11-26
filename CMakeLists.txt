cmake_minimum_required(VERSION 3.20)

project(lua VERSION 5.1 LANGUAGES C)

option(LUA_BUILD_INTERPRETER "Build Lua standalone interpreter" OFF)
option(LUA_BUILD_COMPILER    "Build Lua compiler (luac)"        OFF)
option(LUA_BUILD_SHARED      "Build shared library"             OFF)

set(LUA_CORE_SOURCES
    src/lapi.c
    src/lauxlib.c
    src/lbaselib.c
    src/lcode.c
    src/ldblib.c
    src/ldebug.c
    src/ldo.c
    src/ldump.c
    src/lfunc.c
    src/lgc.c
    src/linit.c
    src/liolib.c
    src/llex.c
    src/lmathlib.c
    src/lmem.c
    src/loadlib.c
    src/lobject.c
    src/lopcodes.c
    src/loslib.c
    src/lparser.c
    src/lstate.c
    src/lstring.c
    src/lstrlib.c
    src/ltable.c
    src/ltablib.c
    src/ltm.c
    src/lundump.c
    src/lvm.c
    src/lzio.c
)

if(LUA_BUILD_SHARED)
    add_library(lua SHARED ${LUA_CORE_SOURCES})
else()
    add_library(lua STATIC ${LUA_CORE_SOURCES})
endif()

# C стандарт
target_compile_features(lua PUBLIC c_std_99)

target_include_directories(lua PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include>
)

if(UNIX)
    target_compile_definitions(lua PRIVATE
        LUA_USE_POSIX
        LUA_USE_DLOPEN
    )
    
    if(NOT APPLE)
        target_link_libraries(lua PUBLIC m dl)
    else()
        target_link_libraries(lua PUBLIC m)
    endif()
elseif(WIN32)
    target_compile_definitions(lua PRIVATE
        _CRT_SECURE_NO_WARNINGS
    )
    if(LUA_BUILD_SHARED)
        target_compile_definitions(lua PRIVATE LUA_BUILD_AS_DLL)
    endif()
endif()

add_library(lua::lua ALIAS lua)

if(LUA_BUILD_INTERPRETER)
    add_executable(lua_interpreter src/lua.c)
    target_link_libraries(lua_interpreter PRIVATE lua)
    set_target_properties(lua_interpreter PROPERTIES
        OUTPUT_NAME lua
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
    )
    
    # Для readline поддержки (опционально)
    if(UNIX AND NOT APPLE)
        find_library(READLINE_LIBRARY readline)
        if(READLINE_LIBRARY)
            target_link_libraries(lua_interpreter PRIVATE ${READLINE_LIBRARY})
            target_compile_definitions(lua_interpreter PRIVATE LUA_USE_READLINE)
        endif()
    endif()
endif()

if(LUA_BUILD_COMPILER)
    add_executable(lua_compiler src/luac.c src/print.c)
    target_link_libraries(lua_compiler PRIVATE lua)
    set_target_properties(lua_compiler PROPERTIES
        OUTPUT_NAME luac
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
    )
endif()

message(STATUS "Lua Configuration:")
message(STATUS "  Library type:      ${LUA_BUILD_SHARED}")
message(STATUS "  Build interpreter: ${LUA_BUILD_INTERPRETER}")
message(STATUS "  Build compiler:    ${LUA_BUILD_COMPILER}")