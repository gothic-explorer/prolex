# Spécifications Fonctionnelles : Traduction IA et Édition (Écran Scindé)

## 1. Objectif du Module
Ce module régit la brique la plus innovante de l'application : la traduction et la synthèse automatique de documents juridiques rédigés en arabe littéraire vers le français ou l'anglais. L'interface propose une vue scindée (côte à côte) permettant à l'avocat de comparer la pièce originale avec le résultat de l'IA, de corriger manuellement le texte et de valider une version officielle pour son client.

---

## 2. Parcours Utilisateur Dépliant (Workflow)

```
[Fiche Dossier] ──> [Clic "Traduire"] ──> [Sélection Langue Cible] ──> [Vérification Cache BDD] 
                                                                             │
                    ┌───────────────── (N'existe pas) ───────────────────────┤
                    ▼                                                        ▼ (Existe en base)
[Appel API OCR + LLM (4-8s)] ──> [Sauvegarde BDD] ──> [Affichage Écran Scindé Instantané (<100ms)]
                                                             │
                                                             ▼
                                              [Édition / Correction Manuelle] ──> [Bouton "Valider pour le Client"]
```

---

## 3. Spécifications de l'Interface Graphique (UI/UX)

### Écran Scindé (Split-Screen / Vue Côte à Côte)
Cet écran s'active lorsque l'avocat clique sur une pièce pour l'analyser. Il est optimisé pour le format Web et Tablette (en mode paysage).

#### Panneau Gauche (50% de la largeur) : Document Original
*   **Visionneuse Intégrée :** Affiche le document PDF original ou l'image téléversée par le cabinet.
*   **Contrôles :** Zoom avant/arrière, rotation, et sélection de texte si le PDF source n'est pas un scan brut.

#### Panneau Droite (50% de la largeur) : Hub d'Assistance IA
Ce panneau comporte deux onglets de navigation :
1.  **Onglet 1 : Résumé Exécutif**
    *   Affiche une liste à puces (3 à 5 points clés) générée par le LLM détaillant l'Objet, les Dates clés et les Ordonnances.
    *   Chaque puce dispose d'un éditeur de texte en ligne au clic.
2.  **Onglet 2 : Traduction Intégrale**
    *   Affiche le texte juridique complet traduit dans la langue choisie.
    *   **Zone d'Édition Rich-Text :** Le texte n'est pas statique ; il est injecté dans un éditeur WYSIWYG permettant à l'avocat d'ajuster des tournures ou des termes du droit tunisien.

#### Barre d'Actions Inférieure
*   **Indicateur d'édition :** Un badge indique si le document est d'origine (`Généré par l'IA`) ou s'il a été modifié (`Modifié manuellement`).
*   **Bouton "Régénérer" :** Relance le pipeline complet (OCR + LLM) en ignorant le cache (écrase la version actuelle).
*   **Bouton "Publier / Valider" :** Change le statut en `validated_by_lawyer`, ce qui rend instantanément le résumé et la traduction visibles sur l'application du client.

---

## 4. Règles de Gestion Métier (Business Rules)

*   **RG-TRAD-01 (Politique de Cache Strict) :** Avant tout appel aux API externes (Google Cloud Vision / Gemini), le backend doit requêter la table `document_translations` avec le couple `(document_id, target_language)`. Si une ligne existe, le traitement IA est ignoré et le contenu de la base est renvoyé.
*   **RG-TRAD-02 (Verrouillage Client) :** Le client final n'a **jamais** accès à une traduction ayant le statut `draft`. Seuls les documents passés au statut `validated_by_lawyer` par une action explicite de l'avocat apparaissent dans l'espace client.
*   **RG-TRAD-03 (Traçabilité des Modifications) :** Dès qu'un avocat modifie un seul caractère dans l'éditeur WYSIWYG de la traduction ou du résumé, le booléen `is_edited_by_user` passe à `true` en base de données et l'en-tête graphique affiche la mention "Modifié par Maître X".

---

## 5. Spécifications C# (Use Case & Payload .NET 10)

Voici la structure du Handler .NET gérant la récupération ou le déclenchement de la traduction avec gestion de cache.

```csharp
using MediatR;
using System.Text.Json;
using ProLex.Application.Common.Interfaces;
using ProLex.Domain.Entities;
using Microsoft.EntityFrameworkCore;

namespace ProLex.Application.Translations.Queries.GetTranslation;

public record GetTranslationQuery : IRequest<TranslationResultDto>
{
    public Guid DocumentId { get; init; }
    public required string TargetLanguage { get; init; }
}

public class GetTranslationQueryHandler : IRequestHandler<GetTranslationQuery, TranslationResultDto>
{
    private readonly IProLexDbContext _context;
    private readonly IAiService _aiService;

    public GetTranslationQueryHandler(IProLexDbContext context, IAiService aiService)
    {
        _context = context;
        _aiService = aiService;
    }

    public async Task<TranslationResultDto> Handle(GetTranslationQuery request, CancellationToken cancellationToken)
    {
        // 1. Vérification du cache en BDD via l'interface du contexte abstrait
        var existingTranslation = await _context.DocumentTranslations
            .FirstOrDefaultAsync(t => t.DocumentId == request.DocumentId && t.TargetLanguage == request.TargetLanguage, cancellationToken);
        
        if (existingTranslation != null)
        {
            return new TranslationResultDto
            {
                Id = existingTranslation.Id,
                Summary = JsonSerializer.Deserialize<List<string>>(existingTranslation.TranslatedSummary ?? "[]") ?? new(),
                FullText = existingTranslation.TranslatedFullText,
                IsEditedByUser = existingTranslation.IsEditedByUser,
                Status = existingTranslation.Status
            };
        }

        // 2. Si inexistant : Déclenchement du vrai pipeline IA (OCR + LLM)
        var document = await _context.Documents.FindAsync(new object[] { request.DocumentId }, cancellationToken);
        if (document == null) throw new KeyNotFoundException("Document introuvable.");

        var aiResult = await _aiService.ProcessAndTranslateDocumentAsync(document.FilePath, request.TargetLanguage);

        // 3. Persistance immédiate dans la table PostgreSQL via EF Core
        var newTranslation = new DocumentTranslation
        {
            DocumentId = request.DocumentId,
            TargetLanguage = request.TargetLanguage,
            RawOcrText = aiResult.RawOcrText,
            TranslatedTitle = $"Traduction_{document.Title}",
            TranslatedSummary = JsonSerializer.Serialize(aiResult.Summary),
            TranslatedFullText = aiResult.FullText,
            Status = "draft",
            IsEditedByUser = false
        };

        _context.DocumentTranslations.Add(newTranslation);
        await _context.SaveChangesAsync(cancellationToken);

        return new TranslationResultDto
        {
            Id = newTranslation.Id,
            Summary = aiResult.Summary,
            FullText = aiResult.FullText,
            IsEditedByUser = false,
            Status = "draft"
        };
    }
}
```
