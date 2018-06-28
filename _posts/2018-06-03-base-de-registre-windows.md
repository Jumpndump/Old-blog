---
layout: default
permalink: /fiches/windows/bases-de-registres
title: La base de registre de Windows 
tag: fichewindows
category: post
---

## La base de registre de Windows 

La base de registre Windows est une immense base de données contenant les fichiers de configuration nécessaires au système, au matériel, aux applications et aux utilisateurs.

A l'origine, Windows stockait ces informations dans les fichiers **system.ini** et **win.ini** (du temps de Windows 3.1... ). Ces deux fichiers ont ensuite été remplacés par **user.dat** et **system.dat** (on trouve même sur Windows Me le fichier **classes.dat**) pour les systèmes Windows 95/98. Mais depuis, les fichiers sont stockés dans <code>%SYSTEMROOT%\System32\Config</code> sous la forme de 6 ruches principales :

- HKU\DEFAULT
- HKLM\HARDWARE
- HKLM\SAM
- HKLM\SECURITY
- HKLM\SOFTWARE
- HKLM\SYSTEM

La première ruche contient les données de configuration utilisateurs. Chaque utilisateur possède un fichier **ntuser.dat** contenant la configuration de son profil, situé dans <code>%USERPROFILE%\Ntuser.dat</code>.

Les quatre autres ruches contiennent les informations suivantes :

|   **Ruche**  |                                                             **Description**|
|:--------:|-----------------------------------------------------------------------------------------------------------------------------------|
| HARDWARE | Contient les informations sur la configuration matérielle à chaque démarrage                                                        |
| SAM      | Contient les paramètres de configuration des programmes installés sur la machine                                                    |
| SECURITY | Contient les paramètres de sécurité locaux                                                                                          |
| SOFTWARE | Contient les paramètres de configuration des programmes installés sur la machine                                                    |
| SYSTEM   | Contient les paramètres de configuration du matériel, des services ainsi que pilotes de périphériques et le nom RAW de ces derniers |

Sur les systèmes Vista, 7, 8 et 2008 Server, le backup des ruches est effectué tous les 10 jours par la tâche **RegIdleBackup** et sont stockés dans <code>%SYSTEMROOT%\System32\Config\RegBack</code>.

Lorsqu'une clé de registre est supprimée, elle est désallouée. Certains outils, tel que YARU, sont capable de restaurer les clés de registre.

## Les clés de liste MRU

La base de registre peut se révéler très utile lors d'une investigation numérique. Il est par exemple possible de retracer l'ordre des activités effectuées sur les artefacts grâce aux listes MRU (Most Recently Used).

Ces listes peuvent contenir des informations tels que l'historique des derniers fichiers accédés ou encore les dernières saisies de l'utilisateur dans le navigateur ou dans la boîte de dialogue Exécuter.

Quelques clés de liste MRU utiles :
<font size="6">
  
| Liste MRU      | Description                                                                                                                                                                                                                                                                                                                                                                                                                               | Emplacement                                                                                                                                                                                                          |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OpenSaveMRU    | Liste les fichiers ayant été ouverts et sauvegardé via une fenêtre Windows, leur localisation et leur date et heure d'ouverture et de sauvegarde.<br><br>Particulièrement utile afin de connaître la date de début de compromission d'une machine via l'ouverture d'un fichier malveillant par exemple. A noter que les fichiers sont classés par extension, le dossier \"\*\" listant les fichiers les plus récents toutes extensions confondues. | **Sur Windows XP** :<br> NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\ Explorer\ComDlg32\OpenSaveMRU<br><br>**Sur Windows 7** :<br>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\ Explorer\ComDlg32\OpenSavePIDlMRU        |
| LastVisitedMRU | Liste les exécutables utilisés pour ouvrir et sauvegarder les fichiers listés dans OpenSaveMRU.<br><br>Il est possible de retrouver la localisation du dernier fichier à avoir été accédé par un exécutable donné.                                                                                                                                                                                                                              | **Sur Windows XP** :<br> NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\ LastVisitedMRU<br><br>**Sur Windows 7** :<br> NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\ LastVisitedPidlMRU |
| RunMRU         | Liste les commandes saisies via la fenêtre de dialogue Exécuter. Les lettres correspondent à l'ordre dans lequel les commandes ont été exécutées.                                                                                                                                                                                                                                                                                         | NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU                                                                                                                                                 |
| ACMRU          | Windows mémorise les termes recherchés par l'utilisateurs afin de pouvoir les proposer plus tard à l'utilisateur. ACMRU est l'abréviation de Auto Complete MRU. Les valeurs sont classées en fonction du type de recherche ayant été effectuée :<br><br>5001 : recherche Internet<br>5603 : nom ou nom partiel d'un document<br>5604 : mot contenu dans un fichier<br>5647 : ordinateurs, imprimantes ou personnes                                       | **Sur Windows XP** :<br>NTUSER.DAT\Software\Microsoft\Search Assistant\ACMru\                                                                                                                                               |
</font>
