Voici le code de mon dashboard pour la partie photovolatique/MsunPv</br></br>
Pour réaliser le même design vous aurez besoin de plusieurs dépots HACS et de 3 scripts pour le fonctionnement des boutons de commande du routeur:</br>

Depot HACS utilisés pour la page Production photovoltaique</br>
https://github.com/iantrich/config-template-card</br>
https://github.com/piitaya/lovelace-mushroom</br>
https://github.com/thomasloven/lovelace-card-mod</br>
https://github.com/thomasloven/lovelace-layout-card</br>
https://github.com/RomRider/apexcharts-card</br>
https://github.com/custom-cards/stack-in-card</br>
https://github.com/bramkragten/swipe-card</br></br>

Les scripts</br>
[script.msunpv_bouton_cumulus](script.msunpv_bouton_cumulus.yaml)</br>
[script.msunpv_bouton_s2](script.msunpv_bouton_s2.yaml)</br>
[script.msunpv_bouton_routage](script.msunpv_bouton_routage.yaml)</br></br>

Vue générale de la page :</br></br>
![](page_complete.jpg)</br></br>
La page est découpée en 4 blocs :
- L'entête (bloc 1)
- Une zone de notification (bloc 2)
- Une premier bloc vertical reprenant les infos de production (bloc 3)
- Un second bloc vertical reprenant le reste des infos (bloc 4)</br></br>

Pour arriver à ce résultat il convient dans un prmier temps de configurer la vue :</br></br>
![](vue_pv.jpg)</br>
[Le code de la vue](vue_pv.yaml)</br></br></br>
L'entête :</br></br>
![](bloc_1.jpg)</br>
[Le code de l'entête](bloc_1.yaml)</br></br></br>
La zone de notification :</br></br>
![](bloc_2.jpg)</br>
[Le code de la zone notification](bloc_2.yaml)</br></br></br>
Le bloc vertical gauche :</br></br>
![](bloc_3.jpg)</br>
[Le code du bloc gauche](bloc_3.yaml)</br></br></br>
Le bloc vertival droite:</br></br>
![](bloc_4.jpg)</br>
[Le code du bloc droite](bloc_4.yaml)</br></br></br>

