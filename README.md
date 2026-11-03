# HASurveillance

Un blueprint Home Assistant qui détecte les **pannes silencieuses** de la maison :
celles qui ne déclenchent aucune erreur, comme un capteur de porte resté sur
« fermé » pendant des mois parce que sa pile est morte.

## Ce qui est surveillé

| Rubrique | Signalé quand | Couvre |
|---|---|---|
| Sans contact | L'entité « Last seen » d'un appareil date de plus de 48 h | Z-Wave sur pile, Zigbee2MQTT |
| Indisponibles | Une entité est « indisponible » depuis plus de 60 min (regroupé par appareil) | ESPHome, ZHA, Z-Wave sur secteur, Zigbee2MQTT avec disponibilité |
| Capteurs figés | Une entité choisie n'a pas changé depuis 2 jours | Capteur qui répond mais ne détecte plus rien |
| Piles faibles | Batterie sous 20 %, ou capteur de pile « faible » | Tous les réseaux |
| Sauvegarde | Dernière sauvegarde automatique réussie de plus de 48 h | Home Assistant |
| Entités orphelines | Entité restée dans le registre après la suppression de son appareil | Ménage à faire |

Toutes les durées et tous les seuils se règlent dans l'automatisation. Les entités des
téléphones (application Home Assistant) sont ignorées : leur pile et leur réseau
varient par nature.

## Comment on est prévenu

- **Dans Home Assistant :** une notification « Maison : N problèmes », mise à jour
  toutes les heures et retirée d'elle-même quand tout est réglé.
- **Sur les téléphones :** un rapport chaque jour à l'heure choisie (9 h par défaut),
  seulement s'il y a un problème. L'option « Rapport même sans problème » envoie aussi
  « Aucun problème » : un message quotidien qui prouve que la surveillance tourne.

Pour un rapport immédiat : ouvrir l'automatisation, menu **⋮ › Exécuter les actions**.

## Installer

La procédure complète est dans [docs/INSTALLATION.md](docs/INSTALLATION.md) :
préparation des appareils (Z-Wave, Zigbee2MQTT), import du blueprint, création de
l'automatisation, premier rapport et dépannage.

Lien d'import du blueprint :

```
https://github.com/cnuma/HASurveillance/blob/main/blueprints/automation/hasurveillance/surveillance.yaml
```
