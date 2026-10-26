# get_next_line

*[English version](README.md)*

Lire un descripteur de fichier ligne par ligne, en C. Projet de l'école 19 (réseau 42).

```c
char *get_next_line(int fd);
```

Chaque appel renvoie la ligne suivante, retour à la ligne compris, ou `NULL` en fin de fichier ou en cas
d'erreur de lecture. C'est à l'appelant de libérer chaque ligne.

```c
int   fd = open("notes.txt", O_RDONLY);
char *line;

while ((line = get_next_line(fd)) != NULL)
{
    printf("%s", line);
    free(line);
}
close(fd);
```

## Fonctionnement

- `read` remplit un tampon de `BUFFER_SIZE` octets, fixé à la compilation (`-D BUFFER_SIZE=42`, 200 par
  défaut, plafonné à 1 000 000).
- Ce qui a été lu sans avoir encore été renvoyé est gardé d'un appel à l'autre dans une chaîne `static`.
  La lecture s'arrête dès que cette chaîne contient un `\n`.
- La fonction la coupe alors en deux : la ligne est renvoyée, le reste attend l'appel suivant. Quand il ne
  reste plus rien, la chaîne est libérée : un fichier lu jusqu'au bout ne laisse aucune allocation.
- Un seul descripteur à la fois : la version bonus, qui en suit plusieurs, n'en fait pas partie.

## Tests

La sortie d'un petit programme de lecture a été comparée octet par octet au fichier d'origine, avec
`BUFFER_SIZE` à 1, 42, 4 096 et 1 000 000, et AddressSanitizer activé :

| Entrée | Résultat |
|---|---|
| fichier vide, une longue ligne sans `\n`, des lignes avec et sans `\n` final | identique |
| 100 000 lignes courtes, une seule ligne de 27 Ko | identique |
| entrée standard (`fd` 0), un descripteur dont l'ouverture a échoué | lignes lues / `NULL` |

Aucune erreur mémoire ni fuite n'a été signalée.

## Compilation

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c
```

`main.c` lit les fichiers d'exemple du dossier `files/`.
