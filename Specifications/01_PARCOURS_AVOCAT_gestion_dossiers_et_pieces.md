# Spécifications Fonctionnelles : Gestion des Dossiers et des Pièces (Espace Cabinet)

## 1. Objectif du Module
Ce module permet à l'avocat (ou à son secrétariat) de centraliser la création des affaires judiciaires selon les usages des tribunaux tunisiens et de gérer l'archivage numérique des pièces (PDF, photos de PV, jugements). Chaque document doit être obligatoirement rattaché à son affaire. La numérisation s'effectue soit par upload de fichier (scan traditionnel), soit par prise de photo directe via smartphone.

---

## 2. Parcours Utilisateur Dépliant (Workflow)

```
[Dashboard Principal] ──> [Bouton "Nouveau Dossier"] ──> [Formulaire Saisie Affaire] ──> [Vue Fiche Dossier]
                                                                                             │
                    ┌─────────────────── (Choix du canal de numérisation) ───────────────────┤
                    ▼                                                                        ▼
   [Upload Fichier (PDF/Scan)]                                                  [Capture Photo Mobile]
                    │                                                                        │
                    └─────────────────────> [Attachement Strict à l'Affaire] <───────────────┘
```

1. **Dashboard Cabinet :** L'avocat ou l'assistant visualise la liste globale de ses dossiers avec un moteur de recherche en temps réel et un bouton proéminent `+ Nouvelle Affaire`.
2. **Création de la Fiche :** L'utilisateur remplit le formulaire bilingue contenant les métadonnées officielles de l'affaire (Tribunal, Numéro de rôle, etc.).
3. **Numérisation et Archivage :** Depuis la fiche de l'affaire, l'utilisateur numérise une pièce (glisser-déposer d'un scan ou capture photo en direct depuis un smartphone). Le document est stocké et encapsulé sous l'ID de cette affaire.

---

## 3. Spécifications des Interfaces et Formulaires

### Écran A : Le Tableau de Bord Central (Dashboard)
*   **Barre de Recherche Réactive :** Un champ de saisie unique permettant un filtrage instantané côté client (< 50ms) sur le nom du client, le numéro d'affaire ou la juridiction.
*   **Tableau de Synthèse des Dossiers :** Affichage des colonnes clés : `N° Rôle`, `Client`, `Partie Adverse`, `Tribunal`, `Dernière Activité` (Date) et `Statut` (Badge de couleur).

### Écran B : Formulaire de Création / Édition de Dossier
Cet écran comporte un formulaire bilingue (Français/Arabe) structuré selon les usages des tribunaux tunisiens :

| Libellé du Champ (FR / AR) | Type de Composant | Règles de Gestion & Validations Métier |
| :--- | :--- | :--- |
| **Numéro d'affaire / Rôle** <br>*(رقم القضية)* | Champ Texte (Court) | **Obligatoire.** Format type tunisien recommandé pour la cohérence : `Année/Numéro` (ex: `2026/14523`). |
| **Tribunal concerné** <br>*(المحكمة)* | Liste Déroulante | **Obligatoire.** Liste standardisée des juridictions tunisiennes (ex: *Tribunal de Première Instance de Tunis, Tribunal de Canton de l'Ariana, Cour d'Appel de Sousse*). |
| **Nom du Client** <br>*(اسم الموكل)* | Champ Texte (100) | **Obligatoire.** Nom complet du client (personne physique ou morale). |
| **Partie Adverse** <br>*(الخصم)* | Champ Texte (100) | **Obligatoire.** Nom de l'opposant ou de son entreprise. |
| **Statut de l'Affaire** <br>*(حالة القضية)* | Boutons Radio | **Obligatoire.** Valeurs : `En cours (جاري)`, `Jugé (محكوم)`, `En appel (في الإستئناف)`. |

### Écran C : La Fiche Dossier & Double Canal de Numérisation (GED)
*   **Canal 1 - Upload Classique :** Zone interactive en pointillés (Drag & Drop) optimisée pour le format Web de bureau. Formats autorisés : `.pdf`, `.jpg`, `.jpeg`, `.png`. Poids maximum : `10 Mo`.
*   **Canal 2 - Caméra Smartphone (Mobile-First) :** Un bouton `[ Prendre une photo ]` ouvre nativement l'appareil photo du smartphone. Une interface de rognage automatique détecte les bords du document papier avant validation.
*   **Liste des Pièces Jointes :** Tableau récapitulatif affichant le titre de la pièce, la date d'import et un groupe de boutons d'action : `[ Visualiser ]`, `[ Traduire via IA ]`, `[ Supprimer ]`.

---

## 4. Règles de Gestion Métier (Business Rules)

*   **RG-DOS-01 (Intégrité de Liaison d'Affaire) :** Aucun document ne peut exister de manière isolée dans le système. L'ID du dossier (`DossierId`) est un champ obligatoire non modifiable à la création du document. La suppression ou l'archivage d'un dossier entraîne l'application en cascade des mêmes règles sur l'ensemble de ses pièces jointes.
*   **RG-DOS-02 (Nettoyage et UUID) :** Lors du téléversement ou de la capture, le système assainit le nom du fichier et génère un identifiant de stockage unique (UUID) pour éviter les collisions dans le stockage cloud sécurisé.
*   **RG-DOS-03 (Filtre d'image pour l'OCR Arabe) :** Pour les captures effectuées par smartphone, un traitement automatique (conversion en nuances de gris, normalisation du contraste) est appliqué à l'image avant l'envoi vers le serveur. Cela permet de garantir un taux de réussite de l'OCR de plus de 95% sur l'arabe littéraire juridique dactylographié, souvent imprimé sur papier administratif de faible qualité.

---

## 5. Spécifications C# (Couche Application .NET 10)

Voici le cas d'utilisation MediatR gérant l'archivage d'une pièce avec son flux binaire, compatible à la fois avec un import de scan de bureau ou un tableau d'octets d'une photo smartphone.

```csharp
using MediatR;
using ProLex.Application.Common.Interfaces;
using ProLex.Domain.Entities;

namespace ProLex.Application.Dossiers.Commands.AddDocument;

public record AddDocumentCommand : IRequest<Guid>
{
    public Guid DossierId { get; init; }
    public required string Title { get; init; }
    public required string Extension { get; init; }
    public required byte[] FileContent { get; init; } // Reçoit le flux binaire (Scan ou Photo)
}

public class AddDocumentCommandHandler : IRequestHandler<AddDocumentCommand, Guid>
{
    private readonly IProLexDbContext _context;
    private readonly IFileStorageService _storageService;

    public AddDocumentCommandHandler(IProLexDbContext context, IFileStorageService storageService)
    {
        _context = context;
        _storageService = storageService;
    }

    public async Task<Guid> Handle(AddDocumentCommand request, CancellationToken cancellationToken)
    {
        // 1. Sauvegarde du fichier binaire sur le stockage Cloud sécurisé
        string secureFilePath = await _storageService.UploadAsync(request.FileContent, request.Title, request.Extension);

        // 2. Création et attachement strict de l'entité à l'affaire
        var document = new Document
        {
            DossierId = request.DossierId,
            Title = request.Title,
            FilePath = secureFilePath,
            CreatedAt = DateTime.UtcNow
        };

        _context.Documents.Add(document);
        await _context.SaveChangesAsync(cancellationToken);

        return document.Id;
    }
}
```
