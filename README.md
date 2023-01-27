# Solutions pour le numérique



**Etape 1 - Connexion de la carte ESP8266 et du ASAIR AM2302**

![image](https://user-images.githubusercontent.com/47890166/215096393-1fed3aed-eb4d-4f93-8dcd-2b1f0fcc5072.png)

Voici ci-dessus les branchements effectués.

**Etape 2 - Installation des librairies utiliées en Arduino**

```
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <Hash.h>
#include <ESPAsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
```

Le premier objectif de notre projet a été de récupérer les données des capteurs, puis connecter la carte au Wifi afin de les rendre accessibles sur une page internet mise à jour en temps réel et hébergée par la carte elle même sur localhost (soit l'adresse IP du Wifi auquel elle était connectée)

![image](https://user-images.githubusercontent.com/47890166/215098297-d678e409-883f-4426-b123-893fa7ddad32.png)

**Etape 3 - **Broker MQTT**

Le second objectif fut d'envoyer sur le broker mis à notre dispotion les informations des capteurs que nous recueillons.
Il s'agissait jusque là d'envoyer la temperature et l'humidité sur un tropic chacun (donc 2 topics)

Je n'ai pas de captures d'écran à fournir pour cette partie car le code a évolué pour les parties suivates.
En substance, depuis MQTT Explorer nous voiyons quelque chose proche de ceci:

```
grpMash
    | temperature = 32
    | humidity = 21
```

**Etape 4 - Node Red (#1)**

Une fois ceci fait, le nouvel objectif fut via Node Red d'accéder à nos valeurs et les afficher.
Je rappelle qu'à ce moment là de notre projet, les données envoyées **n'étaient pas encore au format JSON**

Voici notre schéma Node:

![image](https://user-images.githubusercontent.com/47890166/215099867-3fdf9449-b4d6-412d-a5a4-83832b559746.png)

Voici le résulat affiché:

![image](https://user-images.githubusercontent.com/47890166/215099920-e143df28-6380-452b-b736-1a8f41cc6666.png)

**Etape 5 - JSON et SparkplugB**

Nous avons ensuite eu à envoyer nos données sur le broker MQTT au format JSON afin de s'en servir sur Node Red pour adapter le format de notre payload aux conventions SparkPlugB.

Le code Arduino a donc du être modifié pour envoyer un JSON, ce dernier est présent dans le repo et peut être consulté.
Nous obtenions alors ceci (Regardez "GrpMash", c'est le nom de notre groupe car je ne savais pas encore à ce moment là que nous étions le groupe 7)

![image](https://user-images.githubusercontent.com/47890166/215100615-47ca4412-44da-4de8-b7d5-bd3f3f86fb22.png)

Une fois ce format JSON obtenu et respectant le format "ESIEA/grpN" nous avons eu à nous occuper de la partie Red Node.
Il s'agissait de faire en sorte d'avoir une payload dans le format:

```
  metrics:[{
          name: "Humidity",
          floatValue: msg.payload.Humidity
      },
      {
          name : "Temperature",
          floatValue: msg.payload.Temperature
      }]
```

Une fois ceci fait, nous devions adapter notre payload à la surcouche SparkplugB. Pour cela nous avons eu à installer un module `Protobuf` afin d'avoir accès aux nodes "Decode" et "Encode". Nous nous sommes servis du prototype récupéré à l'adresse suivante `https://github.com/Cirrus-Link/Sparkplug/blob/master/sparkplug_b/sparkplug_b.proto` afin de configurer notre encodeur et décodeur que nous utiliserons pour notre payload.

Voici à quoi ressemble notre Flow en Red Node, vous pouvez accéder au fichier JSON associé à ce dernier dans le repo

![image](https://user-images.githubusercontent.com/47890166/215102432-a37c1042-1fa2-463a-9458-9d8dc81ada83.png)
Nous avons décodé les données AVANT de les envoyer sous votre demande car ces dernieres sont illisibles sur MQTTExplorer, surement du à un problème de reconnaissance de l'UTF-8. Autrement, nous ne les décoderions pas

Voici à quoi ressemblaient depuis MQTTExplorer les données lorsqu'elles sont encodées:

Une ceci fait, à défaut d'avoir un RIO fonctionnel en classe vers lequel envoyer nos données via MQTT, nous choisirons de renvoyer au même broker les informations dans le format SparkplugB.

![image](https://user-images.githubusercontent.com/47890166/215102052-c5819ec3-0aaf-4206-9435-3c17504130f1.png)


