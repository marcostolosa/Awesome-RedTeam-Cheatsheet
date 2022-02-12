### Emotet Malware 

Emotet is a malware. Originally intended to steal banking information, its malware has since diversified. It was distributed mainly through phishing campaigns. The malware uses Spearphishing emails with malicious attachments sent to user, Users launch the malicious attachments delivered via spearphishing emails. WMI was used to execute powershell.exe


![image](https://user-images.githubusercontent.com/75935486/153730523-892b9b98-2699-48c9-a919-5bc6d6824673.png)




### **EMOTET PPID Spoofing using WMI:**

we can try do it on CLI with Poweshell but it is more detectable because the command will be logged.

```powershell
PS > wmic /namespace:\\root\CIMV2 path Win32_Process call create "notepad.exe"
```
```json
{
   "ProcessId": 23468
   "ReturnValue": 0
}
```
**ReturnValue** is equal which means that the operation is completed successfully on PID 22396 (ProcessId).

![image](https://user-images.githubusercontent.com/75935486/153729571-33b13901-b82b-4307-95be-1ab6530fdeb0.png)



### **Using COM API and WMI for PPID Spoofing**


![image](https://user-images.githubusercontent.com/75935486/153729993-192b6fff-e24f-40fa-9756-0f1d2d14339c.png)
