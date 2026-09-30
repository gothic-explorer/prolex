# Guide d'Entretien Avocats : Validation Go / No-Go (ProLex)

## 1. Objectif
Comprendre la situation réelle des avocats tunisiens avant tout développement, afin de décider **go, no-go ou pivot**. Format court : 15 questions, environ 15 minutes. Si l'avocat n'a que 5 minutes, poser uniquement les questions marquées ⭐.

## 2. Conseils de conduite
- **Ne pas présenter ProLex en début d'entretien.** Décrire le problème, pas la solution, sinon les réponses seront polies mais peu fiables.
- **Interroger sur le passé** (« raconte-moi la dernière fois que... ») plutôt que sur l'avenir (« utiliserais-tu... ? »).
- **Varier les profils** : avocat seul, petit cabinet, cabinet plus grand, jeune avocat, avocat expérimenté, et si possible une secrétaire (future utilisatrice réelle).
- **Viser 8 à 10 entretiens.** Formats possibles : téléphone, vocal WhatsApp, échange à la sortie d'une audience.
- Les réponses sur le cadre juridique (Q11) sont des indications, **pas un avis juridique**. Elles doivent être confirmées par un juriste tunisien.

---

## 3. Les 15 questions

### A. La douleur : le problème existe-t-il vraiment ?

1. ⭐ **Raconte-moi la dernière fois qu'un client t'a demandé un détail sur son affaire et que tu n'avais pas le dossier sous la main. Que s'est-il passé ?**
   *Cherche une anecdote concrète et récente. Sans anecdote, la douleur est probablement faible.*

2. ⭐ **Combien de temps par jour perds-tu à répondre aux clients (appels, messages) sur des questions de suivi ?**
   *Valide l'intérêt des rappels automatiques et de l'espace client.*

3. **As-tu déjà manqué une audience ou un délai ? Comment ça s'est passé ?**

4. **Quand tu cherches une pièce ou un jugement ancien, combien de temps cela te prend-il, et comment le retrouves-tu ?**
   *Valide la recherche plein texte dans les pièces.*

### B. L'existant : pourquoi ne pas utiliser ce qui existe ?

5. ⭐ **Quels outils utilises-tu aujourd'hui pour tes dossiers et ton agenda ? As-tu déjà essayé un logiciel, par exemple Avocat Tunisie ? Pourquoi l'as-tu gardé ou abandonné ?**
   *Ce qui a fait abandonner un outil est ce que ProLex doit éviter.*

### C. L'adoption : qui va saisir les données ?

6. ⭐ **Qui classe et saisit les dossiers dans ton cabinet ? Serait-il prêt à scanner ou photographier chaque nouvelle pièce ?**
   *Risque n°1 du projet. Si personne n'a le temps ni l'envie, l'outil restera vide.*

7. **Préférerais-tu ne numériser que les nouveaux dossiers, sans reprendre l'historique papier ?**

### D. Le terrain : offline et mobile

8. **Le réseau est-il mauvais dans les tribunaux où tu vas, et as-tu besoin de consulter ou noter des choses sur ton téléphone sur place ?**

### E. Le client : espace client et rappels

9. **Quelles informations les clients te demandent-ils le plus souvent, et que ne voudrais-tu surtout pas qu'ils voient eux-mêmes ? Préfèrent-ils un SMS, un lien, WhatsApp ou une application ?**
   *Définit le contenu de l'espace client (Lot 02) et les règles de visibilité.*

### F. L'IA et la confidentialité : le point qui peut tout bloquer

10. ⭐ **Accepterais-tu que le contenu d'une pièce, sans les noms des parties, soit traité par un service d'IA à l'étranger pour produire une traduction ou un résumé ? Sinon, qu'est-ce qui te rassurerait (hébergement en Tunisie, accord du client, désactivation par dossier) ?**
    *Un refus net et récurrent impose de repenser le pipeline IA.*

11. **Dois-tu informer le client ou obtenir son accord avant de stocker ou traiter ses pièces en ligne ? L'Ordre a-t-il des règles ou des recommandations sur ce point ? Connais-tu un confrère ou un juriste à consulter ?**
    *Oriente la vérification juridique (secret professionnel, INPDP).*

12. **As-tu vraiment besoin de traductions ou de résumés (clients étrangers, confrères, dossiers en français) ? À quelle fréquence, et comment les obtiens-tu aujourd'hui ?**
    *Vérifie que la traduction est un vrai besoin.*

13. ⭐ **Peux-tu me montrer ou m'envoyer 3 à 5 pièces typiques, anonymisées (photo d'un jugement, d'un PV, d'une ordonnance) ?**
    *Données de test pour l'OCR arabe. Noter si elles sont manuscrites, tamponnées, photocopiées ou de mauvaise qualité.*

### G. Le prix et l'engagement

14. ⭐ **Si un outil réglait ces problèmes, combien paierais-tu par mois, et accepterais-tu de l'essayer gratuitement pendant 1 à 2 mois en me donnant tes retours ?**
    *Un prix cité et un oui au pilote valent plus que tous les compliments.*

15. **Qui décide de l'achat d'un outil dans ton cabinet (toi seul, les associés) et comment paies-tu habituellement (virement, chèque, espèces) ?**
    *Anticipe un blocage sur la décision ou l'encaissement (paiement en ligne exclu du MVP).*

---

## 4. Critères de décision (après 8 à 10 entretiens)

### Go si la majorité de ces signaux sont réunis
- [ ] La plupart des avocats racontent spontanément une anecdote concrète (Q1).
- [ ] La saisie des dossiers est jugée faisable (Q6).
- [ ] L'IA est acceptée sous des conditions réalisables (Q10).
- [ ] Au moins 3 à 4 avocats acceptent un pilote et citent un prix couvrant les coûts (Q14).
- [ ] Au moins une quinzaine de pièces réelles sont obtenues pour tester l'OCR (Q13).

### No-go ou pivot si l'un de ces signaux domine
- Personne ne ressent de vraie douleur.
- Personne n'a le temps de saisir les dossiers.
- L'IA étrangère est refusée en bloc.
- Personne n'accepte de payer ni de tester.

### Pivot possible
Si la douleur est forte mais que l'IA est refusée : lancer d'abord une version sans IA (dossiers, agenda, recherche, rappels, espace client) et ajouter la traduction plus tard.

---

## 5. Tableau de synthèse (à remplir au fil des entretiens)

| # | Profil (seul / cabinet / secrétaire) | Anecdote de douleur (Q1) | Outils actuels (Q5) | Saisie faisable ? (Q6) | IA acceptée ? (Q10) | Prix cité (Q14) | Pilote ? | Pièces reçues (Q13) |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | | | | | | | | |
| 2 | | | | | | | | |
| 3 | | | | | | | | |
| 4 | | | | | | | | |
| 5 | | | | | | | | |
| 6 | | | | | | | | |
| 7 | | | | | | | | |
| 8 | | | | | | | | |
| 9 | | | | | | | | |
| 10 | | | | | | | | |

---

## 6. Fiche de notes par entretien (à dupliquer)

```
Entretien n° : ____        Date : ____________        Format : tél / vocal / présentiel
Profil : avocat seul / petit cabinet / grand cabinet / secrétaire
Ville / tribunaux fréquentés : ____________________

Douleur principale (ses mots) :
_____________________________________________

Anecdote marquante :
_____________________________________________

Outils actuels / logiciel déjà essayé :
_____________________________________________

Qui saisit les dossiers ? Faisable ? :
_____________________________________________

Réseau au tribunal / usage du téléphone :
_____________________________________________

Informations demandées par les clients / à ne pas exposer :
_____________________________________________

Position sur l'IA et l'hébergement (conditions posées) :
_____________________________________________

Besoin réel de traduction (fréquence, méthode actuelle) :
_____________________________________________

Règles de l'Ordre / juriste suggéré :
_____________________________________________

Prix cité : ________ TND / mois      Pilote : oui / non / peut-être
Décideur d'achat / mode de paiement : ____________________
Pièces fournies : ____ (type, qualité) : ____________________

Personnes à rencontrer ensuite :
_____________________________________________

Signal global : 🟢 fort   🟠 moyen   🔴 faible
```

---

## 7. Étapes suivantes
1. Mener 8 à 10 entretiens et remplir le tableau de synthèse.
2. Tester l'OCR et la traduction sur les pièces collectées (avec relecture par un juriste bilingue).
3. Consulter un juriste tunisien (secret professionnel, INPDP, transfert de données à l'étranger).
4. Tester Avocat Tunisie (démo ou essai) et comparer.
5. Décider : go, no-go ou pivot (version sans IA).
