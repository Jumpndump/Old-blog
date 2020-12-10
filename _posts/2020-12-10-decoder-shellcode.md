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

# Décoder le code Powershell

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

```Python
>>> base64.b64decode(data).decode("UTF-16")
'$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("H4sIAAAAAAAAAK1W73PaOBP+HP4KfciM7SlQEnJp6E1mym/MC4TGJKHlGEbIMjERFkiyw[REEDACTED]"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

Nous obtenons un script Powershell qui semble contenir encore une couche d'encodage de base64.

```powershell
$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("H4sIAAAAAAAAAK1W73PaOBP+HP4KfciM7SlQEnJp6E1mym/MC4TGJKHlGEbIMjERFkiyw[REDATCED]")); IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```






![ProcessTree.png](/img/ProcessTree.png)
