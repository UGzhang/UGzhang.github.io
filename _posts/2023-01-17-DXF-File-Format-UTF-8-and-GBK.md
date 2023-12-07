---
layout:     post
title:      DXF File Format:UTF-8 and GBK
subtitle:   Encoding format
date:       2023-01-17
author:     UG
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - Qt
    - Issue
---

## Problem
When importing a DXF file, chinese characters may appear as messy codes and the distance between them is incorrect. (Note: This latter issue will not be addressed in this context.)
![image](/img/20230117/4.2.png) 

The correct display in AutoCAD.
![image](/img/20230117/4.1.png) 

## Analysis
The DXF text is initially converted from std::string to QString use [QString::fromUtf8](https://doc.qt.io/qt-6/qstring.html#fromUtf8-3) function without considering file encoding format. This causes messy code issues when the DXF file use the GBK encoding format.


## Solution
Check if the string format is UTF-8 or GBK(forget screenshot the code!!). If is GBK, use [QString::fromLocal8Bit](https://doc.qt.io/qt-6/qstring.html#fromLocal8Bit).
```
//判断编码text的编码格式
if (is_str_utf8(entity.text.data()))
	context = QString::fromUtf8(entity.text.c_str()); //utf-8
else
	context = QString::fromLocal8Bit(entity.text.c_str()); //gbk
```

```
//判断str是不是utf-8编码格式
bool is_str_utf8(const char* str)
{
	unsigned int nBytes = 0;//UFT8可用1-6个字节编码,ASCII用一个字节
	unsigned char chr = *str;
	bool bAllAscii = true;
	for (unsigned int i = 0; str[i] != '\0'; ++i) {
		chr = *(str + i);
		//判断是否ASCII编码,如果不是,说明有可能是UTF8,ASCII用7位编码,最高位标记为0,0xxxxxxx
		if (nBytes == 0 && (chr & 0x80) != 0) {
			bAllAscii = false;
		}
		if (nBytes == 0) {
			//如果不是ASCII码,应该是多字节符,计算字节数
			if (chr >= 0x80) {
				if (chr >= 0xFC && chr <= 0xFD) {
					nBytes = 6;
				}
				else if (chr >= 0xF8) {
					nBytes = 5;
				}
				else if (chr >= 0xF0) {
					nBytes = 4;
				}
				else if (chr >= 0xE0) {
					nBytes = 3;
				}
				else if (chr >= 0xC0) {
					nBytes = 2;
				}
				else {
					return false;
				}
				nBytes--;
			}
		}
		else {
			//多字节符的非首字节,应为 10xxxxxx
			if ((chr & 0xC0) != 0x80) {
				return false;
			}
			//减到为零为止
			nBytes--;
		}
	}
	//违返UTF8编码规则
	if (nBytes != 0) {
		return false;
	}
	if (bAllAscii) { //如果全部都是ASCII, 也是UTF8
		return true;
	}
	return true;
}
```
![image](/img/20230117/4.3.png) 


## Fixed
![image](/img/20230117/4.4.png) 
