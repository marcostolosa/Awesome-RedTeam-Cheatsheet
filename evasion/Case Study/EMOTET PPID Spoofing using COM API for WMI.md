### Emotet Malware 





**EMOTET PPID Spoofing using WMI:**


```powershell
PS > wmic /namespace:\\root\CIMV2 path Win32_Process call create "notepad.exe"
```
```json
{
   "ProcessId": 22396
   "ReturnValue": 0
}
```
**ReturnValue** is equal which means that the operation is completed successfully on PID 22396 (ProcessId).

![image](https://user-images.githubusercontent.com/75935486/153729571-33b13901-b82b-4307-95be-1ab6530fdeb0.png)






![image](https://user-images.githubusercontent.com/75935486/153729993-192b6fff-e24f-40fa-9756-0f1d2d14339c.png)
