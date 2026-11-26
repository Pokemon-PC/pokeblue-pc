# Pokémon Rouge / Bleu — port PC natif par recompilation statique

Même méthode et mêmes outils que le port de Cristal
(`../pokecrystal/pc/PORTING_ROADMAP.md`) : la ROM reconstruite par ce
désassemblage est traduite en C (une fonction par banque), un runtime
Game Boy écrit à la main fait le matériel. Rien n'est modifié hors de `pc/`
(sauf `/build/` et `/dist/` dans `.gitignore`) ; `make compare` reste OK.

Un exécutable par version, comme HeartGold/SoulSilver : `pokered.exe`,
`pokeblue.exe`.

Particularités de la 1re génération :
- jeu monochrome (header CGB 0x00) : le runtime lui donne les couleurs que la
  Game Boy Color choisit pour lui (Rouge : décor rouge, sprites verts/rouges ;
  Bleu : décor bleu) ; `palette=green` (Game Boy d'origine) ou `palette=gray`
  dans `pokered.ini` ;
- MBC3 sans horloge, sauvegarde de 32 Kio (fichier `.sav` d'émulateur) ;
- 598 morceaux de code dans les textes (`text_asm`) trouvés automatiquement.

## Jalons

| Jalon | Contenu | État |
|---|---|---|
| Outils | rgbds 1.0.4, ROM Rouge et Bleu reconstruites identiques à celles du joueur (USA/Europe) | ✅ |
| Traducteur | 57 101 instructions, 31 banques, routine DMA en HRAM traduite | ✅ |
| Compilation | Windows (Clang + SDL2) | ✅ |
| Démarrage | copyright, intro, titre « Red Version », NEW GAME | ✅ (0 instruction interprétée) |
| Contrôle | `GB_DIFF=1` : 156 490 blocs, 0 écart avec l'interpréteur | ✅ |
| Jeu | Rouge : noms, chambre, maison, Bourg Palette, Chen, labo, Salamèche, combat contre le rival, sauvegarde, CONTINUE au même endroit ; Bleu : jusqu'à la chambre (couleurs bleues) | ✅ |
| Français | ROM Rouge/Bleue (France) traduites directement (`match_roots.py` : 6 416 fonctions sur 6 657 ; adresses relevées à l'exécution dans `pc/roots_*_fr_runtime.txt`). Un exécutable par version, anglais + français, choix par la ROM (`--lang`, `rom_fr=` dans l'ini). Rouge FR : Salamèche, combat rival, SAUVER, CONTINUER ✅ ; Bleue FR : jusqu'à la chambre ✅ | ✅ |
| Paquet, Linux | `pc/tools/package.py`, même CMake que Cristal | — |

## Commandes

```
PATH=/d/LUC/pokemonall/tools/rgbds-1.0.4:$PATH mingw32-make -j8 red blue
python pc/tools/recomp/recomp.py --rom pokered.gbc --sym pokered.sym --src . \
       --out build/rc_red --name red --prefix red --lang en
cmake -S pc -B build/pc_red -G Ninja -DCMAKE_C_COMPILER=clang -DRC_DIRS=build/rc_red \
      -DGAME_NAME=pokered -DSDL2_PREBUILT_DIR=<SDL2 statique>
cmake --build build/pc_red
build/pc_red/pokered.exe "<ROM Rouge>.gb"
```
(idem `blue`.)

## Journal

- 2026-11-25 : parcours scripté de Rouge jusqu'à la sauvegarde/rechargement ; Bleu jusqu'à la chambre ; 0 instruction interprétée.
- 2026-11-25 : versions françaises (ROM du joueur, MBC5) ajoutées aux deux exécutables.
- 2026-11-25 (suite) : les ROM de l'utilisateur sont toutes dans un seul dossier
  (`D:/LUC/pokemonall/roms`, à plat). Le jeu les trouve seul : chemin explicite
  (argument, `rom_en=`/`rom_fr=`), sinon dossier du programme, sinon `rom_dir`
  du `.ini` (`--rom-dir`, `GB_ROM_DIR`) ; le fichier est choisi à son contenu
  (CRC de la ROM entière = version + langue), jamais à son nom. Testé : 10 cas
  EN/FR trouvés depuis `rom_dir` seul. Paquets Windows (.zip) et Linux
  (.tar.gz, glibc >= 2.34 grâce à `pc/cmake/glibc_compat.h` et
  `-DGB_PORTABLE_GLIBC=ON`) faits avec `pc/tools/package.py`.
- 2026-11-26 : tests de fin de jeu (anglais) avec `pc/tools/savedit.py` (édition de
  sauvegarde + checksum, `teleport1` en 1re gén.) et `play.py` (`poke`, `warp`) :
  Cristal, Or, Rouge, Jaune → Ligue Pokémon, Panthéon, générique, « THE END » ✅.
  Cristal : Pokédex, PC, salle de connexion (message hors ligne), échange avec Kyle,
  horloge jour/nuit ✅. Reste : mêmes tests en français, Bleu/Argent, autres systèmes.
- 2026-11-26 (suite) : fin de jeu en **français** validée pour Cristal, Or,
  Rouge, Jaune (Ligue → Panthéon → générique) ; Argent FR jusqu'à Marion ;
  Bleu EN : Ligue jusqu'au combat d'Olga. Adresses FR : identiques en 2e gén.,
  +5 octets en 1re gén. Nouveau : `GB_POKE_KEEP` / `keep`. Guide complet des
  tests et outils : `pc/GUIDE_TESTS.md`.
