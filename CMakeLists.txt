cmake_minimum_required(VERSION 3.14)

project(padcr)

if(WIN32)
    enable_language(ASM_MASM)
endif()

add_subdirectory(vendor/zlib EXCLUDE_FROM_ALL)

set(mini_chromium_src vendor/mini_chromium/base/debug/alias
    vendor/mini_chromium/base/files/file_path
    vendor/mini_chromium/base/files/scoped_file
    vendor/mini_chromium/base/logging
    vendor/mini_chromium/base/process/memory
    vendor/mini_chromium/base/rand_util
    vendor/mini_chromium/base/strings/string16
    vendor/mini_chromium/base/strings/string_number_conversions
    vendor/mini_chromium/base/strings/stringprintf
    vendor/mini_chromium/base/strings/utf_string_conversion_utils
    vendor/mini_chromium/base/strings/utf_string_conversions
    vendor/mini_chromium/base/synchronization/lock
    vendor/mini_chromium/base/third_party/icu/icu_utf
    vendor/mini_chromium/base/threading/thread_local_storage
)

if(WIN32)
    set(mini_chromium_src ${mini_chromium_src}
        vendor/mini_chromium/base/process/process_metrics_win
        vendor/mini_chromium/base/strings/string_util_win
        vendor/mini_chromium/base/synchronization/lock_impl_win
        vendor/mini_chromium/base/threading/thread_local_storage_win
    )
elseif(APPLE)
    set(mini_chromium_src ${mini_chromium_src}
        vendor/mini_chromium/base/files/file_util_posix
        vendor/mini_chromium/base/posix/safe_strerror
        vendor/mini_chromium/base/process/process_metrics_posix
        vendor/mini_chromium/base/synchronization/condition_variable_posix
        vendor/mini_chromium/base/synchronization/lock_impl_posix
        vendor/mini_chromium/base/threading/thread_local_storage_posix
        vendor/mini_chromium/base/mac/close_nocancel
        vendor/mini_chromium/base/mac/foundation_util
        vendor/mini_chromium/base/mac/mach_logging
        vendor/mini_chromium/base/mac/scoped_mach_port
        vendor/mini_chromium/base/mac/scoped_mach_vm
        vendor/mini_chromium/base/mac/scoped_nsautorelease_pool
        vendor/mini_chromium/base/strings/sys_string_conversions_mac
    )
else()
    set(mini_chromium_src ${mini_chromium_src}
        vendor/mini_chromium/base/files/file_util_posix
        vendor/mini_chromium/base/posix/safe_strerror
        vendor/mini_chromium/base/process/process_metrics_posix
        vendor/mini_chromium/base/synchronization/condition_variable_posix
        vendor/mini_chromium/base/synchronization/lock_impl_posix
        vendor/mini_chromium/base/threading/thread_local_storage_posix
    )
endif()

add_library(
    mini_chromium
    STATIC
    ${mini_chromium_src}
)

if(WIN32)
    target_compile_definitions(
        mini_chromium
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        mini_chromium
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )

    target_link_libraries(
        mini_chromium
        PUBLIC
        advapi32
    )
elseif(UNIX)
    set_target_properties(mini_chromium
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        mini_chromium
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    target_link_libraries(
        mini_chromium
        PUBLIC
        pthread
        dl
    )

    if(NOT APPLE)
        target_compile_definitions(
            mini_chromium
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    else()
        target_link_libraries(
            mini_chromium
            PUBLIC
            -framework ApplicationServices
            -framework CoreFoundation
            -framework Foundation
            -framework IOKit
            -framework Security
        )
    endif()
endif()

target_include_directories(
    mini_chromium
    PUBLIC
    vendor/mini_chromium
)

if(WIN32)
    add_library(
        getopt
        STATIC
        vendor/crashpad/third_party/getopt/getopt
    )

    target_compile_definitions(
        getopt
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        getopt
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )

    target_include_directories(
        getopt
        PUBLIC
        vendor/crashpad/third_party/getopt
    )
endif()

if(WIN32)
    add_library(
        crashpad_compat
        STATIC
        vendor/crashpad/compat/win/strings.cc
        vendor/crashpad/compat/win/time.cc
        vendor/crashpad/compat/win/time.h
    )

    target_compile_definitions(
        crashpad_compat
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_compat
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )

    target_include_directories(
        crashpad_compat
        PUBLIC
        vendor/crashpad/compat/win
    )
elseif(UNIX AND NOT APPLE)
    add_library(
        crashpad_compat
        STATIC
        vendor/crashpad/compat/linux/sys/mman.cc
        vendor/crashpad/compat/linux/sys/mman.h
    )

    target_link_libraries(
        crashpad_compat
        PUBLIC
        dl
    )

    set_target_properties(crashpad_compat
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_compat
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    target_compile_definitions(
        mini_chromium
        PRIVATE
        _FILE_OFFSET_BITS=64
    )

    target_include_directories(
        crashpad_compat
        PUBLIC
        vendor/crashpad/compat/linux
    )
endif()

add_library(
    crashpad_tools
    STATIC
    vendor/crashpad/tools/tool_support
)

target_link_libraries(
    crashpad_tools
    PUBLIC
    mini_chromium
)

if(WIN32)
    target_compile_definitions(
        crashpad_tools
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_tools
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )
elseif(UNIX)
    set_target_properties(crashpad_tools
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_tools
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_compile_definitions(
            crashpad_tools
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    endif()
endif()

target_include_directories(
    crashpad_tools
    PUBLIC
    vendor/crashpad
)

set(crashpad_util_src vendor/crashpad/util/file/delimited_file_reader
    vendor/crashpad/util/file/file_io
    vendor/crashpad/util/file/file_reader
    vendor/crashpad/util/file/file_seeker
    vendor/crashpad/util/file/file_writer
    vendor/crashpad/util/file/scoped_remove_file
    vendor/crashpad/util/file/string_file
    vendor/crashpad/util/misc/initialization_state_dcheck
    vendor/crashpad/util/misc/lexing
    vendor/crashpad/util/misc/metrics
    vendor/crashpad/util/misc/pdb_structures
    vendor/crashpad/util/misc/random_string
    vendor/crashpad/util/misc/range_set
    vendor/crashpad/util/misc/reinterpret_bytes
    vendor/crashpad/util/misc/scoped_forbid_return
    vendor/crashpad/util/misc/time
    vendor/crashpad/util/misc/uuid
    vendor/crashpad/util/misc/zlib
    vendor/crashpad/util/net/http_body
    vendor/crashpad/util/net/http_body_gzip
    vendor/crashpad/util/net/http_multipart_builder
    vendor/crashpad/util/net/http_transport
    vendor/crashpad/util/net/url
    vendor/crashpad/util/numeric/checked_address_range
    vendor/crashpad/util/process/process_memory
    vendor/crashpad/util/process/process_memory_range
    vendor/crashpad/util/stdlib/aligned_allocator
    vendor/crashpad/util/stdlib/string_number_conversion
    vendor/crashpad/util/stdlib/strlcpy
    vendor/crashpad/util/stdlib/strnlen
    vendor/crashpad/util/string/split_string
    vendor/crashpad/util/stream/zlib_output_stream
    vendor/crashpad/util/thread/thread
    vendor/crashpad/util/thread/thread_log_messages
    vendor/crashpad/util/thread/worker_thread
)

if(WIN32)
    set(crashpad_util_src ${crashpad_util_src} vendor/crashpad/util/file/directory_reader_win
        vendor/crashpad/util/file/file_io_win
        vendor/crashpad/util/file/filesystem_win
        vendor/crashpad/util/misc/capture_context_win.asm
        vendor/crashpad/util/misc/clock_win
        vendor/crashpad/util/misc/paths_win
        vendor/crashpad/util/misc/time_win
        vendor/crashpad/util/net/http_transport_win
        vendor/crashpad/util/process/process_memory_win
        vendor/crashpad/util/synchronization/semaphore_win
        vendor/crashpad/util/thread/thread_win
        vendor/crashpad/util/win/command_line
        vendor/crashpad/util/win/critical_section_with_debug_info
        vendor/crashpad/util/win/exception_handler_server
        vendor/crashpad/util/win/get_function
        vendor/crashpad/util/win/get_module_information
        vendor/crashpad/util/win/handle
        vendor/crashpad/util/win/initial_client_data
        vendor/crashpad/util/win/module_version
        vendor/crashpad/util/win/nt_internals
        vendor/crashpad/util/win/ntstatus_logging
        vendor/crashpad/util/win/process_info
        vendor/crashpad/util/win/registration_protocol_win
        vendor/crashpad/util/win/safe_terminate_process.asm
        vendor/crashpad/util/win/scoped_handle
        vendor/crashpad/util/win/scoped_local_alloc
        vendor/crashpad/util/win/scoped_process_suspend
        vendor/crashpad/util/win/scoped_set_event
        vendor/crashpad/util/win/session_end_watcher
    )
elseif(APPLE)
    set(crashpad_util_src ${crashpad_util_src}
        vendor/crashpad/util/file/directory_reader_posix
        vendor/crashpad/util/file/file_io_posix
        vendor/crashpad/util/file/filesystem_posix
        vendor/crashpad/util/misc/clock_posix
        vendor/crashpad/util/posix/close_stdio
        vendor/crashpad/util/posix/scoped_dir
        vendor/crashpad/util/posix/scoped_mmap
        vendor/crashpad/util/posix/signals
        vendor/crashpad/util/synchronization/semaphore_posix
        vendor/crashpad/util/thread/thread_posix
        vendor/crashpad/util/posix/close_multiple
        vendor/crashpad/util/posix/double_fork_and_exec
        vendor/crashpad/util/posix/drop_privileges
        vendor/crashpad/util/posix/symbolic_constants_posix
        vendor/crashpad/util/mac/launchd
        vendor/crashpad/util/mac/mac_util
        vendor/crashpad/util/mac/service_management
        vendor/crashpad/util/mac/xattr
        vendor/crashpad/util/mach/child_port_handshake
        vendor/crashpad/util/mach/child_port_server
        vendor/crashpad/util/mach/composite_mach_message_server
        vendor/crashpad/util/mach/exc_client_variants
        vendor/crashpad/util/mach/exc_server_variants
        vendor/crashpad/util/mach/exception_behaviors
        vendor/crashpad/util/mach/exception_ports
        vendor/crashpad/util/mach/exception_types
        vendor/crashpad/util/mach/mach_extensions
        vendor/crashpad/util/mach/mach_message
        vendor/crashpad/util/mach/mach_message_server
        vendor/crashpad/util/mach/notify_server
        vendor/crashpad/util/mach/scoped_task_suspend
        vendor/crashpad/util/mach/symbolic_constants_mach
        vendor/crashpad/util/mach/task_for_pid
        vendor/crashpad/util/misc/capture_context_mac.S
        vendor/crashpad/util/misc/clock_mac
        vendor/crashpad/util/misc/paths_mac
        vendor/crashpad/util/net/http_transport_mac
        vendor/crashpad/util/posix/process_info_mac
        vendor/crashpad/util/process/process_memory_mac
        vendor/crashpad/util/synchronization/semaphore_mac
    )
    set_property(SOURCE vendor/crashpad/util/misc/capture_context_mac.S PROPERTY LANGUAGE C)
else()
    set(crashpad_util_src ${crashpad_util_src}
        vendor/crashpad/util/file/directory_reader_posix
        vendor/crashpad/util/file/file_io_posix
        vendor/crashpad/util/file/filesystem_posix
        vendor/crashpad/util/misc/clock_posix
        vendor/crashpad/util/posix/close_stdio
        vendor/crashpad/util/posix/scoped_dir
        vendor/crashpad/util/posix/scoped_mmap
        vendor/crashpad/util/posix/signals
        vendor/crashpad/util/synchronization/semaphore_posix
        vendor/crashpad/util/thread/thread_posix
        vendor/crashpad/util/posix/close_multiple
        vendor/crashpad/util/posix/double_fork_and_exec
        vendor/crashpad/util/posix/drop_privileges
        vendor/crashpad/util/posix/symbolic_constants_posix
        vendor/crashpad/util/net/http_transport_socket
        vendor/crashpad/util/linux/auxiliary_vector
        vendor/crashpad/util/linux/direct_ptrace_connection
        vendor/crashpad/util/linux/exception_handler_client
        vendor/crashpad/util/linux/exception_handler_protocol
        vendor/crashpad/util/linux/memory_map
        vendor/crashpad/util/linux/proc_stat_reader
        vendor/crashpad/util/linux/proc_task_reader
        vendor/crashpad/util/linux/ptrace_broker
        vendor/crashpad/util/linux/ptrace_client
        vendor/crashpad/util/linux/ptracer
        vendor/crashpad/util/linux/scoped_pr_set_dumpable
        vendor/crashpad/util/linux/scoped_pr_set_ptracer
        vendor/crashpad/util/linux/scoped_ptrace_attach
        vendor/crashpad/util/linux/socket
        vendor/crashpad/util/linux/thread_info
        vendor/crashpad/util/misc/capture_context_linux.S
        vendor/crashpad/util/misc/paths_linux
        vendor/crashpad/util/posix/process_info_linux
        vendor/crashpad/util/process/process_memory_linux
        vendor/crashpad/util/process/process_memory_sanitized
    )
    set_property(SOURCE vendor/crashpad/util/misc/capture_context_linux.S PROPERTY LANGUAGE C)
endif()

add_library(
    crashpad_util
    STATIC
    ${crashpad_util_src}
)

target_compile_definitions(
    crashpad_util
    PRIVATE
    ZLIB_CONST
    CRASHPAD_ZLIB_SOURCE_EXTERNAL
    CRASHPAD_LSS_SOURCE_EXTERNAL
)

target_link_libraries(
    crashpad_util
    PUBLIC
    mini_chromium
)

if(WIN32)
    target_compile_definitions(
        crashpad_util
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_util
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )

    target_link_libraries(
        crashpad_util
        PUBLIC
        crashpad_compat
        zlibstatic

        user32
        version
        winhttp
    )
elseif(UNIX)
    set_target_properties(crashpad_util
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_util
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_link_libraries(
            crashpad_util
            PUBLIC
            crashpad_compat
            zlibstatic
        )

        target_compile_definitions(
            crashpad_util
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    else()
        target_link_libraries(
            crashpad_util
            PUBLIC
            zlibstatic
            bsm
            -framework CoreFoundation
            -framework Foundation
            -framework IOKit
        )
    endif()
endif()

target_include_directories(
    crashpad_util
    PUBLIC
    vendor/crashpad
    vendor/zlib
    vendor
    "$<TARGET_FILE_DIR:zlibstatic>/../"
)

if(APPLE)
    target_include_directories(
        crashpad_util
        PUBLIC
        vendor/crashpad/third_party/xnu/EXTERNAL_HEADERS/mach-o
        vendor/crashpad/compat/mac
    )
endif()

set(crashpad_client_src vendor/crashpad/client/annotation
    vendor/crashpad/client/annotation_list
    vendor/crashpad/client/crash_report_database
    vendor/crashpad/client/crashpad_info
    vendor/crashpad/client/prune_crash_reports
    vendor/crashpad/client/settings
)

if(WIN32)
    set(crashpad_client_src ${crashpad_client_src}
        vendor/crashpad/client/crash_report_database_win
        vendor/crashpad/client/crashpad_client_win
    )
elseif(APPLE)
    set(crashpad_client_src ${crashpad_client_src}
        vendor/crashpad/client/crash_report_database_mac
        vendor/crashpad/client/crashpad_client_mac
        vendor/crashpad/client/simulate_crash_mac
    )
else()
    set(crashpad_client_src ${crashpad_client_src}
        vendor/crashpad/client/crashpad_client_linux
        vendor/crashpad/client/client_argv_handling
        vendor/crashpad/client/crashpad_info_note.S
        vendor/crashpad/client/crash_report_database_generic
    )
    set_property(SOURCE vendor/crashpad/client/crashpad_info_note.S PROPERTY LANGUAGE C)
endif()

add_library(
    crashpad_client
    STATIC
    ${crashpad_client_src}
)

target_link_libraries(
    crashpad_client
    PUBLIC
    mini_chromium
)

target_compile_definitions(
    crashpad_client
    PRIVATE
    CRASHPAD_LSS_SOURCE_EXTERNAL
)

if(WIN32)
    target_compile_definitions(
        crashpad_client
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_client
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
        /wd4201  # nonstandard extension used : nameless struct/union
    )

    target_link_libraries(
        crashpad_client
        PUBLIC
        crashpad_compat
        crashpad_util
        rpcrt4
    )
elseif(UNIX)
    set_target_properties(crashpad_client
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_client
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_link_libraries(
            crashpad_client
            PUBLIC
            crashpad_compat
            crashpad_util
        )

        target_compile_definitions(
            crashpad_client
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    else()
        target_link_libraries(
            crashpad_client
            PUBLIC
            crashpad_util
        )
    endif()
endif()

target_include_directories(
    crashpad_client
    PUBLIC
    vendor/crashpad
    vendor
)

add_library(
    crashpad_minidump
    STATIC
    vendor/crashpad/minidump/minidump_annotation_writer
    vendor/crashpad/minidump/minidump_byte_array_writer
    vendor/crashpad/minidump/minidump_context_writer
    vendor/crashpad/minidump/minidump_crashpad_info_writer
    vendor/crashpad/minidump/minidump_exception_writer
    vendor/crashpad/minidump/minidump_extensions
    vendor/crashpad/minidump/minidump_file_writer
    vendor/crashpad/minidump/minidump_handle_writer
    vendor/crashpad/minidump/minidump_memory_info_writer
    vendor/crashpad/minidump/minidump_memory_writer
    vendor/crashpad/minidump/minidump_misc_info_writer
    vendor/crashpad/minidump/minidump_module_crashpad_info_writer
    vendor/crashpad/minidump/minidump_module_writer
    vendor/crashpad/minidump/minidump_rva_list_writer
    vendor/crashpad/minidump/minidump_simple_string_dictionary_writer
    vendor/crashpad/minidump/minidump_stream_writer
    vendor/crashpad/minidump/minidump_string_writer
    vendor/crashpad/minidump/minidump_system_info_writer
    vendor/crashpad/minidump/minidump_thread_id_map
    vendor/crashpad/minidump/minidump_thread_writer
    vendor/crashpad/minidump/minidump_unloaded_module_writer
    vendor/crashpad/minidump/minidump_user_stream_writer
    vendor/crashpad/minidump/minidump_writable
    vendor/crashpad/minidump/minidump_writer_util
)

target_include_directories(
    crashpad_minidump
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_minidump
    PUBLIC
    mini_chromium
)

if(WIN32)
    target_compile_definitions(
        crashpad_minidump
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_minidump
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
        /wd4201  # nonstandard extension used : nameless struct/union
        /wd4324  # 'struct' : structure was padded due to __declspec(align())
    )

    target_link_libraries(
        crashpad_minidump
        PUBLIC
        crashpad_compat
        crashpad_util
    )
elseif(UNIX)
    target_include_directories(
        crashpad_minidump
        PUBLIC
        vendor/crashpad/compat/non_win
    )

    set_target_properties(crashpad_minidump
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_minidump
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_link_libraries(
            crashpad_minidump
            PUBLIC
            crashpad_compat
            crashpad_util
        )

        target_compile_definitions(
            crashpad_minidump
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    else()
        target_link_libraries(
            crashpad_minidump
            PUBLIC
            crashpad_util
        )
    endif()
endif()

set(crashpad_snapshot_src vendor/crashpad/snapshot/annotation_snapshot
    vendor/crashpad/snapshot/capture_memory
    vendor/crashpad/snapshot/cpu_context
    vendor/crashpad/snapshot/crashpad_info_client_options
    vendor/crashpad/snapshot/handle_snapshot
    vendor/crashpad/snapshot/memory_snapshot
    vendor/crashpad/snapshot/unloaded_module_snapshot
    vendor/crashpad/snapshot/crashpad_types/crashpad_info_reader
    vendor/crashpad/snapshot/minidump/exception_snapshot_minidump
    vendor/crashpad/snapshot/minidump/memory_snapshot_minidump
    vendor/crashpad/snapshot/minidump/minidump_annotation_reader
    vendor/crashpad/snapshot/minidump/minidump_context_converter
    vendor/crashpad/snapshot/minidump/minidump_simple_string_dictionary_reader
    vendor/crashpad/snapshot/minidump/minidump_string_list_reader
    vendor/crashpad/snapshot/minidump/minidump_string_reader
    vendor/crashpad/snapshot/minidump/module_snapshot_minidump
    vendor/crashpad/snapshot/minidump/process_snapshot_minidump
    vendor/crashpad/snapshot/minidump/system_snapshot_minidump
    vendor/crashpad/snapshot/minidump/thread_snapshot_minidump
    vendor/crashpad/snapshot/x86/cpuid_reader # remove if not compiling on x86
)

if(WIN32)
    set(crashpad_snapshot_src ${crashpad_snapshot_src}
        vendor/crashpad/snapshot/win/capture_memory_delegate_win
        vendor/crashpad/snapshot/win/cpu_context_win
        vendor/crashpad/snapshot/win/exception_snapshot_win
        vendor/crashpad/snapshot/win/memory_map_region_snapshot_win
        vendor/crashpad/snapshot/win/module_snapshot_win
        vendor/crashpad/snapshot/win/pe_image_annotations_reader
        vendor/crashpad/snapshot/win/pe_image_reader
        vendor/crashpad/snapshot/win/pe_image_resource_reader
        vendor/crashpad/snapshot/win/process_reader_win
        vendor/crashpad/snapshot/win/process_snapshot_win
        vendor/crashpad/snapshot/win/process_subrange_reader
        vendor/crashpad/snapshot/win/system_snapshot_win
        vendor/crashpad/snapshot/win/thread_snapshot_win
        vendor/crashpad/snapshot/crashpad_types/crashpad_info_reader
    )
elseif(APPLE)
    set(crashpad_snapshot_src ${crashpad_snapshot_src}
        vendor/crashpad/snapshot/posix/timezone
        vendor/crashpad/snapshot/mac/cpu_context_mac
        vendor/crashpad/snapshot/mac/exception_snapshot_mac
        vendor/crashpad/snapshot/mac/mach_o_image_annotations_reader
        vendor/crashpad/snapshot/mac/mach_o_image_reader
        vendor/crashpad/snapshot/mac/mach_o_image_segment_reader
        vendor/crashpad/snapshot/mac/mach_o_image_symbol_table_reader
        vendor/crashpad/snapshot/mac/module_snapshot_mac
        vendor/crashpad/snapshot/mac/process_reader_mac
        vendor/crashpad/snapshot/mac/process_snapshot_mac
        vendor/crashpad/snapshot/mac/process_types
        vendor/crashpad/snapshot/mac/process_types/custom
        vendor/crashpad/snapshot/mac/system_snapshot_mac
        vendor/crashpad/snapshot/mac/thread_snapshot_mac
    )
else()
    set(crashpad_snapshot_src ${crashpad_snapshot_src}
        vendor/crashpad/snapshot/linux/cpu_context_linux
        vendor/crashpad/snapshot/linux/debug_rendezvous
        vendor/crashpad/snapshot/linux/exception_snapshot_linux
        vendor/crashpad/snapshot/linux/process_reader_linux
        vendor/crashpad/snapshot/linux/process_snapshot_linux
        vendor/crashpad/snapshot/linux/system_snapshot_linux
        vendor/crashpad/snapshot/linux/thread_snapshot_linux
        vendor/crashpad/snapshot/sanitized/memory_snapshot_sanitized
        vendor/crashpad/snapshot/sanitized/module_snapshot_sanitized
        vendor/crashpad/snapshot/sanitized/process_snapshot_sanitized
        vendor/crashpad/snapshot/sanitized/sanitization_information
        vendor/crashpad/snapshot/sanitized/thread_snapshot_sanitized
        vendor/crashpad/snapshot/posix/timezone
        vendor/crashpad/snapshot/crashpad_types/crashpad_info_reader
        vendor/crashpad/snapshot/crashpad_types/image_annotation_reader
        vendor/crashpad/snapshot/elf/elf_dynamic_array_reader
        vendor/crashpad/snapshot/elf/elf_image_reader
        vendor/crashpad/snapshot/elf/elf_symbol_table_reader
        vendor/crashpad/snapshot/elf/module_snapshot_elf
        vendor/crashpad/snapshot/crashpad_types/crashpad_info_reader
    )
endif()

add_library(
    crashpad_snapshot
    STATIC
    ${crashpad_snapshot_src}
)

target_include_directories(
    crashpad_snapshot
    PUBLIC
    vendor/crashpad
)

target_link_libraries(
    crashpad_snapshot
    PUBLIC
    mini_chromium
)

if(WIN32)
    target_compile_definitions(
        crashpad_snapshot
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_snapshot
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )

    target_link_libraries(
        crashpad_snapshot
        PUBLIC
        crashpad_compat

        powrprof
    )
elseif(UNIX)
    target_include_directories(
        crashpad_snapshot
        PUBLIC
        vendor/crashpad/compat/non_win
    )

    set_target_properties(crashpad_snapshot
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_snapshot
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_link_libraries(
            crashpad_snapshot
            PUBLIC
            crashpad_compat
        )

        target_compile_definitions(
            crashpad_snapshot
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    endif()
endif()

set(crashpad_handler_src vendor/crashpad/handler/crash_report_upload_thread
    vendor/crashpad/handler/handler_main
    vendor/crashpad/handler/main
    vendor/crashpad/handler/minidump_to_upload_parameters
    vendor/crashpad/handler/prune_crash_reports_thread
    vendor/crashpad/handler/user_stream_data_source
)

if(WIN32)
    set(crashpad_handler_src ${crashpad_handler_src}
        vendor/crashpad/handler/win/crash_report_exception_handler
    )
elseif(APPLE)
    set(crashpad_handler_src ${crashpad_handler_src}
        vendor/crashpad/handler/mac/crash_report_exception_handler
        vendor/crashpad/handler/mac/exception_handler_server
        vendor/crashpad/handler/mac/file_limit_annotation
    )
else()
    set(crashpad_handler_src ${crashpad_handler_src}
        vendor/crashpad/handler/linux/capture_snapshot
        vendor/crashpad/handler/linux/crash_report_exception_handler
        vendor/crashpad/handler/linux/exception_handler_server
    )
endif()

add_executable(
    crashpad_handler
    WIN32
    ${crashpad_handler_src}
)

target_link_libraries(
    crashpad_handler
    PRIVATE
    crashpad_client
    crashpad_minidump
    crashpad_snapshot
    crashpad_tools
)

if(WIN32)
    target_compile_definitions(
        crashpad_handler
        PRIVATE
        NOMINMAX
        UNICODE
        WIN32
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )

    target_compile_options(
        crashpad_handler
        PRIVATE
        /FS
        /W4
        /WX
        /Zi
        /bigobj  # Support larger number of sections in obj file.
        /wd4100  # Unreferenced formal parameter.
        /wd4127  # Conditional expression is constant.
        /wd4324  # Structure was padded due to alignment specifier.
        /wd4351  # New behavior: elements of array will be default initialized.
        /wd4577  # 'noexcept' used with no exception handling mode specified.
        /wd4996  # 'X' was declared deprecated.
    )
elseif(UNIX)
    set_target_properties(crashpad_handler
        PROPERTIES
        CXX_STANDARD 14
        C_STANDARD 11
    )

    target_compile_options(
        crashpad_handler
        PRIVATE
        -Wno-multichar
        -fPIC
    )

    if(NOT APPLE)
        target_compile_definitions(
            crashpad_handler
            PRIVATE
            _FILE_OFFSET_BITS=64
        )
    endif()
endif()

target_include_directories(
    crashpad_handler
    PRIVATE
    vendor/crashpad
    vendor/mini_chromium
)

if(WIN32)
    target_include_directories(
        crashpad_handler
        PRIVATE
        vendor/crashpad/compat/win
    )

    target_link_libraries(
        crashpad_handler
        PRIVATE
        getopt
    )

    # Make a .com file copy from the .exe with subsystem console
    add_custom_command(
        TARGET crashpad_handler POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:crashpad_handler>
            "$<TARGET_FILE_DIR:crashpad_handler>/crashpad_handler.com"
    )

    add_custom_command(
        TARGET crashpad_handler POST_BUILD
        COMMAND editbin
            -nologo
            -subsystem:console
            "$<TARGET_FILE_DIR:crashpad_handler>/crashpad_handler.com"
    )

    set_source_files_properties(
        vendor/crashpad/util/misc/capture_context_win.asm
        PROPERTIES
        COMPILE_FLAGS "/safeseh"
    )

    set_source_files_properties(
        vendor/crashpad/util/win/safe_terminate_process.asm
        PROPERTIES
        COMPILE_FLAGS "/safeseh"
    )
elsif(APPLE)
    target_include_directories(
        crashpad_handler
        PRIVATE
        vendor/crashpad/compat/mac
    )
else()
    target_include_directories(
        crashpad_handler
        PRIVATE
        vendor/crashpad/compat/linux
    )
endif()
