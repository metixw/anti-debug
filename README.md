# anti debug tricks

just a collection of some undocumented anti debug tricks i've found while reversing and messing around with debuggers.

i'll add more stuff here whenever i find something interesting.

## 1. executable lock detection

found this one while doing some ttd analysis. it's not an actual anti debug feature or anything, just a weird file sharing behavior that can be used to detect x64dbg.

from what i've seen, this has been around for like 10 years and still hasn't been fixed in x64dbg.

i've used this in some of my anticheats before and never really saw anyone talking about it. it's nothing crazy tho, pretty easy to patch once you know what's going on.

basically, it tries to open its own executable with exclusive access. if something else is holding a conflicting handle to the file, `CreateFileA` fails with `ERROR_SHARING_VIOLATION`.

this doesn't necessarily mean a debugger is attached, but it can work as a detection trick in certain cases.

### code

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

more tricks soon.
