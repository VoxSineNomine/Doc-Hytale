# 📘 Commande Player – Propulsion verticale et pose de bloc différée (Hytale)

Cette documentation décrit une commande joueur permettant :
- d'analyser l'environnement immédiat du joueur,
- d'appliquer une impulsion verticale contrôlée,
- puis de poser un bloc précis après un court délai,
tout en respectant les contraintes du **thread monde** et de l’**ECS Hytale**.

---

## 🧩 Bloc 1 — Validation du contexte et récupération des données joueur

Objectif : garantir que la commande s’exécute dans un contexte valide et que les composants nécessaires sont disponibles.

```java
if (!ref.isValid()) return; 
// Vérifie que l'entité référencée existe toujours dans le Store

Player player = store.getComponent(ref, Player.getComponentType());
// Récupère le composant Player (vue logique / gameplay)

if (player == null) return; 
// Sécurité : si le joueur n'existe plus, on stop immédiatement

TransformComponent transform = store.getComponent(ref, TransformComponent.getComponentType());
// Récupère le composant Transform lié à la position et rotation

if (transform == null) {
    errorMessage(player); 
    // Envoie un message d'erreur au joueur si le Transform est absent
    return;
}
```

### Schéma fonctionnel

```
[Commande Joueur]
        |
        v
   [Ref valide ?] -- non --> STOP
        |
       oui
        |
   [Player présent ?] -- non --> STOP
        |
       oui
        |
 [TransformComponent ?] -- non --> Message erreur
        |
       oui
```

---

## 📐 Bloc 2 — Analyse de l’environnement (position, fluide, bloc cible)

Objectif : déterminer si les conditions environnementales permettent l’action.

```java
Vector3d pos = transform.getPosition();
// Récupère la position du joueur en coordonnées monde (double)

int bx = (int) Math.floor(pos.getX());
// Conversion coordonnée X monde -> coordonnée bloc

int by = (int) Math.floor(pos.getY()) + 1;
// Conversion coordonnée Y monde -> bloc juste au-dessus du joueur

int bz = (int) Math.floor(pos.getZ());
// Conversion coordonnée Z monde -> coordonnée bloc

int fluidHere = world.getFluidId(bx, (int) Math.floor(pos.getY()), bz);
// Vérifie la présence d’un fluide à la position du joueur

int fluidAbove = world.getFluidId(bx, (int) Math.floor(pos.getY()) + 2, bz);
// Vérifie qu’aucun fluide n’est présent au-dessus du joueur

if (!(fluidHere != 0 && fluidAbove == 0)) {
    errorMessage(player); 
    // Conditions non remplies : on empêche l’exécution
    return;
}

BlockType block = world.getBlockType(bx, by, bz);
// Récupère le type de bloc ciblé dans le monde

if (block == null) {
    errorMessage(player); 
    // Sécurité : bloc invalide ou chunk non chargé
    return;
}

if (!block.getGroup().equals("AIR")) {
    errorMessage(player); 
    // Le bloc ciblé n'est pas compatible avec l'action
    return;
}
```

### Schéma fonctionnel

```
[Position Joueur]
        |
        v
[Conversion en blocs bx/by/bz]
        |
        v
[Fluide présent ici ?] -- non --> STOP
        |
       oui
        |
[Fluide au-dessus ?] -- oui --> STOP
        |
       non
        |
[Bloc cible == AIR ?] -- non --> STOP
        |
       oui
```

---

## 🚀 Bloc 3 — Exécution sur le thread monde et actions différées

Objectif : appliquer une impulsion verticale et poser un bloc après un délai non bloquant.

### Application de la vélocité

```java
world.execute(() -> {
    // Planifie l'exécution sur le thread monde (thread-safe)

    if (!ref.isValid()) return;
    // Re-vérifie la validité de l'entité

    Velocity vel = store.getComponent(ref, Velocity.getComponentType());
    // Récupère le composant de vélocité du joueur

    if (vel == null) return;
    // Sécurité : si le composant est absent, on stop

    double up = 30.0;
    // Intensité de la poussée verticale (à calibrer selon le gameplay)

    vel.addInstruction(
        new Vector3d(0.0, up, 0.0),
        // Vecteur de vélocité uniquement sur l'axe Y

        new VelocityConfig(),
        // Configuration de la modification de vélocité

        ChangeVelocityType.Add
        // Ajoute la vélocité sans écraser le mouvement existant
    );
});
```

### Pose du bloc après un délai

```java
CompletableFuture
    .runAsync(() -> { }, CompletableFuture.delayedExecutor(250, TimeUnit.MILLISECONDS))
    // Lance une tâche différée sans bloquer le thread monde

    .thenRun(() -> world.execute(() -> {
        // Retour contrôlé sur le thread monde

        if (!ref.isValid()) return;
        // Vérifie que le joueur est toujours valide

        BlockType block2 = world.getBlockType(bx, by, bz);
        // Relecture du bloc ciblé au moment de l'exécution

        if (block2 == null) return;
        // Sécurité : bloc ou chunk indisponible

        if (!block2.getGroup().equals("AIR")) return;
        // Évite d'écraser un bloc apparu entre temps

        world.setBlock(bx, by, bz, "Rock_Ice");
        // Pose du bloc définitif à la position calculée
    }))
    .exceptionally(ex -> {
        // Capture et log toute erreur survenue dans la chaîne asynchrone
        ex.printStackTrace();
        return null;
    });
```

---

## ✅ Résumé technique

- Toutes les mutations ECS et monde sont exécutées sur le thread monde.
- Aucun blocage du tick serveur.
- Les délais sont gérés de manière asynchrone et sécurisée.
- Les vérifications sont répétées aux points critiques pour éviter les états invalides.
