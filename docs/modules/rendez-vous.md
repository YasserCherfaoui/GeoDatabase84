# Réservation de rendez-vous

[← Sommaire](../README.md) · Lot 2 · Route `/$lang/rendez-vous?service=…`

## Objectif

Permettre de réserver un créneau pour un service BatiVex, une séance de coaching IZIRA ou un échange formation, en présentiel ou en ligne.

## Paramétrage côté administratrice

```mermaid
flowchart LR
  SO[Services réservables<br/>nom, durée, modalités autorisées,<br/>livrables proposés, prix indicatif]
  AR[Règles de disponibilité<br/>ex. Lu–Je 9h–12h et 14h–17h]
  EX[Exceptions<br/>congés, jours fériés, créneaux ajoutés]
  BUF[Marges<br/>battement entre RDV, délai minimum de réservation]
  SO & AR & EX & BUF --> CALC[Calcul des créneaux libres]
```

## Calcul des créneaux

```mermaid
flowchart TB
  A[Plages des règles<br/>sur la période demandée] --> B[− exceptions]
  B --> C[− RDV déjà PENDING/CONFIRMED<br/>+ marges]
  C --> D[− créneaux avant<br/>maintenant + délai minimum]
  D --> E[Découpage selon<br/>la durée du service]
  E --> F[Créneaux proposés<br/>fuseau Africa/Algiers]
```

La contrainte d'unicité en base (`starts_at`, `service_offer_id`) empêche deux réservations simultanées sur le même créneau : la seconde reçoit `409 SLOT_TAKEN` et voit les créneaux rechargés.

## Parcours

```mermaid
sequenceDiagram
  autonumber
  actor C as Client
  participant W as Web
  participant API as API
  participant G as Google Calendar API
  participant M as Email
  participant S as Cloud Scheduler

  C->>W: Service, modalité
  W->>API: GET /availability
  API-->>W: Créneaux libres
  C->>W: Créneau, livrable attendu, coordonnées
  W->>API: POST /appointments
  API->>API: Verrou + contrôle du créneau
  alt En ligne
    API->>G: Créer événement avec Meet
    G-->>API: Lien Meet
  end
  API->>M: Confirmation (+ .ics, lien Meet ou adresse, liens annuler/reporter)
  API->>M: Notification administratrice
  API-->>W: 201 RDV-2026-0017
  S->>API: /jobs/reminders (toutes les 15 min)
  API->>M: Rappel 24 h avant
```

## Annulation et report

- Chaque email contient deux liens signés (jeton unique) : **Annuler** et **Reporter**.
- Annulation possible jusqu'à 12 h avant (paramétrable) ; au-delà, le lien invite à contacter directement.
- Report : réouvre le sélecteur de créneaux, libère l'ancien créneau, envoie un nouvel email.

## Vue back-office

```
┌─────────────────────────────────────────────────────────────┐
│ Rendez-vous            [Semaine ▾]  ◀ 2–6 nov. 2026 ▶       │
├──────┬──────────┬──────────┬──────────┬──────────┬──────────┤
│      │ Lun 2    │ Mar 3    │ Mer 4    │ Jeu 5    │ Ven 6    │
│ 09:00│          │■ Coaching│          │■ BatiVex │          │
│      │          │ en ligne │          │présentiel│          │
│ 10:00│■ BatiVex │          │          │          │          │
│ 14:00│          │          │░ congé ░ │          │          │
├──────┴──────────┴──────────┴──────────┴──────────┴──────────┤
│ Clic → détail : client, livrable, description, lien Meet,   │
│ [Confirmer] [Reporter] [Annuler] [Marquer effectué/absent]  │
└─────────────────────────────────────────────────────────────┘
```

## Critères d'acceptation

- [ ] Aucun créneau hors disponibilités n'est proposé.
- [ ] Deux personnes ne peuvent pas réserver le même créneau.
- [ ] Le lien Meet est présent pour les RDV en ligne.
- [ ] Le rappel part 24 h avant ; annulation et report fonctionnent depuis l'email.
