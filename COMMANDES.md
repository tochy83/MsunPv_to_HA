<p align="center"><img src="/images/msunpv_to_ha.jpg?token=GHSAT0AAAAAACH6VDSSWYHXPXZIN7LRLMMQZJWRDEQ"></p>
    
# Les scripts des commandes du MsunPv
</br>

- [Explication du fonctionnement](#introduction)
- [Fonctionnalités](#-fonctionnalités)
- [Installation](#%EF%B8%8F-installation)
- [Comment ça fonctionne ?](#-comment-%C3%A7a-fonctionne-)
- [Exemple du résultat dans mon dashboard Home Assistant](#exemple-du-r%C3%A9sultat-dans-mon-dashboard-home-assistant)
- [FAQ](#faq)
</br>

## Explication du fonctionnement
Pour intéragir avec le MsunPv on peut utiliser des commandes 'curl' par le biais des 'shell_command' de Home Assistant.</br>

Par exemple :
```yml
shell_command:
  msunpv_manurad_on: "curl -X POST -d 'parS=4;0;0;0;0;0;0;2;' http://192.168.0.38/index.xml"
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
Le 2 à la fin 'pars=9;0;0;0;0;0;0;2;' correspond quand à lui à la commande 'Test routeur' du MsunPv. Elle est codée comme ceci:
| Valeur | Cmd | 
| --------- | --- |
| 0 | Inject |
| 1 | Zéro |
| 2 | Moyen |
| 4 | Fort |

Sur notre exemple du dessus avec 'pars=9;0;0;0;0;0;0;2;' celà correspondrait donc à l'image ci-dessous sur le routeur :
![](images/manubal_autorad_on.png)



</br></br></br>
