### I - Introduction

User Interface Privilege Isolation (UIPI) is a feature in Windows aimed at reducing attacks such as Shatter Attacks, Code Injections... To do this, UIPI will prevent processes with low IL from talking to windows owned my medium/high integrity level processes.



### 2 - Disable UIPI

```powershell
sls 'uiAccess="true"' *.exe
```

![image](https://user-images.githubusercontent.com/75935486/155860927-e30bb28c-0d6c-45d9-8f19-af4e0cf4605e.png)
