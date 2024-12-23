---
title: 宽窄字符转换
date: 2024-07-14
---

宽窄字符转换

<!-- more -->

## code

~~~c++
#include <iostream>
#include <string>
#include <locale>
#include <codecvt>

class CharConverter {
public:
    CharConverter() = delete;
    ~CharConverter() = delete;

    static std::wstring str_to_wstr(const std::string& str) {
        return converter.from_bytes(str);
    }

    static std::string wstr_to_str(const std::wstring& wstr) {
        return converter.to_bytes(wstr);
    }

private:
    static std::wstring_convert<std::codecvt_utf8_utf16<wchar_t>> converter;
};

std::wstring_convert<std::codecvt_utf8_utf16<wchar_t>> CharConverter::converter;
~~~

## use

~~~c++
int main() {
    std::string original = "你好，世界！";

    std::cout << "Narrow to Wide to Narrow: " << 
    CharConverter::wstr_to_str(CharConverter::str_to_wstr(original)) << std::endl;

    return 0;
}
~~~