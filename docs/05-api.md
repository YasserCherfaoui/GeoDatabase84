# 5. API REST

[← Sommaire](README.md)

Base : `https://api.geodatabase84.dz/v1` (nom de domaine à confirmer). Format JSON, dates ISO 8601, géométries en **GeoJSON**. Documentation interactive générée par Swagger sur `/v1/docs` (protégée en production).

## Conventions

| Sujet | Règle |
| --- | --- |
| Authentification | `Authorization: Bearer <accessToken>` (15 min) + cookie `refreshToken` HttpOnly (7 j) |
| Langue | `Accept-Language: fr` ou `?lang=fr` |
| Pagination | `?page=1&pageSize=20` → `{ data, meta: { page, pageSize, total } }` |
| Filtres | `?status=NEW&voletSlug=bativex&from=2026-10-01` |
| Erreurs | `{ statusCode, error, message, details? }` |
| Anti-spam | Champ `captchaToken` (reCAPTCHA v3 ou Turnstile) sur toutes les routes publiques en écriture |
| Limite de débit | 30 requêtes/min/IP sur les routes publiques en écriture |

## Routes publiques

| Méthode | Route | Description |
| --- | --- | --- |
| GET | `/volets` | Liste des volets publiés |
| GET | `/volets/:slug` | Volet avec catégories et prestations |
| GET | `/prestations/:id` | Fiche prestation (livrables, partenaire) |
| GET | `/trainings?domain=&level=&format=` | Catalogue de formations |
| GET | `/trainings/:slug` | Fiche formation |
| GET | `/pages/:key` | Blocs de texte d'une page (accueil, à propos, izira) |
| GET | `/hazards` | Types d'aléas (code, libellé, couleur, icône) |
| GET | `/layers` | Couches carto visibles (métadonnées) |
| GET | `/layers/:id/geojson` | Données GeoJSON d'une couche (mise en cache CDN) |
| GET | `/reports/published?bbox=w,s,e,n` | Signalements publiés dans une emprise |
| GET | `/partners?specialty=` | Annuaire partenaires |
| GET | `/service-offers` | Services réservables |
| GET | `/availability?serviceId=&from=&to=` | Créneaux libres |
| POST | `/files/upload-url` | URL signée pour déposer un fichier |
| POST | `/requests` | Créer une demande d'étude / formation |
| POST | `/appointments` | Réserver un RDV |
| POST | `/appointments/:ref/cancel?token=` | Annuler depuis l'email |
| POST | `/appointments/:ref/reschedule?token=` | Reporter depuis l'email |
| POST | `/reports` | Signaler un risque |
| POST | `/contact` | Message de contact |

### Exemple — créer une demande d'étude

```http
POST /v1/requests
Content-Type: application/json
Accept-Language: fr
```

```json
{
  "type": "STUDY",
  "prestationId": "8c1e…",
  "fullName": "Karim B.",
  "organization": "BET Atlas",
  "orgType": "BET",
  "email": "karim@bet-atlas.dz",
  "phone": "+213550000000",
  "location": {
    "type": "Polygon",
    "coordinates": [[[3.05,36.75],[3.06,36.75],[3.06,36.76],[3.05,36.76],[3.05,36.75]]]
  },
  "wilaya": "Alger",
  "commune": "Bab Ezzouar",
  "deliverables": ["MAP", "SECTION", "REPORT"],
  "description": "Étude géotechnique préliminaire pour un R+5.",
  "wishedDate": "2026-12-15",
  "attachments": ["uploads/2026/10/abc123-sondages.pdf"],
  "consent": true,
  "captchaToken": "…"
}
```

```json
HTTP/1.1 201 Created
{ "id": "f5a0…", "reference": "DE-2026-0042", "status": "NEW" }
```

### Exemple — réserver un RDV

```json
POST /v1/appointments
{
  "serviceOfferId": "bativex-interpretation",
  "mode": "ONLINE",
  "startsAt": "2026-11-03T10:00:00+01:00",
  "fullName": "Sara M.",
  "email": "sara@exemple.dz",
  "phone": "+213661000000",
  "expectedDeliverable": "Synthèse des essais",
  "description": "Interprétation de 4 essais pressiométriques.",
  "captchaToken": "…"
}
```

Réponse `409 Conflict` si le créneau vient d'être pris : le front recharge les disponibilités.

## Routes protégées (back-office)

| Méthode | Route | Rôle minimum |
| --- | --- | --- |
| POST | `/auth/login` · `/auth/refresh` · `/auth/logout` | — |
| POST | `/auth/forgot-password` · `/auth/reset-password` | — |
| GET | `/admin/dashboard` | EDITOR |
| CRUD | `/admin/volets`, `/admin/categories`, `/admin/prestations`, `/admin/deliverables` | EDITOR |
| CRUD | `/admin/trainings`, `/admin/pages` | EDITOR |
| GET | `/admin/requests?status=&volet=&q=` | ADMIN |
| PATCH | `/admin/requests/:id/status` | ADMIN |
| POST | `/admin/requests/:id/notes` | ADMIN |
| GET | `/admin/requests/export.csv` | ADMIN |
| CRUD | `/admin/availability-rules`, `/admin/service-offers` | ADMIN |
| GET/PATCH | `/admin/appointments` | ADMIN |
| GET | `/admin/reports?moderation=PENDING` | ADMIN |
| PATCH | `/admin/reports/:id/moderate` | ADMIN |
| POST | `/admin/layers/import` (SHP zip, KML, GeoJSON) | ADMIN |
| PATCH/DELETE | `/admin/layers/:id` | ADMIN |
| CRUD | `/admin/partners`, `/admin/plans`, `/admin/subscriptions` | ADMIN |
| CRUD | `/admin/users` | ADMIN |
| GET | `/admin/audit` | ADMIN |
| GET/PATCH | `/partner/me` | PARTNER |
| POST | `/jobs/reminders` · `/jobs/subscriptions` | Jeton Cloud Scheduler |

## Codes d'erreur métier

| Code | Cas |
| --- | --- |
| `SLOT_TAKEN` (409) | Créneau déjà réservé |
| `SLOT_OUTSIDE_AVAILABILITY` (422) | Créneau hors des plages ouvertes |
| `GEOMETRY_INVALID` (422) | Polygone auto-intersecté ou hors Algérie (si restriction activée) |
| `GEOMETRY_TOO_LARGE` (422) | Zone > seuil (ex. 10 000 km²) |
| `FILE_TYPE_NOT_ALLOWED` (415) | Extension refusée |
| `FILE_TOO_LARGE` (413) | > 25 Mo (50 Mo pour SHP) |
| `CAPTCHA_FAILED` (403) | Score anti-spam trop bas |
| `LAYER_IMPORT_FAILED` (422) | Fichier SIG illisible ou sans projection |
