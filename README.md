# Stacktrace Extraction Process
** Debug ve Release ** Birlikte Kullanım
- RelWithdebInfo modda build al
- `extract_symbols_auto.sh -b ./rftt` ile ~/.local/share/Ustek/.syms 'ye sembolleri çıkar.
- `objcopy --only-keep-debug rftt rftt.debug` ile debugları ayrı dosyaya al
- `strip --strip-debug rftt` ile debug symbolleri binaryden çıkar.
- hafifletilmiş rftt yi müşteriye ver core dumped olursa `~/.local/share/Ustek/.crashdumps` pathine git dumpları al 
- `minidump_stackwalk ~/.local/share/Ustek/.crashdumps/*.dmp ~/.local/share/Ustek/.syms > ~/Desktop/crash_log.txt` ile stacktrace'i gör.


---

# Build Pipeline

-  `git submodule add git submodule add https://github.com/hafikanyesilyurt-collab/crashpad-cmake-qt.git `
- git submodule update --init --recursive
- **CMake Configuration**    
    - add_subdirectory (crashpad-cmake-qt)
    - Compiler Flags: `set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g -DNDEBUG -fno-omit-frame-pointer")`
    - Check submodule: `if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/crashpad-cmake-qt/CMakeLists.txt")
    message(FATAL_ERROR "Crashpad submodule not founded")
endif()`
    - Crashpad variables:
     `set(CRASHPAD_ZLIB_SYSTEM ON CACHE BOOL "Use system zlib library")
set(CRASHPAD_ENABLE_STACKTRACE ON CACHE BOOL "Activate stacktrace support`
    - Crashpad server needs Thread package `Threads::Threads`
    Include crashpad headers : `target_include_directories(CrashQt PRIVATE
    ${CMAKE_SOURCE_DIR}/crashpad-cmake-qt
    ${CMAKE_SOURCE_DIR}/crashpad-cmake-qt/vendor/third-party/mini_chromium/mini_chromium
)`
    - Linking project with statis libs : `target_link_libraries(CrashQt PRIVATE
    Qt${QT_VERSION_MAJOR}::Widgets
    crashpad_snapshot
    crashpad_client
    crashpad_tools
    crashpad_util
    mini_chromium
    Threads::Threads
    ${CMAKE_DL_LIBS}
    rt
    m
)
- Build Exceptions
    - Reinterpreted error: `#include <cstdint>`;
    - kMaxValue error: target_link_libraries ( crashpad_client Submodule > CMakeLists.txt . 980.`
    

--- 



***Note***
- rftt-debug size: 113.3 mb
- rftt-release size: 17.4 mb
- rftt-relwithdebinfo size (Release + Debug symbols) : 148 mb
- rftt-relwithdebinfo > only_debug_file size : 132 mb
- rftt-relwithdebinfo > only_release_file size: 17 mb


## Önemli Callback ve Sinyaller Özeti

### Callback Mekanizmaları:
1. **FirstChanceHandler**: Crash olmadan önce çağrılır, exception'ı handle etme şansı verir

### Qt Sinyalleri:
1. **handlerStarted()**: Handler başarıyla başlatıldığında
2. **handlerFailed(QString)**: Handler başlatılamadığında
3. **crashDumpCreated(QString)**: Dump oluşturulduğunda
4. **uploadCompleted(QString)**: Upload tamamlandığında
5. **uploadFailed(QString, QString)**: Upload başarısız olduğunda
6. **databaseStatusChanged()**: Database durumu değiştiğinde

# To-Do

** toollar **
- dump_syms, minidump_stackwalk customer ortamında indir kur
- toolları /home/cerberus/.local/share/Ustek yazdırmalısın
- toolları PATH'E eklemelisin

