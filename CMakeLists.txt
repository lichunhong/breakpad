# CMake wrapper for Google Breakpad
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
include(CheckIncludeFile)
project(breakpad-cmake VERSION 0.0.1)

# check for various headers
check_include_file("a.out.h" HAVE_A_OUT_H)
check_include_file("inttypes.h" HAVE_INTTYPES_H)
check_include_file("memory.h" HAVE_MEMORY_H)
check_include_file("pthread.h" HAVE_PTHREAD_H)
check_include_file("stdint.h" HAVE_STDINT_H)
check_include_file("stdlib.h" HAVE_STDLIB_H)
check_include_file("strings.h" HAVE_STRINGS_H)
check_include_file("string.h" HAVE_STRING_H)
check_include_file("sys/random.h" HAVE_SYS_RANDOM_H)
check_include_file("sys/random.h" HAVE_SYS_STAT_H)
check_include_file("sys/random.h" HAVE_SYS_TYPES_H)
check_include_file("unistd.h" HAVE_UNISTD_H)

# breakpad_client must be build static (shared library calls ARE BAD in exception land)
add_library(client STATIC
    # linux specific sources
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/crash_generation/crash_generation_client.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/crash_generation/crash_generation_server.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/dump_writer_common/thread_info.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/dump_writer_common/ucontext_reader.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/handler/exception_handler.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/handler/minidump_descriptor.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/log/log.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/microdump_writer/microdump_writer.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/minidump_writer/linux_core_dumper.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/minidump_writer/linux_dumper.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/minidump_writer/linux_ptrace_dumper.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/client/linux/minidump_writer/minidump_writer.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/elf_core_dump.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/elfutils.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/file_id.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/guid_creator.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/linux_libc_support.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/memory_mapped_file.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/safe_readlink.cc>
    $<$<PLATFORM_ID:Linux>:${CMAKE_CURRENT_LIST_DIR}/src/common/linux/http_upload.cc>

    # mac specific sources
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/handler/breakpad_nlist_64.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/crash_generation/ConfigFile.mm>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/crash_generation/crash_generation_client.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/crash_generation/crash_generation_server.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/handler/dynamic_images.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/handler/exception_handler.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/handler/minidump_generator.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/handler/protected_memory_allocator.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/arch_utilities.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/bootstrap_compat.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/file_id.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/launch_reporter.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/MachIPC.mm>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/macho_id.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/macho_utilities.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/macho_walker.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/common/mac/string_utilities.cc>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/Framework/Breakpad.mm>
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/mac/Framework/OnDemandServer.mm>

    # windows specific sources
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/crash_generation/client_info.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/crash_generation/crash_generation_client.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/crash_generation/crash_generation_server.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/crash_generation/minidump_generator.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/handler/exception_handler.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/client/windows/sender/crash_report_sender.cc>
    #$<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/dia_util.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/guid_string.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/http_upload.cc>
    #$<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/omap.cc>
    #$<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/pdb_source_line_writer.cc>
    $<$<PLATFORM_ID:Windows>:${CMAKE_CURRENT_LIST_DIR}/src/common/windows/string_utils.cc>

    # non windows platforms need the minidump writer
    $<$<NOT:$<PLATFORM_ID:Windows>>:${CMAKE_CURRENT_LIST_DIR}/src/client/minidump_file_writer.cc>

    #${CMAKE_CURRENT_LIST_DIR}/src/common/convert_UTF.c
    ${CMAKE_CURRENT_LIST_DIR}/src/common/md5.cc
    ${CMAKE_CURRENT_LIST_DIR}/src/common/simple_string_dictionary.cc
    ${CMAKE_CURRENT_LIST_DIR}/src/common/string_conversion.cc
)

add_library(breakpad::client ALIAS client)

# add definitions related to system configuration
target_compile_definitions(client PRIVATE
    HAVE_CXX11
    $<$<BOOL:HAVE_A_OUT_H>:HAVE_A_OUT_H>
    $<$<BOOL:HAVE_SYS_RANDOM_H>:HAVE_GETRANDOM>
    $<$<BOOL:HAVE_INTTYPES_H>:HAVE_INTTYPES_H>
    $<$<BOOL:HAVE_MEMORY_H>:HAVE_MEMORY_H>
    $<$<BOOL:HAVE_PTHREAD_H>:HAVE_PTHREAD>
    $<$<BOOL:HAVE_STDINT_H>:HAVE_STDINT_H>
    $<$<BOOL:HAVE_STDLIB_H>:HAVE_STDLIB_H>
    $<$<BOOL:HAVE_STRINGS_H>:HAVE_STRINGS_H>
    $<$<BOOL:HAVE_STRING_H>:HAVE_STRING_H>
#    $<$<BOOL:HAVE_SYS_RANDOM_H>:HAVE_SYS_RANDOM_H>
    $<$<BOOL:HAVE_SYS_STAT_H>:HAVE_SYS_STAT_H>
    $<$<BOOL:HAVE_SYS_TYPES_H>:HAVE_SYS_TYPES_H>
    $<$<BOOL:HAVE_UNISTD_H>:HAVE_UNISTD_H>
)

target_compile_definitions(client PUBLIC
    $<$<PLATFORM_ID:Windows>:UNICODE>
    $<$<PLATFORM_ID:Windows>:_UNICODE>
)

# internal interface
target_include_directories(client PRIVATE
    $<$<PLATFORM_ID:Darwin>:${CMAKE_CURRENT_LIST_DIR}/src/client/apple/Framework>
)

# include public interface
target_include_directories(client PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/src
)

# dependencies
target_link_libraries(client PRIVATE
    $<$<PLATFORM_ID:Linux>:dl>
    $<$<PLATFORM_ID:Windows>:wininet>
)
