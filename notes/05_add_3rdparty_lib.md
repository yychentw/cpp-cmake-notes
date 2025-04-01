# 在 C++ 專案中使用 third-party libraries 的 5+1 種方法

在軟體開發中，我們不可能所有功能都從零開始寫，有時候我們需要借助其他公司、組織或個人寫的軟體程式，讓我們的專案可以使用特定的功能，這就是 third-party library。

實務上如果沒有特殊的理由，推薦多使用發展成熟的 third-party library，避免閉門造車。

以下會介紹幾個在自己專案加入 third-party library 的工具或方法，但使用上盡可能依照自己的專案狀況使用其中一個工具就好，避免多種工具並用，至於如何選擇合適的工具可以看最後的[選擇哪種方式引入外部 library 比較好？](#選擇哪種方式引入外部-library-比較好？)

## 方法一：Git submodule

Git submodule 是 Git 的進階功能，它可以在一個 Git repository 中嵌入其他 Git repositories（例如 third-party libraries）作為 submodules，而保持 commits 分開，讓專案更方便維護。

透過 Git submodule 使用 third-party library，必須符合以下條件： 
* 自己的專案本身是 Git repository
* Third-party library 專案也是 Git repository

我們可以透過以下的 Git command 在自己的專案加入 third-party library：
```bash
git submodule add <3rdparty_lib_url> <3rdparty_lib_dir>
```

### 使用 CMake 自動化加入並建構 third-party Git repository

如果 third-party library 是 CMake 專案，且我們的專案已經有 .gitsubmodules 的記錄，我們可以寫一個 CMake module 以更自動化的加入並建構這個 third-party library，流程大略如下：
* 檢查系統是否有安裝 Git
* 檢查 third-party library directory（在 .gitsubmodules 會定義好）是否存在，以及底下是否有 `CMakeLists.txt`，若無，則呼叫 Git submodule command 更新 third-party library
* 如果 third-party library 為 CMake 專案，則將 third-party library directory 加入 CMake subdirectory，就會與主要專案一起建構了

這裡假設讀者已瞭解基本的 CMake 語法，加入 Git submodule 的 CMake module（`./cmake/AddGitSubmodule.cmake`）的範例程式如下：
```cmake
function(add_git_submodule relative_dir)
    find_package(Git REQUIRED)

    set(FULL_DIR ${CMAKE_SOURCE_DIR}/${relative_dir})

    if (NOT EXISTS ${FULL_DIR}/CMakeLists.txt)
        execute_process(COMMAND ${GIT_EXECUTABLE}
            submodule update --init --recursive -- ${relative_dir}
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
    endif()

    if (EXISTS ${FULL_DIR}/CMakeLists.txt)
        message("Submodule is CMake Project: ${FULL_DIR}/CMakeLists.txt")
        add_subdirectory(${FULL_DIR})
    else()
        message("Submodule is NO CMake Project: ${FULL_DIR}")
    endif()
endfunction(add_git_submodule)
```

然後我們只要在專案最上層的 `CMakeLists.txt` 加入以下幾行即可：
```cmake
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")
include(AddGitSubmodule)
add_git_submodule(<third_party_lib_dir>)
```

## 方法二：FetchContent

FetchContent 是 CMake 內建的 module，用於在 configure time 自動新增內容（例如 third-party libraries）。

要透過 FetchContent 加入並使用 third-party library，必須符合以下條件：
1. Third-party library 是 Git repository
2. Third-party library 是 CMake project

符合以上這兩個條件的話，FetchContent 會比前面提到的 Git submodule 更好用，不過使用 Git submodule 的優點是可以處理 third-party library 不是 CMake project 的情況。

而 FetchContent 的使用方式如下：
```cmake
include(FetchContent)

FetchContent_Declare(
    <name，通常使用 project name> 
    GIT_REPOSITORY <URL> 
    GIT_TAG <version or other tags> 
    (optional) GIT_SHALLOW <TRUE/FALSE>)
    
FetchContent_MakeAvailable(<name> [<name2>...])
```
* `FetchContent_Declare()`：用於宣告 third-party library 的資訊
* `FetchContent_MakeAvailable()`：根據宣告資訊取得 third-party library

注意當要取得多個 third-party libraries 時，要先用 `FetchContent_Declare()` 都一一宣告完，最後再用 `FetchContent_MakeAvailable()` 一次取得所有 libraries。

另外，雖然課程只涵蓋到用 FetchContent 加入 Git repository，但其實 FetchContent 也能用來加入其他 source 的 third-party libraries，關於 FetchContent 的詳細用法請參考 [FetchContent CMake Documentation](https://cmake.org/cmake/help/latest/module/FetchContent.html)。

最後，在加入 third-party library 後，我們需要將它與自己的程式 link 起來，CMake 語法如下：
```cmake
target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> 
    <third_party_project_name>::<third_party_lib_target_name>)
```
這裡需要 third_party_project_name 和 third_party_lib_target_name 兩個資訊，兩者並不一定相同，必須從 library 的 `CMakeLists.txt` 自行查找。

## 方法三：CPM

CPM 是一個可以管理 C++ package 的 CMake script，它無須安裝，只要在 [CPM 的 GitHub repository](https://github.com/cpm-cmake/CPM.cmake) 下載最新發布的 `CPM.cmake`，並加入到自己專案的 `./cmake` 底下，就可以使用。

CPM 其實是基於前面提到的 FetchContent 寫成的，因此使用條件與 FetchContent 相同（third-party library 必須同時是 Git repository 及 CMake project），但 CPM 提供了更簡單的 API，使用上更加方便。

在引入 CPM 後，只需要一行程式就能加入 third-party library：
```cmake
include(cmake/CPM.cmake)
CPMAddPackage("<third_party_lib_expression>")
```

而 third-party library expression 的格式可以是：
* `uri@version`
* `uri#tag`
* 如果是 GitHub repository，URI 的部分可以簡寫成 `gh:user/name`

因此，假設我們要在專案加入 nlohmann json 3.11.3，寫法如下：
```
CPMAddPackage("gh:nlohmann/json#v3.11.3")
```

最後，在將我們的程式與 third-party library link 起來時，CMake 語法與使用 FetchContent 相同，這裡繼續以 link nlohmann json 為例：
```cmake
target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> 
    nlohmann_json::nlohmann_json)
```

## 方法四：Conan

Conan 也是一個開源、跨平台的 C++ Package Manager，它的特點在於能夠建立和使用預先編譯（pre-compiled）好的套件 binaries，並且也有公開的套件庫，因此如果有符合需求的 pre-compiled binaries 就可以直接拉下來使用，省略在 local 端編譯的步驟。

### 安裝

由於 Conan 是由 Python 寫成，因此安裝 Conan 之前需要確保先安裝好 Python3，然後用 pip 即可安裝：
```bash
pip install --user -U conan
```

然後在 terminal 執行 `conan` command 即可確定是否安裝成功。

另外我們還需要有定義各種 Conan configuration 的 Conan profile，要讓 Conan 依照現在的作業系統和環境自動偵測 Conan profile，必須先執行以下：
```bash
conan profile detect --force
```

### 使用 Conan 加入 third-party library

Conan 預設會從 [Conan Center](https://conan.io/center) 下載安裝 libraries，因此可以先在 Conan Center 搜尋是否有所需版本的 libraries，以我們要加入 `zlib/1.2.11` 為例，最簡單的方式就是寫一個 `conanfile.txt`：
```
[requires]
zlib/1.2.11

[generators]
CMakeDeps
CMakeToolchain
```

課程中還提供另一種方法是寫 `conanfile.py`：
```python
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain


class CompressorRecipe(ConanFile):
    settings = "os", "compiler", "build_type", "arch"
    generators = "CMakeDeps"

    def requirements(self):
        self.requires("zlib/1.2.11")

    def generate(self):
        tc = CMakeToolchain(self)
        tc.user_presets_path = False
        tc.generate()
```

其中 `CMakeDeps` 是用來產生要加入的 third-party libraries 的資訊，而 `CMakeToolchain` 則是用來傳遞建構資訊給 CMake。

最後，在 `CMakeLists.txt` 加上以下兩行程式碼即可：
```cmake
include({CMAKE_BINARY_DIR}/conan-toolchain.cmake)
find_package(<lib>)
```

## 方法五：VCPKG

VCPKG 是 Microsoft 打造的跨平台 C/C++ package manager，和 Conan 相似的是也可以使用 pre-compiled libraries。

### 安裝

1. 先從 GitHub repository 複製 VCPKG repository
```bash
git clone https://github.com/microsoft/vcpkg.git
```
2. 執行啟動程式腳本
```
.\vcpkg\bootstrap-vcpkg.bat # windows
./vcpkg/bootstrap-vcpkg.sh # Unix
```

完成以上兩個步驟就可以使用 VCPKG 了。

### 使用 VCPKG 加入 third-party library

步驟和使用 Conan 相似，首先需要先建立含有要加入的 libraries 資訊的 manifest 檔 `vcpkg.json`，以要加入 `fmt` 為例，格式如下：
```jsonld
{
    "dependencies": [
        {
            "name": "fmt",
            "version>=": "9.1.0"
        }
    ]
}
```

除了手動建立 `vcpkg.json` 外，也可以使用 commands 建立並定義 `vcpkg.json`。

而依照課程說明關於 `vcpkg.json`，比較特別的是一般只會定義最低需要的版本，VCPKG 會自動安裝最新的版本，但這在需要指定特定版本時就比較麻煩，會需要另外定義 override 和加上這個版本的 commit SHA。

最後，一樣在 `CMakeLists.txt` 加上以下兩行程式碼即可：
```cmake
include(<VCPKG_DIR>/vcpkg.cmake)
find_package(<lib>)
```

其實有不少 VCPKG 教學文章寫的使用流程有略微的不同，比較推薦的使用流程建議參考 Microsoft 的官方教學 [Tutorial: Install and use packages with CMake](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-bash)。

## 補充方法：使用自行預先編譯的 third-party libraries

有些時候我們要在很特定的平台使用 third-party library，而如果每次使用都要重新編譯會太花時間的話，就會考慮自行預先編譯好套件的 binary 檔案，在建構自己的專案時再加入。

一個最簡單的加入做法是直接在 `CMakeLists.txt` 寫下該 library 的路徑：
```cmake
target_include_directories(<target_name> [PUBLIC|PRIVATE] <third_party_include_dir>)
target_link_libraries(<target_name> [PUBLIC|PRIVATE] <third_party_lib_path>) # applicable to both static and dynamic libraries
```

但更專業的做法是需要針對每一個 third-party library 定義各自的 CMake script `Find<package>.cmake`，以告訴 `CMakeLists.txt` 如何找到對應的 library，如以下格式：
```cmake
# FindYourLibrary.cmake
set(YourLibrary_ROOT_DIR ${PROJECT_SOURCE_DIR}/external/)
set(YourLibrary_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/external/inc/)

if(MSVC)
    set(YourLibrary_LIBRARIES ${PROJECT_SOURCE_DIR}/external/lib/YourLibrary.lib)
else()
    set(YourLibrary_LIBRARIES ${PROJECT_SOURCE_DIR}/external/lib/libYourLibrary.a)
endif()

if(EXISTS ${YourLibrary_LIBRARIES})
    set(YourLibrary_FOUND TRUE)
else()
    set(YourLibrary_FOUND FALSE)
    message("YourLibrary Lib not found!")
endif()
```

注意 `Find<package>.cmake` 需要包含以下資訊
1. Library 的 root directory、include directory, library path，注意此處設定的路徑變數會在 `CMakeLists.txt` 被引用
2. 如果 library path 存在，設定 `<package>_FOUND` 為 TRUE

接著，我們就可以用 `find_package()` 找到 library：
```cmake
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")
find_package(<package> REQUIRED)
```

不過自行編譯的 binary 就會需要考慮存放與版本管理的問題，在讀 Conan 相關資訊的時候發現，Conan 也能在私有機器上進行套件 binary management，不知道是否就能解決這個問題？

## 選擇哪種方式引入外部 library 比較好？

工具的選擇必須根據專案的使用情境以及工具本身的特性：
* **Git submodule**：適合 non-CMake project，例如有些 header-only library 也適用
* **FetchContent、CPM**：兩者底層是相同的，都適用於 GitHub / GitLab CMake project，但建議使用高階的 CPM 會更方便
* **Conan**：如果想使用的 library 自己編譯會很耗時，且 Conan 剛好有的話，可以考慮使用，但使用者需要額外安裝 Python
* **VCPKG**：如果想使用的 library 自己編譯會很耗時，且 VCPKG 剛好有的話，可以考慮使用。但使用者需要另外 clone VCPKG，而且如果需要指定使用特定 library 版本會需要複雜的 configuration

如何要選擇最簡單泛用的方法，至少依照課程的建議，選擇 CPM 會比較好。

## 補充：課程介紹的好用 C++ libraries

* nlohmann_json：use JSON like STL containers
* fmt：string formatting library，在 C\++20 後內建的功能，但在 C\++14 或 17 可引入使用
* cxxopts：lightweight C++ command line option parser，用法如 Python 的 argparser
* spdlog：fast C++ logging library
