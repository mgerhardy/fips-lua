fips_begin_lib(lua)
    include_directories(lua/src .)
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
        luac.c luaconf.h
        lua.c lua.h lua.hpp
        lualib.h lundump.c
        lundump.h
        lvm.c lvm.h
        lzio.c lzio.h
    )
    if (FIPS_LINUX)
        add_definitions("-DLUA_USE_LINUX")
    elseif(FIPS_MACOS)
        add_definitions("-DLUA_USE_MACOSX")
    elseif(FIPS_WINDOWS)
    endif()
fips_end_lib()
