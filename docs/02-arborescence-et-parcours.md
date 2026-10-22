# 2. Arborescence et parcours utilisateurs

[← Sommaire](README.md)

## Plan du site

```mermaid
flowchart TB
  H[En-tête : logo · menu · langue · Contact]
  H --> ACC[/ Accueil/]
  H --> APR[/a-propos/]
  H --> IZ[/izira/]
  H --> GDB[/geodatabase/]
  H --> BV[/bativex/]
  H --> TS[/tessera/]
  H --> BR[/batirecycle/]
  H --> GA[/geoassistance/]
  H --> CT[/contact/]

  GDB --> GDBc[/geodatabase/:categorie/]
  GDBc --> GDBp[/geodatabase/:categorie/:prestation/]
  BV --> BVp[/bativex/:prestation/]
  BR --> BRp[/batirecycle/:prestation/]
  TS --> TSc[/tessera/carte/]
  TS --> TSs[/tessera/:prestation/]
  GA --> GAf[/geoassistance/:formation/]

  GDBp & BVp & BRp & TSs --> DE[/demande-etude?prestation=…/]
  BV & IZ & GA --> RDV[/rendez-vous?service=…/]
  TSc --> SIG[/tessera/signaler/]

  ADM[/admin/*  back-office/]
```

## Routes front-end (TanStack Router)

Toutes les routes publiques sont préfixées par la langue : `/fr/...`, `/en/...` (ou `/ar/...`).

| Route | Écran | Rendu |
| --- | --- | --- |
| `/$lang` | Accueil | Pré-rendu (SSG) |
| `/$lang/a-propos` | À propos | SSG |
| `/$lang/izira` | IZIRA | SSG |
| `/$lang/geodatabase` | Catalogue GéoDataBase | SSG |
| `/$lang/geodatabase/$categorie/$prestation` | Fiche prestation | SSG |
| `/$lang/bativex` · `/$lang/bativex/$prestation` | BatiVex | SSG |
| `/$lang/tessera` | Présentation Tessera | SSG |
| `/$lang/tessera/carte` | Carte 3D | Client (SPA) |
| `/$lang/tessera/signaler` | Signalement | Client |
| `/$lang/batirecycle` · `/$prestation` | BatiRecycle | SSG |
| `/$lang/geoassistance` · `/$formation` | Catalogue de formations | SSG |
| `/$lang/demande-etude` | Formulaire de demande | Client |
| `/$lang/rendez-vous` | Réservation | Client |
| `/$lang/contact` | Contact | SSG + formulaire client |
| `/admin/*` | Back-office | Client, protégé |

## Navigation

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ [LOGO]  Accueil  À propos  Moi  GéoDataBase  BatiVex  Tessera  BatiRecycle   │
│         GeoAssistance                         [FR ▾]  [Contact]  [Demander]  │
└──────────────────────────────────────────────────────────────────────────────┘
```

Sur mobile : logo + bouton menu (tiroir latéral). Le sélecteur de langue et le bouton **« Demander une étude »** restent visibles en permanence.

## Parcours 1 — Un BET demande une étude de sol

```mermaid
journey
  title Bureau d'études : de la recherche à la demande
  section Découverte
    Arrive depuis Google sur "interprétation essais CBR": 4: Visiteur
    Lit la fiche BatiVex: 4: Visiteur
  section Demande
    Clique "Demander une étude": 5: Visiteur
    Pointe son terrain sur la carte: 4: Visiteur
    Coche les livrables attendus: 4: Visiteur
    Joint un rapport de sondage PDF: 3: Visiteur
    Envoie et reçoit un email de confirmation: 5: Visiteur
  section Traitement
    Reçoit la notification: 5: Administratrice
    Passe la demande "En cours" puis "Devis envoyé": 4: Administratrice
```

## Parcours 2 — Un porteur de projet réserve un coaching

```mermaid
sequenceDiagram
  actor P as Porteur de projet
  participant S as Site (IZIRA)
  participant R as Module RDV
  participant API as API NestJS
  participant M as Email

  P->>S: Lit "Coaching entrepreneurial"
  P->>R: Clique "Réserver une séance"
  R->>API: GET /availability?service=coaching
  API-->>R: Créneaux libres
  P->>R: Choisit "En ligne", mardi 10h
  R->>API: POST /appointments
  API-->>M: Confirmation + lien Google Meet
  M-->>P: Email + fichier .ics
  Note over API,M: Rappel automatique 24 h avant
```

## Parcours 3 — Un citoyen signale un glissement de terrain

```mermaid
flowchart LR
  A[Ouvre Tessera / carte] --> B[Clique sur le lieu]
  B --> C[Choisit « Signaler un risque »]
  C --> D[Type : glissement<br/>photos, date, description]
  D --> E{Anonyme ?}
  E -- oui --> F[Envoi sans coordonnées]
  E -- non --> G[Envoi avec email]
  F & G --> H[Statut : À modérer]
  H --> I{Modération}
  I -- validé --> J[Publié sur la carte]
  I -- rejeté --> K[Archivé]
```

## Parcours 4 — Un étudiant s'inscrit à une formation

```mermaid
flowchart LR
  A[GeoAssistance] --> B[Filtre : Géotechnique · Débutant]
  B --> C[Fiche « Essais Proctor et CBR »]
  C --> D[« S'inscrire / Demander des infos »]
  D --> E[Formulaire court<br/>nom, email, niveau, message]
  E --> F[Demande enregistrée<br/>type = formation]
  F --> G[Email de confirmation]
```
