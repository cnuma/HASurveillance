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

## Préparer la maison

**Z-Wave JS.** Un appareil sur pile dont la pile est morte n'est jamais déclaré
« mort » : seule son entité « Last seen » le révèle. Elle est désactivée par défaut.
Pour chaque appareil sur pile : **Paramètres › Appareils**, l'appareil, **Entités
masquées ou désactivées**, puis activer « Last seen ».

**Zigbee2MQTT.** Dans l'interface de Zigbee2MQTT :
- **Paramètres › Disponibilité :** activer. Un appareil sur pile passe
  « indisponible » après 25 h sans nouvelles.
- **Paramètres › Avancé › last_seen :** `ISO_8601`. L'entité « Last seen » de chaque
  appareil apparaît alors dans Home Assistant, désactivée : l'activer comme pour Z-Wave.

**ZHA et ESPHome.** Rien à faire : leurs appareils passent « indisponible » d'eux-mêmes.

## Installer

**Paramètres › Automatisations et scènes › Blueprints › Importer un blueprint**, puis
coller :

```
https://github.com/cnuma/HASurveillance/blob/main/blueprints/automation/hasurveillance/surveillance.yaml
```

Créer ensuite **une seule** automatisation depuis **HASurveillance — santé de la
maison**.

| Champ | À choisir |
|---|---|
| Téléphones | Les appareils de l'application Home Assistant qui reçoivent le rapport |
| Capteurs à activité quotidienne | Ce qui doit bouger chaque jour : porte d'entrée, lumière très utilisée |
| Dernière sauvegarde réussie | Proposé par défaut ; vider le champ pour ne pas surveiller les sauvegardes |
| Appareils ignorés / Entités ignorées | Ce qui est éteint volontairement et ne doit pas être signalé |

Au premier rapport, la liste peut être longue : la plupart des anciens problèmes se
règlent en supprimant les entités orphelines ou en ajoutant aux exclusions ce qui est
éteint volontairement.

## Limites

- Le nom de l'action de notification est déduit du nom de l'appareil du téléphone
  (`notify.mobile_app_<nom>`). Un téléphone renommé après son enregistrement peut ne
  rien recevoir : vérifier l'action dans **Outils de développement › Actions**.
- 10 lignes au plus par rubrique dans un message ; la suite est résumée en « … et N
  autres ».
