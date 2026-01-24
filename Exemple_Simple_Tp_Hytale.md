# ECS Hytale – Exemple de commande découpé (format brut)

> **But** : illustrer, à travers un exemple simple de **téléportation d’un joueur**, le flux ECS standard côté serveur Hytale, en conservant le code strictement intact et en explicitant le cheminement `World → Store → Component`.


---

## Contexte, intention et accès ECS

- Toute Event Hytale est intrinsèquement liée à l’ECS.
- L’ECS est architecturalement rattaché au `World`.
- Les annotations `@Nonnull` définissent un contrat de non-null mais n’excluent pas les vérifications runtime.
- L’accès aux données ECS passe obligatoirement par le `World`, puis l’`EntityStore`, puis le `Store`.

```java
public class Commands extends AbstractPlayerCommand {

    /*
     Exemple d'une fonction Hytale, toute Event est intrinsèquement liée à l'ECS, qui lui est architecturalement liée au world.
     Une fonction moteur Hytale comportera un World au minimum car elle représente la base de l'ECS et donc du moteur de jeu.
     Ces fonctions sont définis comme @Nonnull, cela permet de prévenir Java que ces variables n'ont pas la capacité d'être null.
    */
    public void exemple(@Nonnull Ref<EntityStore> ref, @Nonnull World world) {

        if (ref.isValid()) { // On vérifie obligatoirement que ref ne renvoie pas à nulle

            EntityStore e_store = world.getEntityStore(); // Récupération de l'EntityStores liée au world.
            Store<EntityStore> Store = e_store.getStore(); // Récupération du store des entités.
```

---

## Lecture des components et données

- Les données de l’entité sont récupérées exclusivement via le `Store`.
- `TransformComponent` expose la position de l’entité.
- `Player` représente l’entité visible du joueur.
- Les vérifications `null` sont nécessaires car les getters ne garantissent aucune sûreté statique.

```java
            TransformComponent trComponent = Store.getComponent(ref, TransformComponent.getComponentType()); // Récupération du Component Transform de l'entité ref.
            Player p = Store.getComponent(ref, Player.getComponentType()); // Récupération de l'entité Player d'après le Store d'entités.

            if (p != null && trComponent != null) { // Vérifications de sûreté

                /*
                 Pourquoi s'embêter à vérifier que ref est valide, que p et trComponent ne soit pas null,
                 Par défaut nos variables proviennent de getters, rien dans notre code actuel ne permet à Java
                 De déterminer avec assurances que ces variables ne soit pas égual à null,
                 Afin d'éviter tout bug ou fuite de mémoire.
                */

                Vector3d Pos = trComponent.getPosition(); // Position de l'entité joueur

                /*
                 World (architecture centrale du système ECS)
                         -> EntityStore (Store des Entités liées au world)
                         -> Store<EntityStores> (Récupération du store des Entités)
                         -> TransformComponent
                         -> Vector3D (position)
                */

                double x = Pos.getX();
                double y = Pos.getY();
                double z = Pos.getZ();

                Vector3d Pos2 = new Vector3d(x + 25.0, y + 30.0, z + 45.0);
                Vector3f rot = new Vector3f(0f, 0f, 0f);
```

---

## Application de l’action via le Store

- L’action n’est jamais appliquée directement à l’entité.
- La création d’un component représente l’intention.
- L’ajout du component au `Store` rend l’action effective.

```java
                Teleport tp = new Teleport(world, Pos2, rot); // Création du component Teleport
                Store.addComponent(ref, Teleport.getComponentType(), tp); // Écriture dans le Store

                /*
                 Pour rendre la téléportation du joueur effective nous allons ajouter le Component de Teleport
                 au Store, ceci rend effectif la téléportation.
                */
            }
        }
    }
}
```

---

# Synthèse brute


- **Entrées** : `@Nonnull Ref<EntityStore> ref`, `@Nonnull World world`
- **Garde-fous** : `ref.isValid()`, `p != null`, `trComponent != null`
- **Lecture data** : `TransformComponent -> getPosition()`
- **Transformation** : `x/y/z -> Pos2`, création `rot`
- **Action ECS** : création `Teleport` puis `Store.addComponent(...)`

- **Entrées** : `@Nonnull Ref<EntityStore> ref`, `@Nonnull World world`
- **Accès ECS** : `World → EntityStore → Store`
- **Lecture données** : `TransformComponent`, `Player`
- **Sécurité** : `ref.isValid()`, vérifications `null`
- **Action** : création d’un `Teleport` puis `Store.addComponent(...)`

Ce document constitue une référence interne directe pour comprendre et reproduire le flux ECS standard côté serveur Hytale, produit avec amour et dévouement par Vox.

