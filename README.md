# Breakpad

项目fork自[google breakpad](https://chromium.googlesource.com/breakpad/breakpad/).

Breakpad是一组实现崩溃报告系统的客户端和服务器组件。

Breakpad 可以在移除编译器调试信息后，抓取、压缩 minidump 信息，将其发送回你的服务器，然后为 C/C++ 生成调用栈。

Breakpad 的特点主要在于崩溃报告部分支持无符号抓取。 整套工具实现了在客户使用无符号的发布版应用前提下， 开发者也能以较低代价恢复应用崩溃现场的调用栈。

Breakpad 包含三大组件：

client：读取当前线程的状态、加载的可执行文件、共享库等信息，写入到 minidump 中。可以放到应用中，当崩溃发生时自动使用，或者显式调用。

symbol dumper(`dump_syms`)：读取编译器生成的调试信息，产生基于Breakpad格式的symbol file。

processor(`minidump_stackwalk`)：读取minidump寻找适合的symbol file，生成可读的C/C++调用栈。

![breakpad](./docs/breakpad.png)


## 如何在Linux应用中添加Breakpad（客户端）
1.  将Breakpad作为第三方库添加进你的工程
    
    在bash中，添加breakpad作为submodule.
    ```sh
    cd project_source_path/third_party
    git submodule add http://10.10.42.100:8888/toolbox/breakpad.git
    ```
    
    然后在CmakeLists中添加breakpad路径.
    ```cmake
    include_directories(${PROJECT_SOURCE_DIR}/third_party/breakpad)
    add_subdirectory(${PROJECT_SOURCE_DIR}/third_party/breakpad)
    ```

    在CmakeLists中链接breakpad.
    ```cmake
    target_link_libraries(test
        ...
        client
        )
    ```

2.  在main中使用breakpad

    ```c++
    #include "client/linux/handler/exception_handler.h"
    
    static bool dumpCallback(const google_breakpad::MinidumpDescriptor& descriptor,
    void* context, bool succeeded) {
        // printf("Dump path: %s\n", descriptor.path());
        return succeeded;
    }

    void crash() { volatile int* a = (int*)(NULL); *a = 1; }
    
    int main(int argc, char* argv[]) {
        // set dump file save path
        google_breakpad::MinidumpDescriptor descriptor("/tmp"); 
        google_breakpad::ExceptionHandler eh(descriptor, NULL, dumpCallback, NULL, true, -1);
        
        // crash trigger, just as example
        crash();
        return 0;
    }
    ```
    
    程序崩溃时, 会在/tmp文件夹下生成xxx.dmp文件,该文件中保存了程序崩溃时的调用栈信息.
    (在实际应用程序中，可以通过将dmp文件发送到服务器进行分析。
    Breakpad 源代码树包含一些您可能会发现有用的 HTTP 上传源，
    以及一个 minidump 上传工具。)

## 如何分析dmp文件，定位到源代码具体位置（服务器端）

1.  生成服务器端分析工具`dump_syms`和`minidump_stackwalk`.

    ```sh
    cd breakpad
    ./configure && make
    make install
    ```

2.  生成应用程序符号文件
    
    首先，确保编译选项中加入了-g选项。
    ```cmake
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")
    ```    

    为了生成有用的堆栈跟踪，Breakpad要求将二进制文件中的调试符号转换为文本格式的符号文件。
    
    ```sh
    dump_syms ./test > test.sym
    ```

    为了在`minidump_stackwalk`工具中使用这些符号，您需要将它们放在特定的目录结构中。
    符号文件的第一行包含生成此目录结构所需的信息，例如（您的输出会有所不同）：
    ```sh
    head -n1 test.sym #输出 MODULE Linux x86_64 6EDC6ACDB282125843FD59DA9C81BD830
    mkdir -p ./symbols/test/6EDC6ACDB282125843FD59DA9C81BD830
    mv test.sym ./symbols/test/6EDC6ACDB282125843FD59DA9C81BD830
    ```
    
3. 生成堆栈信息
   
   下面的命令生成跟踪栈，并打印。
    ```sh
    minidump_stackwalk minidump.dmp ./symbols
    ```
   
4. 配合`addr2line`定位到行。

    ```sh
    addr2line -e ./test  addr #具体的address
    ```


## 其他工具

1.  去除可执行文件中的调试信息

    ```sh
    strip --strip-debug filename
    ```