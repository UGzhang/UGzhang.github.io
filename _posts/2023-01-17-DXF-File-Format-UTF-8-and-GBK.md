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
// Determine the encoding format of the text
if (is_str_utf8(entity.text.data()))
	context = QString::fromUtf8(entity.text.c_str()); //utf-8
else
	context = QString::fromLocal8Bit(entity.text.c_str()); //gbk
```

```
// Check if the string is in UTF-8 encoding
bool is_str_utf8(const char* str)
{
	// UTF-8 can be encoded in 1 to 6 bytes, while ASCII uses one byte.
	unsigned int nBytes = 0; 
	unsigned char chr = *str;
	bool bAllAscii = true;
	for (unsigned int i = 0; str[i] != '\0'; ++i) {
		chr = *(str + i);
		// To determine if it's ASCII encoding, 
		// if not, it may be UTF-8. ASCII uses 7 bits for encoding, with the highest bit marked as 0 (0xxxxxxx)
		if (nBytes == 0 && (chr & 0x80) != 0) {
			bAllAscii = false;
		}
		if (nBytes == 0) {
			//If it's not ASCII code, it should be a multi-byte character. Calculate the number of bytes
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
			// Non-initial bytes of multi-byte characters should start with 10xxxxxx
			if ((chr & 0xC0) != 0x80) {
				return false;
			}
			// Keep subtracting until reaching zero.
			nBytes--;
		}
	}
	// Violates UTF-8 encoding rules
	if (nBytes != 0) {
		return false;
	}
	if (bAllAscii) { 
		//If it's all ASCII, it is also UTF-8
		return true;
	}
	return true;
}
```

## Fixed
![image](/img/20230117/4.4.png) 

## References
![cnblogs](https://www.cnblogs.com/Toney-01-22/p/993529.html)
