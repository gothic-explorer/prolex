# Spécifications Fonctionnelles Détaillées (SFD) - MVP ProLex Tunisie

Ce dossier contient l'ensemble des spécifications techniques et fonctionnelles pour le développement itératif du Produit Minimum Viable (MVP) de la plateforme ProLex, outil de gestion et de numérisation pour les cabinets d'avocats en Tunisie.

## 1. Vision du Projet & Problématique
Actuellement en Tunisie, la gestion des cabinets d'avocats repose quasi-intégralement sur le support papier et la mémoire humaine. Cette absence de numérisation engendre :
* Une perte de temps massive lors de la recherche physique des pièces et des jugements.
* Une friction permanente entre l'avocat et ses clients due à des appels incessants pour le suivi des audiences.
* Un risque élevé d'oubli de dates d'audiences par les clients.

**La Solution :** Une application Web Responsive Mobile-First (ProLex) permettant d'archiver, de traduire instantanément le jargon juridique arabe via une IA sécurisée, et d'automatiser les rappels d'audiences par SMS/Push.

---

## 2. Périmètre du MVP (Règles d'Inclusion / Exclusion)

### 🟢 Inclus dans le MVP (Priorité Haute)
* **Gestion des dossiers :** Enregistrement des références judiciaires tunisiennes.
* **Double canal de numérisation :** Upload de scans ou capture photo depuis un smartphone, rattachés strictement à l'affaire.
* **Pipeline IA :** Extraction OCR (Arabe) + Traduction et Résumé automatique (Français/Anglais) avec sauvegarde en base de données.
* **Édition humaine :** Possibilité pour l'avocat de corriger la traduction de l'IA avant partage.
* **Agenda Judiciaire :** Suivi des audiences et de leurs issues (reports, délibérés).
* **Mode Offline :** Consultation et saisie locale pour l'avocat au sein des tribunaux.
* **Rappels Automatiques :** Envoi automatique de SMS et notifications Push à J-3 et J-1 aux clients.

### 🔴 Exclus du MVP (Règles d'Exclusion - Version 2.0)
* **Chat en temps réel :** Aucun système de messagerie instantanée (WhatsApp-like) pour préserver le temps de l'avocat.
* **Facturation complexe :** Pas de gestion des fiches de frais ou de comptabilité avancée.
* **Paiement en ligne :** Exclusion des passerelles de paiement (ClicToPay, Konnect) à ce stade en raison de la complexité réglementaire.

---

## 3. Glossaire Juridique Tunisien (Termes Métiers)

Pour faciliter le développement et la cohérence de l'IA, voici les correspondances strictes des termes utilisés dans l'application :

| Terme en Arabe | Transliteration / Usage | Équivalent Français Certifié | Définition / Contexte Applicatif |
| :--- | :--- | :--- | :--- |
| **رقم القضية / رقم الجلسة** | Numéro de rôle | Numéro d'affaire / rôle | L'identifiant unique d'une affaire au tribunal. |
| **محكمة الإبتدائية** | Tribunal d'Instance | Tribunal de Première Instance | Juridiction de premier degré. |
| **محكمة الاستئناف** | Isti'naf | Tribunal d'Appel | Juridiction de second degré. |
| **محكمة الناحية** | Mahkamat Al Nahiya | Tribunal de Canton | Juridiction de proximité pour les petits litiges. |
| **تأجيل الجلسة** | Ta'jil | Report d'audience | Décision du juge de reporter l'affaire à une date ultérieure. |
| **حكم غيابي** | Hokm Ghiabi | Jugement par défaut | Jugement rendu en l'absence de l'une des parties. |
| **نسخة تنفيذية** | Noskha Tanfizya | Grosse du jugement / Copie exécutoire | Copie officielle du jugement permettant de faire exécuter la décision. |

---

## 4. Matrice de Traçabilité des Fonctionnalités (Ordre d'Implémentation)

Le développement doit suivre cet ordre strict pour valider le socle technique avant les interfaces :

```
[Étape 1 : Socle BDD & Sécurité] ──> [Étape 2 : Gestion des Dossiers] ──> [Étape 3 : Pipeline IA & Édition] ──> [Étape 4 : Rappels & Espace Client]
```

1. **Lot 00 (Socle Transverse) :** Configuration de la BDD PostgreSQL, architecture .NET 10 Clean Architecture, et pipeline d'API pour l'IA.
2. **Lot 01 (Espace Avocat) :** Écrans de création de dossiers, numérisation/import de fichiers, agenda et écran scindé de traduction.
3. **Lot 02 (Espace Client) :** Écrans de consultation simplifiée et script de déclenchement des tâches automatisées (SMS/Push).
