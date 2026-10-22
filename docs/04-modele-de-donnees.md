# 4. Modèle de données

[← Sommaire](README.md)

Base : **Neon (PostgreSQL 16 + PostGIS)**. Les géométries sont stockées en `geometry(…, 4326)` (WGS 84, le système de Google Maps).

## Diagramme entité-relation

```mermaid
erDiagram
  USER ||--o{ AUDIT_LOG : "effectue"
  USER ||--o{ REQUEST_NOTE : "écrit"

  VOLET ||--o{ CATEGORY : "contient"
  CATEGORY ||--o{ PRESTATION : "contient"
  PRESTATION }o--o{ DELIVERABLE : "propose"
  PRESTATION }o--o| PARTNER : "réalisée avec"

  VOLET ||--o{ TRAINING : "propose"
  TRAINING_DOMAIN ||--o{ TRAINING : "classe"

  PRESTATION ||--o{ STUDY_REQUEST : "fait l'objet de"
  STUDY_REQUEST }o--o{ DELIVERABLE : "demande"
  STUDY_REQUEST ||--o{ ATTACHMENT : "joint"
  STUDY_REQUEST ||--o{ REQUEST_NOTE : "annoté par"
  STUDY_REQUEST ||--o{ STATUS_HISTORY : "trace"

  SERVICE_OFFER ||--o{ APPOINTMENT : "réservé pour"
  AVAILABILITY_RULE ||--o{ APPOINTMENT : "encadre"

  HAZARD_TYPE ||--o{ RISK_REPORT : "qualifie"
  HAZARD_TYPE ||--o{ MAP_LAYER : "qualifie"
  RISK_REPORT ||--o{ ATTACHMENT : "photos"

  PARTNER ||--o{ SUBSCRIPTION : "souscrit"
  PLAN ||--o{ SUBSCRIPTION : "définit"

  USER {
    uuid id PK
    string email UK
    string password_hash
    enum role "ADMIN | EDITOR | PARTNER"
    uuid partner_id FK "nullable"
    timestamptz last_login_at
  }
  VOLET {
    uuid id PK
    string slug UK "geodatabase, bativex..."
    jsonb title "fr/en/ar"
    jsonb tagline
    string hero_image
    string accent_color
    int sort_order
    bool published
  }
  CATEGORY {
    uuid id PK
    uuid volet_id FK
    string slug
    jsonb title
    jsonb description
    int sort_order
  }
  PRESTATION {
    uuid id PK
    uuid category_id FK
    string slug
    jsonb title
    jsonb description
    bool requires_partner
    bool requires_location
    enum status "DRAFT | PUBLISHED"
  }
  DELIVERABLE {
    uuid id PK
    string code "MAP, SECTION, GIS_DB, MODEL_3D, REPORT..."
    jsonb label
  }
  TRAINING {
    uuid id PK
    uuid domain_id FK
    string slug
    jsonb title
    enum level "BEGINNER | INTERMEDIATE | ADVANCED"
    enum format "ONSITE | ONLINE | HYBRID"
    int duration_hours
  }
  STUDY_REQUEST {
    uuid id PK
    string reference UK "DE-2026-0042"
    enum type "STUDY | TRAINING | INFO"
    uuid prestation_id FK
    string full_name
    string organization
    enum org_type "COMPANY | BET | INSTITUTION | STUDENT | OTHER"
    string email
    string phone
    geometry location "Point ou Polygon, 4326"
    string wilaya
    string commune
    text description
    date wished_date
    enum status "NEW | ANALYSIS | QUOTE_SENT | ACCEPTED | DELIVERED | CLOSED | REJECTED"
    string locale
    bool consent
    timestamptz created_at
  }
  APPOINTMENT {
    uuid id PK
    string reference UK "RDV-2026-0017"
    uuid service_offer_id FK
    enum mode "ONSITE | ONLINE"
    timestamptz starts_at
    int duration_min
    string full_name
    string email
    string phone
    string expected_deliverable
    text description
    string meet_url
    string cancel_token
    enum status "PENDING | CONFIRMED | CANCELLED | DONE | NO_SHOW"
  }
  RISK_REPORT {
    uuid id PK
    uuid hazard_type_id FK
    geometry location "Point ou Polygon"
    date observed_on
    text description
    string reporter_email "nullable"
    bool anonymous
    enum moderation "PENDING | APPROVED | REJECTED"
    bool published_on_map
  }
  MAP_LAYER {
    uuid id PK
    jsonb name
    uuid hazard_type_id FK
    string source_file
    string geojson_file
    jsonb style "couleurs, opacité, légende"
    bool visible_by_default
    int sort_order
  }
  HAZARD_TYPE {
    uuid id PK
    string code "EARTHQUAKE, LANDSLIDE..."
    jsonb label
    string color
    string icon
  }
  PARTNER {
    uuid id PK
    string name
    string logo
    jsonb specialties
    geometry location
    string website
    bool featured
  }
  SUBSCRIPTION {
    uuid id PK
    uuid partner_id FK
    uuid plan_id FK
    date starts_on
    date ends_on
    enum status "ACTIVE | EXPIRED | CANCELLED"
  }
  ATTACHMENT {
    uuid id PK
    string owner_type "REQUEST | REPORT"
    uuid owner_id
    string storage_key
    string mime
    int size_bytes
  }
```

Tables secondaires non détaillées dans le schéma : `TRAINING_DOMAIN`, `SERVICE_OFFER` (services réservables : coaching, BatiVex, formation), `AVAILABILITY_RULE` (plages horaires récurrentes + exceptions), `PLAN` (paliers d'abonnement), `REQUEST_NOTE`, `STATUS_HISTORY`, `CONTACT_MESSAGE`, `AUDIT_LOG`, `PAGE_CONTENT` (blocs de texte libres des pages Accueil, À propos, IZIRA).

## Cycle de vie d'une demande d'étude

```mermaid
stateDiagram-v2
  [*] --> NEW : soumission
  NEW --> ANALYSIS : prise en charge
  NEW --> REJECTED : hors périmètre / spam
  ANALYSIS --> QUOTE_SENT : devis envoyé
  ANALYSIS --> REJECTED
  QUOTE_SENT --> ACCEPTED : client accepte
  QUOTE_SENT --> CLOSED : sans suite (30 j)
  ACCEPTED --> DELIVERED : livrables remis
  DELIVERED --> CLOSED
  REJECTED --> [*]
  CLOSED --> [*]
```

Chaque changement de statut crée une ligne dans `STATUS_HISTORY` (qui, quand, commentaire) et peut déclencher un email au demandeur.

## Cycle de vie d'un rendez-vous

```mermaid
stateDiagram-v2
  [*] --> PENDING : réservation
  PENDING --> CONFIRMED : validation (auto ou manuelle)
  PENDING --> CANCELLED
  CONFIRMED --> CANCELLED : annulation (lien email)
  CONFIRMED --> CONFIRMED : report
  CONFIRMED --> DONE : après la séance
  CONFIRMED --> NO_SHOW
  DONE --> [*]
  CANCELLED --> [*]
  NO_SHOW --> [*]
```

## Requêtes spatiales utiles

```sql
-- Aléas qui touchent la zone d'une demande
SELECT h.code, l.name
FROM study_request r
JOIN map_layer_feature f ON ST_Intersects(f.geom, r.location)
JOIN map_layer l ON l.id = f.layer_id
JOIN hazard_type h ON h.id = l.hazard_type_id
WHERE r.id = $1;

-- Signalements publiés dans la vue de la carte
SELECT id, hazard_type_id, ST_AsGeoJSON(location) AS geojson
FROM risk_report
WHERE published_on_map
  AND location && ST_MakeEnvelope($west, $south, $east, $north, 4326);

-- Surface d'une zone dessinée, en hectares
SELECT ST_Area(location::geography) / 10000 AS hectares FROM study_request WHERE id = $1;
```

## Index

| Table | Index | Raison |
| --- | --- | --- |
| `study_request` | GIST(`location`), BTREE(`status`, `created_at`) | Carte des demandes, liste back-office |
| `risk_report` | GIST(`location`), BTREE(`moderation`) | Affichage par emprise, file de modération |
| `map_layer_feature` | GIST(`geom`) | Intersections |
| `appointment` | UNIQUE(`starts_at`, `service_offer_id`) WHERE status IN (PENDING, CONFIRMED) | Empêche la double réservation |
| `prestation` | UNIQUE(`category_id`, `slug`) | URL lisibles |

## Textes multilingues

Les champs traduisibles sont en `jsonb` : `{"fr": "Cartographie géologique", "en": "Geological mapping"}`. L'API renvoie la langue demandée (en-tête `Accept-Language` ou paramètre `?lang=`) et retombe sur le français si la traduction manque.
