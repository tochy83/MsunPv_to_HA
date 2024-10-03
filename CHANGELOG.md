
#### 2024-10-04 - Bug Fixes
* Correction d'une petite coquille qui empêchait les sauvegardes des fichiers CSV de septembre dans le fichier 'msunpv_save_sd_csv.yaml'. ([637a0e3](https://github.com/tochy83/MsunPv_to_HA/commit/637a0e3953eb1c184afbf797d3ab185edb5dead8))
  ```yml
  La ligne :
    {% set month_list = ["JANV", "FEVR", "MARS", "AVRI", "MAI", "JUIN", "JUIL", "AOUT", "SETP", "OCTO", "NOVE", "DECE"] %}
  
  Devient :
    {% set month_list = ["JANV", "FEVR", "MARS", "AVRI", "MAI", "JUIN", "JUIL", "AOUT", "SEPT", "OCTO", "NOVE", "DECE"] %}
  ```
  Attention cette ligne est présente à 2 endroit dans le fichier et il faut bien corriger les deux.

#### 2023-11-08 - Bug Fixes
* Correction d'une petite coquille qui empêchait les scripts de fonctionner dans les fichiers 'msunpv_scripts_2_2.yaml' et 'msunpv_scripts_4_4.yaml'. ([3905024](https://github.com/tochy83/MsunPv_to_HA/commit/3905024966374c015111fc06c1692965957da1d9))
  ```yml
  La ligne :
    msunpv_commandes_routeur: #Envoi des commandes au MsunPv
  
  Devient :
    msunpv_commande_routeur: #Envoi des commandes au MsunPv
  ```
