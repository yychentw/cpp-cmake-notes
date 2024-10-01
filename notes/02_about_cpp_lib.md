# 常見的 C++ library 類型

Library 是為其他程式提供可重複使用程式碼的集合。Library 本身不會被拿來執行；而應用程式會使用 library 中的資料結構、功能，藉此達成特定的應用。

C++ library 根據其類型，在使用或建立上會有不同的特性：

## Static library

* 預先編譯好的 binary 檔案，會與 header 檔案一起釋出
* Client code 在連結（link） static library 時，library 的內容會被直接複製到 client code 中的對應位置。
* 因此 client code 最終編譯出來的 binary 檔案較大，不過執行速度較快，而且在執行階段就不需要 static library。
* 要維護使用 static library 的應用程式比較困難，當 library 更新時，整個應用程式會需要重新編譯
* 檔案格式
    * Linux/MacOS: \*.a
    * Windows: \*.lib
* 如何使用 CMake 建立 static library：
    ```cmake
    add_library(<name> STATIC [<source>...])
    # link libraries & include directories as needed
    ```


## Dynamic library

* 預先編譯好的 binary 檔案，會與 header 檔案一起釋出
* 在 client 的應用程式編譯完要執行時，才會動態的載入 dynamic library 中的內容
* 因為在執行階段才動態載入 dynamic library，所以 client 的應用程式執行速度會略慢，且 library 需要與應用程式放在一起，或安裝到執行環境中
* 多個 client binaries 可以共用一個 dynamic library，比較節省空間
* Dynamic library 有更新時，如果介面沒有改變，只要抽換 library 即可，client binaries 不用重新編譯
* 檔案格式
    * Linux: \*.so
    * MacOS: \*.dylib
    * Windows: \*.dll
* 如何使用 CMake 建立 dynamic library：
    ```cmake
    add_library(<name> SHARED [<source>...])
    # link libraries & include directories as needed
    ```

## Header-only library

* 只由 header 檔案組成，不需要預先編譯就可以直接加到 client project 中
* 使用 template 開發的 library 通常為 header-only
* 因為不會預先編譯，所以使用者可以看到 library 所有的實作細節
* Client code 的編譯時間成本較高，因為需要與 header-only library 一起編譯
* 與使用 static library 類似的是，每當 header-only library 有更新，client code 就需要重新編譯
* 如何使用 CMake 建立 header-only library 的 target：
    ```cmake
    add_library(<name> INTERFACE)
    target_include_directories(<name> INTERFACE [<header_dir>...])
    ```

## 參考資源

* [Header-only vs. Static vs. Dynamic Libraries](https://ctfchan.github.io/posts/header-static-dynamic-libraries/)
* [C++ Development Tutorial 4: Static and Dynamic Libraries](https://domiyanyue.medium.com/c-development-tutorial-4-static-and-dynamic-libraries-7b537656163e#:~:text=We%20learned%20the%20basics%20of,loaded%20and%20linked%20at%20runtime.)
* [[Day 11] Library 類型](https://ithelp.ithome.com.tw/articles/10317010)
