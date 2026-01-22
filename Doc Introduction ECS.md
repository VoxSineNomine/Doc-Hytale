<!--
Documentation interne – ECS & Stores (Hytale Server)
Format : GitHub Markdown (README.md)
Usage : Onboarding développeurs / Modding
-->

# Architecture ECS (Entity Component System)

## Table des matières
- Présentation générale
- Objectif de l’ECS
- Vue d’ensemble conceptuelle
- Les éléments fondamentaux de l’ECS
  - Entity
  - Components
  - Systems
- Interaction Entity / Components / Systems
- Usage interne – Onboarding et modding

---

# Architecture ECS (Entity Component System)

> **Documentation interne développeurs – Onboarding Hytale Server**
>
> Objectif : fournir une compréhension rapide, structurée et exploitable de l’architecture ECS et des Stores utilisée par Hypixel Studios, afin de permettre à une nouvelle équipe de développeurs d’être opérationnelle rapidement sur un serveur Hytale.
>
> Public visé : développeurs backend / gameplay / systèmes, débutants ou intermédiaires sur l’ECS.

---


## Présentation générale

> Hypixel Studios à choisit pour son jeu de développé une architecture connu du monde du jeu vidéo, l' ECS, soit Entity Components Systems, cette architecture permet de séparer les Entités de leurs datas et fonction système, Il se focalise sur la capacité à séparer ces entités de tout ce qui les rattache et via ceci on sort du cadre habituelle d'entités dont les datas et fonctions systèmes sont hiérarchisés, celles si sont contenus dans des Stores qui sont facilement accessible et réutilisable.

---

## Objectif de l’ECS

> Entre autre l'ECS permet de décharger certaines fonctions qui ne sont pas nécessaire au code mais liées a l'entité. Je m'explique pour prendre exemple dans Minecraft une EntityPlayer est une classe qui est elle même sous hérité d'une classe et qui elle même possède d'innombrables classes ou méthodes sous hérité, l'ECS permet de choisir spécifiquement quel data pour quel entité et quel thread system pour quel entité.

---

# Les éléments fondamentaux de l’ECS

---

# Vue d’ensemble conceptuelle

```
┌───────────┐        ┌──────────────────┐        ┌───────────────┐
│  Entity   │ ────▶  │      Stores      │ ◀────  │    Systems    │
│ (ID / Key)│        │ (Components data)│        │ (Logic only)  │
└───────────┘        └──────────────────┘        └───────────────┘
```

- L’**Entity** est une clé
- Les **Stores** contiennent les données (Components)
- Les **Systems** lisent et modifient les données via les stores

Aucune logique n’est stockée dans l’entité. Aucune donnée n’est stockée dans les systèmes.

---


## Entity

### Rôle dans l’architecture


> Une Entity reprèsente la réference, ce n'est qu'un identifiant une clé d'accès pour les stores.
> Elle ne contient aucune data ni aucune méthodes

> Voyez ceci comme la clé qui ouvre le tiroir des data.

### Schéma conceptuel

```
Entity (ID)
   │
   ├──▶ PositionComponent
   ├──▶ HealthComponent
   ├──▶ InventoryComponent
```

L’entité n’existe que par son identifiant. Sans component associé dans un store, elle ne représente rien de fonctionnel.


---

## Components

### Rôle dans l’architecture


> Les components représente uniquement les data, elles ne contiennent que des valeurs et aucune méthode, seulement des données liée à la clé (réference) qu'on a indiquer dans notre variable pour récuperer ses datas.

---

## Systems

### Rôle dans l’architecture


> Ici on stock tout ce qui touche à la logique fonctionnelles, les méthodes y sont stockés, c'est ici que l'on va pouvoir effectuer une action sur une entité, elle nous permet d'isoler chaque méthodes et interaction qui pourrait ralentir le jeu si elle était simplement sous hérité à une classe entité.

---

# Interaction Entity / Components / Systems (flux logique)

```
[ System Tick / Thread ]
        │
        ▼
┌─────────────────────┐
│     System Logic    │
│ (ex: MovementSystem)│
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│      Store          │
│ PositionComponent   │
│ VelocityComponent   │
└─────────┬───────────┘
          │
          ▼
      Entity ID
```

- Le système sélectionne les entités possédant les components requis
- Il lit les données
- Il applique une logique
- Il écrit les nouvelles valeurs dans les stores

---

# Lecture visuelle simplifiée (récapitulatif)

- **Entity** : clé / identifiant
- **Components** : tiroirs de données
- **Systems** : logique et actions

L’entité n’agit jamais seule, elle est manipulée par les systèmes à partir des données stockées dans les components via les stores.

---

# Usage interne – Onboarding et modding

Ce découpage permet une compréhension claire pour les développeurs impliqués dans le modding :
- aucune logique cachée dans une entité
- aucune méthode dans les données
- une séparation stricte entre structure et comportement

Cette organisation favorise la lisibilité, la performance et la réutilisation du code dans un contexte serveur Hytale.

