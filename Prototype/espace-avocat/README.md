# Prototype UI — Espace Avocat (Lot 01)

Prototype cliquable, mobile-first, du parcours avocat décrit dans `Specifications/PARCOURS_AVOCAT/`.
Un seul fichier HTML autonome (HTML/CSS/JS sans dépendance). Ouvrir `index.html` dans un navigateur.

Données fictives, IA simulée, état conservé dans le `localStorage` du navigateur (réinitialisable depuis l'écran Synchro).

| Spécification | Écran du prototype |
| :--- | :--- |
| 01 · Écran A — Dashboard | Liste des dossiers : recherche instantanée (temps affiché), filtres par statut, tableau (desktop) / cartes (mobile) |
| 01 · Écran B — Formulaire | Création / modification bilingue FR/AR, validation des champs obligatoires, format `Année/Numéro` recommandé, détection de doublon |
| 01 · Écran C — Fiche & GED | Double canal : glisser-déposer (PDF/JPG/PNG, 10 Mo max) et « Prendre une photo » (recadrage + filtre gris/contraste RG-DOS-03), nom de stockage UUID, dossier non modifiable (RG-DOS-01), suppression logique |
| 02 · Écran scindé | Original (zoom, rotation) / Assistant IA (résumé éditable, traduction WYSIWYG + glossaire certifié), cache (RG-TRAD-01), brouillon invisible client (RG-TRAD-02), « Modifié par Maître X » (RG-TRAD-03), Régénérer, Publier |
| 03 · Agenda & compte-rendu | Vue jour / 7 jours, indicateur réseau, rappels SMS J-3 / Push J-1, compte-rendu express (statut segmenté, date conditionnelle, issue en arabe, note interne) |
| 03 · Mode hors-ligne | Bouton réseau du bandeau pour simuler la coupure : écriture locale « Modifié localement », file `Sync_Queue` (payload `POST /api/hearings/sync`), synchronisation par lot au retour du réseau |

Bascule FR / ع dans le bandeau : inversion LTR/RTL de l'interface.
