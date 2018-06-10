---
layout: default
permalink: /fiches/windows/bases-de-registres
title: La base de registre de Windows 
tag: fichewindows
category: post
---

## La base de registre de Windows 

La base de registre Windows est une immense base de données contenant les fichiers de configuration nécessaires au système, au matériel, aux applications et aux utilisateurs.

A l'origine, Windows stockait ces informations dans les fichiers system.ini et win.ini (du temps de Windows 3.1... ). Ces deux fichiers ont ensuite été remplacés par user.dat et system.dat (on trouve même sur Windows Me le fichier classes.dat) pour les systèmes Windows 95/98. Mais depuis, les fichiers sont stockés dans %SYSTEMROOT%\System32\Config sous la forme de 6 ruches principales :

- HKU\DEFAULT
- HKLM\HARDWARE
- HKLM\SAM
- HKLM\SECURITY
- HKLM\SOFTWARE
- HKLM\SYSTEM

La première ruche contient les données de configuration utilisateurs. Chaque utilisateur possède un fichier ntuser.dat contenant la configuration de son profil, situé dans %USERPROFILE%\Ntuser.dat.

Les quatre autres ruches contiennent les informations suivantes :