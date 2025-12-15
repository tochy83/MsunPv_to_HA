#### 2025-12-15 - Nouveaux sensors
* Ajout de nouveaux sensors pour prendre en compte les ajouts du dashboard energy depuis la version 2025-12-0 de HA.

  Pour la msunpv_2_2.yaml (lignes 148 à 160)
  ```yml
      - name: "energie_msunpv_powreso" #Puissance du réseau pour le dashboard energie
        unique_id: "energie_msunpv_powreso"
        unit_of_measurement: "W"
        state: "{{ (states('sensor.msunpv_powreso')|float(0)) }}"
        device_class: power
        state_class: measurement

      - name: "energie_msunpv_powpv" #Puissance des panneaux pour le dashboard energie
        unique_id: "energie_msunpv_powpv"
        unit_of_measurement: "W"
        state: "{{ min(0, states('sensor.msunpv_powpv')|float(0))|abs }}" #Valeur absolue de powpv si powpv négative, sinon 0
        device_class: power
        state_class: measurement   
  ```
  
  Pour la msunpv_4_4.yaml (lignes 186 à 212)
  ```yml
      - name: "energie_msunpv_powreso" #Puissance du réseau pour le dashboard energie
        unique_id: "energie_msunpv_powreso"
        unit_of_measurement: "W"
        state: "{{ (states('sensor.msunpv_powreso')|float(0)) }}"
        device_class: power
        state_class: measurement

      - name: "energie_msunpv_powpv" #Puissance des panneaux pour le dashboard energie
        unique_id: "energie_msunpv_powpv"
        unit_of_measurement: "W"
        state: "{{ min(0, states('sensor.msunpv_powpv')|float(0))|abs }}" #Valeur absolue de powpv si powpv négative, sinon 0
        device_class: power
        state_class: measurement        

      - name: "energie_msunpv_powbal" #Puissance du cumulus pour le dashboard energie
        unique_id: "energie_msunpv_powbal"
        unit_of_measurement: "W"
        state: "{{ (states('sensor.msunpv_powbal')|float(0)) }}"
        device_class: power
        state_class: measurement

      - name: "energie_msunpv_powrad" #Puissance du radiateur pour le dashboard energie
        unique_id: "energie_msunpv_powrad"
        unit_of_measurement: "W"
        state: "{{ (states('sensor.msunpv_powrad')|float(0)) }}"
        device_class: power
        state_class: measurement 
  ```
Vous pouvez simplement rajouter les lignes ci dessus à votre fichier msunpv_2_2.yaml ou msunpv_4_4.yaml.
</br></br>




#### 2025-08-04 - Bug Fixes
* Mise à jour des fichiers msunpv_2_2.yaml, msunpv_4_4.yaml, msunpv_addons_progh_2_2.yaml, msunpv_addons_progh_4_4.yaml, msunpv_addons_thermostats.yaml et msunpv_addons_moresensors.yaml pour ajouter un encodage lors de l'appel de l'intégration Restful, qui sans cet encodage, déclenchait une erreur dans les logs de Home Assistant sous certaines conditions. ([e15af79](https://github.com/tochy83/MsunPv_to_HA/commit/e15af798bf9c4f8d19aa5a69bc17ed9a873c9cad), [4e4a3be](https://github.com/tochy83/MsunPv_to_HA/commit/4e4a3bedbca51194ec0af0bb969c47fbd184894c), [b2b0302](https://github.com/tochy83/MsunPv_to_HA/commit/b2b03029e23fa05d8e15a125b68f0a670a1438e3), [eed83ef](https://github.com/tochy83/MsunPv_to_HA/commit/eed83ef0519a95962c47ccff0c0d34b9e127e84b), [16caa88](https://github.com/tochy83/MsunPv_to_HA/commit/16caa885a6af704c8652f85c015c3b20a57841ad))
  ```yml
  Les lignes :
    method: GET
    sensor:
  
  Deviennent :
    method: GET
    encoding: iso-8859-1
    sensor:
  ```
  Correction à apporter dans tous les fichiers cités si vous les utilisez.
  
  Merci à EMqa de me l'avoir signalé.

#### 2025-01-19 - MaJ
* Mise à jour de toutes les automatisations et scripts pour prendre en compte la nouvelle syntaxe de HA.

#### 2024-10-04 - Bug Fixes
* Correction d'une petite coquille qui empêchait les sauvegardes des fichiers CSV de septembre dans le fichier 'msunpv_save_sd_csv.yaml'. ([637a0e3](https://github.com/tochy83/MsunPv_to_HA/commit/637a0e3953eb1c184afbf797d3ab185edb5dead8))
  ```yml
  La ligne :
    {% set month_list = ["JANV", "FEVR", "MARS", "AVRI", "MAI", "JUIN", "JUIL", "AOUT", "SETP", "OCTO", "NOVE", "DECE"] %}
  
  Devient :
    {% set month_list = ["JANV", "FEVR", "MARS", "AVRI", "MAI", "JUIN", "JUIL", "AOUT", "SEPT", "OCTO", "NOVE", "DECE"] %}
  ```
  Attention cette ligne est présente à 2 endroits dans le fichier et il faut bien corriger les deux.

#### 2023-11-08 - Bug Fixes
* Correction d'une petite coquille qui empêchait les scripts de fonctionner dans les fichiers 'msunpv_scripts_2_2.yaml' et 'msunpv_scripts_4_4.yaml'. ([3905024](https://github.com/tochy83/MsunPv_to_HA/commit/3905024966374c015111fc06c1692965957da1d9))
  ```yml
  La ligne :
    msunpv_commandes_routeur: #Envoi des commandes au MsunPv
  
  Devient :
    msunpv_commande_routeur: #Envoi des commandes au MsunPv
  ```
