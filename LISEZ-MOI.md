# Quiz Histoire — mode d'emploi

## Ce qui a changé

Avant, chaque quiz était un dossier contenant deux fichiers de 500 lignes, dont 400 identiques d'un quiz à l'autre. Maintenant, la machinerie vit dans `moteur/` (écrite une fois) et chaque quiz est un simple fichier de contenu dans `quiz/`.

```
quiz-histoire/
├── index.html              Page d'accueil
├── editeur.html            Pour créer et corriger les quiz sans toucher au code
├── moteur/
│   ├── config.js           L'adresse de ta base Firebase (à remplir une fois)
│   ├── prof.html           Vue professeur
│   └── eleve.html          Vue élève
├── quiz/
│   ├── etudiants.json      Ta liste de classe
│   └── revolution-francaise-1.json
└── (tes 7 anciens dossiers, intacts et fonctionnels)
```

Tes anciens quiz continuent de fonctionner exactement comme avant. Rien n'a été touché.

---

## Installation, une seule fois

### 1. Créer la base Firebase

1. Va sur console.firebase.google.com, **Ajouter un projet**, nomme-le `revolutions-quiz`. Désactive Google Analytics, c'est inutile ici.
2. Menu de gauche → **Realtime Database** → **Créer une base de données**.
3. Emplacement : `us-central1`. Mode : **test**.
4. Copie l'adresse affichée en haut (`https://…firebaseio.com/`) et colle-la dans `moteur/config.js`.

Une seule base pour les onze quiz du cours. Chaque quiz y occupe son propre tiroir.

### 2. Verrouiller ce qui doit l'être

Onglet **Règles** de la base, remplace tout par ceci :

```json
{
  "rules": {
    "live":      { ".read": true, ".write": true },
    "resultats": { ".read": true, ".write": true, "$quiz": { "$seance": { "$eleve": { ".write": "!data.exists() || newData.child('reponses').val() != null" } } } }
  }
}
```

C'est volontairement simple. Ça empêche l'effacement accidentel des résultats archivés, sans transformer la mise en place en projet informatique. Un étudiant déterminé qui ouvrirait la console du navigateur pourrait encore bricoler son score affiché — mais **pas sa note**, qui est calculée à partir des réponses archivées au moment de la révélation, pas du score du jeu.

### 3. Déposer les fichiers sur GitHub

Glisse `index.html`, `editeur.html`, le dossier `moteur/` et le dossier `quiz/` à la racine de ton dépôt `quiz-histoire`. GitHub Pages publie en une minute environ.

### 4. Remplir la liste de classe

Ouvre `quiz/etudiants.json` et remplace les tableaux vides :

```json
"groupes": {
  "Groupe 01": ["Bouchard, Léa", "Nguyen, Kim", "Tremblay, Antoine"],
  "Groupe 02": ["Côté, Sarah", "Roy, Jérémie"]
}
```

Format : `"Nom, Prénom"`, une virgule entre chaque étudiant, pas de virgule après le dernier.

**Tant que la liste est vide, le quiz fonctionne quand même** : l'étudiant tape son nom et choisit son groupe. Tu peux donc tester dès aujourd'hui et remplir la liste en septembre. Dès qu'elle est remplie, le mode bascule tout seul.

---

## Faire passer un quiz

1. Ouvre `moteur/prof.html?quiz=revolution-francaise-1` et projette l'écran.
2. Les étudiants scannent le QR, cherchent leur nom, rejoignent. Un nom déjà pris se grise pour les autres.
3. **Lancer le quiz**. Pour chaque question : le chronomètre tourne (`＋15 s` si la classe traîne, `Arrêter le temps` si tout le monde a répondu), puis **Résultats** pour la distribution, **Révéler** pour la correction, **Suivante**.
4. Au podium : **Télécharger le CSV**.

L'archivage se fait à chaque révélation, pas à la fin. Si le navigateur plante à la question 7, les six premières sont déjà en sécurité. Le témoin en bas à droite confirme chaque enregistrement.

Si tu oublies d'exporter : rien n'est perdu. Retourne à l'accueil du quiz, **Exporter une séance antérieure**, choisis la date.

---

## Le fichier de notes

Une ligne par étudiant, ouvrable dans Excel :

| Groupe | Nom | Q1 | Q2 | … | Total sur 10 | Pourcentage |
|---|---|---|---|---|---|---|

`1` = bonne réponse, `0` = mauvaise ou aucune. La rapidité n'apparaît nulle part : elle ne sert qu'au classement affiché pendant la partie.

Pour le calcul final, colle les onze colonnes « Total » côte à côte dans une feuille, garde les huit meilleures par étudiant avec `=SOMME(GRANDE.VALEUR(plage;{1;2;3;4;5;6;7;8}))`, et divise par 8 pour ramener sur 10 %.

---

## Créer les autres quiz

Ouvre `editeur.html`, remplis le formulaire, clique **Télécharger le fichier**. Dépose le `.json` obtenu dans `quiz/` et ajoute son nom dans la liste `QUIZ_REVOLUTIONS`, au début du fichier `index.html`.

Pour corriger une coquille : **Ouvrir un quiz existant**, corrige, retélécharge, remplace le fichier.
