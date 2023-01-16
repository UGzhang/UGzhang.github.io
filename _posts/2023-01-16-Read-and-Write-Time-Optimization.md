---
layout:     post
title:      Read and Write Time Optimization
subtitle:   QString, std::wtring and std::string
date:       2023-01-16
author:     Yves
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - I/O
    - Optimization
---

## Describe
Extract date from a DXF file and write it to a txt file.

## Test File
A DXF file has 1422390 lines and a size of 10.5M.

## Result
| Replication Times  | Before Opt (Avg.)| After Opt (Avg.)  | 
|------------------|-----------|-------------|
| 100              | 1.3329s   | 0.6027s     |

## Debugging
**Tool**: Visual Studio 2019 - CPU Usage - Call Tree  
**Version**: C++11  
**Mode**: [RelWithDebug](https://learn.microsoft.com/en-us/cpp/build/how-to-debug-a-release-build?view=msvc-170)

## Solution
**Find the expensive cost function in call tree**  
### 1. readDouble()  
- **Describe**: read a string and convert it to double  
- **Analysis**: Converting std::string  -> std::istringstream -> double is too expensive. A simple an direct way is to use [std::stod](https://cplusplus.com/reference/string/stod/) function to convert std::string to double. And a faster way to convert *const char** to float is to use [atof](https://cplusplus.com/reference/cstdlib/atof/).
The double precision is unnecessary as the render precision is set to float.
- **Before Opt**: ***663***/1350(ms) = 49.1%  
![image](/img/20230116/2.1.png)

- **After Opt**:  ***203***/862(ms) = 23.55%
![image](/img/20230116/2.2.png)


### 2. pointToValue()  
- **Describe**: convert double to std::string
- **Analysis**: The most expensive part is `str = std::to_string(x) + ","` which involves two string allocations and a copy from the previous string to the new allocation. To reduce time, I replace [std::to_string()](https://en.cppreference.com/w/cpp/string/basic_string/to_string) to [sprintf_s()](https://en.cppreference.com/w/c/io/fprintf). And std::to_chars(C++17) has better performent than sprintf_s(), but c++17 is not supported by our application.
- **Before Opt**: ***218***/862(ms) = 26.49%  
![image](/img/20230116/2.3.png)

- **After Opt**:  ***141***/726(ms) = 19.42%
![image](/img/20230116/2.4.png)


### 3. setBasic()
- **Describe**: convert double to std::string and ASCII to std::string
- **Analysis**: Use [std::stringstream](https://cplusplus.com/reference/sstream/stringstream/) to convert double to std::string is expensive, I use the sprintf_s function mentioned previously. Then, the ASCII value of a char is convrt to int and then to std::string. This is a [blog](https://www.zverovich.net/2013/09/07/integer-to-string-conversion-in-cplusplus.html) compares different ways to convert int to std::string. 

- **Before Opt**: ***94***/726(ms) = 12.95%  
![image](/img/20230116/2.5.png)

- **After Opt**:  ***17***/659(ms) = 2.58%
![image](/img/20230116/2.6.png)


### 4. getHandleString()
- **Describe**: Convert hexadecimal number std::string to a decimal int.
- **Analysis**: It is redundant that convert std::string to std::isstringstream and then to hexadecimal integer. The [std::stoi](https://cplusplus.com/reference/string/stoi/) can direct convert type and number system.

- **Before Opt**: ***43***/659(ms) = 6.53%  
![image](/img/20230116/2.7.png)

- **After Opt**:  ***1***/627(ms) = 0.16%
![image](/img/20230116/2.8.png)
 

### 5. Difference
- **Describe**: There are some differences between previous output file and new file.
![image](/img/20230116/2.9.png)
