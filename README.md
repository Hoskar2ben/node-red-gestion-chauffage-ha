# node-red-gestion-chauffage-ha
Voici mon Flow permettant de gerer mes radiateurs de ma maison.

Voici un screen du Flow, j'ai mis des couleurs sur les differentes brique de celui-ci pour que ça soit plus simple à expliquer.

![Screen](/img/2021-11-08_20-05.png)

**Pré-requis**

- node-red-contrib-home-assistant-websocket
- node-red-contrib-simple-message-queue

Explication du Flow
-------------


***Récupératon des informations utile pour le fonctionnement des radiateurs***

le groupe vert en haut.

Permet de générer les données dans un context flow :
![Screen](/img/context_flow.png)

Description des six state node :

"Température choisi manuellement" permet de récupérer la température cible qu'on souhaite avoir dans la pièce.

input_number commençant tcible_ par exemple pour la chambre de ma fille Lylou : 

`input_number.tcible_lylou`

"Calendrier" permet de récupérer la température cible souhaitait dans la pièce.

calendar commençant par chauffage_ : 

`calendar.chauffage_lylou`

"TemperatureCapteur" et "TemperatureCapteur2" permet de récupérer la température de la pièce.

Pour "TemperatureCapteur" :

sensor commençant par temperature_sa_t_ch_ : 

`sensor.temperature_sa_t_ch_lylou`

Pour "TemperatureCapteur" même principe qu'au dessus pour d'autre capteur ne portant pas le même format.

"Fenetre" permet de récupérer l'information si la fênetre est ouverte ou non.

binary_sensor commençant par fenetre_ : 

`binary_sensor.fenetre_lylou`

"ModeAuto" permet activer la gestion auto en utilisant la donnée du calendrier.

ON => Information du `calendar.chauffage_`

OFF => Données du `input_number.tcible_`

input_boolean commençant par mode_auto_ : 

`input_boolean.mode_auto_lylou`

"Infos" permet de récupérer à quelle étape est le flow

Les sensors seront générées automatiquement via le node api.

sensor commençant par infos_ : 

`sensor.infos_lylou`
