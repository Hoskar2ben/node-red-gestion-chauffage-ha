# node-red-gestion-chauffage-ha
Voici mon Flow permettant de gerer mes radiateurs de ma maison.

Voici un screen du Flow, j'ai mis des couleurs sur les differentes brique de celui-ci pour que ça soit plus simple à expliquer.

![Screen](/img/2021-11-08_23.49.png)

**Pré-requis**

- node-red-contrib-home-assistant-websocket
- node-red-contrib-simple-message-queue

# Explication du Flow


## ***Récupératon des informations utile pour le fonctionnement des radiateurs***

le groupe vert en haut.

Description des six state node :

- "Température choisi manuellement" permet de récupérer la température cible qu'on souhaite avoir dans la pièce.

    input_number commençant tcible_ par exemple pour la chambre de ma fille Lylou : 

    `input_number.tcible_lylou`

- "Calendrier" permet de récupérer la température cible souhaitait dans la pièce.

    calendar commençant par chauffage_ : 

    `calendar.chauffage_lylou`

- "TemperatureCapteur" et "TemperatureCapteur2" permet de récupérer la température de la pièce.

    Pour "TemperatureCapteur" :

    sensor commençant par temperature_sa_t_ch_ : 

- `sensor.temperature_sa_t_ch_lylou`

    Pour "TemperatureCapteur" même principe qu'au dessus pour d'autre capteur ne portant pas le même format.

    "Fenetre" permet de récupérer l'information si la fênetre est ouverte ou non.

    binary_sensor commençant par fenetre_ : 

- `binary_sensor.fenetre_lylou`

    "ModeAuto" permet activer la gestion auto en utilisant la donnée du calendrier.

    ON => Information du `calendar.chauffage_`

    OFF => Données du `input_number.tcible_`

    input_boolean commençant par mode_auto_ : 

    `input_boolean.mode_auto_lylou`

- "Infos" permet de récupérer à quelle étape est le flow

    Les sensors seront générées automatiquement via le node api.

    sensor commençant par infos_ : 

    `sensor.infos_lylou`

Tous les blocs Transform Data permettent de stocker les données dans le context flow

Exemple pour Lylou :

![Screen](/img/context_flow.png)

## ***MAJ Input_number permettant d'afficher la valeur de chauffe***

le groupe jaune.

Permet la mise à jour des `input_number.tcible_` lors de la reception d'information d'un calendrier lorsque le mode auto est actif.

## ***Lancement manuel de l'alimentation des variables FLOW***

le groupe vert clair.

Permet de l'alimentation des context flow manuellement en récupérant toutes les données.

## **Permet de donner l'ordre de chauffe + affichage de celui ci dans un input***

le groupe en bleu

Permet de donner l'ordre de chauffe suivant les conditions suivantes : 

```js
if(msg.fenetre == true){
    msg.etat = 'fenetre_ouverte';
    msg.etat_radiateur = 'off';
    msg.pass=true;
}else{
    if(msg.value_input_number >msg.value_temperature){
        if(msg.timer < Date.now() || msg.timer == undefined){
            msg.etat = 'Actif '+(msg.timer_chauffage/60000)+' min'; 
            msg.etat_radiateur = 'on';
            msg.pass=true;
        }else{
            //msg.etat = 'wait'
            msg.pass=false;
        }
    }else{
        msg.etat = 'off'; 
        msg.etat_radiateur = 'off';
        msg.pass=true
    }
}
```

Le function node permet également de paramétrer le nom de l'entity id du radiateur via le bloc suivant :

```js
if(msg.person == 'elyot'){
    msg.entity_radiateur='light.qubino_goap_zmnhjd1_flush_dimmer_pilot_wire_level_3';
    msg.nom_temperature = msg.person+'.sensor.temperature_sa_t_ch_'+msg.person;
}else if(msg.person == 'lylou'){
    //msg.entity_radiateur='light.qubino_goap_zmnhjd1_flush_dimmer_pilot_wire_level_2'
    msg.entity_radiateur='input_boolean.alumage_chauffage_lylou' ;
    msg.nom_temperature = msg.person+'.sensor.temperature_sa_t_ch_'+msg.person;
}else if(msg.person == 'parents'){
    msg.entity_radiateur='light.qubino_goap_zmnhjd1_flush_dimmer_pilot_wire_level';
    msg.nom_temperature = msg.person+'.sensor.temperature_sa_t_ch_'+msg.person;
}else if(msg.person == 'bureau'){
    msg.entity_radiateur='switch.on_off_light_5';
    msg.nom_temperature = msg.person+'.sensor.temp_bureau_bt_temperature';
}else if(msg.person == 'salonaquarium'){
    msg.entity_radiateur='switch.radiateur_salon_d';
    msg.nom_temperature = msg.person+'.sensor.temp_salonaquarium_bt_temperature';
}else if(msg.person == 'salontele'){
    msg.entity_radiateur='switch.radiateur_salon_g';
    msg.nom_temperature = msg.person+'.sensor.temp_tele_bt_temperature';
}
```

Dans ce node function il y a également le paramétrage du timer pour la durée de chauffe et d'attente après extinction du radiateur.

Les timers sont paramétrés avec les deux msg. suivant 

```
msg.timer_relance = 60000*15; // relance dans 15 min
msg.timer_chauffage = 60000*10; // 10min
```

Le node API permet de créer des entités sensor pour avoir l'infos du fonctionnement.

## ***Execution chauffage X min maxi + wait X min***

Permet d'allumer le radiateur pendant X min puis d'attendre X min. 

Une fois le traitement réaliser celui ci va vérifier s'il faut recommencer le cycle.


## ***Pour corriger bug raditeur LYLOU***

Utile pour mon utilisation, j'ai un soucis sur un qubino qui indique qu'il est en off alors qu'il fonctionne.