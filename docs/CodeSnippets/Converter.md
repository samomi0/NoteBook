---
title: 字符转换
---

## string & wstring

=== "C++"

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

    !!! example

        ~~~c++
        int main() {
            std::string original = "你好，世界！";

            std::cout << "Narrow to Wide to Narrow: " << 
            CharConverter::wstr_to_str(CharConverter::str_to_wstr(original)) << std::endl;

            return 0;
        }
        ~~~

## UTF & unicode

=== "C++"

    === "utf32 to utf8"

        ~~~c++
        #include <iostream>
        #include <string>
        #include <vector>
        #include <stdexcept>

        std::string utf32ToUtf8(char32_t utf32Char) {
            std::string utf8Char;
            if (utf32Char <= 0x7F) {
                utf8Char.push_back(static_cast<char>(utf32Char));
            } else if (utf32Char <= 0x7FF) {
                utf8Char.push_back(static_cast<char>((utf32Char >> 6) | 0xC0));
                utf8Char.push_back(static_cast<char>((utf32Char & 0x3F) | 0x80));
            } else if (utf32Char <= 0xFFFF) {
                utf8Char.push_back(static_cast<char>((utf32Char >> 12) | 0xE0));
                utf8Char.push_back(static_cast<char>(((utf32Char >> 6) & 0x3F) | 0x80));
                utf8Char.push_back(static_cast<char>((utf32Char & 0x3F) | 0x80));
            } else if (utf32Char <= 0x10FFFF) {
                utf8Char.push_back(static_cast<char>((utf32Char >> 18) | 0xF0));
                utf8Char.push_back(static_cast<char>(((utf32Char >> 12) & 0x3F) | 0x80));
                utf8Char.push_back(static_cast<char>(((utf32Char >> 6) & 0x3F) | 0x80));
                utf8Char.push_back(static_cast<char>((utf32Char & 0x3F) | 0x80));
            } else {
                throw std::runtime_error("Invalid UTF-32 character");
            }
            return utf8Char;
        }

        std::vector<std::string> toUtf8Sequence(const std::string& input) {
            std::vector<std::string> utf8Sequence;

            for (char c : input) {
                char32_t utf32Char = static_cast<unsigned char>(c);
                std::string utf8Char = utf32ToUtf8(utf32Char);
                utf8Sequence.push_back(utf8Char);
            }

            return utf8Sequence;
        }
        ~~~

    === "unicode to utf8"

        ~~~c++
        std::vector<std::string> toUtf8Sequence(const std::string &text) {
            std::vector<std::string> utf8Sequence;
            int num = text.size();
            int i = 0;
            while (i < num) {
                int size = 1;
                if (text[i] & 0x80) {
                char temp = text[i];
                temp <<= 1;
                do {
                    temp <<= 1;
                    ++size;
                } while (temp & 0x80);
                }
                std::string subWord = text.substr(i, size);
                utf8Sequence.push_back(subWord);

                i += size;
            }
            return utf8Sequence;
        }
        ~~~

    !!! example

            ~~~c++
            int main() {
                std::string input = "Hello, 你好, 世界!";

                std::vector<std::string> utf8Sequence = toUtf8Sequence(input);

                std::cout << "UTF-8 Character Sequence:" << std::endl;
                for (const std::string& utf8Char : utf8Sequence) {
                    printf("%X ", utf8Char.c_str());
                }
                printf("\n");

                return 0;
            }
            ~~~