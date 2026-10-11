# Volley 5-1 en 3D

Application web pour apprendre la formation 5-1 au volley-ball à des débutants, quel que soit leur rôle : terrain et joueurs en 3D animés, pour chaque rotation, en attaque, en défense et en rally aléatoire.

- Réponds en français.
- Hébergement visé : GitHub Pages (dépôt public, branche `main`, dossier racine). Toute modification poussée sur `main` est publiée.

## Fichiers

- `index.html` : **toute l'application** dans un seul fichier (HTML, CSS, JS). Pas de build, pas de dépendances locales, pas de `package.json`.
- `LICENSE` : tous droits réservés.
- `README.md` : présentation du dépôt sur GitHub.
- Dépendances chargées par CDN (connexion internet nécessaire) :
  - three.js r147 UMD : `https://cdn.jsdelivr.net/npm/three@0.147.0/build/three.min.js` et `examples/js/controls/OrbitControls.js` (variable globale `THREE`, pas de modules ES). Garder cette version : `examples/js` n'existe plus après r148.
  - Polices Barlow depuis Google Fonts.

Pour tester : ouvrir `index.html` dans un navigateur, ou servir le dossier (`npx serve .` ou `python -m http.server`).

## Conventions du code

- Repère : `x` = largeur du terrain (-4.5 à 4.5), `z` = distance au filet (notre camp de 0 à 9, l'adversaire en négatif), `y` = hauteur.
- Cible du passeur `T = [1.2, 0.8]`, côté adverse `T_OPP = [-1.2, -0.8]`.
- Rotations dans l'ordre S1, S6, S5, S4, S3, S2, affichées P1 à P6. `lineup(rot, mode)` avec `mode` = `attack`, `defense` ou `rally`.
- Le libéro remplace le central arrière, sauf en défense quand ce central sert. L'exception P1 (pointu attaque en 4, R4 en 2) ne s'applique qu'en mode attaque.
- Scénarios construits par `Builder` : chaque étape vaut `{key, title, text, dur, pos, jump, ball:{to, apex}, ctx}`, positions interpolées avec `easeIO`. `relax()` garantit au moins 0,9 m entre joueurs.
- Fonctions principales :
  - `buildAttack`, `buildDefense`, `buildRally` ;
  - couverture 3-2 : `coverPlan` et `bestAssign` ;
  - défense périphérique « 6 arrière » : `addDefenseCycle` ;
  - passe et attaque : `addSetAttack` (option `cont` pour enchaîner en rally, `relay` pour une passe faite par un autre que le passeur) ;
  - attaque adverse : `OPP_HIT` (côtés `L`, `M`, `R` et pipe `B`), frappes `OPP_SHOTS` (diagonale, ligne, feinte, balle longue ; zones 5/1/6 et feinte pour le central et la pipe), `oppShotPlan` donne le défenseur. Si le passeur défend le premier ballon, le pointu fait la passe de relais (`RELAY_SPOT`) et `validAtts` retire 2e main, balle rapide et attaques du passeur ou du pointu.
- Animation : le ballon vole pendant la fenêtre `ball.win` de l'étape (`arc` pour bomber une frappe, `bounce` pour le rebond après un point) ; `computeVel` + `pathPoint` (Hermite) évitent que les joueurs s'arrêtent entre deux étapes quand ils continuent dans la même direction.
- Rally aléatoire : générateur à graine `mulberry`, tirages pondérés `pickW` (poids `ATT_W`), 10 tours par défaut, jusqu'à 30.
- Joueurs : rigs articulés, postures dans la table `POSES`, `poseFor` choisit la posture, `applyPose` l'applique (IK 2 os pour les jambes, `aimLimb` pour les bras), `animatePlayers` gère la course, le saut et l'orientation.
- Bulles d'info au survol ou au toucher : `tipFor`, `tipHTML`, `zoneReason`, `coverTip`, `moveDesc`.
- Réglages sauvegardés dans `localStorage`, clés préfixées `v51.`, toujours dans un try/catch.
- Trois mises en page, choisies par `applyUI` et posées dans `body[data-ui]` (`desk`, `port`, `land`) ; un petit script en début de `<body>` fixe la valeur avant le premier affichage.
  - Téléphone vertical (`port`) : la page ne défile pas ; terrain et lecture en haut, contenu de l'onglet au milieu, barre d'onglets `#mNav` en bas.
  - Téléphone horizontal (`land`, hauteur ≤ 520 px) : terrain plein écran, lecture en surimpression, rail d'onglets à droite, panneau en tiroir (`#app.drawerOpen`).
  - Onglets : Étape, Jeu, Rôle, Plus. Chaque bloc du panneau porte `data-tab` ; `setTab` change d'onglet. Sur téléphone, la légende passe dans `#capSlot` et la liste des étapes dans `#stepsSlot`.
  - Sur téléphone : puce de contexte `#ctxChip` (ouvre Jeu), menu caméra `#camToggle`, vitesse en une touche `#speedCycle`.
  - Aucun défilement horizontal sur mobile.

## Choix déjà faits par l'utilisateur

- Étiquettes au-dessus des joueurs combinables : rôle, position, prénom.
- Règle de chevauchement **décochée** par défaut.
- **Pas d'ombres** de joueurs.
- Choisir un rôle **ne rend pas les autres transparents** : le joueur choisi est seulement mis en avant (halo et surbrillance).
- Pas de mention « Inspiré de learnvolley.com ».
- Thème de l'interface Auto / Jour / Nuit, **Nuit par défaut** (`#themeSw`, clé `v51.theme`, appliqué dès le `<head>` pour éviter un flash). Le terrain 3D garde ses couleurs.
- Sidebar : chaque section est une carte (`.sec`, `details`) avec un titre à repère orange.

## Vérifications à refaire après une modification

- Aucun chevauchement illégal au service adverse, quand la règle est activée.
- Au moins 0,9 m entre joueurs à chaque étape, et tout le monde dans le terrain.
- Trois joueurs proches de l'attaquant en couverture.
- En rally, plusieurs centaines de graines sans erreur dans la console.
- Mobile (390 × 844) et paysage : pas de défilement horizontal.

## Idées proposées mais pas encore choisies

Mode quiz, rotations enchaînées, défense sur contre, autres systèmes de défense, fiche imprimable par rôle, fautes courantes. Ne pas les commencer sans demande.
