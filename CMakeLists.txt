cmake_minimum_required(VERSION 3.6)

if(${EMSCRIPTEN})
	set(OPT_FLAG "-Oz")
  #set(OPT_FLAG "-O2 -g4 --source-map-base http://localhost:8080/")
	set(USE_PORTS "-s USE_ZLIB=1")
	set(CMAKE_C_FLAGS "${OPT_FLAG} -w -Werror=return-type ${USE_PORTS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} ${USE_PORTS} -Wno-c++11-narrowing")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} \
		${USE_PORTS} \
		${OPT_FLAG} \
		-s TOTAL_STACK=1MB \
		-s MALLOC=emmalloc \
		--emit-symbol-map \
		-s EXIT_RUNTIME=1 \
		-s ASSERTIONS=0 \
		-s SAFE_HEAP=0 \
		-s TOTAL_MEMORY=67108864 \
		-s ALLOW_MEMORY_GROWTH=1 \
		-s INVOKE_RUN=0 \
		-s MODULARIZE=1 \
		-s EXPORT_NAME='WDOSBOX' \
		-s FORCE_FILESYSTEM=1 -lidbfs.js -lwebsocket.js \
		-s EXTRA_EXPORTED_RUNTIME_METHODS=\"['getMemory', 'addRunDependency', 'removeRunDependency','FS', 'FS_createPath', 'FS_createPreloadedFile', \
			'FS_createDataFile', 'lengthBytesUTF8', 'stringToUTF8', 'UTF16ToString', 'UTF8ToString', 'callMain']\" \
		-s NO_EXIT_RUNTIME=1 \
		-s ERROR_ON_UNDEFINED_SYMBOLS=0")
	add_definitions(-DEMSCRIPTEN)		
else()
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror=return-type -Wno-deprecated")
	add_definitions(-DC_DEBUG=1 -DC_HEAVY_DEBUG=1 -DEMSCRIPTEN_KEEPALIVE=)
	add_definitions(-DEMTERPRETER_SYNC)

	include_directories("${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/linux")
endif()

add_definitions(-DHAVE_CONFIG_H -DGET_X86_FUNCTIONS -DJSDOS)

include(
	"${CMAKE_CURRENT_LIST_DIR}/3rd-party/3rd-party.cmake"
)

include_directories(
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/include"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/include"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2"
)

set(SOURCES_CXX11 
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-ci.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-events.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-json.cpp"
#sdlmain
#	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/sdlmain.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/gui/sdlmain.cpp"
)
set_source_files_properties(${SOURCES_CXX11} PROPERTIES COMPILE_FLAGS "-std=c++11")

set(SOURCES 
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-3rdparty.c"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-debug.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-debug-mem.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-asyncify.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/js-dos-core.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dosbox.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_simple.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/paging.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_prefetch.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_dyn_x86.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_full.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_dynrec.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/core_normal.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/cpu.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/callback.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/flags.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/cpu/modrm.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/fpu/fpu.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_pal.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/ems.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/xms.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_put_pixel.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/bios_disk.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/mouse.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/bios.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_misc.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_char.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_video_state.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_memory.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_vptable.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/bios_keyboard.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_vesa.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/ints/int10_modes.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drive_local.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drive_cache.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_keyboard_layout.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom_ioctl_linux.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_tables.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_devices.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_mscdex.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom_ioctl_win32.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_execute.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_programs.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drive_virtual.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drives.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_ioctl.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_memory.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_classes.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom_aspi_win32.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom_image.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_misc.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drive_fat.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/cdrom_ioctl_os2.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/drive_iso.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/dos/dos_files.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/shell/shell.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/shell/shell_cmds.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/shell/shell_batch.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/shell/shell_misc.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/curstxt.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/sdl_mapper.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/midi.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/sdl_gui.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/render_scalers.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/gui/render.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/debug/debug.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/debug/debug_win32.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/debug/debug_disasm.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/debug/debug_gui.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/pcspeaker.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/sblaster.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/gus.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_attr.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_paradise.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/ipxserver.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/dma.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_xga.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/timer.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_dac.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/pic.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_s3.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/cmos.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_memory.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/dbopl.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_crtc.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/hardware.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_draw.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/ipx.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/adlib.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/mixer.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/iohandler.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/joystick.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_seq.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/pci_bus.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/mpu401.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/keyboard.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_misc.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/tandy_sound.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_tseng.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_gfx.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/vga_other.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/gameblaster.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/disney.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/memory.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/directserial.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/serialdummy.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/nullmodem.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/serialport.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/libserial.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/softmodem.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/serialport/misc_util.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/misc/programs.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/misc/setup.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/misc/cross.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/misc/support.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/misc/messages.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/libs/gui_tk/gui_tk.cpp"
#	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/hardware/opl.cpp"
#	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/libs/zmbv/drvproc.cpp"
#	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/libs/zmbv/zmbv.cpp"
#	"${CMAKE_CURRENT_LIST_DIR}/dreamlayers-em-dosbox-em-dosbox-svn-sdl2/src/libs/zmbv/zmbv_vfw.cpp"
)

if(${EMSCRIPTEN})
	add_library(bin-asyncify OBJECT ${SOURCES} ${SOURCES_CXX11} ${SOURCES_3RDPARTY})
        target_compile_definitions(bin-asyncify PUBLIC -DEMTERPRETER_SYNC -DASYNCIFY -Demscripten_sleep_with_yield=asyncify_sleep -Demscripten_sleep=asyncify_sleep)
	add_executable(wdosbox $<TARGET_OBJECTS:bin-asyncify>)
	set_target_properties(wdosbox PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox PROPERTIES LINK_FLAGS "-s WASM=1 -s ASYNCIFY -s 'ASYNCIFY_IMPORTS=[\"emscripten_sleep\",\"syncSleep\",\"wsipxWait\"]' -s ASYNCIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/asyncify.txt")
  #set_target_properties(wdosbox PROPERTIES LINK_FLAGS "-s WASM=1 -s ASYNCIFY -s 'ASYNCIFY_IMPORTS=[\"emscripten_sleep\",\"syncSleep\",\"wsipxWait\"]'")
	add_executable(wdosbox-profiling $<TARGET_OBJECTS:bin-asyncify>)
	set_target_properties(wdosbox-profiling PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox-profiling PROPERTIES LINK_FLAGS "--profiling-funcs -s WASM=1 -s ASYNCIFY -s 'ASYNCIFY_IMPORTS=[\"emscripten_sleep\", \"syncSleep\"]' -s ASYNCIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/asyncify.txt")
	add_executable(dosbox $<TARGET_OBJECTS:bin-asyncify>)
	set_target_properties(dosbox PROPERTIES SUFFIX .js)
	set_target_properties(dosbox PROPERTIES LINK_FLAGS "-s WASM=0 -s ALLOW_MEMORY_GROWTH=0 --memory-init-file 0 -s ASYNCIFY -s 'ASYNCIFY_IMPORTS=[\"emscripten_sleep\", \"syncSleep\"]' -s ASYNCIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/asyncify.txt")

	add_library(bin-emterp OBJECT ${SOURCES} ${SOURCES_CXX11} ${SOURCES_3RDPARTY})
    target_compile_definitions(bin-emterp PUBLIC -DEMTERPRETER_SYNC)
	add_executable(wdosbox-emterp $<TARGET_OBJECTS:bin-emterp>)
	set_target_properties(wdosbox-emterp PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox-emterp PROPERTIES LINK_FLAGS "-s WASM=1 -s EMTERPRETIFY=1 -s EMTERPRETIFY_ASYNC=1 -s EMTERPRETIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/emterpretify.txt")
	add_executable(wdosbox-emterp-profiling $<TARGET_OBJECTS:bin-emterp>)
	set_target_properties(wdosbox-emterp-profiling PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox-emterp-profiling PROPERTIES LINK_FLAGS "--profiling-funcs -s WASM=1 -s EMTERPRETIFY=1 -s EMTERPRETIFY_ASYNC=1 -s EMTERPRETIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/emterpretify.txt")
	add_executable(dosbox-emterp $<TARGET_OBJECTS:bin-emterp>)
	set_target_properties(dosbox-emterp PROPERTIES SUFFIX .js)
	set_target_properties(dosbox-emterp PROPERTIES LINK_FLAGS "-s WASM=0 -s ALLOW_MEMORY_GROWTH=0 --memory-init-file 0 -s EMTERPRETIFY=1 -s EMTERPRETIFY_ASYNC=1 -s EMTERPRETIFY_WHITELIST=@${CMAKE_CURRENT_LIST_DIR}/js-dos-cpp/emterpretify.txt")

	add_library(bin-nosync OBJECT ${SOURCES} ${SOURCES_CXX11} ${SOURCES_3RDPARTY})
	add_executable(wdosbox-nosync $<TARGET_OBJECTS:bin-nosync>)
	set_target_properties(wdosbox-nosync PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox-nosync PROPERTIES LINK_FLAGS "-s WASM=1")
	add_executable(wdosbox-nosync-profiling $<TARGET_OBJECTS:bin-nosync>)
	set_target_properties(wdosbox-nosync-profiling PROPERTIES SUFFIX .js)
	set_target_properties(wdosbox-nosync-profiling PROPERTIES LINK_FLAGS "--profiling-funcs -s WASM=1")
	add_executable(dosbox-nosync $<TARGET_OBJECTS:bin-nosync>)
	set_target_properties(dosbox-nosync PROPERTIES SUFFIX .js)
	set_target_properties(dosbox-nosync PROPERTIES LINK_FLAGS "-s WASM=0 -s ALLOW_MEMORY_GROWTH=0 --memory-init-file 0")
else()
	add_executable(bin ${SOURCES} ${SOURCES_CXX11} ${SOURCES_3RDPARTY})
	target_link_libraries(bin
		SDL z ncurses
	)

	set_target_properties(bin PROPERTIES COMPILE_FLAGS "-m32" LINK_FLAGS "-m32")
endif()
