# TP2 Cybersécurité
- Nom: Racym Derahmoune
- Matricule: 2393681
- Groupe: 1050

## Attaque (exploit)

Voici les étapes détaillées pour pouvoir modifier le courriel / téléphone sur la page demandée:

### Étape 1
j’ai commencé par utiliser nmap pour identifier les ports ouverts sur le serveur en entrant son adresse ip (192.168.0.15) et j’ai trouvé le port 22 ouvert, qui correspond au service ssh.

<img width="900" height="902" alt="image" src="https://github.com/user-attachments/assets/94bca67b-ee64-48ef-986b-796160d15573" />

### Étape 2
j’ai ensuite tenté de me connecter par ssh au serveur à partir du compte de bob, dont on a plusieurs informations. On sais que le mot de passe de son mail est Kim2012!, qui ne fonctionne pas pour ssh. Cependant, cela nous donne un bon indice car on connait maintenant le format de ses mot de passes qui est le nom d’un de ses enfants, l’année de naissance de ce dit enfant suivi d’un point d’exclamation. En essayant ce format avec les deux autres enfants de bob, j’ai réussi à établir une connexion ssh, le mot de passe étant « Sophie2014! ».

<img width="900" height="672" alt="image" src="https://github.com/user-attachments/assets/d25bdfb1-1f71-4177-b779-b2b053799988" />

### Étape 3
En faisant une commande sudo, (comme sudo systemctl status apache2 par exemple), on se rend compte que bob n’est pas un sudoer, il n’a donc pas de permissions administrateurs sur le serveur ce qui complique la tâche pour notre attaque. Cependant, en faisant un ls -l dans le dossier html, on peut voir que tous les fichiers ont des permissions 777, qui sont les permissions absolues à tout le monde. C’est une vulnérabilité à notre avantage car cela permet à bob de modifier le ficher concerné par l’attaque (contact.html) sans sudo.

<img width="900" height="227" alt="image" src="https://github.com/user-attachments/assets/fdcccaf9-944f-4527-ab96-6326fe019bfc" />

### Étape 4
nous pouvons donc facilement modifier le fichier contact.html avec un simple « nano contact.html » et en sauvegardant les modifications avec un « crtl+s ». 

#### Avant : 

<img width="900" height="684" alt="image" src="https://github.com/user-attachments/assets/876156ee-41c8-4346-be53-74fde9084b5b" />
<img width="900" height="337" alt="image" src="https://github.com/user-attachments/assets/55c3d45f-b430-425e-b917-c594d5da0cc5" />

#### Après :

<img width="900" height="511" alt="image" src="https://github.com/user-attachments/assets/44522e2c-d843-4a6b-ac8f-0ffc1f4bd0b6" />
<img width="900" height="367" alt="image" src="https://github.com/user-attachments/assets/97fd30bb-fb2e-4d1a-b7ba-23653b4e2a08" />

L'attaque a donc réussi.


# Correctif 1 - Connexion SSH
### Étape 1
Pour commencer, je vais boucher la première faille de sécurité que j’ai trouvé dans la partie 1 : le port ssh ouvert. Il y a deux manières de corriger cette faille, le bloquer complètement ou le restreindre à un réseau privé (ou à une adresse ip précise) à travers le firewall. Vu que notre machine windows est sur le même réseau, la seule solution est de bloquer complètement ce port, ce qui va complètement désactiver la fonctionnalité ssh. Nous allons commencer par voir le statut de notre firewall.. :

<img width="800" height="136" alt="image" src="https://github.com/user-attachments/assets/f6258b79-b9ef-4061-8d5a-f9677b315205" />

### Étape 2
Comme on peux le voir sur l’image ci-haut, le firewall est complètement inactif, ce qui représente une faille de sécurité majeure pour un serveur avec des ports ouverts à l’internet. Ceci étant dit, pour l’activer il faut tout d’abord définir des règles par défaut pour les ports que nous n’avont pas configurés. Je vais donc utiliser les commandes « sudo ufw default deny incoming » pour rejeter toute connexion entrante par défaut et « sudo ufw default allow outgoing » pour autoriser toutes les connextions sortantes que le serveur initie. 

<img width="900" height="198" alt="image" src="https://github.com/user-attachments/assets/e3feeef7-3c39-432c-84ae-f9fa6304cedb" />

### Étape 3
je vais ensuite activer le firewall, qui n’a aucun risque puisque toutes les connexions entrantes sont refusées par défaut avec sudo ufw enable.

<img width="878" height="106" alt="image" src="https://github.com/user-attachments/assets/d9974051-b172-4a39-8ec8-b4633b3b25bd" />


Je vais ensuite vérifier le statut du firewall avec sudo ufw status verbose:

<img width="900" height="182" alt="image" src="https://github.com/user-attachments/assets/610bc738-bbd7-41aa-b4c3-47bf5f86ff76" />

### Étape 4
Je vais ensuite ajouter une  règle pour autoriser le traffic http (port 80) pour que le site puisse être accédé depuis l’extérieur avec la commande sudo ufw allow 80/tcp : 

<img width="728" height="152" alt="image" src="https://github.com/user-attachments/assets/dd3807fd-5386-4278-9c6d-31432649e563" />

Voici le nouvel état de notre firewall : 

<img width="900" height="368" alt="image" src="https://github.com/user-attachments/assets/57b78b61-6c53-41a7-9d0b-ac547da2c559" />

Je vais maintenant tenter de reproduire l’attaque en essayant de me connecter avec ssh depuis le poste windows : 

<img width="900" height="139" alt="image" src="https://github.com/user-attachments/assets/2f2eda23-21a4-42ae-8ead-093a4a37216c" />


Comme nous pouvons le voir, la connexion n’a pas pu s’effectuer, car le serveur n’a pas répondu à la demande. L’étape 2 du notre hack ne peux plus donc se faire. D’un autre côté, le site web fonctionne toujours : 

<img width="900" height="362" alt="image" src="https://github.com/user-attachments/assets/1678305e-6e1d-402e-a786-156706e35cfd" />

La faille de sécurité est donc bouchée.


# Correctif 2 - Permissions de fichiers

### Étape 1
nous allons maintenant nous occuper de permisisons de fichiers. Je vais temporairement autoriser les connexions ssh pour pouvoir tester le fix. 
<img width="756" height="177" alt="image" src="https://github.com/user-attachments/assets/bd20f63b-1ed7-475f-a6d3-9051b29c806c" />


Lorsequ’on regarde les permissions de fichiers dans /var/www/html avec ls -l, on se rend compte que le dossier et les fichiers ont des droits absolus (777).

<img width="900" height="328" alt="image" src="https://github.com/user-attachments/assets/08eb7f48-e257-4683-9db9-475e6f66a241" />


Ceci représente une faille de sécutiré majeure, car il ne faut jamais utiliser 777 comme permissions de fichiers, cela laisse la porte grande ouverte à tout le monde. Ce que l’on veut pour un site, c’est uniquement pouvoir visionner les fichiers, mais laisser l’administrateur la possiblilité de les modifier quand-même. Nous allons donc utiliser une commande récursive sur le dossier html pour retirer les droits d’écriture pour les groupes et les autres utilisateurs mais laisser les permissions absolues pour l’administateur avec la commande « sudo chmod -R 744 » pour le dosser html pour modifier tous les fichiers d’un coup, puis sudo chmod 755 (ajouer les droits d’exécution) sur le dossier html pour qu’apache puisse accéder au dossier.


<img width="900" height="225" alt="image" src="https://github.com/user-attachments/assets/0c93d8b6-759b-4a36-b92e-e98c070526c3" />
<img width="900" height="174" alt="image" src="https://github.com/user-attachments/assets/36a0aff4-923d-4cd2-ae57-d1c5514e31e1" />

### Étape 2
La faille de sécurité est maintenant bouchée, nous allons donc tenter de modifier le fichier par ssh depuis notre poste, comme à l’étape 4 de notre hack en faisant la commande « nano /var/www/html/index.html » depuis notre poste windows.

<img width="900" height="472" alt="image" src="https://github.com/user-attachments/assets/3b889aed-e939-400b-b77f-34b5570ff7fa" />


Comme nous pouvons le voir écrit en rouge, le fichier n’est pas modifiable par bob, qui n’est pas un sudoer. La faille est donc bouchée, et le site fonctionne toujours : 


<img width="900" height="376" alt="image" src="https://github.com/user-attachments/assets/3104505e-9e2b-4cf1-b9f0-dadb56705d65" />

# Correctif 3
### Étape 1
Nous allons maintenant fixer le mot de passe de bob. Son mot de passe se devine facilement car il utiise une « recette » qui est basée sur des informations personnelles qu’il expose sur internet. Une « recette » est un patterne que quelqu’un utilise que l’on peut utiliser si on le connaît pour deviner d’autres mots de passes. Cette recette pour le cas de bob est le nom d’une de ses filles avec majuscule, son année de naissance suivi d’un point d’exclamation. Cela nous a permis de facilement deviner le mot de passe de la session de bob car on connaît sa recette, avec le mot de passe de son mail qui a leak. Pour fixer cette faille, bob doit modifier son mot de passe en utilisant aucun paterne, idéalement des mots choisis aléatoirements avec des chiffres et des symboles. J’ai choisi comme exemple « Patate*Louvre&guitatre » comme nouveau mot de passe. Je vais donc utiliser la commande « sudo passwd bob » pour changer le mot de passe :


<img width="764" height="339" alt="image" src="https://github.com/user-attachments/assets/737362b7-4e25-450f-bf98-d7b3b43de35c" />

### Étape 2 
Le mot de passe étant changé, l’étape 2 de notre hack ne peut plus fonctionner, car on ne peux plus se connecter par ssh. Lorsque l’on tente de se connecter par ssh avec l’ancien mot de passe (Sophie2014!), la connexion est refusée : 

<img width="647" height="145" alt="image" src="https://github.com/user-attachments/assets/473b47f7-12ec-4561-a1f0-24354028c93f" />


#### Le site demeure fonctionnel : 


<img width="900" height="404" alt="image" src="https://github.com/user-attachments/assets/2b90df90-b447-4a47-a09d-56b460389a8a" />

### Étape 3 
La faille de sécurité est maintenant bouchée. J’ai pris le soin de supprimer la règle ufw autorisant le traffic ssh.

<img width="659" height="184" alt="image" src="https://github.com/user-attachments/assets/a71fe336-4344-47a3-8e5e-b8cfdaea1aeb" />

<img width="667" height="178" alt="image" src="https://github.com/user-attachments/assets/3db54dac-0144-43e8-aacb-97ecb2f951aa" />

<img width="900" height="312" alt="image" src="https://github.com/user-attachments/assets/422b218b-fb36-41a9-b31f-843d219cb1be" />




