# Anti-Debug Tricks

A collection of undocumented and lesser-known anti-debugging techniques for Windows, discovered through reverse engineering, experimentation, and debugging analysis.

This repository will be updated with additional tricks over time.

## Tricks

### 01 — Executable Lock Detection (ELD)

**Category:** File System / Anti-Debug  
**Platform:** Windows  
**Target:** x64dbg  
**Discovered by:** [@metixw](https://github.com/metixw)  
**Difficulty to bypass:** Easy

#### Description

An undocumented anti-debugging trick discovered through **Time Travel Debugging (TTD)** analysis.

This technique does not rely on traditional anti-debugging APIs such as `IsDebuggerPresent` or `CheckRemoteDebuggerPresent`.

Instead, it exploits file-sharing behavior observed when debugging an executable with **x64dbg**.

The issue has reportedly existed for over 10 years and remains unpatched in certain x64dbg configurations.

This trick has been used in private anti-cheat implementations without being publicly documented, although it can be bypassed relatively easily.

#### How It Works

1. Retrieve the current executable path using `GetModuleFileNameA`.
2. Attempt to open the executable using `CreateFileA` with exclusive sharing access.
3. Check whether the operation fails with `ERROR_SHARING_VIOLATION`.
4. Treat the sharing violation as a potential debugger indicator.

**Note:** This is a heuristic, not a reliable debugger detection mechanism. Other processes can cause sharing violations, and debugger behavior may vary by version and configuration.

#### Proof of Concept

```cpp
// made by metixw ( @metixw )
#include <windows.h>
#include <iostream>

bool walahi()
{
    char path[MAX_PATH];
    GetModuleFileNameA(NULL, path, MAX_PATH);
    HANDLE h = CreateFileA(path, GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    if (h == INVALID_HANDLE_VALUE) return GetLastError() == ERROR_SHARING_VIOLATION;
    CloseHandle(h);
    return false;
}

int main()
{
    while (true)
    {
        if ( walahi() )
        {
            std::cout << "detected walahi" << std::endl;
            system( "pause" );
            Sleep(1000);
            exit(0);
        }
        Sleep( 500 );
    }
    return 0;
}
```

#### Limitations

- Can produce false positives.
- Does not detect every debugger.
- Can be bypassed by patching the check or changing file-sharing behavior.
- Detection depends on how the debugger handles the executable file.

---

## Credits

**Research & Discovery:** metixw (@metixw)

## Disclaimer

This repository is intended for educational purposes, reverse engineering research, and anti-cheat development.

More techniques will be added over time.
