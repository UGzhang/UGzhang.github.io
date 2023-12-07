---
layout:     post
title:      Calling An Executable Within C++
subtitle:   QTextDocument
date:       2023-03-01
author:     UG
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - Python
---

## Describe
 A simple example of calling this exe in C++.

## Effect
Tried to find the DPI of a JPG in Qt, but have trouble getting the right DPI value by using QImage.  The same problem as mentioned in this [link](https://bugreports.qt.io/browse/QTBUG-3618). So I am using Pillow library in python to get the correct dpi. Then turn python code into stand-alone executables by using Pyinstaller library in Python. Finally, call the .exe with C++ code.

## Python Code
```
import sys
from PIL import Image

image_path = sys.argv[0]
with Image.open(image_path) as img:
    dpi = img.info.get('dpi')

sys.stdout = open("output.txt", "w") #write the result to output.txt
print(dpi)
sys.exit(1)

```

## C++ Code
```
#include <iostream>
#include <string>
#include <windows.h>
#include <tchar.h>


int main()
{
    // modified it to your exe path
    TCHAR exePath[] = _T("C:\\Users\\main.exe");
    
    // modified it to your JPG path
    TCHAR cmd[] = _T("C:\\Users\\1.JPG"); 

    // Create a process and pass arguments
    STARTUPINFO si;  
    PROCESS_INFORMATION pi; 
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    if (!CreateProcess(
        exePath,   //Path of the called-exe
        cmd,       //cmd arguments
        NULL,   
        NULL,   
        FALSE,  
        0,
        NULL,
        NULL,
        &si,
        &pi)) 
    {
        std::cout << "Error: CreateProcess failed!" << std::endl;
        return -1;  
    }

    // Wait for process to end
    WaitForSingleObject(pi.hProcess, INFINITE);
    
    // the return code of exe
    DWORD exitCode;
    GetExitCodeProcess(pi.hProcess, &exitCode);
    CloseHandle(pi.hThread);  
    CloseHandle(pi.hProcess); 

    std::cout << "Return value: " << exitCode << std::endl;

    return 0;
}

```

## Summarize
It is not cost-effective to creating a new process and the Python exe is almost 10M. I wouldn't consider this is a practical solution.