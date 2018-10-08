---
layout: default
permalink: /fiches/windows/anti-debug-tech
title: Techniques d'anti-debug communes
tag: malwaretuto
category: post
---

# Common Anti-debug technics Cheat Sheet

Ref: http://antukh.com/blog/2015/01/19/malware-techniques-cheat-sheet/

## Breakpoints check

### 0xCC bytes
When set a breakpoint in a debugger, that modify the code inserting 0xCC byte.
Can be a xored value, but some anti-debugg check it too (i.e 0xCC ^ 0x99, 0xCC ^ 0x55…)

### Hardware breakpoints
Calls GetThreadContext / SetThreadContext API and check for presence of DR0…DR4 debug registers.

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
assembly'''
pushf                                                    ; Push flag on stack
mov dword [esp], 0x100                ; Set TrapFlag flag (0x100)
popf                                                      ; Restore flag register
assembly'''

### IsDebugged
API call IsDebuggerPresent() check for the 2nd byte of PEB. Return 0 if is not debugged.

### NtGlobalFlag
Check the offset offset 0x68/0xBC (x86/x64) in PEB is set to 0x70. This value is set when a process is created with a debugger.

[TBC]