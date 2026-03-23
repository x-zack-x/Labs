# Lab Name — WebStrike

**Platform:** CyberDefenders  
**Category:** Network Forensics  
  

---

## Scenario
A suspicious file was identified on a company web server, raising alarms within the intranet. The Development team flagged the anomaly, suspecting potential malicious activity. To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.
Your task is to analyze the provided PCAP file to uncover how the file appeared and determine the extent of any unauthorized activity.
---

## Tools Used
- wireshark
- iplocation.net
---

## Walkthrough

### Q1 — Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?


**What I did:** La première chose à faire est d'ouvrir le fichier PCAP dans Wireshark, car ce fichier contient l'ensemble du trafic réseau, autrement dit la chronologie complète des activités de l'attaquant.
**What I found:** Logiquement, dans une attaque, c'est l'attaquant qui est à l'origine du premier paquet SYN. Son adresse IP est donc **117.11.88.124**. En la recherchant sur iplocation.net, j'ai pu déterminer sa localisation exacte ainsi que des informations supplémentaires.
**Answer:** Tianjin

---

### Q2 — Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?


**What I did:** Le User-Agent identifie le navigateur utilisé par une machine. Connaître cette information est utile pour affiner les règles de blocage. J'ai filtré le trafic dans Wireshark avec le filtre **http** afin d'isoler uniquement les requêtes HTTP et localiser le User-Agent.

**What I found:** En inspectant un paquet HTTP, j'ai analysé les différentes couches du modèle réseau. La couche application, utilisant le protocole HTTP, contenait plusieurs informations sur la requête, notamment le champ User-Agent.
**Answer:** Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

---

### Q3 — We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?

**What I did:** Si l'attaquant a réussi à accéder à la machine, il a probablement utilisé un upload malveillant via une méthode POST pour déposer un web shell. J'ai donc filtré le trafic avec **http.request.method == "POST"** pour isoler ces requêtes.
 
**What I found:** Parmi les requêtes POST, deux provenaient de l'attaquant et une de la victime, ce qui suggère un upload réussi suivi d'une communication sortante. Les paquets étaient fragmentés en raison de la segmentation TCP. J'ai utilisé **Follow TCP Stream** pour reconstituer l'échange complet. Le contenu révèle un script PHP de type reverse shell, placé dans un fichier nommé **image.jpg.php** avec une double extension pour tromper les utilisateurs Windows, qui masquent par défaut les extensions de fichiers. La troisième requête confirme le succès de l'upload.
**Answer:** image.jpg.php

---

### Q4 — Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?

**What I did:** Une fois le fichier uploadé, l'attaquant doit l'exécuter via une requête GET. J'ai utilisé le filtre Wireshark suivant pour localiser cette requête : **http.request.method == "GET" && http.request.uri contains "image.jpg.php".
**What I found:** La requête correspondante est apparue, révélant le chemin complet du répertoire où le fichier malveillant a été stocké.
**Answer:** /reviews/uploads/

---

### Q5 — Which port, opened on the attacker's machine, was targeted by the malicious web shell for establishing unauthorized outbound communication?

**What I did:** Pour identifier le port d'écoute de l'attaquant, j'ai analysé le contenu du script PHP extrait via Follow TCP Stream.
**What I found:** En lisant le script, on distingue clairement l'adresse IP de l'attaquant ainsi que le port cible sur lequel la connexion sortante est établie.
**Answer:** 8080

---

### Q6 — Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?

**What I did:** Pour identifier le fichier ciblé par l'attaquant, j'ai filtré les requêtes HTTP dont la source est l'adresse IP de la victime, ce qui correspond au trafic généré par le web shell exécuté côté serveur.
**What I found:** Le filtrage a mis en évidence une requête HTTP contenant le fichier que l'attaquant tentait d'exfiltrer.
**Answer:** /etc/passwd


## Incident Timeline
- t=26.922s  : Première tentative d'upload du web shell
- t=49.758s  : Deuxième tentative d'upload du fichier malveillant
- t=57.538s  : Navigation vers /admin/uploads à la recherche du fichier uploadé
- t=63.058s  : Navigation vers /uploads
- t=69.755s  : Navigation vers /admin/
- t=75.201s  : Navigation vers /reviews/uploads
- t=75.207s  : Navigation vers /reviews/uploads/
- t=84.150s  : Exécution du web shell via GET /reviews/uploads/image.jpg.php
- t=191.372s : Tentative d'exfiltration de /etc/passwd — échouée car le serveur de l'attaquant ne supporte pas la méthode POST

## What I Learned
- Les champs d'upload doivent valider strictement le type et l'extension des fichiers côté serveur.
- Un attaquant doit s'assurer que son infrastructure supporte les méthodes HTTP nécessaires avant de lancer une attaque.
- Les filtres Wireshark permettent d'optimiser considérablement l'analyse du trafic réseau.
- Une règle de pare-feu limitant la taille des réponses HTTP sortantes peut prévenir l'exfiltration de fichiers sensibles comme /etc/passwd.
