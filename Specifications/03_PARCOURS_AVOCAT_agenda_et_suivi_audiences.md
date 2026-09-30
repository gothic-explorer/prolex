# Spécifications Fonctionnelles : Agenda Judiciaire et Suivi des Audiences (Mode Offline)

## 1. Objectif du Module
Ce module régit la planification et le suivi des audiences des tribunaux tunisiens. L'avocat passe une grande partie de ses journées au tribunal, où la connexion internet (3G/4G) est souvent inexistante ou coupée. Ce module spécifie le fonctionnement du mode déconnecté (Offline) : l'avocat doit pouvoir consulter son planning et saisir l'issue d'une audience directement sur le terrain. La synchronisation s'effectue automatiquement dès le retour du réseau.

---

## 2. Parcours Utilisateur Dépliant (Workflow de Terrain)

```
[Avocat au Tribunal (Sans Réseau)] ──> [Consultation Agenda Local] ──> [Saisie de l'Issue de l'Audience] 
                                                                                   │
                                                                                   ▼
[Stockage immédiat en mémoire locale (SQLite/Hive)] <── [Statut d'audience : "Modifié localement"]
                                                                                   │
                     ┌─────────────────── (Retour de la 4G / Wi-Fi) ───────────────┘
                     ▼
[Déclenchement du Worker de Synchronisation Backend .NET] ──> [Mise à jour PostgreSQL] ──> [Envoi Notifications Client]
```

---

## 3. Spécifications des Interfaces (UI/UX)

### Écran A : L'Agenda Judiciaire Hebdomadaire / Journalier
*   **Vue Liste Chronologique :** Un affichage épuré des audiences du jour, triées par heure ou par priorité. Chaque carte d'audience affiche : `Heure`, `Numéro de rôle`, `Tribunal`, `Nom du Client` et `Salle`.
*   **Indicateur de Réseau :** Un petit témoin visuel en haut de l'écran indique l'état de la connexion (`En ligne` [Vert] / `Hors-ligne` [Orange]).

### Écran B : Formulaire de Compte-Rendu d'Audience (Mise à Jour Express)
Au sortir de la salle d'audience, l'avocat ouvre cet écran rapide pour renseigner l'issue du passage devant le juge :

| Élément d'Interface | Type de Composant | Description & Règles de Saisie |
| :--- | :--- | :--- |
| **Statut de l'Audience** | Boutons Segmentés | Options : `Effectuée (تمت)`, `Reportée (تأجلت)`, `Mise en délibéré (حجزت للمجال)`. |
| **Nouvelle Date d'Audience** | Sélecteur de Date | Conditionnel : Affiché uniquement si le statut est `Reportée`. Permet de planifier directement le prochain passage. |
| **Issue / Décision (Arabe)** | Champ texte libre | Saisie de l'ordonnance ou de la raison du report en langue arabe (ex: *تأجيل لتقديم الجواب*). |
| **Note Interne (Optionnel)** | Champ texte libre | Notes personnelles de l'avocat pour son cabinet (non visibles par le client). |

---

## 4. Mécanisme Technique du Mode Hors-ligne (Offline Sync)

Pour que l'expérience utilisateur soit fluide et transparente pendant le PoC et en production, l'application applique la logique suivante :

1.  **Mise en cache préventive (Read Cache) :** Chaque matin (ou lors de la dernière connexion réseau réussie), l'application télécharge et stocke l'intégralité des dossiers et du calendrier des 30 prochains jours dans la base de données locale du smartphone (`SQLite` ou `Hive`).
2.  **Écriture Locale (Write Queue) :** Lorsque l'avocat valide un compte-rendu d'audience sans réseau, l'application :
    *   Met à jour la ligne dans la base locale avec un drapeau `is_dirty = true`.
    *   Ajoute la requête de modification dans une file d'attente locale (`Sync_Queue`).
3.  **Algorithme de Synchronisation (Background Synchronization) :** L'application écoute les changements d'état du réseau de l'appareil. Dès que la connexion est rétablie :
    *   La file d'attente `Sync_Queue` envoie les modifications au backend .NET par lots (Batch Processing).
    *   Le serveur applique les modifications, réinitialise le drapeau `is_dirty = false` et déclenche le pipeline de notification pour le client final.

---

## 5. Spécifications C# (Couche API .NET 10)

Voici le endpoint d'API qui reçoit la synchronisation des données d'audience depuis le terminal de l'avocat.

```csharp
using MediatR;
using Microsoft.AspNetCore.Mvc;
using ProLex.Application.Hearings.Commands;

namespace ProLex.WebAPI.Controllers;

[ApiController]
[Route("api/hearings")]
public class HearingsController : ControllerBase
{
    private readonly IMediator _mediator;

    public HearingsController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost("sync")]
    public async Task<IActionResult> SyncHearingOutcome([FromBody] SyncHearingOutcomeCommand command)
    {
        var result = await _mediator.Send(command);
        
        if (!result.IsSuccess)
            return BadRequest(result.ErrorMessage);

        return Ok(new { Message = "Audience mise à jour et synchronisée avec succès." });
    }
}
```
