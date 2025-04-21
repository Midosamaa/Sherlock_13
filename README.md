# Sherlock 13

## Objectif

Ce projet consiste à implémenter une version **multijoueur réseau** du jeu de déduction *Sherlock 13*, en langage C.

Il met en œuvre plusieurs concepts OS: sockets, processus, threads, mutex, FSM, et rendu graphique via SDL2.

---

## Contenu du dépôt

```
bash
CopyEdit
.
├── img/              # Ressources graphiques (cartes, objets, boutons)
├── src/
│   ├── sh13.c        # Client SDL2
│   ├── server.c      # Serveur multijoueur
│   └── makefile      # Compilation client + serveur
├── regles_du_jeu.pdf # Règles officielles
└── README.md         # Ce fichier
```

---

## Compilation

### Préparation de l’environnement

Installez les bibliothèques nécessaires :

```bash
bash
CopyEdit
sudo apt install libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev
```

### Clone repository

Clonez le repository du projet:

```bash
git clone https://github.com/Midosamaa/Sherlock_13
```

### Compiler

Depuis le dossier `src` :

```bash
bash
CopyEdit
make
```

Deux exécutables sont générés :

- `sh13` : client SDL2
- `server` : serveur multijoueur

### Lancement d'une partie

1. Trouver l’adresse IP du serveur

```bash
ifconfig
```

1. **Lancer le serveur** sur un port (ex : 12345) :

```bash
bash
CopyEdit
./server 12345
```

1. Trouver l’adresse IP du client

```bash
ifconfig
```

1. **Lancer les clients** (dans différents terminaux ou machines) :

```bash
bash
CopyEdit
./sh13 <IP_SERVEUR> <PORT_SERVEUR> <IP_CLIENT> <PORT_CLIENT> <NomJoueur>
```

où <IP_SERVEUR> doit être remplacé par l’adresse IP du serveur trouvée précedemment

où :

- `<IP_SERVEUR>` : adresse IP du serveur (ex : `127.0.0.1` si en local)
- `<PORT_SERVEUR>` : port sur lequel le serveur écoute (ex : `2500`)
- `<IP_CLIENT>` : adresse IP locale du client (souvent `127.0.0.1` aussi si en local)
- `<PORT_CLIENT>` : port d’écoute du client (ex : `3000`, `3001`, etc.)
- `<NomJoueur>` : pseudo du joueur (ex : `Sherlock`, `Watson`...)

Exemple :

```bash
bash
CopyEdit
./sh13 127.0.0.1 2500 127.0.0.1 2501 Watson
```

---

## Règles du jeu (version courte)

- Il y a **13 personnages suspects**, chacun associé à **3 objets uniques** (pipe, couronne, collier...).
- Chaque joueur reçoit **3 cartes suspects**. Le **13e personnage est le coupable**, inconnu de tous.
- À son tour, un joueur peut :
    - **Demander à un joueur** combien de fois un objet apparaît dans ses cartes.
    - **Demander combien de fois un objet apparaît chez les autres joueurs.**
    - **Accuser** un personnage.
- Une **bonne accusation** remporte la partie. Une **mauvaise** élimine le joueur.
- Le but est de **déduire qui est le coupable** grâce aux objets visibles et aux questions posées.

Pour plus de détails aller voir le fichier des règles (`regles_du_jeu.pdf`)

---

## Construction du programme

### Côté serveur (`server.c`)

- Le serveur est une application **TCP** qui accepte jusqu’à **4 clients** en simultané.
- Chaque client connecté est associé à une **structure** contenant ses informations (nom, socket, cartes, etc.).
- Une **machine à états (FSM)** a été mise en place pour gérer les différentes phases du jeu :
    - **Attente des connexions**
    - **Distribution aléatoire des cartes**
    - **Gestion du tour de chaque joueur**
    - **Traitement des questions et des accusations**
    - **Détection de victoire ou défaite**
- La FSM permet une **logique claire et séquentielle** malgré l’aspect asynchrone des connexions.
- La logique du jeu (vérification des objets, validation des accusations) est **centralisée côté serveur** pour éviter toute triche ou désynchronisation.

### Ce que j’ai complété :

- Lecture des messages reçus et décodage des actions (type de question ou accusation)
- Gestion de l’envoi des bonnes réponses aux bons joueurs
- Détection du gagnant et notification à tous les clients
- Intégration des règles du jeu dans le code serveur

---

### Côté client (`sh13.c`)

- Le client est une **interface graphique** réalisée avec **SDL2**, affichant dynamiquement :
    - Les cartes visibles (images)
    - Les objets disponibles
    - Les boutons d’action (question ciblée, question globale, accusation)
    - Les messages du serveur (ex : "Le joueur X a posé une question", "Vous avez gagné", etc.)
- Pour gérer la communication réseau en parallèle de l'affichage, un **thread dédié** est lancé dès le début :
    - Ce thread écoute en permanence les messages du serveur
    - Lorsqu’un message est reçu, il est stocké dans une variable partagée
- Pour éviter les conflits entre le **thread réseau** et la **boucle SDL principale**, un **mutex** protège l’accès à cette variable.

### Ce que j’ai complété :

- Initialisation de la SDL (fenêtre, images, police, etc.)
- Création et gestion des boutons cliquables
- Thread de réception réseau avec synchronisation par mutex
- Mise à jour dynamique de l'affichage selon les messages serveur

---

## Concepts OS USER utilisés

| Concept OS USER | Implémentation concrète dans le projet |
| --- | --- |
| **Sockets TCP** | Serveur écoute via `socket`, `bind`, `listen` ; chaque client se connecte via `connect` pour envoyer ses actions et recevoir les réponses |
| **Processus (client/serveur)** | Le serveur fonctionne en **processus séparé**, lancé avant les clients, ce qui simule une architecture distribuée (comme dans un vrai jeu en réseau) |
| **Threads** | Côté client : un thread est lancé pour écouter en continu les messages du serveur sans bloquer l’interface graphique |
| **Mutex** | Synchronisation entre le thread réseau et la boucle SDL2, pour **éviter les accès concurrents** à des données partagées (ex : messages reçus, états du jeu) |
| **Machine à états (FSM)** | Implémentée côté serveur pour organiser la logique du jeu étape par étape, en évitant les bugs liés à l'ordre des actions |
| **Gestion mémoire** | Allocation dynamique de structures pour chaque joueur, libération propre en fin de partie |
