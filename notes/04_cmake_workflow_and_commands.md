# CMake 專案的建構流程與常用指令

課程中採用 CMake 作為 C++ 專案的建構工具，然而 CMake 的建構流程在課程中都是分段介紹，並非一個完整的主題。

我認為對於初接觸 CMake 的開發者來說，能先瞭解大致的執行流程與對應的指令，對於如何使用或開發一個 CMake 專案，會有比較完整的認識，因此整理了本篇筆記。

為了不要所有資訊都混在一起，筆記會先整理概念性的 CMake 執行流程，呈現流程中包含哪些步驟後，再介紹各個步驟對應的指令（與命令列參數）。

## CMake 簡介

CMake 是開源的跨平台自動化建構工具，它本身不是建構系統（build system），而是會根據平台、編譯器產生對應的建構檔並建構程式。

CMake 屬於高階的建構工具，使用者利用同一套 CMake 語法和指令就可以在不同平台編譯程式，而即使只在單一平台使用，也比直接使用個別建構系統（build system）來得簡單方便。

## CMake 專案的建構流程

### 在專案資料夾底下建立 CMakeLists.txt，

CMakeLists.txt 是由 CMake 語法寫成的 build configuration file，用以控制專案如何建構。

在 CMakeLists.txt 中，會定義程式碼檔案的位置、要建構的函式庫（library）與執行檔，也包含第三方套件的管理，甚至自動化測試、安裝的規則都可以寫在 CMakeLists.txt。

不過在本篇文章中不會介紹到 CMake 語法，如果想了解怎麼撰寫簡單的 CMakeLists.txt，可以參考官方的學習資源 [Writing CMakeLists Files](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Writing%20CMakeLists%20Files.html) 和 [CMake 入門](https://zh.wikibooks.org/wiki/CMake_%E5%85%A5%E9%96%80)。

### 產生 build files

在這個階段 CMake 會藉由 CMakeLists.txt 中定義的內容、建構的平台與使用的編譯器，產生對應的建構檔。

例如如果想在 Ubuntu 系統上用 GNU Make 建構專案的話，指定 CMake generator 為 `Unix Makefiles`，在 configure 後即可產生建構檔 Makefile（後面會有相關的例子）。

### Build

產生的建構檔後，就可以依照一般建構方式或 CMake 建構命令產生最終的軟體。

例如在 configure 的例子中我們已產生了 Makefile，接著就可以用 `make` 指令建構程式了。

### Install

CMake 可以把建構好的最終軟體安裝到系統預設路徑或特定的路徑。

## In-source build & out-of-source build

在正式進到 CMake 指令的介紹前，想先補充 in-source build 和 out-of-source build 的概念。

In-source build 指的是在程式原始碼所在的位置直接編譯程式，而過程中所產生的所有建構相關檔案（包括建構檔、函式庫、執行檔）都會存放在和原始碼相同的目錄（directory）底下。這樣的做法雖然很直接了當，但混合了原始碼與建構檔案，會讓專案變得不容易瀏覽，而且重新建構程式時，會很難清理先前的建構相關檔案。

Out-of-source build 則是讓建構相關檔案與程式原始碼分開在不同目錄底下的做法，一般來說會在另外建立一個 `build/` 目錄以存放建構檔案。這樣原始碼的目錄能保持乾淨，要重新建構程式時，只要把 `build/` 刪掉即可。而且如果要針對不同的 build configurations 進行建構，只要建立多個不同的建構目錄（build directories）即可達成。

綜合來說，out-of-source build 是目前建議的做法。雖然流程上會比較複雜，但是可以讓建構相關檔案更容易管理，也讓建構本身更有彈性。

## 常用的 CMake commands

這裡我將根據剛剛介紹的 CMake workflow，依序介紹產生 build files、build、install 所使用的 CMake commands。完整的 CMake commands 可以參考官方文件 [CMake Command-Line Tools](https://cmake.org/cmake/help/latest/manual/cmake.1.html)。

### 產生 build files

CMake 產生 build files 的指令一般形式如下：
```bash  
cmake [<options>] -B <path-to-build> [-S <path-to-source>]
```

`-B` 指定建構目錄（build directory），`-S` 指定程式原始碼目錄（source directory），也是最上層 CMakeLists.txt 的位置。如果依照剛剛介紹的 out-of-source build 的做法，建構目錄與原始碼目錄會是不同的。

而其他選項 `[<options>]` 的部分，我們可以用 `-D<var>=<value>` 指定或變更現有的 CMake 變數，例如 `CMAKE_BUILD_TYPE` 是 CMake 既有的變數，一般預設為 `Debug`，如果我們要使用編譯優化以產生適合發布的軟體，就可以加入 `-DCMAKE_BUILD_TYPE=Release` 這個指令選項。

而假設當前工作目錄為建構目錄，就不用指定 `-B`，只要標明程式原始碼目錄即可，此時指令會如以下形式：
```bash 
cmake [<options>] <path-to-source>
```

最後補充一個參數 `-G`，用於指定產生建構系統/建構檔的 generator，指令會如以下形式：
```bash  
cmake -S <path-to-source> -B <path-to-build> -G <generator-name>
```
在前面建構流程有提到，如果想在 Ubuntu 系統上用 GNU Make 建構專案的話，就要透過 `-G Unix Makefiles` 指定 CMake generator。關於 CMake 在不同平台支援的 generators 可以參考 [CMake Generator](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html#manual:cmake-generators(7))。

### Build

產生建構檔案以後，就可以建構專案了。在這個步驟除了可以直接依照對應的 build system 的流程進行建構，也可以使用 CMake 的指令，其一般形式如下：
```bash
cmake --build <path-to-build> [<options>] [-- <build-tool-options>]
```
值得注意的是 `--build` 一定要是第一個參數，不能與後面的參數調換順序。

這裡要介紹一個常用的參數 `--target`，用來指定要建構的 target，適合只需要建構專案中的一部份時使用，指令形式如下：
```bash    
cmake --build <path-to-build> --target <target-name>
```
如果不指定 target，CMake 會建構預設的所有 targets（libraries 和 executables）。

### Install

建構出最終的軟體後，就可以將它安裝到系統預設路徑或特定的路徑，CMake 指令的一般形式如下：
```bash
cmake --install <dir> [<options>]
```
和建構時的指令類似，`--install` 一定要是第一個參數，不能與後面的參數調換順序。

## 學習資源

* [CMake 入門](https://zh.wikibooks.org/wiki/CMake_%E5%85%A5%E9%96%80)
* [Writing CMakeLists Files](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Writing%20CMakeLists%20Files.html)
* [CMake Command-Line Tools](https://cmake.org/cmake/help/latest/manual/cmake.1.html)
