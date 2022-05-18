# Door-control-BLE
Contrôler une porte grâce à la proximité d’un appareil BLE et partager des données par Wifi entre esp32 et esp8266.


# I-	Conception électronique 


a)	esp32 BLE



Le Bluetooth Low Energy est une technologie de réseau personnel sans fil utilisée pour transmettre des données sur de courtes distances. Comme son nom l'indique, il est conçu pour une faible consommation d'énergie et un faible coût, tout en conservant une portée de communication similaire au Classic Bluetooth. 
Nous utiliserons le module esp32 Wrover-CAM possédant cette technologie pour notre projet. Celui-ci agira en tant que client, attendant de recevoir des données issues d’un serveur en scannant les appareils autour de lui.
Le serveur sera une technologie Ibeacons : Les écouteurs Bluetooth BUDS Live agissent en tant que serveur émettant en permanence un signal appelé advertising signal et transmettant ses données aux appareils clients à proximité.
Lorsque le boitier des écouteurs sera allumé, le module esp32 détectera sa présence et récupéra comme donnée la valeur RSSI du signal soit la puissance du signal reçu.
Nous pourrons ainsi connaitre la proximité entre le serveur et le client.



b)	esp32 BLE + ESP8266



Le module esp32 ne sera pas situé à côté de la porte, il nous sera donc impossible de le brancher au vérin électrique pour commander son ouverture.
Il nous faudra communiquer en Wifi avec un esp8266 qui sera, lui, charger de contrôler le mécanisme.



c)	Vérin électrique



Le vérin électrique est commandé en 12V à l’aide d’une batterie et d’un relai branché à l’esp8266.
Lorsque la carte reçoit par Wifi l’indication si les écouteurs sont proches ou éloignés, elle agira sur le vérin.



# II-	Conception informatique



d)	Mise en place du client BLE 



Pour notre projet, nous utiliserons la bibliothèque Esp32 BLE Arduino. Nous souhaitons donc mettre en place un client de manière à récupérer les informations du boitier des écouteurs.
Dans un  premier temps, nous devons récupérer l’adresse MAC de nos écouteurs afin d’identifier le bon appareil. Nous pouvons télécharger une application comme BLE scanner pour récupérer cet UUID et la mettre dans une variable.
Nous emploierons principalement le code fournit en exemple dans la bibliothèque utilisée. Une fonction callback est appelée à chaque fois que des appareils disposant de la technologie Bluetooth sont détectés à un intervalle que l’on choisira. Celle-ci les scanne et nous renvoie leurs informations. 
Dans le loop, nous ne regardons alors que leur adresse MAC pour les comparer à l’adresse de nos écouteurs. Ce n’est que lorsque celles-ci sont identiques que l’on affiche une nouvelle information de l’appareil : le RSSI soit la force du signal reçu par le client.
Ainsi grâce au RSSI, nous pouvons déterminer la proximité ou non de l’appareil avec l’esp32 puis déterminer une action à réaliser en fonction.



a)	Communication Wifi entre appareil



Comme mentionné, l’actionneur et l’esp32 ne seront pas situés à côté. Il sera donc nécessaire à l’esp32 d’envoyer des données à une carte esp8266 qui, elle, interagira avec le vérin. Pour les faire communiquer, nous utilisons le protocole Esp-Now.



Esp32 : sender

L’esp32 va être chargé d’envoyer la donnée suivante : les écouteurs sont à proximité de l’esp32 oui/non.
Préalablement, tout comme pour le boitier, nous devons récupérer l’adresse MAC de l’esp8266. Nous utiliserons un programme Arduino qui permettra de l’obtenir.
Pour envoyer nos données, nous déclarons une structure pouvant contenir n’importe quel type de variable. Dans notre cas, simplement un type char pour 0 ou 1.
Nous mettons également en place une structure type esp_now_peer_info_t nommé peer_info pour stocker les informations de l’appareil avec lequel on cherche à communiquer. Dans le setup, nous copions l’adresse MAC de l’esp8266 pour la mettre dans la variable peer_addr de la structure peer_info. Nous appelons la fonction esp_now_add_peer() en mettant l’adresse de peer_info pour coupler les appareils.

Enfin dans le loop, nous envoyons dans la fonction esp_now_send(),  l’adresse MAC de l’appareil avec lequel on cherche à communiquer, l’adresse de la structure contenant nos données et sa taille.



Esp8266 : receiver

On créé une structure avec le même nom que celle de l’envoyeur.
Dans le setup, on initialise l’esp-Now. On met également en place une callback fonction qui sera appelée à chaque fois que la carte recevra des informations de l’envoyeur.
Les paramètres de la fonction callback correspondent directement aux paramètres envoyés par la fonction send() de l’envoyeur : l’adresse mac du récepteur, la structure contenant les données, la taille de la structure.
Pour manipuler la structure de données envoyée par la carte, on la copie dans notre propre structure créée.


b)	Activation de la porte


Nous recevons donc dans notre structure les informations 1 ou 0 et avec une simple condition, nous actionnons le vérin électrique.
En s’approchant avec les écouteurs, celui-ci déverrouille la porte et inversement en s’éloignant 


![ezgif com-gif-maker (6)](https://user-images.githubusercontent.com/92324336/169152523-1fde116c-a971-4a41-909e-4755c3ce8719.gif)

![ezgif com-gif-maker (7)](https://user-images.githubusercontent.com/92324336/169152541-9dbc8e03-631b-4c4d-b821-3ed2c6dfcece.gif)
