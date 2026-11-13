# 🃏 nsi-cardBattle — Bataille à plusieurs joueurs

Projet NSI de **Thomas Deynoux** (Terminale) : simulation d'une partie de bataille à plusieurs joueurs, jouable dans le terminal ou dans le navigateur.

## Sommaire

1. [Principe général](#principe-général)
2. [Lancement](#lancement)
3. [Architecture du code](#architecture-du-code)
   - [Classe `cardDeck`](#classe-carddeck)
   - [Classe `playerDeck`](#classe-playerdeck)
   - [Fonction `valeur_carte`](#fonction-valeur_carte)
   - [Fonction `init_game`](#fonction-init_game)
   - [Fonction `in_game`](#fonction-in_game)
4. [Tests réalisés](#tests-réalisés)
5. [Difficultés rencontrées](#difficultés-rencontrées)
6. [Algorithme global (pseudo-code)](#algorithme-global-pseudo-code)

---

## Principe général

Le programme simule une partie de bataille à plusieurs joueurs. Il repose sur :

- **deux classes** : `cardDeck` et `playerDeck` ;
- **deux fonctions utilitaires** : `valeur_carte` et `init_game` ;
- **une fonction principale** : `in_game`, qui déroule la partie.

## Lancement

La fonction `cli_or_web` propose de jouer :

- dans le **terminal** (mode CLI) ;
- dans le **navigateur** (mode Web). Dans ce cas, le jeu est relancé via l'outil [`ttyd`](https://github.com/tsl0922/ttyd) avec l'option `--jeu`, ce qui démarre directement la partie sur le port `440`.

## Architecture du code

### Classe `cardDeck`

Construit le jeu de cartes. Elle prend en paramètre sa longueur, qui doit valoir **32** ou **52**.

**Construction du deck.** Pour chaque famille, on ajoute d'abord les cartes numérotées (de 2 à 10 pour un jeu de 52 cartes, de 7 à 10 pour un jeu de 32), puis les cartes spéciales : valet, dame, roi et as. On obtient une liste de la forme :

```
[nombres, cartes spéciales, ...]
```

répétée quatre fois. Une fois cette liste primaire constituée, on ajoute les familles : la méthode `set_families` découpe la liste en quatre quarts égaux et concatène à chaque carte du quart le texte « de cœur », « de carreau », « de trèfle » ou « de pique ». Les cartes prennent ainsi la forme « 7 de cœur » ou « roi de pique ». Le deck est enfin mélangé avec `random.shuffle`.

| Méthode | Rôle |
|---|---|
| `__init__` | Prend la longueur du jeu, construit la liste de base, appelle `add_royal` et `set_families`, puis mélange le deck |
| `add_royal` | Appelée par `__init__`, ajoute les cartes spéciales |
| `set_families` | Appelée par `__init__`, ajoute les familles |
| `get_deck` | Accesseur : renvoie la liste complète du jeu de cartes |

### Classe `playerDeck`

Représente la main d'un joueur. Elle prend en paramètre une liste de cartes.

| Méthode | Rôle |
|---|---|
| `get_player_deck` | Renvoie les cartes actuellement détenues par le joueur |
| `player_lose_cards(n)` | Retire les `n` premières cartes du paquet, les range dans la liste `lost_cards`, puis la renvoie |
| `player_win_cards(cartes)` | Ajoute à la fin du paquet une liste de cartes gagnées. Elle prend obligatoirement une **liste de cartes** (et non un nombre), car il faut connaître les cartes précises pour les ajouter |

### Fonction `valeur_carte`

Les cartes spéciales sont stockées sous forme de texte (« valet », « dame », « roi », « as »). Elles n'ont donc aucune valeur numérique et ne peuvent pas être comparées lors d'une bataille.

`valeur_carte` joue le rôle de dictionnaire de correspondance : elle prend le nom d'une carte, en extrait le premier mot, et renvoie sa valeur.

| Carte | Valeur |
|---|---|
| Valet | 11 |
| Dame | 12 |
| Roi | 13 |
| As | 14 |
| Carte numérotée | le nombre converti en entier |

### Fonction `init_game`

Initialise une partie en faisant collaborer les deux classes : `playerDeck` a besoin des cartes fournies par `cardDeck`, qui elle-même a besoin de la longueur du jeu. Elle prend en paramètres **la longueur du jeu** et **le nombre de joueurs**.

1. Elle crée un objet `game_deck` de la bonne longueur et récupère la liste de cartes avec `get_deck()`.
2. Elle crée un dictionnaire `players_decks` contenant un objet `playerDeck` vide par joueur. La clé du joueur `x` est `"x_deck"`, ce qui permet ensuite d'accéder facilement à ses cartes.
3. Elle distribue les cartes avec un index qui varie de façon cyclique de `0` au nombre de joueurs moins 1. Cela reproduit une vraie distribution : une carte au premier joueur, une au deuxième, et ainsi de suite jusqu'à épuisement du paquet.

### Fonction `in_game`

Cœur du jeu. Elle demande d'abord la longueur du deck et le nombre de joueurs, et vérifie que les paramètres sont valides :

- deck de 32 ou 52 cartes ;
- au moins 2 joueurs ;
- au plus la moitié du nombre de cartes.

Si ce n'est pas le cas, la saisie est redemandée. Elle initialise ensuite la partie avec `init_game`, puis définit la fonction interne `tirer_carte`, qui joue un tour.

#### Déroulement d'un tour (`tirer_carte`)

1. Tous les joueurs encore en lice tirent leur première carte. Les cartes tirées sont placées sur la table (`surplus_cards`).
2. On convertit chaque carte en valeur avec `valeur_carte` et on repère la valeur maximale.
3. Si un seul joueur possède cette valeur, il remporte le tour.
4. **En cas d'égalité**, seuls les joueurs à égalité rejouent, et leurs nouvelles cartes s'ajoutent à la table. Un joueur qui n'a plus de carte ne peut pas continuer l'égalité. S'il ne reste qu'un joueur capable de jouer, il gagne. Si aucun ne le peut, le gagnant est tiré au hasard parmi les joueurs à égalité.
5. Le gagnant récupère toutes les cartes de la table. Elles sont **mélangées** avant d'être ajoutées à son paquet, ce qui évite les parties infinies.
6. Enfin, les joueurs qui n'ont plus aucune carte sont éliminés.

#### Boucle principale

Les tours s'enchaînent tant qu'il reste plus d'un joueur, avec une pause d'une seconde entre chaque tour. Pour éviter une partie interminable, la limite est fixée à **30 tours** : au-delà, on annonce qu'il n'y a pas de gagnant. Si un seul joueur reste, il est déclaré vainqueur.

## Tests réalisés

Les tests ont été exécutés avec Python 3.

| Élément testé | Test effectué | Résultat |
|---|---|---|
| `cardDeck(32)` et `cardDeck(52)` | Le deck contient bien 32 puis 52 cartes, toutes différentes | ✅ Réussi |
| `set_families` | Chaque famille (cœur, carreau, trèfle, pique) contient 8 cartes (deck de 32) ou 13 cartes (deck de 52) | ✅ Réussi |
| Valeurs extrêmes | Plus petite carte : 7 (deck de 32) ou 2 (deck de 52) ; plus grande : 14 (as) | ✅ Réussi |
| `random.shuffle` | Deux decks créés successivement sont dans un ordre différent | ✅ Réussi |
| `valeur_carte` | « 7 de coeur » = 7, « 10 de pique » = 10, valet = 11, dame = 12, roi = 13, as = 14 | ✅ Réussi |
| `player_lose_cards(2)` | Renvoie bien les deux premières cartes et les retire du paquet | ✅ Réussi |
| `player_win_cards` | Les cartes gagnées sont ajoutées à la fin du paquet | ✅ Réussi |
| Paquet vide | Tirer une carte dans un paquet vide lève une `IndexError` : `in_game` doit donc éliminer les joueurs sans carte avant de les faire jouer | ✅ Comportement confirmé, corrigé |
| `init_game` | Distribution testée pour (52, 2), (52, 3), (52, 4), (32, 2), (32, 5) et (32, 16) : toutes les cartes sont distribuées une seule fois et l'écart entre deux joueurs est au maximum d'une carte | ✅ Réussi |
| Clés des joueurs | Les clés du dictionnaire sont bien `0_deck`, `1_deck`, `2_deck`… | ✅ Réussi |
| Saisies invalides | Les entrées « 40 » cartes, « abc » (texte au lieu d'un nombre) et « 1 » joueur sont refusées et la question est reposée. Le jeu ne démarre qu'avec des paramètres valides | ✅ Réussi |
| Limite des tours | Le nombre maximum de tours est de 30 | ✅ Réussi |

## Difficultés rencontrées

La principale difficulté résidait dans la conception de l'algorithme et dans la réflexion qu'elle impliquait : quelles classes créer, et pour répondre à quel besoin ?

Par souci d'optimisation et de minimalisme, j'ai choisi de **ne pas créer une classe par carte**, ce qui m'aurait obligé à gérer 52 objets distincts. Il a également fallu soigner la gestion des erreurs et renforcer la structure du code afin de prévenir les cas problématiques.

## Algorithme global (pseudo-code)

<details>
<summary>Afficher le pseudo-code des classes principales</summary>

```text
importation de tous les modules

-- CLASSE JEUX DE CARTES
classe jeux_de_cartes:
    definir init qui prend en compte la longueur du jeu de cartes
        tableau = vide
        si la longueur est de 52:
            on répète 4 fois:
                ajouter les nombres de 2 à 10 au tableau
                ajouter les cartes spéciales (valet, dame, roi)
        sinon si la longueur vaut 32:
            on répète 4 fois:
                ajouter les nombres de 7 à 10 au tableau
                ajouter les cartes spéciales
        sinon, retourner une erreur
        exécuter set_families()
        mélanger le jeu de cartes

    definir ajouter_speciales (aucun argument):
        ajouter au tableau valet, dame, roi, as

    definir appliquer_familles (aucun argument):
        créer un tableau familles qui contient coeur, carreau, trèfle, pique
        définir le quart sur la longueur de self.jeux_de_cartes divisé par 4
        répéter 4 fois:
            répéter pour la valeur du quart:
                position = tour_de_boucle_2 + tour_de_boucle_1 * valeur_quart
                on remplace tableau[position] par ce qu'il y a actuellement + famille[index]

    definir obtenir_jeux_de_cartes (aucun argument):
        retourner la liste du jeu de cartes

-- CLASSE CARTES DU JOUEUR
classe cartes_du_joueur:
    definir init qui prend en paramètre jeux_de_cartes:
        self.jeux_de_cartes = jeux_de_cartes

    definir obtenir_cartes_du_joueur (aucun paramètre):
        renvoyer jeux_de_cartes

    definir perdre_carte qui prend en paramètre nombre_cartes:
        initialiser le tableau tab_pertes vide
        répéter nombre_cartes fois:
            définir cartes_perdues égal à la suppression de la dernière carte de jeux_de_cartes
            on ajoute cartes_perdues à tab_pertes
        on renvoie tab_pertes

    definir gagner_carte qui prend en argument liste_cartes:
        agrandir jeux_de_cartes avec liste_cartes

-- FONCTION VALEUR_DES_CARTES
definir valeur_des_cartes qui prend en argument carte:
    initialiser la variable mot sur le premier mot de carte
    créer un tableau avec le poids des cartes spéciales:
        valet: 11, dame: 12, roi: 13, as: 14
    retourner le numéro correspondant

-- FONCTION INITIALISER_PARTIE
definir initialiser_partie:
    [...]
```

</details>

## Auteur

**Deynoux Thomas** — Terminale, spécialité NSI.
