# FAQ

</br></br>

## C'est bien beau ton truc mais mes compteurs sont en positifs, comment je fais ?
Une première solution serait de passer vos compteurs en négatif par l'outil msapp_pv fournit sur le site [Ard-tek](https://ard-tek.com/) pour paramétrer en profondeur le MsunPv.</br></br>

Sinon il faut modifier les lignes suivantes :

```yml
      - name: msunpv_eninj #Production injectée journalière
        unique_id: "msunpv_eninj"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ (0 if cptVals.split(";")[1]|int(base=16)|int == 0 else ((0xFFFFFFFF - cptVals.split(";")[1]|int(base=16)) * -1)/10) |float }}
        unit_of_measurement: "Wh"
        device_class: energy

      - name: msunpv_enpv_j #Production panneaux journalière
        unique_id: "msunpv_enpv_j"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ (0 if cptVals.split(";")[2]|int(base=16)|int == 0 else ((0xFFFFFFFF - cptVals.split(";")[2]|int(base=16)) * -1)/10) |float }}
        unit_of_measurement: "Wh"
        device_class: energy

      - name: msunpv_enpv_p #Production panneaux totale
        unique_id: "msunpv_enpv_p"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ (0 if cptVals.split(";")[3]|int(base=16)|int == 0 else ((0xFFFFFFFF - cptVals.split(";")[3]|int(base=16)) * -1)/10) |float }}
        unit_of_measurement: "Wh"
        device_class: energy
```

Et les remplacer par celles-ci :

```yml
      - name: msunpv_eninj #Production injectée journalière
        unique_id: "msunpv_eninj"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ cptVals.split(";")[1]|int(base=16)/10 |float }}
        unit_of_measurement: "Wh"
        device_class: energy

      - name: msunpv_enpv_j #Production panneaux journalière
        unique_id: "msunpv_enpv_j"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ cptVals.split(";")[2]|int(base=16)/10 |float }}
        unit_of_measurement: "Wh"
        device_class: energy

      - name: msunpv_enpv_p #Production panneaux totale
        unique_id: "msunpv_enpv_p"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ cptVals.split(";")[3]|int(base=16)/10 |float }}
        unit_of_measurement: "Wh"
        device_class: energy
```

</br>De même pour les sensors du dashboard energy où il faut remplacer les lignes :

```yml
      - name: "energie_enpv_j" #Production panneaux journalière
        unit_of_measurement: "kWh"
        state: "{{ (states('sensor.msunpv_enpv_j')|float /1000 *-1) }}"
        device_class: energy
        state_class: total_increasing

      - name: "energie_eninj" #Production injectée journalière
        unit_of_measurement: "kWh"
        state: "{{ (states('sensor.msunpv_eninj')|float /1000 *-1) }}"
        device_class: energy
        state_class: total_increasing
```
Par :

```yml
      - name: "energie_enpv_j" #Production panneaux journalière
        unit_of_measurement: "kWh"
        state: "{{ (states('sensor.msunpv_enpv_j')|float /1000) }}"
        device_class: energy
        state_class: total_increasing

      - name: "energie_eninj" #Production injectée journalière
        unit_of_measurement: "kWh"
        state: "{{ (states('sensor.msunpv_eninj')|float /1000) }}"
        device_class: energy
        state_class: total_increasing
```
</br></br>
## J'ai une sonde qui mesure la puissance du cumulus, comment je fais ?
Je remplace simplement la ligne :
```yml
      - name: msunpv_outbal #% routage cumulus
        unique_id: "msunpv_outbal"
        state: >-
          {{ (state_attr('sensor.msunpv_xml', 'inAns')|replace(" ","")|replace(",",".")).split(";")[2] |int }}
        unit_of_measurement: "%"
```
Par :
```yml
      - name: msunpv_outbal #puissance routage cumulus
        unique_id: "msunpv_outbal"
        state: >-
          {{ (state_attr('sensor.msunpv_xml', 'inAns')|replace(" ","")|replace(",",".")).split(";")[2] |float }}
        unit_of_measurement: "W"
```
L'unité de mesure devient W à la place de % et la valeur devient un nombre décimal et non plus un nombre entier.

</br></br>
## Moi j'ai une version 4entrées et 4 sorties, comment je fais ?
Sur un MsunPv4_4, il y'a plus d'entrées, plus de sortie et donc forcement plus de sensors potentiels à créer. Pour savoir comment les rajouter il faut recupérer les fichiers 'http://IP_DU_MSUNPV/index.xml' et 'http://IP_DU_MSUNPV/status.xml' qui vont permettre de comprendre qu'est ce qu'il faut récupérer.

- index.xml :
```yml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<xml>
      <paramSys>16:11:04;23/10/2023;On;01:00;0,0;MS_PV2_2b;5.0.1;0000220;104a;104a;00:00;00:00</paramSys>
      <typAns>PowReso;1;6:PowPV;1;6:OutBal;0;3:OutRad;0;3:VoltRes;1;4:T_Bal1;1;18:T_SDB;1;18:T_Amb;1;18:S9;0;0:S10;0;0:S11;0;0:S12;0;0:S13;0;0:S14;0;0:S15;0;0:S16;0;0:</typAns>
      <typouts>Cumulus;0;2:Sortie2;0;1:A3;0;0:A4;0;0:A5;0;0:A6;0;0:A7;0;0:A8;0;0:A9;0;0:A10;0;0:A11;0;0:A12;0;0:A13;0;0:A14;0;0:A15;0;0:A16;0;0:</typouts>
      <cmdM0>3;0;Comd Manu/Auto;ManuBal;AutoBal;ManuRad;AutoRad;</cmdM0>
      <cmdM1>0;0;Commande 2;Param1;Param2;Param3;Param4;</cmdM1>
      <cmdM2>0;0;Commande 3;Param1;Param2;Param3;Param4;</cmdM2>
      <cmdM3>0;0;Commande 4;Param1;Param2;Param3;Param4;</cmdM3>
      <cmdM4>0;0;Commande 5;Param1;Param2;Param3;Param4;</cmdM4>
      <cmdM5>0;0;Commande 6;Param1;Param2;Param3;Param4;</cmdM5>
      <cmdM6>0;0;Commande 7;Param1;Param2;Param3;Param4;</cmdM6>
      <cmdM7>1;2;Test routeur;Inject;Zero;Moyen;Fort;</cmdM7>
      <typCpt>EnConso;1;16:EnInj;1;16:EnPV_J;1;16:EnPV_P;1;16:Compt 5;0;0:Compt 6;0;0:Compt 7;0;0:Compt 8;0;0:</typCpt>
</xml>
```
- statuts.xml :
```yml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<xml>
      <rtcc>16:12:45 LU</rtcc>
      <rssi>100;-37</rssi>
      <paramSys>16:12:46;23/10/2023;On;01:00;0,0;MS_PV2_2b;5.0.1;0000220;104a;104a;00:00;00:00</paramSys>
      <inAns>0,2;-609,5;44; 0;233,8;0,0;0,0;0,0; 0; 0; 0; 0; 0; 0; 0; 0;</inAns>
      <survMm>0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;</survMm>
      <cmdPos>0;0;0;0;0;0;0;2;</cmdPos>
      <outStat>10;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;</outStat>
      <cptVals>6333;fffffff6;ffff4e6a;fdd33e02;0;0;0;0;</cptVals>
      <chOutVal>0;0;0;ff;:0,0;0,0;0,0;0,0;</chOutVal>
</xml>
```
Et plus particulièrement ces lignes qui vont nous indiquer quelle est la position de chaque éléments dans le fichier 'status.xml' et donc pouvoir en extraire les bonnes valeurs :
```yml
      <typAns>PowReso;1;6:PowPV;1;6:OutBal;0;3:OutRad;0;3:VoltRes;1;4:T_Bal1;1;18:T_SDB;1;18:T_Amb;1;18:S9;0;0:S10;0;0:S11;0;0:S12;0;0:S13;0;0:S14;0;0:S15;0;0:S16;0;0:</typAns>
      <typCpt>EnConso;1;16:EnInj;1;16:EnPV_J;1;16:EnPV_P;1;16:Compt 5;0;0:Compt 6;0;0:Compt 7;0;0:Compt 8;0;0:</typCpt>
```
Qui une fois reduite au infos qui nous interresse nous donne :
```yml
      <typAns>PowReso:PowPV:OutBal:OutRad:VoltResT_Bal1:T_SDB:T_Amb:S9:S10:S11:S12:S13:S14:S15:S16:</typAns>
      <typCpt>EnConso:EnInj:EnPV_J:EnPV_P:Compt 5:Compt 6:Compt 7:Compt 8:</typCpt>
```
On sait donc maintenant que si je veux récupérer la valeur de 'Outbal' il faudra que je récupère la 3éme valeur de la ligne correspondante dans le fichier 'status.xml'. Outbal pourrais très bien s'appeler tartampion que celà fonctionnerai de la même façon.
</br>De la même manière si je veux la valeur du compteur 'Compt 6' je récupère la 5éme valeur de la ligne correspondante dans le fichier 'status.xml'.
</br>Les correspondance sont :
| index.xml | <=> | status.xml | 
| --------- | --- | ---------- | 
| typAns | <=> | inAns |
| typCpt | <=> | cptVals |

'inAns' correspondant aux 'entrées' et 'cptVals' aus compteurs.

</br>Exemple de code pour récupérer la valeur de 'Compt 5' qui n'ai pas dans la version mSunPv2_2 de base et imaginons que cela corresponde au compteur de la pince qui mesure la puissance en W envoyée au cumulus chaque jour, le code à ajouter serait :
```yml
      - name: msunpv_encumulus #Consommation cumulus journalière
        unique_id: "msunpv_encumulus"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ cptVals.split(";")[4]|int(base=16)/10 |float }}
        unit_of_measurement: "Wh"
        device_class: energy
```
On peut noté que j'ai mis [4] :
```yml
          {{ cptVals.split(";")[4]|int(base=16)/10 |float }}
```
Alors que je veux récupérer la 5ème valeur de la liste ! Ceci est du au fait que le premier élément d'une liste est l'élément 0, le ssecond est l'élément 1 et ainsi de suite. Du coup pour le 5ème je mets 4 entre les corchets.
</br>Si la puissance est en kW :
```yml
      - name: msunpv_encumulus #Consommation cumulus journalière
        unique_id: "msunpv_encumulus"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ cptVals.split(";")[4]|int(base=16)/10 |float }}
        unit_of_measurement: "kWh"
        device_class: energy
```
</br>Et si la puissance est en W et que je veux ma valeur en kWh, je divise par 1000 et je fais un arrondi à 3 chiffre après la virgule :
```yml
      - name: msunpv_encumulus #Consommation cumulus journalière
        unique_id: "msunpv_encumulus"
        state: >-
          {% set cptVals =state_attr('sensor.msunpv_xml', 'cptVals')|replace(" ","") %}
          {{ (cptVals.split(";")[4]|int(base=16)/10 |float /1000)|round(3) }}
        unit_of_measurement: "kWh"
        device_class: energy
```
Pour le 'name' du sensor vous pouvez mettre ce que bon vous semble ainsi que pour son 'unique_id'. Il faut juste que 'unique_id' soit unique par contre.

