# TP - Sortir du fichier unique

**Travail** : individuel

**Prérequis** : le TP de révision, et en particulier l'exercice 6 sur les listes chaînées. Vous allez reprendre ce code et le découper.

**Ce TP ne tient pas dans un seul fichier**, c'est tout son objet. Vous allez produire un dossier contenant `liste.h`, `liste.c`, `main.c`, un `Makefile` et un `.gitignore`.

**Un compilateur installé est nécessaire.** Un compilateur en ligne ne permet ni la compilation séparée ni `make`. Si l'installation a échoué en séance de révision, réglez-la avant de commencer : c'est le moment.

## Organisation du TP

| Partie | Travail |
| --- | --- |
| Exercice 0 | Le dossier de projet et le premier commit |
| Exercice 1 | Découper la liste chaînée en trois fichiers |
| Exercice 2 | Compiler en deux temps, à la main |
| Exercice 3 | Provoquer les trois erreurs classiques |
| Exercice 4 | Écrire un premier Makefile |
| Exercice 5 | La dépendance au fichier d'en-tête |
| Exercice 6 | Trouver une fuite sans aucun outil |
| Exercice 7 | Installer valgrind, et reprendre la même fuite |
| Exercice 8 | Ce que le compteur ne peut pas voir |
| Exercice 9 | Ajouter une fonction au module |

## Méthode de travail

Les trois règles du TP de révision restent valables : compilez souvent, compilez avec `-Wall -Wextra -Werror`, lisez les messages d'erreur.

Une quatrième s'y ajoute : **faites un commit à la fin de chaque exercice**. Pas un seul à la fin du TP. Un commit qui ne contient qu'une chose se relit ; un commit qui contient huit exercices ne sert à rien ! 

## Exercice 0 - Le dossier de projet et le premier commit

### Consigne

1. Créez un dossier `liste-chainee` et placez-vous dedans.
2. Initialisez un dépôt : `git init`.
3. Créez un fichier `.gitignore` contenant les lignes ci-dessous.
4. `git add .gitignore`, puis `git commit -m "Depart : .gitignore"`.
5. Question : pourquoi ne versionne-t-on jamais les fichiers `.o` ni l'exécutable ?

### Le `.gitignore`

```
*.o
demo
demo.exe
```

## Exercice 1 - Découper la liste chaînée en trois fichiers

| Fichier | Contenu |
| --- | --- |
| `liste.h` | La garde d'inclusion, le type `Maillon`, les déclarations des fonctions publiques |
| `liste.c` | Les définitions de ces fonctions, et rien d'autre |
| `main.c` | Le programme qui utilise le module |

### Consigne

1. Reprenez l'exercice 6 de `revisions.c` : le type `Maillon` et les cinq fonctions.
2. Répartissez-les dans les trois fichiers d'après le squelette ci-dessous.
3. Renommez les fonctions avec un préfixe `liste_` : `liste_inserer`, `liste_longueur`, `liste_contient`, `liste_afficher`, `liste_liberer`.
4. Aucune fonction ne doit porter `static` dans `liste.c` : elles sont toutes publiques ici.
5. Question A : pourquoi le type `Maillon` doit-il être dans le `.h` et non dans le `.c` ?
6. Question B : pourquoi `liste.c` inclut-il son propre `liste.h` ?

### Squelette de `liste.h`

```c
#ifndef ______
#define ______

#include <stdbool.h>

typedef struct Maillon {
    int valeur;
    struct Maillon *suivant;
} Maillon;

Maillon *liste_inserer(Maillon *tete, int valeur);
int      liste_longueur(const Maillon *tete);
bool     ______;      /* liste_contient */
void     ______;      /* liste_afficher */
void     ______;      /* liste_liberer  */

#endif /* ______ */
```

### Squelette de `liste.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include "______"

Maillon *liste_inserer(Maillon *tete, int valeur)
{
    /* le corps vient de revisions.c */
}

/* ... les quatre autres fonctions ... */
```

### Squelette de `main.c`

```c
#include <stdio.h>
#include "______"

int main(void)
{
    Maillon *liste = NULL;
    for (int i = 1; i <= 5; i++) liste = liste_inserer(liste, i * 10);

    printf("liste     : ");
    liste_afficher(liste);
    printf("longueur  : %d\n", liste_longueur(liste));
    printf("contient 30 : %s\n", liste_contient(liste, 30) ? "oui" : "non");

    liste_liberer(liste);
    printf("liberee\n");
    return 0;
}
```

*Commit (si en CLI) : `git add . && git commit -m "Decoupage en trois fichiers"`*

### Format attendu

| Question | Réponse |
| --- | --- |
| A |Car les fichiers .c ont besoin du type Maillon, si Maillon etait dans liste.c, main.c n'arriverai pas à compilé  |
| B | liste.c a besoin de Maillon qui vient du .h. |

## Exercice 2 - Compiler en deux temps, à la main

| Commande | Ce qu'elle fait |
| --- | --- |
| `gcc -Wall -Wextra -c main.c` | Compile `main.c` et produit `main.o`, sans lier |
| `gcc -o demo main.o liste.o` | Assemble les deux objets en un exécutable |

### Consigne

1. Compilez chaque `.c` séparément avec `-c`. Vérifiez que `main.o` et `liste.o` existent.
2. Liez-les en un exécutable `demo`. Lancez-le.
3. Question A : combien de fichiers `.o` obtenez-vous ? Y a-t-il un `liste.h.o` ? Pourquoi ?
4. Question B : la commande `gcc -Wall -Wextra -o demo main.c liste.c` fonctionne aussi. Qu'est-ce qu'elle fait de moins bien que les deux étapes ?

### Format attendu

Sortie du programme :

```
```

| Question | Réponse |
| --- | --- |
| A | Deux fichiers .o, non, car quand on lie les deux ça donne demo |
| B | Cette commande ne créer pas de fichiers individuelles liste.o et main.o |

## Exercice 3 - Provoquer les trois erreurs classiques

L'objectif n'est pas de réussir mais de **reconnaître un message**. Pour chaque cas, notez le message exact, puis rétablissez le code avant de passer au suivant.

### Consigne

**Cas 1, le fichier objet oublié.** Recompilez `main.c` seul, puis liez sans `liste.o` :

```bash
gcc -Wall -Wextra -c main.c
gcc -o demo main.o
```

**Cas 2, l'inclusion oubliée.** Retirez la ligne `#include "liste.h"` de `main.c`, puis compilez `main.c`.

**Cas 3, la garde d'inclusion oubliée.** Retirez `#ifndef`, `#define` et `#endif` de `liste.h`. Ajoutez au début de `main.c` une seconde ligne `#include "liste.h"`. Compilez.

### Questions

1. Lequel de ces trois messages vient de l'éditeur de liens et non du compilateur ? À quoi le voyez-vous ?
2. Dans le cas 2, le compilateur émet plusieurs messages. Lequel est le seul utile ?
3. Le cas 3 ne peut pas se produire dans un projet à trois fichiers aussi simple. À partir de quelle situation devient-il possible ?

### Format attendu

| Cas | Premier message exact | Compilation ou lien |
| --- | --- | --- |
| 1 | | |
| 2 | | |
| 3 | | |

| Question | Réponse |
| --- | --- |
| 1 | |
| 2 | |
| 3 | |

Rétablissez le code, vérifiez que tout compile, puis commit.

## Exercice 4 - Écrire un premier Makefile

**Attention** : les lignes de commande d'un Makefile commencent par une **tabulation**, jamais par des espaces. Configurez votre éditeur en conséquence, ou vérifiez avec `cat -A Makefile` : une tabulation s'affiche `^I`.

### Consigne

1. Écrivez le `Makefile` d'après le squelette.
2. Lancez `make clean` puis `make`. Observez les trois commandes exécutées.
3. Relancez `make` sans rien modifier. Notez ce qu'il affiche.
4. Question A : pourquoi la seconde exécution ne recompile-t-elle rien ?
5. Question B : remplacez la tabulation de la première commande par quatre espaces et relancez `make`. Notez le message.

### Squelette

```makefile
demo: main.o liste.o
→   gcc -Wall -Wextra -std=c11 -g -o demo main.o liste.o

main.o: ______ ______
→   gcc -Wall -Wextra -std=c11 -g -c main.c

liste.o: ______ ______
→   gcc -Wall -Wextra -std=c11 -g -c liste.c

clean:
→   rm -f main.o liste.o demo

.PHONY: clean
```

*Le caractère `→` représente une tabulation. Ne le tapez pas.*

### Format attendu

| Question | Réponse |
| --- | --- |
| A | |
| B (message exact) | |

## Exercice 5 - La dépendance au fichier d'en-tête

### Consigne

1. Vérifiez que le projet compile : `make clean && make`.
2. `touch liste.h`, puis `make`. Notez ce qui est recompilé.
3. Retirez `liste.h` des dépendances de `main.o` dans le Makefile : la ligne devient `main.o: main.c`.
4. `touch liste.h`, puis `make`. Notez ce qui est recompilé.
5. Question A : quelle différence observez-vous entre les étapes 2 et 4 ?
6. Question B : imaginez que vous ajoutiez un champ à la structure `Maillon` dans `liste.h`. Avec la version fautive du Makefile, que contient l'exécutable produit ?
7. Rétablissez la dépendance.

### Format attendu

| Étape | Ce que make recompile |
| --- | --- |
| 2, avec la dépendance | |
| 4, sans la dépendance | |

| Question | Réponse |
| --- | --- |
| A | |
| B | |

### Le Makefile complet

Une fois l'exercice compris, remplacez votre Makefile par cette version, qui sert jusqu'à la fin du module.

```makefile
CC     = gcc
CFLAGS = -Wall -Wextra -std=c11 -g
OBJ    = main.o liste.o

demo: $(OBJ)
→   $(CC) $(CFLAGS) -o $@ $^

%.o: %.c liste.h
→   $(CC) $(CFLAGS) -c $< -o $@

clean:
→   rm -f $(OBJ) demo

.PHONY: clean
```

Vérifiez qu'il produit exactement le même résultat que le précédent, puis commit.

## Exercice 6 - Trouver une fuite sans aucun outil

Vous n'avez pas encore d'outil de détection, et c'est voulu : dans cet exercice, vous en construisez un. Douze lignes de C ordinaire suffisent à savoir si votre programme rend toute la mémoire qu'il prend.

**L'idée** : `malloc` et `free` vont toujours par paires. Si on les enveloppe dans deux fonctions à nous, qui incrémentent et décrémentent un compteur, il suffit de regarder ce compteur en fin de programme. Zéro, tout est rendu. Autre chose, il manque autant de `free`.

### Consigne

1. Ajoutez le compteur à `liste.c` d'après le squelette ci-dessous, puis remplacez les appels à `malloc` et `free` du module par `suivi_malloc` et `suivi_free`.
2. Ajoutez à `liste.h` la déclaration de `liste_blocs_en_circulation`, et affichez le compteur dans `main.c` après la construction de la liste, puis après sa libération.
3. Compilez, lancez. Notez les deux valeurs.
4. Introduisez une fuite : construisez dans `main.c` une **seconde** liste de trois éléments et ne la libérez pas. Recompilez, relancez. Notez la valeur finale.
5. Question A : pourquoi le compteur, et les fonctions `suivi_malloc` et `suivi_free`, doivent-ils être `static` dans `liste.c` plutôt que déclarés dans `liste.h` ?
6. Question B : `suivi_malloc` n'incrémente que si `malloc` a réussi, et `suivi_free` ne décrémente que si le pointeur n'est pas `NULL`. Que se passerait-il si on retirait ces deux tests ?
7. Question C : le compteur vous dit **combien** de blocs manquent. Vous dit-il **où** ? Comment feriez-vous pour le retrouver avec ce seul outil ?
8. Corrigez la fuite, vérifiez que le compteur revient à 0, commit.

### Squelette, à ajouter en haut de `liste.c`

```c
/* --- compteur d'allocations, interne au module --- */

static int blocs = 0;

static void *suivi_malloc(size_t taille)
{
    void *p = malloc(taille);
    if (______) blocs++;
    return p;
}

static void suivi_free(void *p)
{
    if (______) { ______; free(p); }
}

int liste_blocs_en_circulation(void)
{
    return ______;
}
```

### À ajouter dans `liste.h`

```c
int liste_blocs_en_circulation(void);
```

### Format attendu

| Mesure | Sans la fuite | Avec la fuite |
| --- | --- | --- |
| Compteur après construction | | |
| Compteur après libération | | |

| Question | Réponse |
| --- | --- |
| A | |
| B | |
| C | |

## Exercice 7 - Installer valgrind, et reprendre la même fuite

Votre compteur vous dit qu'il manque trois `free`. Il ne vous dit pas lesquels. C'est exactement la limite qu'un outil dédié lève, et c'est le moment de l'installer.

### Installation

| Système | Commande |
| --- | --- |
| Debian, Ubuntu, WSL | `sudo apt install valgrind` |
| Fedora | `sudo dnf install valgrind` |
| Arch | `sudo pacman -S valgrind` |

Vérifiez avec `valgrind --version`.

**Si valgrind n'existe pas chez vous** (Windows natif, macOS sur puce Apple), n'essayez pas de le forcer : il n'y est pas porté. Utilisez à la place le désinfecteur d'adresses intégré au compilateur, qui donne la même information. Il ne s'installe pas, il s'active à la compilation :

```bash
gcc -Wall -Wextra -std=c11 -g -fsanitize=address -o demo main.c liste.c
./demo
```

Toutes les questions de l'exercice se répondent avec l'une ou l'autre version. La correspondance des deux sorties est donnée en fin d'exercice.

### Consigne

1. Vérifiez que votre `Makefile` contient bien `-g` dans `CFLAGS`. Sans cela, le rapport ne donnera aucun numéro de ligne, et ne servira à rien.
2. `make clean && make`, puis `valgrind --leak-check=full ./demo` sur la version **sans** fuite. Notez les deux dernières lignes du rapport.
3. Réintroduisez la fuite de l'exercice 6, recompilez, relancez valgrind.
4. Notez le nombre d'octets perdus, le nombre de blocs, et la pile d'appels indiquée.
5. Question A : la pile d'appels se lit de bas en haut. Quelle ligne de `main.c` y figure ? Est-ce celle où vous avez oublié le `free` ? Laquelle est-ce alors ?
6. Question B : votre compteur affichait `3`. Combien de blocs valgrind classe-t-il en `definitely lost`, et combien en `indirectly lost` ? Pourquoi cette répartition sur une liste chaînée ?
7. Question C : `total heap usage: N allocs, M frees`. Que valent N et M, et pourquoi N n'est-il pas égal au nombre de maillons que vous avez créés ?
8. Corrigez la fuite, vérifiez que valgrind ne signale plus rien, commit.

### Correspondance des deux outils

| valgrind | Désinfecteur d'adresses |
| --- | --- |
| `definitely lost` | `Direct leak` |
| `indirectly lost` | `Indirect leak` |
| `--leak-check=full` | Activé d'office à la fin du programme |
| Code de sortie inchangé | Code de sortie 1 en cas de fuite |

### Format attendu

| Mesure | Sans la fuite | Avec la fuite |
| --- | --- | --- |
| `definitely lost` | | |
| `indirectly lost` | | |
| `total heap usage` | | |
| Votre compteur | | |

| Question | Réponse |
| --- | --- |
| A | |
| B | |
| C | |

## Exercice 8 - Ce que le compteur ne peut pas voir

Votre compteur et valgrind ont donné la même réponse à l'exercice précédent. Voici un cas où ils ne la donnent pas.

### Consigne

1. Dans un fichier séparé `deborde.c`, écrivez le programme ci-dessous.
2. Compilez-le avec `-Wall -Wextra -g` et lancez-le. Signale-t-il quelque chose d'anormal ? Quel est son code de sortie (`echo $?` sous Linux et macOS, `echo %ERRORLEVEL%` sous Windows) ?
3. Question A : ce programme fait-il un `malloc` de plus qu'un `free` ? Votre compteur de l'exercice 6 aurait-il détecté quoi que ce soit ?
4. Lancez-le sous valgrind, ou compilez-le avec `-fsanitize=address` et lancez-le. Notez le message.
5. Question B : le message dit `0 bytes after a block of size 20`. D'où vient le 20, et d'où vient le 0 ? Que dirait-il pour `t[6]` ?
6. Question C : le programme est faux et il affiche pourtant le résultat attendu. Quelle conclusion en tirez-vous sur la valeur d'un test qui « passe » ?
7. Question D : au vu des exercices 6, 7 et 8, dans quels cas gardez-vous le compteur, et dans quels cas allez-vous chercher valgrind ?

### Le programme

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *t = malloc(5 * sizeof(int));
    if (t == NULL) return 1;
    t[5] = 42;                    /* une case trop loin */
    printf("%d\n", t[5]);
    free(t);
    return 0;
}
```

### Format attendu

| Observation | Réponse |
| --- | --- |
| Sortie affichée, sans outil | |
| Code de sortie, sans outil | |
| Message de l'outil | |

| Question | Réponse |
| --- | --- |
| A | |
| B | |
| C | |
| D | |

## Exercice 9 - Ajouter une fonction au module

### Consigne

1. Ajoutez au module une fonction qui donne la plus grande valeur de la liste.
2. La difficulté est le cas de la liste vide : il n'y a pas de maximum. La signature proposée renvoie `false` dans ce cas et ne touche pas à `*resultat`.
3. Déclarez-la dans `liste.h`, définissez-la dans `liste.c`, appelez-la dans `main.c`.
4. Testez sur la liste de cinq éléments et sur `NULL`.
5. Vérifiez une dernière fois que le compteur vaut 0 et que valgrind ne signale rien.
6. Question A : combien de fichiers avez-vous modifiés ? Lesquels `make` a-t-il recompilés ?
7. Question B : pourquoi ne pas simplement renvoyer `-1` quand la liste est vide ?

### Squelette

```c
/* dans liste.h */
bool liste_maximum(const Maillon *tete, int *resultat);

/* dans liste.c */
bool liste_maximum(const Maillon *tete, int *resultat)
{
    if (______) return false;          /* liste vide */

    int max = ______;                  /* le premier element */
    for (const Maillon *m = ______; m != NULL; m = m->suivant)
        if (______) max = m->valeur;

    *resultat = max;
    return true;
}
```

### Format attendu

Sortie du programme :

```
```

| Question | Réponse |
| --- | --- |
| A | |
| B | |

*Commit final : `git add . && git commit -m "Ajout de liste_maximum"`. Puis `git log --oneline` : vous devez avoir un commit par exercice.*


## Pour aller plus loin

1. **Deux modules.** Sortez le banc de test de `main.c` dans un module `test.h` / `test.c` qui compte les réussites et les échecs. Le projet passe à cinq fichiers ; ajoutez `test.o` à `OBJ`. Vérifiez que `make` gère le nouveau fichier sans autre modification.
2. **Cacher les détails.** Ajoutez à `liste.c` une fonction interne `static Maillon *dernier(Maillon *tete)` et utilisez-la dans une nouvelle fonction publique `liste_inserer_en_queue`. Essayez d'appeler `dernier` depuis `main.c` : notez le message. C'est ce que `static` apporte.
3. **La cible test.** Ajoutez au Makefile une cible `test` qui construit puis exécute le programme sous valgrind. N'oubliez pas de la déclarer dans `.PHONY`. Faites-la échouer quand valgrind trouve quelque chose : par défaut il renvoie 0 quoi qu'il arrive, et `--error-exitcode=1` change cela.
4. **Un compteur qui dit où.** Faites afficher par `suivi_malloc` la taille demandée et une étiquette passée en paramètre supplémentaire. Vous vous rapprochez de ce que fait valgrind, et vous mesurez le travail que l'outil vous épargne.
