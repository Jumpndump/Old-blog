---
layout: default
permalink: /malwarelab/tuto/anti-debug-tech
title: Techniques d'anti-debug communes
tag: tuto_malware
category: post
---

La plupart des malwares implémentent des mécanismes d'anti-debug. Cela à pour conséquence de crasher le malware lorsque celui-ci détecte qu'il est en train d'être analysé.

Il existe plusieurs techniques d'anti-debug, mais les plus communes et classiques sont les suivantes.

## Vérification de la présence de breakpoints
Lorsque l'on positionne un breakpoint dans le débuggeur, on insert dans le code une instruction spécifique au breakpoint. Cette instruction lève une interruption et le CPU se met en pause (en réalité, il sauvegarde son état (adresse, mémoire...) puis s'arrête, mais ne termine pas le programme).

### Breakpoint logiciel : int 3
Il s'agit tout simplement de l'instruction correspondant au breakpoint (0xCC en hexadécimal). On peut la rencontrer aussi sous forme d'une valeur xorée (0xCC ^ 0x99, 0xCC ^ 0x55…) afin d'éviter d'être repérée.

### Breakpoint matériel (hardware)
Le but ici est de vérifier la présence des registres de débug DR0...DR4. Pour cela, soit on appelle les API GetThreadContext / SetThreadContext, soit on lève une exception (par exemple avec un xor eax, eax / div eax).

### Guard pages
Creation of PAGE_GUARD mem page and access it to causes access violation. If STATUS_GUARD_PAGE_VIOLATION occurs there is no debug. Imitation of debugger behavior.

## API calls

### IsDebuggerPresent / CheckRemoteDebuggerPresent
Checks a specific flag in PEB and return non-zero if the process is being debugged.

### FindWindows
Used to detect specific debuggers. For OllyDbg window class is named “OLLYDBG”. Other debuggers classes checks include “WinDbgFrameClass”, “ID”, “Zeta Debugger”, “Rock Debugger” and “ObsidianGUI”.

### NtQueryObject
Queries for various object information. Check if object type is a “debug object”.

### NtQuerySystemInformation (ZwQuerySystemInformation)
Checks if debug object handle exists and returns true if it’s the case.

### NtSetInformationThread (ZwSetInformationThread)
Checks if class HideThreadFromDebugger has been passed as an argument. This class prevent the debugger from receiving events (breakpoints, exiting the program, …) from any thread that has this API called on it.

### NtContinue
Used to modify current context or load a new one in the current thread, which can confuse debugger.

### CloseHandle / NtClose
Calling of ZwClose with invalid handle generates STATUS_INVALID_HANDLE exception when the process is debugged.

### GenerateConsoleCtrlEvent
Invokes a Ctrl+C signal. If EXCEPTION_CTL_C exception is raised and is true, that’s mean the process is being debugged.

### OutputDebugString
Sends a string to the debugger to display it.

## Flags

### Trap flag
If this flag is set, instructions are executed in single-step mode (step-by-step) and raises SINGLE_STEP exception.

Ex:
```assembly
pushf                                                    ; Push flag on stack
mov dword [esp], 0x100                ; Set TrapFlag flag (0x100)
popf                                                      ; Restore flag register
```

### IsDebugged
API call IsDebuggerPresent() check for the 2nd byte of PEB. Return 0 if is not debugged.

### NtGlobalFlag
Check the offset offset 0x68/0xBC (x86/x64) in PEB is set to 0x70. This value is set when a process is created with a debugger.

### Heap Flags



Ref: http://antukh.com/blog/2015/01/19/malware-techniques-cheat-sheet/
