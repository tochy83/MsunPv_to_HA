## 🧐 C'est bien beau ton truc mais mes compteurs sont en positifs, comment je fais ?
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
## 🧐 J'ai une sonde qui mesure la puissance du cumulus, comment je fais ?
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
## 🧐Moi j'ai une version 4entrées et 4 sorties, comment je fais ?
