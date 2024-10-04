# C++ 專案常見的檔案結構

課程中依照 source files 和 header files 存放位置的安排，介紹了三種常見的 C++ 專案結構：

## Source files 和 header files 依照 module 放在一起

`src/` 底下依照 module 分成一到數個 subdirectories，所有相關的 source files 和 header files 都存放於底下，是最精簡的結構。

[TensorFLow](https://github.com/tensorflow/tensorflow) 就屬於這種專案結構。

簡單的結構範例如下：
```
- app/
    - CMakeLists.txt
    - main.cpp
- src/
    - my_lib/
        - CMakeLists.txt
        - my_lib.cpp
        - my_lib.h
    - CMakeLists.txt
- CMakeLists.txt
```

## 完全分開 source files 與 header files

`src/` 底下依照 module subdirectory 存放 source files，`include/` 底下也有相同的 module directory，用以存放 header files。

不過目前也有建議的做法是，只讓 `include/` 存放 public header files，private header files （project 內部使用）要和 source files 一起存放在 `src/`。

這個做法的好處是方便區分專案中公開與非公開的部分，我們可以很容易的取得 public header files，也能透過權限設定只讓開發者有存取 `src/` 的權限。

[Spdlog](https://github.com/gabime/spdlog) 就是採用這種專案結構的例子。

簡單的結構範例如下：
```
- app/
    - CMakeLists.txt
    - main.cpp
- include/
    - my_lib/
        - CMakeLists.txt
        - my_lib.h
    - CMakeLists.txt
- src/
    - my_lib/
        - CMakeLists.txt
        - my_lib.cpp
    - CMakeLists.txt
- CMakeLists.txt
```

## 在 module 底下分開 source files 與 header files

介於前兩種專案結構之間，在 module 底下分開 source files 與 header files，[OpenCV](https://github.com/opencv/opencv) 屬於採用這種專案結構的經典例子。

根據我的觀察，這種專案結構和前一種結構雖然都將 source files 與 header files 分開，但前一種完全分開的方式也許比較適用於中小型、module 數量較少較單純的專案，如果專案龐大複雜，開發時要對照 source files 與 header files 就比較麻煩；而依照 module 分開 source files 與 header files 也許就適用在 module 多且複雜的大型專案，開發者可以專注於個別的 module 即可。

簡單的結構範例如下：
```
- app/
    - CMakeLists.txt
    - main.cpp
- src/
    - my_lib/
        - include/
            - CMakeLists.txt
            - my_lib.h
        - src/
            - CMakeLists.txt
            - my_lib.cpp
        - CMakeLists.txt
    - CMakeLists.txt
- CMakeLists.txt
```

---

不過實際上專案結構並非絕對，也可以看到有些開源專案的檔案結構可能介於剛剛介紹的幾種結構之間，重點是了解這幾個主要類型的特點，並依照專案需求設計合適的檔案結構。

---

## See also

其實 C++ project structuring 不只有 directory structure 的設計，還包含 build system 的使用、dependency 管理、coding standards 等很多面向，這些主題也許未來會整理到，也可以先從下列的文章瞭解：
* [Best Practices for Structuring C++ Projects (Industry Standards)](https://medium.com/@gtech.govind2000/best-practices-for-structuring-c-projects-industry-standards-71b82f6b145c)
* [Modern C++ Project Structuring](https://gregorykelleher.com/interview_practice_questions)
* [Back to basics: How to organize your C/C++ project](https://trussel.ch/cpp/2020/03/30/how-to-organize-your-cpp-project.html)
