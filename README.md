# WIP-Listing-devices-win32-api

```
C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\shared
```
## TCC

```
-nostdinc    do not use standard system include paths
```		


```
C:\Users\Juozas\Desktop>tcc "-IC:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\shared" "-IC:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\um" -run test.c
In file included from test.c:1:
In file included from C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0/um/windows.h:167:
In file included from c:/users/juozas/desktop/latest-built/include/excpt.h:9:
c:/users/juozas/desktop/latest-built/include/_mingw.h:160: warning: WINAPI_FAMILY_PARTITION redefined
In file included from test.c:1:
In file included from C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0/um/windows.h:171:
In file included from C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0/shared/windef.h:24:
In file included from C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0/shared/minwindef.h:182:
C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0/um/winnt.h:986: error: '(' expected (got "{")
```

```
#include <windows.h>
#include <commctrl.h>
#include <winioctl.h>

typedef struct _STORAGE_DEVICE_NUMBER {
  DEVICE_TYPE  DeviceType;
  ULONG  DeviceNumber;
  ULONG  PartitionNumber;
} STORAGE_DEVICE_NUMBER, *PSTORAGE_DEVICE_NUMBER;

void PrintVolumes()
{
    char volName[MAX_PATH];
    HANDLE hFVol;
    DWORD bytes;

    hFVol = FindFirstVolume(volName, sizeof(volName));
    if (!hFVol)
    {
        printf("error...\n");
        return;
    }
    do
    {
        size_t len = strlen(volName);
        if (volName[len-1] == '\\')
        {
            volName[len-1] = 0;
            --len;
        }

        /* printf("OpenVol %s\n", volName); */
        HANDLE hVol = CreateFile(volName, 0, FILE_SHARE_READ|FILE_SHARE_WRITE, NULL, OPEN_EXISTING, 0, NULL);
        if (hVol == INVALID_HANDLE_VALUE)
            continue;

        STORAGE_DEVICE_NUMBER sdn = {0};
        if (!DeviceIoControl(hVol, IOCTL_STORAGE_GET_DEVICE_NUMBER, NULL,
                    0, &sdn, sizeof(sdn), &bytes, NULL))
        {
            printf("error...\n");
            continue;
        }
        CloseHandle(hVol);

        printf("Volume Type:%d, Device:%d, Partition:%d\n", (int)sdn.DeviceType, (int)sdn.DeviceNumber, (int)sdn.PartitionNumber);
        /* if (sdn.DeviceType == FILE_DEVICE_DISK)
            printf("\tIs a disk\n");
            */
    } while (FindNextVolume(hFVol, volName, sizeof(volName)));
    FindVolumeClose(hFVol);
}

INT WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
    PSTR lpCmdLine, INT nCmdShow)
{
	PrintVolumes();
    return 0;
}
```

### GCC 
```
C:\Users\Juozas\Desktop>gcc test.c
test.c: In function 'PrintVolumes':
test.c:20:9: warning: implicit declaration of function 'printf' [-Wimplicit-function-declaration]
         printf("error...\n");
         ^~~~~~
test.c:20:9: warning: incompatible implicit declaration of built-in function 'printf'
test.c:20:9: note: include '<stdio.h>' or provide a declaration of 'printf'
test.c:41:13: warning: incompatible implicit declaration of built-in function 'printf'
             printf("error...\n");
             ^~~~~~
test.c:41:13: note: include '<stdio.h>' or provide a declaration of 'printf'
test.c:46:9: warning: incompatible implicit declaration of built-in function 'printf'
         printf("Volume Type:%d, Device:%d, Partition:%d\n", (int)sdn.DeviceType, (int)sdn.DeviceNumber, (int)sdn.PartitionNumber);
         ^~~~~~
test.c:46:9: note: include '<stdio.h>' or provide a declaration of 'printf'

C:\Users\Juozas\Desktop>a
Volume Type:7, Device:0, Partition:1
Volume Type:7, Device:1, Partition:4
Volume Type:7, Device:1, Partition:5
Volume Type:7, Device:0, Partition:2
Volume Type:7, Device:0, Partition:3
Volume Type:7, Device:1, Partition:3
Volume Type:2, Device:0, Partition:-1
```
