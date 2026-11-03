# CLAUDE.md — HASurveillance

Toujours répondre en français.

## Invariants

| ID | Règle |
|---|---|
| I1 | **Procédure d'installation versionnée :** tout blueprint, et plus largement tout élément à installer ou à régler dans Home Assistant, est livré avec sa procédure complète dans `docs/INSTALLATION.md` (préparation des appareils, import, mise à jour, création de l'automatisation, premier test, dépannage), écrite ou mise à jour dans le même commit. Une explication donnée dans la conversation y est reportée. Règle commune à tous les dépôts du propriétaire (HAPanel : G11, D029). |
| I2 | **Aucune donnée de la maison dans le dépôt :** pas de nom d'entité, d'appareil ni de secret ; tout se choisit dans les champs du blueprint. |
| I3 | **Modèles vérifiés :** tout modèle Jinja modifié est rendu hors de Home Assistant avec de faux états avant livraison ; ce qui n'a pas pu tourner dans Home Assistant est signalé. |
