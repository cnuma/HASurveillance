# Installer HASurveillance

Procédure complète, de la préparation des appareils au premier rapport. Les libellés
de Home Assistant peuvent légèrement varier selon sa version.

## 1. Préparer les appareils

### Z-Wave JS : activer « Last seen »

Un appareil Z-Wave sur pile dont la pile est morte n'est jamais déclaré « mort » : ses
entités gardent leur dernier état. Seule son entité **Last seen** (dernier contact)
révèle la panne. Home Assistant la crée pour chaque appareil, **désactivée par
défaut** ; elle ne se règle pas dans Z-Wave JS mais dans Home Assistant.

**Pour un appareil :**

1. **Paramètres › Appareils et services › Z-Wave › Appareils**, ouvrir l'appareil.
2. Dans le cadre **Diagnostic**, cliquer sur le lien gris **« + N entités non
   affichées »**.
3. Cliquer sur **Last seen**, puis sur la roue dentée.
4. Activer **Activé**, puis **Mettre à jour**. L'intégration se recharge : la valeur
   apparaît après une trentaine de secondes.

**Pour tous les appareils d'un coup :**

1. **Paramètres › Appareils et services › Entités**.
2. Dans les filtres, afficher les **entités désactivées**.
3. Rechercher `last seen`.
4. Passer en mode sélection, cocher les entités, puis menu **Activer**.

**Contrôle :** l'identifiant de l'entité doit se terminer par `_last_seen` (par exemple
`sensor.capteur_porte_entree_last_seen`). C'est ainsi que le blueprint la reconnaît.

**Si « Last seen » n'existe pas :** le module complémentaire Z-Wave JS (ou Z-Wave JS UI)
est sans doute trop ancien. Le mettre à jour.

### Zigbee2MQTT : disponibilité et dernier contact

Sans l'option de disponibilité, un appareil Zigbee2MQTT en panne garde lui aussi son
dernier état indéfiniment. Dans l'interface de Zigbee2MQTT :

1. **Paramètres › Disponibilité :** activer. Un appareil sur pile passe
   « indisponible » après 25 h sans nouvelles, un appareil sur secteur après 10 min.
2. **Paramètres › Avancé › last_seen :** choisir `ISO_8601`, puis enregistrer.
3. Dans Home Assistant, activer les entités **Last seen** comme pour Z-Wave (méthode
   « tous les appareils d'un coup »).

### ZHA et ESPHome

Rien à faire : leurs appareils passent « indisponible » d'eux-mêmes (ZHA : après 2 h
sur secteur, 6 h sur pile).

## 2. Importer le blueprint

1. **Paramètres › Automatisations et scènes › Blueprints › Importer un blueprint**.
2. Coller ce lien, puis **Aperçu** et **Importer** :

```
https://github.com/cnuma/HASurveillance/blob/main/blueprints/automation/hasurveillance/surveillance.yaml
```

**Mise à jour :** menu **⋮** du blueprint, puis **Réimporter le blueprint**. Les
automatisations existantes gardent leurs réglages.

## 3. Créer l'automatisation

Depuis le blueprint **HASurveillance — santé de la maison**, créer **une seule**
automatisation pour toute la maison.

| Champ | À choisir | Par défaut |
|---|---|---|
| Téléphones | Appareils de l'application Home Assistant qui reçoivent le rapport | Aucun |
| Heure du rapport | Heure du rapport quotidien sur les téléphones | 9 h |
| Rapport même sans problème | Envoie aussi « aucun problème » : prouve que la surveillance tourne | Non |
| Sans contact depuis | Âge maximal d'une entité « Last seen » ; doit dépasser l'intervalle de réveil des appareils sur pile | 48 h |
| Indisponible depuis | Durée avant de signaler une entité indisponible | 60 min |
| Capteurs à activité quotidienne | Entités qui bougent normalement chaque jour : porte d'entrée, lumière très utilisée | Aucun |
| Sans changement depuis | Durée avant de signaler un de ces capteurs comme figé | 2 j |
| Pile faible sous | Seuil des capteurs de batterie | 20 % |
| Dernière sauvegarde réussie | Capteur des sauvegardes ; vider le champ pour ne pas les surveiller | `sensor.backup_last_successful_automatic_backup` |
| Sauvegarde plus ancienne que | Âge maximal de la dernière sauvegarde | 48 h |
| Appareils ignorés / Entités ignorées | Ce qui est éteint volontairement, prêté, en test | Aucun |

Les entités des téléphones sont toujours ignorées : leur pile et leur réseau varient
par nature.

## 4. Premier rapport

1. Ouvrir l'automatisation, menu **⋮ › Exécuter les actions** : le rapport part
   aussitôt, dans Home Assistant et sur les téléphones.
2. Le premier rapport est souvent long. Pour l'alléger :
   - **Entités orphelines :** les supprimer (**Paramètres › Entités**, l'entité,
     **Supprimer**) ;
   - **appareil éteint volontairement :** l'ajouter aux appareils ignorés ;
   - **pile faible :** la remplacer.
3. Relancer jusqu'à n'avoir que de vrais problèmes.

Ensuite, la notification dans Home Assistant se met à jour toutes les heures (à hh:05)
et disparaît quand tout est réglé ; les téléphones reçoivent le rapport chaque jour.

## 5. Dépannage

| Symptôme | Cause probable | Que faire |
|---|---|---|
| Rubrique « Sans contact » toujours vide | Entités « Last seen » désactivées, ou identifiant sans `_last_seen` à la fin | Voir la section 1 |
| Rien sur le téléphone | Nom de l'action déduit du nom de l'appareil (`notify.mobile_app_<nom>`) : téléphone renommé après son enregistrement | Vérifier l'action dans **Outils de développement › Actions** ; consulter les traces de l'automatisation |
| Rubrique « Sauvegarde : aucune sauvegarde réussie connue » | Sauvegardes automatiques non configurées, ou faites par un autre outil | Configurer les sauvegardes (**Paramètres › Système › Sauvegardes**) ou vider le champ |
| Trop d'indisponibles | Appareils éteints volontairement | Les ajouter aux appareils ignorés |
| Message coupé « … et N autres » | 10 lignes au plus par rubrique | Régler les premiers problèmes, puis relancer |
