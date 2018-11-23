---
layout: default
permalink: /malwarelab/tuto/anti-debug-tech
title: Techniques d'anti-debug communes
tag: tuto_malware
category: post
---

## Techniques d'anti-debugg les plus communes (Cheat Sheet)
La plupart des malwares implémentent des mécanismes d'anti-debug. Cela à pour conséquence de crasher le malware lorsque celui-ci détecte qu'il est en train d'être analysé.

Il existe plusieurs techniques d'anti-debug, mais les plus communes et classiques sont les suivantes.

## Vérification de la présence de breakpoints
Lorsque l'on positionne un breakpoint dans le débuggeur, on insert dans le code une instruction spécifique au breakpoint. Cette instruction lève une interruption et le CPU se met en pause (en réalité, il sauvegarde son état (adresse, mémoire...) puis s'arrête, mais ne termine pas le programme).

### Breakpoint logiciel : int 3
Il s'agit tout simplement de l'instruction correspondant au breakpoint (0xCC en hexadécimal). On peut la rencontrer aussi sous forme d'une valeur xorée (0xCC ^ 0x99, 0xCC ^ 0x55…) afin d'éviter d'être repérée.

### Breakpoint matériel (hardware)
Le but ici est de vérifier la présence des registres de débug DR0...DR4. Pour cela, soit on appelle les API GetThreadContext / SetThreadContext, soit on lève une exception (par exemple avec un xor eax, eax / div eax) afin d'y accéder.

### Détection des "guard pages"
Une guard page est une zone mémoire non mappée, créée automatiquement lors d'opérations d'allocation. Elle permet de se prémunir des heap buffer overflow et se place entre la pile et le tas.
Lorsque qu'on essaie d'accéder à la guard page, une erreur STATUS_GUARD_PAGE_VIOLATION est levée et le processus s'arrête. Or sur une VM, l'erreur est ignorée.

## Obtention d'informations grâce aux appels de fonctions
Lorsqu'un processus est lancé en mode débug, des valeurs spécifiques sont attribuées à certains flags et/ou à certaines de ses propriétés. Les malwares utilisent très souvent des bibliothèques Windows afin d'obtenir ces informations afin de savoir s'ils sont dans un débugger.

### IsDebuggerPresent / CheckRemoteDebuggerPresent
Si le processus est lancé dans un débugger, un flag spécifique dans PEB est activé. Les API IsDebuggerPresent et CheckRemoteDebuggerPresent retournent une valeur différente de 0 si ce flag est activé.

### FindWindow
Cette API cherche si des débugger sont lancés en cherchant dans le noms des fenêtres lancés sur le système (par ex : “OLLYDBG”). Pour certains débuggers, la vérification est également faite sur d'autres éléments de la fenêtre (par ex : “WinDbgFrameClass”, “ID”, “Zeta Debugger”, “Rock Debugger” ou “ObsidianGUI”)

### NtQueryObject
Cette API requêtes des informations sur des objets. Parmi ces informations, un processus en cours de débug est de type "debug object".

### NtQuerySystemInformation (ZwQuerySystemInformation)
L'API vérifie si le le handle du debug object existe et retourne vrai si c'est le cas.

### NtSetInformationThread (ZwSetInformationThread)
L'API vérifie si la classe HideThreadFromDebugger a été passée en argument. Cette classe passe sous silence les évènements (breakpoints, exiting the program, …) des threads ayant invoqué cette API , au lieu de les envoyer au débugger.

### NtContinue
Afin de provoquer des comportements inatendus dans le debugger, NtContinue modifie le contexte ou en charge un différent dans le thread.

### CloseHandle / NtClose
En appelant ces API avec un handle invalide, l'exception STATUS_INVALID_HANDLE est levée si le processus est en cours de débugg.

### GenerateConsoleCtrlEvent
Envoie un Ctrl+C. Si le processus est en cours de debugg, l'exception EXCEPTION_CTL_C est alors levée et est vraie.

### OutputDebugString
Envoie une chaîne de caractères au débugger afin qu'il l'affiche.

## Présence de certains flags
Il existe certaines valeurs, octets, etc... notamment dans le PEB, permettant de vérifier si le processus est en cours de débug ou non.

### Trap flag
Lorsque le processus est débuggué en single-step-mode, l'exception SINGLE_STEP est levée et un flag que l'on appelle trap flag est activé.

Ex:
```
pushf                                                     ; Push flag on stack
mov dword [esp], 0x100                                    ; Set TrapFlag flag (0x100)
popf                                                      ; Restore flag register
```

### IsDebuggerPresent
L'API IsDebuggerPresent() vérifie l'état du 2nd octet du PEB et retourne 0 si le processus n'est pas en cours de débug.

### NtGlobalFlag
Vérifie si l'offset 0x68/0xBC (x86/x64) dans le PEB est à la valeur 0x70. Si cette valeur est présente, cela signifie que le processus a été créé lancé avec un débugger.

### Heap Flags
Il est également possible de vérifier la présence des flags "Flags" et "ForceFlags", situés dans le tas.

## Temps d'exacution des instructions
Dans un cycle d'horloge, une ou plusieurs instructions peuvent être exécutées par le processeur. Il est alors possible de vérifier si le processus est en cours de débug ou non en vérifiant le timing des instructions.

### GetTickCount, GetLocalTime, GetSystemTime, timeGetTime, NtQueryPerformanceCounter
Ces fonctions sont utilisées pour mesurer le temps d'exécution nécessaire pour certaines fonctions ou ensemble d'instructions. Le process est amené à quitter si le temps d'exécution dépasse le seuil qui a été fixé.

### Read Time Stamp Counter (rdtsc)
L'instruction rdtsc fait la même chose que les appels ci-dessus. Elle peut s'utiliser par exemple de la manière suivante :
```
rdtsc                       ; get current timestamp (saved in a 64 bit value: EDX [first half], EAX [second half])
xor ecx,ecx                 ; sets ECX to zero
add ecx,eax                 ; save timestamp to ECX
rdtsc                       ; get another timestamp
sub eax,ecx                 ; compute elapsed ticks
cmp eax,0FFF
jb short bintext.0041B652   ; jump if less than FFF ticks (assumes that program is not running under a debugging tool)
rdtsc
push eax
retn                        ; else, jump to bad location to make the program crash
```

Ref: http://antukh.com/blog/2015/01/19/malware-techniques-cheat-sheet/
