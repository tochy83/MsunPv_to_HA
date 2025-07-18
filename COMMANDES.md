<p align="center"><img src="/images/msunpv_to_ha.jpg?token=GHSAT0AAAAAACH6VDSSWYHXPXZIN7LRLMMQZJWRDEQ"></p>

    
# Les scripts des commandes du MsunPv
</br>

- [Explication du fonctionnement](#explication-du-fonctionnement)
- [Détails du fichier msunpv_scripts_x_x.yaml](#détails-du-fichier-msunpv_scripts_x_xyaml)
- [Exemple du fonctionnement en image](#exemple-du-fonctionnement-en-image)
- [Ajout de la commande Test routeur](#ajout-de-la-commande-test-routeur)
</br>

## Explication du fonctionnement
Pour intéragir avec le MsunPv on peut utiliser des commandes 'curl' par le biais des 'shell_command' de Home Assistant.</br>

Par exemple :
```yml
shell_command:
  msunpv_manurad_on: "curl -X POST -d 'parS=4;0;0;0;0;0;0;2;' http://IP_DU_MSUNPV/index.xml"
```
>'pars=9;0;0;0;0;0;0;2;' contenant la commande à envoyer au routeur.</br>

On pourrait ce dire, ok le 9 correspond à la commande à envoyer sur la sortie 1, le 1er 0 la sortie 2 et ainsi de suite jusqu'a la sortie 8. Cela serait bien trop simple. Les sorties sont liées 2 par 2. Du coup le 9 correspond à la comande envoyée à la sortie 1 et 2, le 1er 0 pour les sorties 3 et 4 et ainsi de suite.

>Je crois qu'il y'a 8 sorties pilotables sur les MsunPv_2_2 et MsunPv_4_4, les 0 supplémentaires au delà du 4ème chiffre, que l'on pourrait imaginer être pour les sorties 9 à 14 sont inutiles dans notre cas. Sans doute un héritage du grand frère du MsunPv.

Voici comment elles sont codées :
| Valeur | Cmd 1 | Cmd 2 | 
| --------- | --- | ---------- | 
| 0 | off | off |
| 1 | manuel | off |
| 2 | auto | off |
| 4 | off | manuel |
| 5 | manuel | manuel |
| 6 | auto | manuel |
| 8 | off | auto |
| 9 | manuel | auto |
| 10 | auto | auto |

On peut répéter ce schéma pour les comandes 3-4, 5-6 et 7-8.</br></br>
Le 2 à la fin de 'pars=9;0;0;0;0;0;0;2;' correspond quand à lui à la commande 'Test routeur' du MsunPv. Elle est codée comme ceci:
| Valeur | Cmd test | 
| --------- | --- |
| 1 | Inject |
| 2 | Zéro |
| 4 | Moyen |
| 8 | Fort |

Sur notre exemple du dessus avec 'pars=9;0;0;0;0;0;0;2;' cela correspondrait donc à l'image ci-dessous sur le routeur :

![](images/manubal_autorad_on.png)

Grâce au tableau détaillant comment sont codées les commandes, on peut se rendre compte que pour passer le sortie 1 de OFF à MANUEL (sans toucher à sortie 2), il faut ajouter 1 à la Cmd1. De même pour passer de OFF à AUTO il faur retrancher 2. Pour la sortie 2 il faudra par contre ajouter 4 pour passer de OFF à MANUEL.</br></br>

C'est ce comportement que reproduisent les scripts. Ils analysent l'état de chaque sorties grâce au sensor 'msunpv_cmdpos' remonté par le fichier 'status.xml' du MsunPv et en fonction du l'état que l'on veut atteindre, ils ajoutent ou retranchent 1,2,4,8 à la commande à envoyer au MsunPv. Pour la commande 'Test routeur' du MsunPv on envoi directement la valeur désirée.

</br>

## Détails du fichier msunpv_scripts_x_x.yaml
J'ai fait le choix pour rester simple de créer 3 scripts par commandes. Un premier script pour faire un reset de la commande, le second pour activer le mode MANUEL et le troisième pour le mode AUTO.</br>

Exemple avec la sortie 1 :
```yml
# Scripts des commandes pour MsunPv en configuration 2_2

script:

  msunpv_s1_off: #Mode routage actif sortie 1
    alias: msunpv_s1_off
    sequence:
      - action: input_select.select_option
        data:
          option: |-
            {% if states('sensor.msunpv_cmd_s1') in ['1','5','9'] %}
              {{ states('sensor.msunpv_cmd_s1')|int -1 }}
            {% elif states('sensor.msunpv_cmd_s1') in ['2','6','10'] %}
              {{ states('sensor.msunpv_cmd_s1')|int -2 }}
            {% else %}
              {{ states('sensor.msunpv_cmd_s1')|int }}
            {% endif %}
        target:
          entity_id: input_select.msunpv_command_sortie_1
      - action: input_select.select_option
        data:
          option: "{{ states('sensor.msunpv_cmd_test') }}"
        target:
          entity_id: input_select.msunpv_command_test_routeur
      - action: script.msunpv_commande_routeur
        data: {}
    mode: single
    
  msunpv_s1_manuel: #Mode forçage sortie 1
    alias: msunpv_s1_manuel
    sequence:
      - action: input_select.select_option
        data:
          option: |-
            {% if states('sensor.msunpv_cmd_s1') in ['0','4','8'] %}
              {{ states('sensor.msunpv_cmd_s1')|int +1 }}
            {% elif states('sensor.msunpv_cmd_s1') in ['2','6','10'] %}
              {{ states('sensor.msunpv_cmd_s1')|int -1 }}
            {% else %}
              {{ states('sensor.msunpv_cmd_s1')|int -1 }}
            {% endif %}
        target:
          entity_id: input_select.msunpv_command_sortie_1
      - action: input_select.select_option
        data:
          option: "{{ states('sensor.msunpv_cmd_test') }}"
        target:
          entity_id: input_select.msunpv_command_test_routeur
      - action: script.msunpv_commande_routeur
        data: {}
    mode: single
    
  msunpv_s1_auto: #Mode programmation horaire sortie 1
    alias: msunpv_s1_auto
    sequence:
      - action: input_select.select_option
        data:
          option: |-
            {% if states('sensor.msunpv_cmd_s1') in ['1','5','9'] %}
              {{ states('sensor.msunpv_cmd_s1')|int +1 }}
            {% elif states('sensor.msunpv_cmd_s1') in ['0','4','8'] %}
              {{ states('sensor.msunpv_cmd_s1')|int +2 }}
            {% else %}
              {{ states('sensor.msunpv_cmd_s1')|int -2 }}
            {% endif %}
        target:
          entity_id: input_select.msunpv_command_sortie_1
      - action: input_select.select_option
        data:
          option: "{{ states('sensor.msunpv_cmd_test') }}"
        target:
          entity_id: input_select.msunpv_command_test_routeur
      - action: script.msunpv_commande_routeur
        data: {}
    mode: single
```
- Pour chacun des scripts on retrouve 3 parties :
```yml
      - action: input_select.select_option
        data:
          option: |-
            {% if states('sensor.msunpv_cmd_s1') in ['1','5','9'] %}
              {{ states('sensor.msunpv_cmd_s1')|int -1 }}
            {% elif states('sensor.msunpv_cmd_s1') in ['2','6','10'] %}
              {{ states('sensor.msunpv_cmd_s1')|int -2 }}
            {% else %}
              {{ states('sensor.msunpv_cmd_s1')|int }}
            {% endif %}
        target:
          entity_id: input_select.msunpv_command_sortie_1
```
La 1ère vérifie l'état actuel des sorties 1 et 2 et choisi la valeur à envoyer en fonction de l'état désiré par le biais d'un 'input_select'.
>'input_select.msunpv_command_sortie_1' est défini dans le fichier 'msunpv_x_x.yaml'

</br>

```yml
      - action: input_select.select_option
        data:
          option: "{{ states('sensor.msunpv_cmd_test') }}"
        target:
          entity_id: input_select.msunpv_command_test_routeur
```
La 2ème récupère l'état de la sortie 'Test routeur' et renvoie sa valeur par le biais d'un 'input_select' également.
>'input_select.msunpv_command_test_routeur' est défini dans le fichier 'msunpv_x_x.yaml'

</br>

```yml
      - action: script.msunpv_commande_routeur
        data: {}
```
La 3ème appelle le script 'msunpv_commande_routeur' qui permet d'envoyer les commandes au routeur.</br>

>Sur la version _4_4 il y'a une partie supplémentaire qui se charge de récupérer l'état des sorties 3 et 4 également.
>```yml
>      - action: input_select.select_option
>        data:
>          option: "{{ states('sensor.msunpv_cmd_s3') }}"
>        target:
>          entity_id: input_select.msunpv_command__sortie_3
>```

</br>

- Le script 'msunpv_routage_on_off':
```yml
  msunpv_routage_on_off: #Bascule entre routage et injection
    alias: msunpv_routage_on_off
    sequence:
      - action: input_select.select_option
        data:
          option: |-
            {% if states('sensor.msunpv_cmd_test') in ['1','4','8'] %}
              2
            {% else %}
              1
            {% endif %}
        target:
          entity_id: input_select.msunpv_command_test_routeur
      - action: input_select.select_option
        data:
          option: "{{ states('sensor.msunpv_cmd_s1') }}"
        target:
          entity_id: input_select.msunpv_command_sortie_1
      - action: script.msunpv_commande_routeur
        data: {}
    mode: single
```
Fait la même chose que les précédents mais pour la sortie 'Test routeur'.</br></br>

- Le script 'msunpv_commande_routeur' :
```yml
  msunpv_commandes_routeur: #Envoi des commandes au MsunPv
    alias: msunpv_commande_routeur
    sequence:
      - action: shell_command.msunpv_commandes
        data: {}
      - delay:
          hours: 0
          minutes: 0
          seconds: 1
          milliseconds: 0
      - action: homeassistant.update_entity
        data: {}
        target:
          entity_id: sensor.msunpv_xml
    mode: single
```
Ce dernier script appelle la 'shell_command' qui va piloter le MsunPv, puis fait une pause et force un update du sensor 'msunpv_xml' pour faire remonter les nouveau états dans l'interface de Home Assistant plus rapidement.
>La 'shell_command.msunpv_commandes' est défini dans le fichier 'msunpv_x_x.yaml'
>```yml
>shell_command:
>   msunpv_commandes: "curl -X POST -d 'parS={{ states('input_select.msunpv_command_sortie_1') }};0;0;0;0;0;0;{{ states('input_select.msunpv_command_test_routeur') }};' http://IP_DU_MSUNPV/index.xml"
>```

</br></br>

## Exemple du fonctionnement en image
![](images/carte_exemple_commandes_sorties.gif)
On peut apercevoir que les commandes fonctionnent dans les 2 sens, aussi bien de Home Assistant vers le MsunPv que l'inverse.
>La vitesse de remontée des infos quand on passe la commande depuis le MsunPv dépend du paramètre 'scan_interval' defini dans le fichier 'msunpv_x_x.yaml'.

</br>

<details>
  <summary>Le code de la carte utilisée dans cet exemple : (Cliquer pour dérouler)</summary>

```yml
#   Les integrations et interfaces HACS utilisées pour cette carte :
#
#   - lovelace-mushroom : https://github.com/piitaya/lovelace-mushroom
#   - lovelace-card-mod : https://github.com/thomasloven/lovelace-card-mod
#
#
#   Les sensors, scripts créés pour cette carte :
#
#   - Tous les sensors sont définis dans l'intégration msunpv_x_x.yaml
#   - Les scripts sont définis dans l'intégration msunpv_scripts_x_x.yaml


type: vertical-stack
cards:
### Titre
  - type: custom:mod-card
    card:
      type: custom:mushroom-title-card
      title: ''
      subtitle: Commandes cumulus
    card_mod:
      style:
        mushroom-title-card$: |
          .subtitle {
            border-bottom: 1px solid var(--ha-card-border-color,var(--divider-color,#e0e0e0));
            padding-bottom: 5px;
          }
          .header {
            margin-bottom: -27px;
          }

### Boutons commandes cumulus          
  - type: custom:mushroom-chips-card
    chips:
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s1_off
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            mdi:circle-outline
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            mdi:circle
          {% elif states('sensor.msunpv_cmd_s1') in ['0','4','8'] %}
            mdi:circle
          {% elif states('sensor.msunpv_cmd_s1') in ['1','5','9'] %}
            mdi:power-on
          {% elif states('sensor.msunpv_cmd_s1') in ['2','6','10'] %}
            mdi:clock-outline
          {% endif %}
        content: |-
          {% if states('sensor.msunpv_cmd_test') in ['4','8'] %}
            Soutirage
          {% else %}
            Routage
          {% endif %}
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            disabled
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            red
          {% elif states('sensor.msunpv_cmd_s1') in ['0','4','8'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s1_manuel
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: mdi:power-on
        content: Manuel
        icon_color: |-
          {% if states('sensor.msunpv_cmd_s1') in ['1','5','9'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s1_auto
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: mdi:clock-outline
        content: Auto
        icon_color: |-
          {% if states('sensor.msunpv_cmd_s1') in ['2','6','10'] %}
            blue
          {% else %}
            disabled
          {% endif %}     
      - type: spacer
    alignment: center

### Titre
  - type: custom:mod-card
    card:
      type: custom:mushroom-title-card
      title: ''
      subtitle: Commandes sortie 2
    card_mod:
      style:
        mushroom-title-card$: |
          .subtitle {
            border-bottom: 1px solid var(--ha-card-border-color,var(--divider-color,#e0e0e0));
            padding-bottom: 5px;
          }
          .header {
            margin-bottom: -27px;
          }

### Boutons commandes sortie 2    
  - type: custom:mushroom-chips-card
    chips:
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s2_off
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            mdi:circle-outline
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            mdi:circle
          {% elif states('sensor.msunpv_cmd_s1') in ['0','1','2'] %}
            mdi:circle
          {% elif states('sensor.msunpv_cmd_s1') in ['4','5','6'] %}
            mdi:power-on
          {% elif states('sensor.msunpv_cmd_s1') in ['8','9','10'] %}
            mdi:clock-outline
          {% endif %}
        content: |-
          {% if states('sensor.msunpv_cmd_test') in ['4','8'] %}
            Soutirage
          {% else %}
            Routage
          {% endif %}
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            disabled
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            red
          {% elif states('sensor.msunpv_cmd_s1') in ['0','1','2'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s2_manuel
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: mdi:power-on
        content: Manuel
        icon_color: |-
          {% if states('sensor.msunpv_cmd_s1') in ['4','5','6'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_s2_auto
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: mdi:clock-outline
        content: Auto
        icon_color: |-
          {% if states('sensor.msunpv_cmd_s1') in ['8','9','10'] %}
            blue
          {% else %}
            disabled
          {% endif %}     
      - type: spacer
    alignment: center

### Titre    
  - type: custom:mod-card
    card:
      type: custom:mushroom-title-card
      title: ''
      subtitle: Commandes routeur
    card_mod:
      style:
        mushroom-title-card$: |
          .subtitle {
            border-bottom: 1px solid var(--ha-card-border-color,var(--divider-color,#e0e0e0));
            padding-bottom: 5px;
          }
          .header {
            margin-bottom: -27px;
          }

### Boutons commandes routeur         
  - type: custom:mushroom-chips-card
    chips:
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_routage_on_off
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            mdi:circle-outline
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            mdi:circle
          {% else %}
            mdi:circle
          {% endif %}
        content: |-
          {% if states('sensor.msunpv_cmd_test') in ['4','8'] %}
            Soutirage
          {% else %}
            Routage
          {% endif %}
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['2'] %}
            blue
          {% elif states('sensor.msunpv_cmd_test') in ['4','8'] %}
            red
          {% else %}
            disabled
          {% endif %}
      - type: spacer
      - type: template
        tap_action:
          action: perform-action
          perform_action: script.msunpv_routage_on_off
          target: {}
        hold_action:
          action: none
        double_tap_action:
          action: none
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['2','4','8'] %}
            mdi:circle-outline
          {% else %}
            mdi:circle
          {% endif %}
        content: Injection
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: spacer
    alignment: center
```
</details>
    
Elle fonctionne avec les scripts fournis dans le fichier 'msunpv_scripts_2_2.yaml'.</br></br>

On peut très bien imaginer un autre comportement, mais celà necessite de créer des scripts supplémentaires.</br>
Par exemple :
![](images/carte_exemple_2_commandes_sorties.gif)
Dans ce second exemple j'ai refait 2 scripts, un pour le bouton 'cumulus' et un pour le bouton 'sortie 2', sur le même principe que les précédents mais avec un comportement différent puisque ici les boutons ne sont plus en On/Off mais suivent un cycle prédéfini.

</br></br>

## Ajout de la commande Test routeur
![](images/cartes_exemple_commandes_test_routeur.gif)
La commande fonctionne dans les 2 sens, aussi bien de Home Assistant vers le MsunPv que l'inverse.
>La vitesse de remontée des infos quand on passe la commande depuis le MsunPv dépend du paramètre 'scan_interval' defini dans le fichier 'msunpv_x_x.yaml'.

</br>

<details>
  <summary>Le code de la carte utilisée dans cet exemple : (Cliquer pour dérouler)</summary>

```yml
#   Les integrations et interfaces HACS utilisées pour cette carte :
#
#   - lovelace-mushroom : https://github.com/piitaya/lovelace-mushroom
#
#
#   Les sensors, scripts créés pour cette carte :
#
#   - Tous les sensors sont définis dans l'intégration msunpv_x_x.yaml
#   - Les scripts sont définis dans l'intégration msunpv_scripts_x_x.yaml


type: vertical-stack
cards:
  - type: custom:mushroom-title-card
    subtitle: Commandes routeur
  - type: custom:mushroom-chips-card
    chips:
      - type: template
        tap_action:
          action: perform-action
          target: {}
          perform_action: script.msunpv_test_routeur
          data:
            choix: 1
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            mdi:circle
          {% else %}
            mdi:circle-outline
          {% endif %}
        content: Injection
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['1'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: template
        tap_action:
          action: perform-action
          target: {}
          perform_action: script.msunpv_test_routeur
          data:
            choix: 2
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['2'] %}
            mdi:circle
          {% else %}
            mdi:circle-outline
          {% endif %}
        content: Zero
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['2'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: template
        tap_action:
          action: perform-action
          target: {}
          perform_action: script.msunpv_test_routeur
          data:
            choix: 4
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['4'] %}
            mdi:circle
          {% else %}
            mdi:circle-outline
          {% endif %}
        content: Moyen
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['4'] %}
            blue
          {% else %}
            disabled
          {% endif %}
      - type: template
        tap_action:
          action: perform-action
          target: {}
          perform_action: script.msunpv_test_routeur
          data:
            choix: 8
        icon: |-
          {% if states('sensor.msunpv_cmd_test') in ['8'] %}
            mdi:circle
          {% else %}
            mdi:circle-outline
          {% endif %}
        content: Fort
        icon_color: |-
          {% if states('sensor.msunpv_cmd_test') in ['8'] %}
            blue
          {% else %}
            disabled
          {% endif %}
    alignment: justify
```
</details>
    
Elle fonctionne avec le scripts 'msunpv_test_routeur' fournis dans les fichier 'msunpv_scripts_2_2.yaml' et 'msunpv_scripts_4_4.yaml'.</br></br>

**Ce script peut également être appeler dans une automatisation**
![](images/automatisation_commande_test_routeur.jpg)

Pour cela il faut lors de l'appel du script dans l'automatisation bien penser à renseigner le choix de l'action à effectuer (Voir la partie entourée en orange sur l'image ci-dessus).</br></br>
Les choix possibles sont:
- 1 pour Injection
- 2 pour Zéro
- 4 pour Moyen
- 8 pour Fort
>Attention à bien mettre 1, 2, 4 ou 8 et non pas 3, 5, 6, 7 sinon cela ne fonctionnera pas et le routeur restera sur la dernière commande utilisée.

</br></br>

[Retour au README.md](README.md#msunpv-to-ha)

</br></br>
