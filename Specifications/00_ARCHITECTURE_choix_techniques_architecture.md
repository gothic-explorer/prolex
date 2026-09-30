# Spécifications Techniques : Architecture Globale, .NET 10 & Persistance Code-First (Audit & Soft Delete)

## 1. Objectif du Document
Ce document formalise les choix technologiques et les principes d'architecture de la plateforme **ProLex**. L'objectif est de garantir une séparation stricte des responsabilités via une **Clean Architecture**, permettant de basculer de manière transparente entre un mode Démo (données éphémères en mémoire) et un mode Production (persistance sur PostgreSQL). Ce document intègre un système d'audit automatisé et de suppression logique globale pour répondre aux exigences réglementaires de traçabilité.

---

## 2. Choix Technologiques Globaux

*   **Frontend :** Application Web Responsive conçue selon l'approche **Mobile-First**. L'interface est adaptative pour garantir une ergonomie optimale sur smartphone (pour l'avocat en déplacement et le client) et sur ordinateur/tablette (pour l'écran scindé de l'avocat au cabinet). Elle prend en charge nativement le bilinguisme et l'inversion de layout **LTR/RTL** (Français/Arabe).
*   **Backend :** Développé avec **.NET 10** et **C#**, offrant des performances de pointe pour le traitement asynchrone des flux de documents et l'intégration des API d'intelligence artificielle.
*   **Base de Données :** **PostgreSQL**, sélectionné pour sa robustesse, sa gestion native du type `jsonb` (idéal pour le stockage des résumés IA) et sa conformité avec les exigences de sécurité.

---

## 3. Structure de la Clean Architecture (ProLex)

La solution .NET `ProLex.sln` est découpée en quatre projets distincts. Les dépendances pointent exclusivement vers l'intérieur, protégeant le cœur métier des frameworks externes.

```
[ ProLex.WebAPI (Présentation) ] ──> [ ProLex.Infrastructure ]
                 │                                   │
                 ▼                                   ▼
     [ ProLex.Application ] ─────────────────────────┘
                 │
                 ▼
       [ ProLex.Domain ]
```

1.  **ProLex.Domain :** Contient les entités métiers, les enums et les logiques pures du droit tunisien. Aucune dépendance externe.
2.  **ProLex.Application :** Définit la logique métier globale, les cas d'utilisation (Use Cases via MediatR) et les interfaces des contrats nécessaires (Repository Pattern, DbContext et services transverses).
3.  **ProLex.Infrastructure :** Gère la persistance réelle (EF Core, PostgreSQL), la sécurité, l'envoi de SMS et la communication avec les API d'IA (Google Cloud Vision et Gemini).
4.  **ProLex.WebAPI :** Point d'entrée de l'application. Reçoit les requêtes HTTP, gère les sessions et orchestre l'Inversion de Contrôle (IoC).

---

## 4. Modèle de Données & Cœur d'Audit (Couche Domain)

Toutes les entités de l'application ProLex dérivent obligatoirement d'une classe abstraite unique. Celle-ci intègre un identifiant unique (GUID), les champs d'audit d'horodatage et d'identité, ainsi que le support de la suppression logique.

### L'Entité de Base : `ProLex.Domain.Common.EntityBase`
```csharp
namespace ProLex.Domain.Common;

public abstract class EntityBase
{
    public Guid Id { get; set; } = Guid.NewGuid();
    
    // Champs d'audit d'horodatage
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    public DateTime? DeletedAt { get; set; }
    
    // Champs d'audit d'identité (Stocke l'ID ou l'Email de l'utilisateur)
    public string? CreatedBy { get; set; }
    public string? ModifiedBy { get; set; }
    public string? DeletedBy { get; set; }
    
    // Marqueur de suppression logique
    public bool IsDeleted { get; set; } = false;
}
```

### Exemple d'application : L'Entité `Dossier`
```csharp
using ProLex.Domain.Common;

namespace ProLex.Domain.Entities;

public class Dossier : EntityBase
{
    public required string CaseNumber { get; set; }
    public required string CourtName { get; set; }
    public required string ClientName { get; set; }
    public required string OpponentName { get; set; }
    public required string Status { get; set; }
}
```

---

## 5. Abstraction & Services (Couche Application)

### Le Contrat d'Identité : `ProLex.Application.Common.Interfaces.ICurrentUserService`
Cette interface permet à la couche de persistance de connaître l'utilisateur actif de la session, qu'il s'agisse d'un vrai jeton JWT en production ou d'un utilisateur simulé en démo.
```csharp
namespace ProLex.Application.Common.Interfaces;

public interface ICurrentUserService
{
    string? UserId { get; } // Renvoie l'ID ou l'email de l'utilisateur connecté
}
```

### Le Contrat du Contexte de Données : `ProLex.Application.Common.Interfaces.IProLexDbContext`
```csharp
using Microsoft.EntityFrameworkCore;
using ProLex.Domain.Entities;

namespace ProLex.Application.Common.Interfaces;

public interface IProLexDbContext
{
    DbSet<Dossier> Dossiers { get; }
    DbSet<Document> Documents { get; }
    DbSet<DocumentTranslation> DocumentTranslations { get; }

    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
}
```

---

## 6. Implémentations & Traitement de l'Audit (Couche Infrastructure)

La couche Infrastructure fournit l'implémentation de production basée sur EF Core. C'est ici que l'interception des états et l'automatisation du Soft Delete sont configurées.

### L'Implémentation de Production : `ProLex.Infrastructure.Persistence.ProLexDbContext`
```csharp
using Microsoft.EntityFrameworkCore;
using ProLex.Application.Common.Interfaces;
using ProLex.Domain.Common;
using ProLex.Domain.Entities;

namespace ProLex.Infrastructure.Persistence;

public class ProLexDbContext : DbContext, IProLexDbContext
{
    private readonly ICurrentUserService _currentUserService;

    public DbSet<Dossier> Dossiers => Set<Dossier>();
    public DbSet<Document> Documents => Set<Document>();
    public DbSet<DocumentTranslation> DocumentTranslations => Set<DocumentTranslation>();

    public ProLexDbContext(
        DbContextOptions<ProLexDbContext> options, 
        ICurrentUserService currentUserService) : base(options)
    {
        _currentUserService = currentUserService;
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configuration globale de l'inspection des entités
        foreach (var entity in modelBuilder.Model.GetEntityTypes())
        {
            // 1. Application automatique du standard PascalCasing pour PostgreSQL
            entity.SetTableName(entity.DisplayName());
            foreach (var property in entity.GetProperties())
            {
                property.SetColumnName(property.Name);
            }

            // 2. Configuration automatique du filtre de suppression logique (Soft Delete)
            if (typeof(EntityBase).IsAssignableFrom(entity.ClrType))
            {
                modelBuilder.Entity(entity.ClrType)
                    .HasQueryFilter(ConvertFilterExpression(entity.ClrType));
            }
        }

        // Configuration pour le format JSONB natif de PostgreSQL
        modelBuilder.Entity<DocumentTranslation>()
            .Property(b => b.TranslatedSummary)
            .HasColumnType("jsonb");
    }

    // Générateur dynamique d'expression de filtre : e => !e.IsDeleted
    private static System.Linq.Expressions.LambdaExpression ConvertFilterExpression(Type type)
    {
        var parameter = System.Linq.Expressions.Expression.Parameter(type, "e");
        var property = System.Linq.Expressions.Expression.Property(parameter, nameof(EntityBase.IsDeleted));
        var falseConstant = System.Linq.Expressions.Expression.Constant(false);
        var comparison = System.Linq.Expressions.Expression.Equal(property, falseConstant);
        return System.Linq.Expressions.Expression.Lambda(comparison, parameter);
    }

    // Interception et automatisation de l'Audit & Soft Delete
    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        var currentUserId = _currentUserService.UserId ?? "System";

        foreach (var entry in ChangeTracker.Entries<EntityBase>())
        {
            switch (entry.State)
            {
                case EntityState.Added:
                    entry.Entity.CreatedAt = DateTime.UtcNow;
                    entry.Entity.CreatedBy = currentUserId;
                    entry.Entity.IsDeleted = false;
                    break;

                case EntityState.Modified:
                    entry.Entity.UpdatedAt = DateTime.UtcNow;
                    entry.Entity.ModifiedBy = currentUserId;
                    break;

                case EntityState.Deleted:
                    // Interception de la suppression physique -> Mutation en suppression logique
                    entry.State = EntityState.Modified;
                    entry.Entity.IsDeleted = true;
                    entry.Entity.DeletedAt = DateTime.UtcNow;
                    entry.Entity.DeletedBy = currentUserId;
                    break;
            }
        }

        return await base.SaveChangesAsync(cancellationToken);
    }
}
```
