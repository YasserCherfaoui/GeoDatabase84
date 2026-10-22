# 9. Planning et lots

[← Sommaire](README.md)

Durée totale : **16 semaines** à compter de l'acompte et de la réception des contenus. Chaque lot est validé par la cliente sous 5 jours ouvrés ; tout retard de validation ou de contenu décale d'autant la livraison.

```mermaid
gantt
  title GéoData Base 84 — planning de réalisation
  dateFormat  YYYY-MM-DD
  axisFormat  S%W
  excludes    weekends

  section Lot 1 — Vitrine
  Charte et maquettes            :l1a, 2026-11-02, 10d
  Validation maquettes           :milestone, m1, after l1a, 0d
  Socle technique (monorepo, CI) :l1b, 2026-11-02, 5d
  API, auth, modèle de données   :l1c, after l1b, 10d
  Pages volets + contenus        :l1d, after l1a, 15d
  Demande d'étude + emails       :l1e, after l1c, 7d
  Back-office contenus/demandes  :l1f, after l1e, 8d
  Recette lot 1                  :milestone, m2, 2026-12-11, 0d

  section Lot 2 — Tessera et RDV
  Carte Google 3D + couches      :l2a, 2026-12-14, 12d
  Import SIG (SHP/KML/GeoJSON)   :l2b, after l2a, 5d
  Signalement + modération       :l2c, 2026-12-14, 6d
  Réservation RDV + Meet         :l2d, after l2c, 8d
  Recette lot 2                  :milestone, m3, 2027-01-15, 0d

  section Lot 3 — Partenaires, langue, SEO
  Annuaire + abonnements         :l3a, 2027-01-18, 7d
  Seconde langue                 :l3b, 2027-01-18, 6d
  SEO + statistiques             :l3c, after l3b, 4d
  Recette lot 3                  :milestone, m4, 2027-02-05, 0d

  section Mise en service
  Recette finale + correctifs    :ms1, 2027-02-08, 5d
  Déploiement + formation        :ms2, after ms1, 5d
  Mise en ligne                  :milestone, m5, 2027-02-19, 0d
```

Les dates sont indicatives (départ supposé le 2 novembre 2026) et seront recalées sur la date réelle de l'acompte.

## Contenu des lots

| Lot | Contenu | Montant HT (proforma) |
| --- | --- | --- |
| 1 | Maquettes (offertes), pages vitrine, API + base, demande d'étude, back-office | 760 000 DA |
| 2 | Carte Google 3D, signalement, réservation de RDV | 470 000 DA |
| 3 | Partenaires, seconde langue (offerte), SEO et statistiques | 170 000 DA |
| Mise en service | Déploiement Google Cloud, CI/CD, formation | 90 000 DA |
| **Total** | | **1 490 000 DA HT** |

## Jalons de paiement

```mermaid
flowchart LR
  A[Commande<br/>40 %] --> B[Livraison lot 1<br/>30 %] --> C[Livraison finale<br/>30 %]
```

## Définition de « terminé » pour chaque fonctionnalité

- Le comportement décrit dans sa fiche est implémenté et ses critères d'acceptation sont vérifiés.
- Responsive testé sur mobile (375 px), tablette et bureau.
- Textes en français en place ; clés de traduction prêtes pour la seconde langue.
- Tests unitaires sur les règles métier, test e2e sur le parcours principal.
- Déployé en recette et présenté à la cliente.
