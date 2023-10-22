# <p align="center">MsunPv to HA</p>
  
<p align="center">Une intégration au format .yaml pour intégrer le routeur solaire MsunPv de [Ard-tek](https://ard-tek.com/) à Home Assistant.</p>

## 🛠️ Installation

- Télécharger le fichier 'msunpv.yaml'
- Créer un dossier nommé 'packages' a la racine du dossier 'config' de Home Assistant
- Copier le fichier téléchargé 'msunpv.yaml' dans le dossier 'packages'
- Ouvrir le fichier 'msunpv.yaml' et remplacer dans celui-ci 'IP_DU_MSUNPV' par l'adresse ip de votre MsunPv <span style="color:orange">partout où cela est nécessaire</span> dans le fichier puis sauvegarder.

    ```js
    - resource: http://IP_DU_MSUNPV/status.xml
    ```
    Devient, si l'adresse ip de votre MsunPv est '192.168.0.111'
    ```js
    - resource: http://192.168.0.111/status.xml
- Ajouter dans le fichier 'configuration.yaml' de Home Assistant les lignes suivantes
    ```
    homeassistant:
      packages: !include_dir_named packages
    ```
    Si la ligne 'homeassistant:' n'est pas déja présente sinon ajouter juste
    ```
      packages: !include_dir_named packages
    ```
    A la suite dessous de celle-ci
- Sauvegarder et redémarrer complétement Home Assistant

