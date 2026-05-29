# 🌞 Protection thermique intelligente multi-fenêtres pour Home Assistant

Automatisation Home Assistant avancée permettant de protéger automatiquement un logement contre la surchauffe estivale en pilotant les volets selon :

* l’orientation du soleil,
* la position solaire réelle,
* la température intérieure,
* l’indice UV,
* les alertes canicule,
* et des seuils de confort personnalisés par fenêtre.

Cette version est conçue pour être :

✅ modulaire
✅ réutilisable
✅ facile à adapter à d'autres logements
✅ compatible avec plusieurs volets/fenêtres
✅ observable grâce à un mode DEBUG intégré

---

# ✨ Fonctionnalités

* Gestion centralisée de plusieurs fenêtres/volets
* Protection thermique pièce par pièce
* Calcul solaire réel via `sun.sun`
* Déclenchement selon :

  * azimut solaire
  * élévation solaire
  * UV
  * température intérieure
  * alerte canicule météo
* Positionnement partiel intelligent des volets
* Retour automatique à l’état normal lorsque le soleil n’impacte plus la fenêtre
* Notifications DEBUG détaillées
* Architecture basée sur des variables facilement extensibles

---

# 🧠 Principe de fonctionnement

Toutes les minutes, l’automatisation :

1. Parcourt la liste des fenêtres définies dans `windows`
2. Vérifie pour chaque ouverture :

   * si le soleil tape réellement sur la façade
   * si la chaleur devient problématique
   * si le volet n’est pas déjà suffisamment fermé
3. Active une protection solaire adaptée
4. Réouvre automatiquement lorsque le soleil quitte la zone

---

# ☀️ Conditions de fermeture des volets

La protection solaire s’active uniquement si :

## 1. Le soleil éclaire réellement la fenêtre

Conditions :

* azimut solaire compris entre `az_min` et `az_max`
* élévation solaire supérieure à `el_min`

Exemple :

```yaml
az_min: 110
az_max: 280
el_min: 5
```

---

## 2. La chaleur est jugée excessive

La protection s’active si :

```yaml
température intérieure >= threshold
ET UV >= 4
```

OU si une alerte canicule est détectée.

---

## 3. Le volet n’est pas déjà suffisamment fermé

```yaml
current_position > 35
```

---

# 🔄 Réouverture automatique

Lorsque le soleil sort de la zone d’exposition :

```yaml
not sun_in_range
```

et que la protection était active, le script de fin de protection est exécuté.

---

# 🧩 Structure du système

Le système repose sur :

## 1. Une automatisation principale

Elle orchestre toute la logique.

---

## 2. Un script de fermeture

```yaml
script.protection_solaire_volet
```

Responsable de :

* fermer le volet à une position cible
* activer le helper associé

---

## 3. Un script de fin de protection

```yaml
script.fin_de_protection_solaire_volet
```

Responsable de :

* restaurer le comportement normal
* désactiver le helper

---

## 4. Des helpers `input_boolean`

Un helper par fenêtre :

```yaml
input_boolean.protection_solaire_cuisine_active
```

Ils servent à mémoriser l’état de protection.

---

# ⚙️ Paramètres à adapter

## 📌 Liste des fenêtres

Tout se configure ici :

```yaml
windows:
```

Chaque entrée représente une ouverture.

---

# 🪟 Paramètres d’une fenêtre

## `name`

Nom lisible utilisé dans les logs/debug.

```yaml
name: cuisine
```

---

## `volet`

Entité du volet roulant.

```yaml
volet: cover.mon_volet
```

---

## `helper`

Helper associé à cette fenêtre.

```yaml
helper: input_boolean.protection_solaire_cuisine_active
```

---

## `temp_sensor`

Capteur de température intérieure.

```yaml
temp_sensor: sensor.temperature_salon
```

---

## `threshold`

Température déclenchant la protection.

```yaml
threshold: 23
```

---

## `comfort_cover`

Position cible du volet.

Exemple :

```yaml
comfort_cover: 27
```

Correspond à :

* 0 = fermé
* 100 = ouvert

---

## `az_min` / `az_max`

Zone d’exposition solaire de la fenêtre.

```yaml
az_min: 110
az_max: 280
```

---

## `el_min`

Hauteur minimale du soleil avant activation.

```yaml
el_min: 5
```

Permet d’éviter les déclenchements au lever/coucher du soleil.

---

# 🧭 Comment déterminer les bons azimuts ?

## Méthode simple

Utiliser :

* les Developer Tools Home Assistant
* l’entité `sun.sun`
* observer les valeurs lorsque le soleil éclaire réellement la fenêtre

Attributs utiles :

```yaml
azimuth
elevation
```

---

# 🐞 Mode DEBUG

Le mode debug repose sur :

```yaml
input_boolean.debug_protection_solaire
```

Lorsqu’il est activé :

* une notification persistante est générée toutes les minutes
* toutes les conditions calculées sont visibles

Très utile pour :

* calibrer les azimuts
* ajuster les seuils
* comprendre les comportements

---

# 🌡️ Gestion de la canicule

Cette automatisation utilise :

```yaml
sensor.75_weather_alert
```

et recherche les niveaux :

* jaune
* orange
* rouge

Vous pouvez remplacer ce capteur par votre propre source météo.

---

# 🔧 Dépendances

## Entités nécessaires

### Soleil

```yaml
sun.sun
```

---

### UV

Exemple :

```yaml
sensor.paris_uv
```

---

### Températures intérieures

Un capteur par pièce/fenêtre.

---

### Volets pilotables

Entités `cover.*`

---

# 🚀 Installation

## 1. Créer les helpers

Créer un `input_boolean` par fenêtre.

Exemple :

```yaml
input_boolean:
  protection_solaire_cuisine_active:
```

---

## 2. Créer le helper DEBUG

```yaml
input_boolean:
  debug_protection_solaire:
```

---

## 3. Ajouter les scripts

Créer :

* `script.protection_solaire_volet`
* `script.fin_de_protection_solaire_volet`

---

## 4. Ajouter l’automatisation

Importer le YAML dans :

```text
Settings → Automations & Scenes
```

---

# 💡 Conseils de calibration

## Commencer avec le DEBUG activé

Observer :

* les azimuts
* les élévations
* les heures réelles d’exposition

---

## Ajuster progressivement

Modifier :

* `az_min`
* `az_max`
* `el_min`

jusqu’à obtenir un comportement parfait.

---

# 🔋 Optimisations possibles

## Ajouter la température extérieure

Pour éviter des fermetures inutiles.

---

## Ajouter la présence

Ne protéger que lorsque le logement est occupé.

---

## Ajouter la météo prévisionnelle

Anticipation des pics thermiques.

---

## Ajouter un mode hiver

Exploiter les apports solaires gratuits.

---

# 🏠 Exemple d’usage réel

Cette automatisation est particulièrement efficace pour :

* les grandes baies vitrées
* les logements RT2012 très vitrés
* les appartements traversants
* les chambres exposées ouest
* les maisons avec forte inertie thermique

---

# 📜 Licence

MIT

---

# ❤️ Remerciements

Merci à la communauté :

* Home Assistant
* Zigbee2MQTT
* ESPHome
* et tous les passionnés de domotique libre ☀️
