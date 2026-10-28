# NSPanel en local (Gitea)

Cette copie du projet NSPanel HA Blueprint est hébergée sur le Gitea local
`http://192.168.10.23:3000/admin/nspanel` et ne dépend plus de GitHub.

## Ce qui a été changé par rapport à l'original

| Élément | Avant (GitHub) | Maintenant (Gitea) |
|---|---|---|
| Code C++ (`external_components`) | `github.com/Blackymas/...` au tag `v<version>` | `${gitea_repo_url}` branche `${gitea_branch}` |
| Fichier TFT de l'écran | `raw.githubusercontent.com/.../v<version>/hmi/nspanel_<modèle>.tft` | `${gitea_raw_url}hmi/nspanel_<modèle>.tft` (branche `main`) |
| URL TFT par défaut | `nspanel_blank.tft` | `nspanel_eu.tft` |
| Blueprint (`source_url`) | GitHub | `http://192.168.10.23:3000/admin/nspanel/raw/branch/main/nspanel_blueprint.yaml` |
| Blueprint | - | Support des aspirateurs (`vacuum`) |
| Design | - | Images du style A dans `hmi/style_A/` |

Les réglages Gitea sont regroupés dans `esphome/nspanel_esphome_base.yaml` (`gitea_url`, `gitea_owner`,
`gitea_repo`, `gitea_branch`). Ils peuvent aussi être surchargés dans le YAML du panel.

Non modifié : `prebuilt/` (firmwares précompilés qui pointent encore vers GitHub, inutiles si tu compiles toi-même)
et `.github/workflows/` (CI de GitHub ; si Gitea Actions est activé sur ce dépôt, désactive les Actions dans les
paramètres du dépôt).

## Config ESPHome du panel

Dans ESPHome, remplace le contenu du YAML de ton panel par :

```yaml
substitutions:
  device_name: "nspanel-salon"            # nom de l'appareil dans ESPHome
  friendly_name: "NSPanel Salon"
  wifi_ssid: !secret wifi_ssid
  wifi_password: !secret wifi_password
  upload_tft_automatically: true

packages:
  remote_package:
    url: http://192.168.10.23:3000/admin/nspanel.git
    ref: main
    refresh: 300s
    files:
      - nspanel_esphome.yaml
      # - esphome/nspanel_esphome_addon_climate_heat.yaml
      # - esphome/nspanel_esphome_addon_cover.yaml
```

Garde la section `api:` (clé de chiffrement) de ton YAML actuel si tu en as une.

`refresh: 300s` : ESPHome relit le dépôt au plus toutes les 5 minutes. Après un push, attends ce délai
(ou supprime le cache dans ESPHome : menu ⋮ > Clean Build Files) avant de recompiler.

## Blueprint dans Home Assistant

Paramètres > Automatisations et scènes > Blueprints > Importer un blueprint, avec l'URL :

```
http://192.168.10.23:3000/admin/nspanel/raw/branch/main/nspanel_blueprint.yaml
```

Si le blueprint NSPanel est déjà installé depuis GitHub, la « ré-importation » ira désormais chercher cette URL.

## Modifier l'écran (TFT)

1. Modifier `hmi/nspanel_eu.HMI` dans Nextion Editor (voir `hmi/style_A/IMPORT_NEXTION.md`).
2. Compiler et exporter le TFT en **`hmi/nspanel_eu.tft`** (même nom, il remplace l'ancien).
3. Commit + push.
4. Dans Home Assistant, appareil NSPanel > bouton **Update TFT display** (ou action
   `esphome.<nom_du_panel>_upload_tft` avec `url: "default"`). Le panel télécharge le TFT depuis Gitea.

## Bon à savoir

- Le dépôt doit être **public** (lisible sans connexion) : le panel et ESPHome le lisent sans identifiants.
- Garde **http://**, l'ESP32 gère mal le https avec un certificat local.
- La **toute première** compilation ESPHome télécharge encore les outils ESP32 (PlatformIO / ESP-IDF) depuis
  Internet ; ils restent ensuite en cache.
- Les contrôles de compatibilité de version (ESPHome / blueprint / TFT, `version: 2026041`) sont inchangés :
  si tu modifies une seule des trois parties sans toucher aux numéros, tout reste compatible.
