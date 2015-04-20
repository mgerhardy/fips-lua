if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 2.8)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-lua)
endif()

get_filename_component(MODULE_ABS_DIR "./lua/cmake" ABSOLUTE)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${MODULE_ABS_DIR}")

## CONFIGURATION
# Default configuration (we assume POSIX by default)
set(LUA_PATH "LUA_PATH" CACHE STRING "Environment variable to use as package.path.")
set(LUA_CPATH "LUA_CPATH" CACHE STRING "Environment variable to use as package.cpath.")
set(LUA_INIT "LUA_INIT" CACHE STRING "Environment variable for initial script.")

option(LUA_ANSI "Use only ansi features." OFF)
option(LUA_USE_RELATIVE_LOADLIB "Use modified loadlib.c with support for relative paths on posix systems." ON)
option(LUA_COMPAT_ALL "Enable backwards compatibility options." ON)
set(LUA_IDSIZE 60 CACHE NUMBER "gives the maximum size for the description of the source.")

#2DO: LUAI_* and LUAL_* settings, for now defaults are used.
set(LUA_DIRSEP "/")
set(LUA_MODULE_SUFFIX ${CMAKE_SHARED_MODULE_SUFFIX})
set(LUA_LDIR ${INSTALL_LMOD})
set(LUA_CDIR ${INSTALL_CMOD})

if (LUA_USE_RELATIVE_LOADLIB)
    # This will set up relative paths to lib
    string(REGEX REPLACE "[^!/]+" ".." LUA_DIR "!/${INSTALL_BIN}/")
else()
    # Direct path to installation
    set(LUA_DIR ${CMAKE_INSTALL_PREFIX} CACHE STRING "Destination from which modules will be resolved. See INSTALL_LMOD and INSTALL_CMOD.")
endif()

set(LUA_PATH_DEFAULT "./?.lua;${LUA_DIR}${LUA_LDIR}/?.lua;${LUA_DIR}${LUA_LDIR}/?/init.lua")
set(LUA_CPATH_DEFAULT "./?${LUA_MODULE_SUFFIX};${LUA_DIR}${LUA_CDIR}/?${LUA_MODULE_SUFFIX};${LUA_DIR}${LUA_CDIR}/loadall${LUA_MODULE_SUFFIX}")

if (FIPS_WINDOWS AND NOT CYGWIN)
    # Windows systems
    option(LUA_WIN "Windows specific build." ON)
    option(LUA_BUILD_WLUA "Build wLua interpretter without console output." ON)
    option(LUA_BUILD_AS_DLL "Build Lua library as Dll." ${BUILD_SHARED_LIBS})

    # Paths (Double escapes ne  option needed)
    set(LUA_DIRSEP "\\\\")
    string(REPLACE " /" ${LUA_DIRSEP} LUA_DIR "${LUA_DIR}")
    string(REPLACE "/" ${LUA_DIRSEP} LUA_LDIR "${LUA_LDIR}")
    string(REPLACE "/" ${LUA_DIRSEP} LUA_CDIR "${LUA_CDIR}")
    string(REPLACE "/" ${LUA_DIRSEP} LUA_PATH_DEFAULT "${LUA_PATH_DEFAULT}")
    string(REPLACE "/" ${LUA_DIRSEP} LUA_CPATH_DEFAULT "${LUA_CPATH_DEFAULT}")
else()
    # Posix systems (incl. Cygwin)
    option(LUA_USE_POSIX "Use POSIX features." ON)
    option(LUA_USE_DLOPEN "Use dynamic linker to load modules." ON)
    option(LUA_USE_MKSTEMP "Use mkstep." ON)
    option(LUA_USE_ISATTY "Use tty." ON)
    option(LUA_USE_POPEN "Use popen." ON)
    option(LUA_USE_ULONGJMP "Use ulongjmp" ON)
    option(LUA_USE_GMTIME_R "Use GTIME_R" ON)
    # Apple and Linux specific
    if (FIPS_LINUX OR FIPS_MACOSX)
        option(LUA_USE_STRTODHEX "Assume 'strtod' handles hexa formats" ON)
        option(LUA_USE_AFORMAT "Assume 'printf' handles 'aA' specifiers" ON)
        option(LUA_USE_LONGLONG "Assume support for long long" ON)
    endif()
endif()

## SETUP
# Optional libraries
find_package(Readline)
if (READLINE_FOUND)
    option(LUA_USE_READLINE "Use readline in the Lua CLI." ON)
endif()

find_package(Curses)
if (CURSES_FOUND)
    option(LUA_USE_CURSES "Use curses in the Lua CLI." ON)
endif()

# Setup needed variables and libraries
if (LUA_USE_POSIX)
    # On POSIX Lua links to standard math library "m"
    list(APPEND LIBS m)
endif()

if (LUA_USE_DLOPEN)
    # Link to dynamic linker library "dl"
    find_library(DL_LIBRARY NAMES dl)
    if (DL_LIBRARY)
        list(APPEND LIBS ${DL_LIBRARY})
    endif()
endif()

if (LUA_USE_READLINE)
    # Add readline
    include_directories(${READLINE_INCLUDE_DIR})
    list(APPEND LIBS ${READLINE_LIBRARY})
endif()

if (LUA_USE_CURSES)
    # Add curses
    include_directories(${CURSES_INCLUDE_DIR})
    list(APPEND LIBS ${CURSES_LIBRARY})
endif()

## SOURCES
# Generate luaconf.h
configure_file(lua/src/luaconf.h.in ${CMAKE_CURRENT_SOURCE_DIR}/luaconf.h)

fips_begin_lib(lua52)
    fips_include_directories(lua/src ${CMAKE_CURRENT_SOURCE_DIR})
    if (NOT FIPS_EXCEPTIONS)
        add_definitions(-DLUA_USE_LONGJMP)
    endif()
    fips_dir(lua/src)
    fips_files(
        lapi.c lapi.h
        lauxlib.c lauxlib.h
        lbaselib.c
        lbitlib.c
        lcode.c lcode.h
        lcorolib.c
        lctype.c lctype.h
        ldblib.c
        ldebug.c ldebug.h
        ldo.c ldo.h ldump.c
        lfunc.c lfunc.h
        lgc.c lgc.h
        linit.c
        liolib.c
        llex.c llex.h
        llimits.h
        lmathlib.c
        lmem.c lmem.h
        loadlib.c
        lobject.c lobject.h
        lopcodes.c lopcodes.h
        loslib.c
        lparser.c lparser.h
        lstate.c lstate.h
        lstring.c lstring.h
        lstrlib.c
        ltable.c ltable.h
        ltablib.c
        ltm.c ltm.h
        luaconf.h
        lualib.h lundump.c
        lundump.h
        lvm.c lvm.h
        lzio.c lzio.h
    )

    if (FIPS_LINUX)
        fips_libs(${DL_LIBRARY})
        add_definitions("-DLUA_USE_LINUX")
    elseif(FIPS_MACOS)
        add_definitions("-DLUA_USE_MACOSX")
    elseif(FIPS_WINDOWS)
    endif()
fips_end_lib()

if (FIPS_MACOS OR FIPS_LINUX OR FIPS_WINDOWS)
    fips_begin_app(luac cmdline)
        fips_dir(lua/src)
        fips_files(luac.c luac.rc)
        fips_libs(${LIBS})
        fips_deps(lua52)
    fips_end_app()

    fips_begin_app(lua cmdline)
        fips_dir(lua/src)
        fips_files(lua.c lua.rc)
        fips_libs(${LIBS})
        fips_deps(lua52)
    fips_end_app()
endif()

if (NOT FIPS_IMPORT)
    fips_finish()
endif()
