
cmake_minimum_required(VERSION 3.18)
project(HalfLifeSDK C CXX)

#-------------------------------------------------------------------------------
# Common options

set(CMAKE_C_STANDARD 90)
set(CMAKE_CXX_STANDARD 98)

# GCC exports all symbols by default, but Half-Life SDK exports them manually.
# Therefore, a symbol should be hidden by default, not exported.
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)

# On Linux, rpath should be avoided, otherwise client.so will be malformed:
# it will be linked to "some_path/vgui.so" instead of just "vgui.so".
# On Windows this doesn't matter.
set(CMAKE_SKIP_RPATH YES)

include_directories(common dlls engine game_shared pm_shared public)

# Most of this definitions are not used.
add_compile_definitions(CLIENT_WEAPONS HL1 NDEBUG NO_MALLOC_OVERRIDE QUAKE2 QUIVER VALVE_DLL)

#-------------------------------------------------------------------------------
# Platform-dependent options

if(WIN32)
    add_compile_definitions(WIN32 _WINDOWS)
elseif(APPLE)
    add_compile_definitions(OSX _OSX)
elseif(UNIX)
    add_compile_definitions(LINUX _LINUX POSIX _POSIX)
endif()

#-------------------------------------------------------------------------------
# Compiler-dependent options

if(NOT MSVC)
    # Force 32-bit build on 64-bit environments.
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        add_compile_options(-m32)
        add_link_options(-m32)
    endif()

    # Relax C++ rules a bit and disable a lot of annoying warnings.
    # Don't try this at home!
    add_compile_options(
        $<$<COMPILE_LANGUAGE:CXX>:-fpermissive>
        $<$<COMPILE_LANGUAGE:CXX>:-fno-strict-aliasing>
        $<$<COMPILE_LANGUAGE:CXX>:-Wno-invalid-offsetof>
        $<$<COMPILE_LANGUAGE:CXX>:-Wno-conversion-null>
        $<$<COMPILE_LANGUAGE:C>:-Wno-incompatible-pointer-types>
        -Wno-pointer-arith
        -Wno-write-strings
        -Wno-format-overflow
        -Wno-format-security
        -Wno-endif-labels)
    
    # Include debug information in DWARF format.
    add_compile_options(-g -gdwarf-2)

    # These function redefinitions should be fixed in the source code,
    # not here.
    add_compile_definitions(GNUC _alloca=alloca
        _snprintf=snprintf _vsnprintf=vsnprintf
        stricmp=strcasecmp strnicmp=strncasecmp)
endif()

#-------------------------------------------------------------------------------
# Server-side library

add_library(hl SHARED
    "dlls/wpn_shared/hl_wpn_glock.cpp"
    "dlls/aflock.cpp"
    "dlls/agrunt.cpp"
    "dlls/airtank.cpp"
    "dlls/animating.cpp"
    "dlls/animation.cpp"
    "dlls/apache.cpp"
    "dlls/barnacle.cpp"
    "dlls/barney.cpp"
    "dlls/bigmomma.cpp"
    "dlls/bloater.cpp"
    "dlls/bmodels.cpp"
    "dlls/bullsquid.cpp"
    "dlls/buttons.cpp"
    "dlls/cbase.cpp"
    "dlls/client.cpp"
    "dlls/combat.cpp"
    "dlls/controller.cpp"
    "dlls/crossbow.cpp"
    "dlls/crowbar.cpp"
    "dlls/defaultai.cpp"
    "dlls/doors.cpp"
    "dlls/effects.cpp"
    "dlls/egon.cpp"
    "dlls/explode.cpp"
    "dlls/flyingmonster.cpp"
    "dlls/func_break.cpp"
    "dlls/func_tank.cpp"
    "dlls/game.cpp"
    "dlls/gamerules.cpp"
    "dlls/gargantua.cpp"
    "dlls/gauss.cpp"
    "dlls/genericmonster.cpp"
    "dlls/ggrenade.cpp"
    "dlls/globals.cpp"
    "dlls/gman.cpp"
    "dlls/h_ai.cpp"
    "dlls/h_battery.cpp"
    "dlls/h_cine.cpp"
    "dlls/h_cycler.cpp"
    "dlls/h_export.cpp"
    "dlls/handgrenade.cpp"
    "dlls/hassassin.cpp"
    "dlls/headcrab.cpp"
    "dlls/healthkit.cpp"
    "dlls/hgrunt.cpp"
    "dlls/hornet.cpp"
    "dlls/hornetgun.cpp"
    "dlls/houndeye.cpp"
    "dlls/ichthyosaur.cpp"
    "dlls/islave.cpp"
    "dlls/items.cpp"
    "dlls/leech.cpp"
    "dlls/lights.cpp"
    "dlls/maprules.cpp"
    "dlls/monstermaker.cpp"
    "dlls/monsters.cpp"
    "dlls/monsterstate.cpp"
    "dlls/mortar.cpp"
    "dlls/mp5.cpp"
    "dlls/multiplay_gamerules.cpp"
    "dlls/nihilanth.cpp"
    "dlls/nodes.cpp"
    "dlls/observer.cpp"
    "dlls/osprey.cpp"
    "dlls/pathcorner.cpp"
    "dlls/plane.cpp"
    "dlls/plats.cpp"
    "dlls/player.cpp"
    "dlls/python.cpp"
    "dlls/rat.cpp"
    "dlls/roach.cpp"
    "dlls/rpg.cpp"
    "dlls/satchel.cpp"
    "dlls/schedule.cpp"
    "dlls/scientist.cpp"
    "dlls/scripted.cpp"
    "dlls/shotgun.cpp"
    "dlls/singleplay_gamerules.cpp"
    "dlls/skill.cpp"
    "dlls/sound.cpp"
    "dlls/soundent.cpp"
    "dlls/spectator.cpp"
    "dlls/squadmonster.cpp"
    "dlls/squeakgrenade.cpp"
    "dlls/subs.cpp"
    "dlls/talkmonster.cpp"
    "dlls/teamplay_gamerules.cpp"
    "dlls/tempmonster.cpp"
    "dlls/tentacle.cpp"
    "dlls/triggers.cpp"
    "dlls/tripmine.cpp"
    "dlls/turret.cpp"
    "dlls/util.cpp"
    "dlls/weapons.cpp"
    "dlls/world.cpp"
    "dlls/xen.cpp"
    "dlls/zombie.cpp"
    "game_shared/voice_gamemgr.cpp"
    "pm_shared/pm_debug.c"
    "pm_shared/pm_math.c"
    "pm_shared/pm_shared.c")

set_target_properties(hl PROPERTIES PREFIX "")

# The engine requires GiveFnptrsToDll() function to have an ordinal #1.
# I'm certainly not a very experienced Windows developer and had no idea
# about this stuff before.
# Anyway, the only way to manually set an ordinal number is to use .def
# file when linking. Otherwise, the ordinal number will be picked automatically
# by the linker and GiveFnptrsToDll() will get some random number. As a result,
# the engine will not be able to load the server-side DLL.

if(WIN32 AND MSVC)
    # MSVC? But can we use any other compiler at all?
    target_link_options(hl PRIVATE "/def:${CMAKE_SOURCE_DIR}\\dlls\\hl.def")
endif()

#-------------------------------------------------------------------------------
# Client-side library

#---------------------------------------
# System dependencies

add_library(client_sys INTERFACE)

if(WIN32)
    target_link_libraries(client_sys INTERFACE wsock32)
else()
    target_link_libraries(client_sys INTERFACE m dl pthread)
endif()

#---------------------------------------
# Prebuilt SDL2

add_library(SDL2 INTERFACE)
target_include_directories(SDL2 INTERFACE "${CMAKE_SOURCE_DIR}/external")

if(WIN32)
    target_link_directories(SDL2 INTERFACE "${CMAKE_SOURCE_DIR}/lib/public")
    target_link_libraries(SDL2 INTERFACE "SDL2.lib")
else()
    target_link_directories(SDL2 INTERFACE "${CMAKE_SOURCE_DIR}/linux")
    target_link_libraries(SDL2 INTERFACE ":libSDL2-2.0.so.0")
endif()

#---------------------------------------
# Prebuilt VGUI library

add_library(vgui INTERFACE)
target_include_directories(vgui INTERFACE "${CMAKE_SOURCE_DIR}/utils/vgui/include")

if(WIN32)
    target_link_directories(vgui INTERFACE "${CMAKE_SOURCE_DIR}/utils/vgui/lib/win32_vc6")
    target_link_libraries(vgui INTERFACE "vgui.lib")
else()
    target_link_directories(vgui INTERFACE "${CMAKE_SOURCE_DIR}/linux")
    target_link_libraries(vgui INTERFACE ":vgui.so")
endif()

#---------------------------------------
# Client-side library itself

add_library(client SHARED
    "cl_dll/hl/hl_baseentity.cpp"
    "cl_dll/hl/hl_events.cpp"
    "cl_dll/hl/hl_objects.cpp"
    "cl_dll/hl/hl_weapons.cpp"
    "cl_dll/ammo_secondary.cpp"
    "cl_dll/ammo.cpp"
    "cl_dll/ammohistory.cpp"
    "cl_dll/battery.cpp"
    "cl_dll/cdll_int.cpp"
    "cl_dll/com_weapons.cpp"
    "cl_dll/death.cpp"
    "cl_dll/demo.cpp"
    "cl_dll/entity.cpp"
    "cl_dll/ev_common.cpp"
    "cl_dll/ev_hldm.cpp"
    "cl_dll/events.cpp"
    "cl_dll/flashlight.cpp"
    "cl_dll/GameStudioModelRenderer.cpp"
    "cl_dll/geiger.cpp"
    "cl_dll/health.cpp"
    "cl_dll/hud_bench.cpp"
    "cl_dll/hud_benchtrace.cpp"
    "cl_dll/hud_msg.cpp"
    "cl_dll/hud_redraw.cpp"
    "cl_dll/hud_servers.cpp"
    "cl_dll/hud_spectator.cpp"
    "cl_dll/hud_update.cpp"
    "cl_dll/hud.cpp"
    "cl_dll/in_camera.cpp"
    "cl_dll/input.cpp"
    "cl_dll/inputw32.cpp"
    "cl_dll/interpolation.cpp"
    "cl_dll/menu.cpp"
    "cl_dll/message.cpp"
    "cl_dll/saytext.cpp"
    "cl_dll/status_icons.cpp"
    "cl_dll/statusbar.cpp"
    "cl_dll/studio_util.cpp"
    "cl_dll/StudioModelRenderer.cpp"
    "cl_dll/text_message.cpp"
    "cl_dll/train.cpp"
    "cl_dll/tri.cpp"
    "cl_dll/util.cpp"
    "cl_dll/vgui_ClassMenu.cpp"
    "cl_dll/vgui_ConsolePanel.cpp"
    "cl_dll/vgui_ControlConfigPanel.cpp"
    "cl_dll/vgui_CustomObjects.cpp"
    "cl_dll/vgui_int.cpp"
    "cl_dll/vgui_MOTDWindow.cpp"
    "cl_dll/vgui_SchemeManager.cpp"
    "cl_dll/vgui_ScorePanel.cpp"
    "cl_dll/vgui_ServerBrowser.cpp"
    "cl_dll/vgui_SpectatorPanel.cpp"
    "cl_dll/vgui_TeamFortressViewport.cpp"
    "cl_dll/vgui_teammenu.cpp"
    "cl_dll/view.cpp"
    "cl_dll/voice_status.cpp"
    "dlls/wpn_shared/hl_wpn_glock.cpp"
    "dlls/crossbow.cpp"
    "dlls/crowbar.cpp"
    "dlls/egon.cpp"
    "dlls/gauss.cpp"
    "dlls/handgrenade.cpp"
    "dlls/hornetgun.cpp"
    "dlls/mp5.cpp"
    "dlls/python.cpp"
    "dlls/rpg.cpp"
    "dlls/satchel.cpp"
    "dlls/shotgun.cpp"
    "dlls/squeakgrenade.cpp"
    "dlls/tripmine.cpp"
    "common/parsemsg.cpp"
    "game_shared/vgui_checkbutton2.cpp"
    "game_shared/vgui_grid.cpp"
    "game_shared/vgui_helpers.cpp"
    "game_shared/vgui_listbox.cpp"
    "game_shared/vgui_loadtga.cpp"
    "game_shared/vgui_scrollbar2.cpp"
    "game_shared/vgui_slider2.cpp"
    "game_shared/voice_banmgr.cpp"
    "public/interface.cpp"
    "pm_shared/pm_debug.c"
    "pm_shared/pm_math.c"
    "pm_shared/pm_shared.c")

set_target_properties(client PROPERTIES PREFIX "")

target_compile_definitions(client PRIVATE CLIENT_DLL HL_DLL)

if(NOT WIN32)
    target_compile_definitions(client PRIVATE DISABLE_JUMP_ORIGIN DISABLE_VEC_ORIGIN)
endif()

target_link_libraries(client PRIVATE client_sys SDL2 vgui)
