---
layout: default
permalink: /forensiclab/tuto/identifier-processus-malveillant-off
title: (DRAFT)Identifier un processus malveillant (offline)
tag: tuto_forensic
category: post
---

## (DRAFT)Identifier un processus malveillant (offline)

L'analyse d'un disque requiert parfois une analyse offline (contrainte d'accessibilité, disponibilité...). Dans ces cas là, il s'agit d'analyser le dump mémoire de la machine infectée afin d'identifier les processus malveillants à un instant t.

Pour effectuer l'analyse, nous utiliserons l'outil **volatility**.

## Mise en place

Afin de débuter l'analyse, volatility a besoin de connaître le profile système du dump mémoire. Pour se faire, nous allons utiliser le plugin imageinfo.
```bash
vol.py --f fichier.raw imageinfo
```

Plusieurs profils possibles sont alors proposés. Le bon est souvent le dernier. Il convient toutefois de vérifier avec le Service Pack detecté dans le résultat de la même commande.
Pour chaque plugin utilisé, il faudra donc évidement fournir à volatility le nom du dump et le profil système adéquat comme ceci :
```bash
vol.py --f fichier.raw --profil=Win7SP1x86 <plugin>
```

**Astuce :** il est assez rébarbatif de taper toute la commande à chaque utilisation d'un plugin. Il peut être alors pratique de créer un alias afin de n'avoir plus que le plugin et ses paramètres à taper.
```bash
alias vol="vol.py --f fichier.raw --profil=Win7SP1x86"
```

## Analyser les processus actifs

Il existe 4 plugins permettant de lister les processus actifs sur la machine au moment du dump.

### Pslist
Liste les processus en cours, les informations les concernant (PID, PPID, Threads, timestamp de création...).
![Pslist.png](/img/Pslist.png)
*<center>Source : https://www.oreilly.com/library/view/digital-forensics-and/9781787288683/369abaa6-8568-4fc4-8e6d-30b072e4520c.xhtml</center>*

### Pstree 
Liste les processus de la même manière que pslist, mais en arborescence.
![Pstree.png](/img/Pstree.png)
*<center>Source : https://www.oreilly.com/library/view/digital-forensics-with/9781788625005/dc17064f-3d4d-4c3f-97f1-66fda0589779.xhtml</center>*

### Psscan
Liste les processus en cours et les processus cachés (scan des structures EPROCESS blocks existantes, donc celles ayant été détachées de la liste également) ainsi que leur timestamp de création et de fin.
![Psscan.png](/img/Psscan.png)
*<center>Source : https://www.oreilly.com/library/view/digital-forensics-and/9781787288683/1e90beab-0052-48f2-835b-7f4bc9fc4810.xhtml</center>*

### Psxview
Liste les processus en cours et cachés (pslist + psscan)
![Psxview.png](/img/Psxview.png)
*<center>Source : https://www.oreilly.com/library/view/digital-forensics-with/9781788625005/4d0403d3-9cba-4a16-bbe4-0a3df437044c.xhtml</center>*

Si certains processus malveillants peuvent être triviaux à trouver (nom suspect, nom de processus légitime avec fautes d'orthographe, lancement inhabituel d'un terminal...), d'autres en revanche peuvent passer presque inaperçu si on ne connaît pas les spécificité du système.

## Une histoire de famille...

Nous allons nous intéresser à deux types de processus : ceux lancés par le **système** et ceux lancés par **userinit**. La plupart du temps, un processus malveillant va se faire passer par un processus système, ces derniers étant parfois obscures et non visibles pour les non initiés. Afin de les débusqués, un petit rappel des processus souvent usurpés :

![ProcessTree.png](/img/ProcessTree.png)

## Chercher l'intrus...

Les anomalies que l'on rencontre le plus fréquemment sont par exemple :

- un processus ayant un père anormal (exemple : iexplorer.exe ne devrait pas être spawné par un autre processus que explorer.exe)
- un processus présent en nombre anormal (exemple : le processus lsass.exe ne doit être présent qu'une seule fois)
- un processus caché (visible uniquement via le plugin psscan)
- un processus avec un temps de démarrage anormal (exemple : le processus lsm.exe est normalement lancé dans les secondes suivant le boot)

Il existe un plugin volatility permettant de détecter les anomalies de ce type : **malsysproc**.

Lorsqu'une anomalie est trouvée par malsysproc, le flag FALSE apparaît dans le tableau. Le risque de faux-positifs n'est pas nul, cependant, nous pouvons porter notre attention sur les processus identifiés par le plugin. Un processus malveillant aura souvent les colonnes Path, PPID, Time voir Cmdline en FALSE.

![Malsysproc.png](/img/volatility/malsysproc.png)

Il peut être également utile de vérifier si un processus a été lancé par un utilisateur ou le système avec le plugin getsid. Un malware aura généralement tendance a être exécuté avec les droits utilisateur.

\<screen getsid\>

**Rappel :**

## Les DLLs et Handles
Le processus malveillant identifié, nous allons maintenant chercher à le localiser sur le disque. Pour cela, le plugin dlllist est utilisé. Il permet de lister les dlls associées au processus dont nous venons d'identifier le PID avec leur localisation.
```bash
vol dlllist -p <PID>
```

\<screen dllist\>

On pourra également vérifier ses handles associés à ce processus grâce au plugin handles.
```bash
vol handles -p <PID>
```

Ce plugin permet de savoir dans quel(s) clé(s) de registre le processus malveillant s'est implanté.

## Les injections de code

[TBC]
