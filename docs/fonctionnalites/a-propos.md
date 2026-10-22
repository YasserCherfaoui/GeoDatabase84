# À propos

[← Sommaire](../README.md) · Lot 1 · Route `/$lang/a-propos`

## Objectif

Expliquer la mission et la démarche de GéoData Base 84. Reprend le texte de l'ancienne page d'accueil.

## Contenu

| Bloc | Contenu (ancienne page Accueil) |
| --- | --- |
| Approche | Visualisation scientifique, modélisation géospatiale, collecte numérique de données de terrain |
| Interdisciplinarité | Comprendre les territoires, anticiper les évolutions géologiques et environnementales |
| Mission | Fédérer une communauté autour de la connaissance ouverte, de l'innovation responsable et du partage d'expertise |
| Origine | Initiative algérienne, approche intégrée, collaborative et durable |

## Maquette fil de fer

```
┌───────────────────────────────────────────────────────┐
│ BANDEAU  « À propos de GéoData Base 84 »              │
├───────────────────────────────────────────────────────┤
│ Notre approche            │  [image terrain / SIG]    │
│ texte                     │                           │
├───────────────────────────────────────────────────────┤
│ ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│ │ Mission  │ │ Vision   │ │ Valeurs  │                │
│ └──────────┘ └──────────┘ └──────────┘                │
├───────────────────────────────────────────────────────┤
│ Une initiative algérienne        [Découvrir IZIRA →]  │
└───────────────────────────────────────────────────────┘
```

## Comportement

- Textes éditables (`PAGE_CONTENT`, clé `about`).
- Lien vers IZIRA pour présenter la fondatrice.
- Si la cliente choisit de fusionner « À propos » et « Moi » ([point ouvert 4](../10-points-ouverts.md)), cette page devient une section de la page IZIRA et la route redirige (301).

## Critères d'acceptation

- [ ] Contenu fourni intégralement repris.
- [ ] Page lisible sans image (textes alternatifs).
