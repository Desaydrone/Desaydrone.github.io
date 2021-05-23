---
title: "Centre de notification : sur la Freebox pop"
date: 2020-12-28 14:00
last_modified_at: 2020-12-28 14:30
excerpt: "Nous allons voir dans cet article comment mettre des notifications en mode PIP (picture in picture) sur la Freebox Pop."
read_time: true
toc: true
toc_label: "Sommaire"
toc_icon: "file-alt"
toc_sticky: true

published: false

gallery:
    - url: /assets/images/notif_freeboxpop/pipup_original.png
      image_path: /assets/images/notif_freeboxpop/pipup_original.png
      alt: "Resultat notification"
      title: "Resultat avec juste un message"
    - url: /assets/images/notif_freeboxpop/pipup_message.png
      image_path: /assets/images/notif_freeboxpop/pipup_message.png
      alt: "Resultat notification avec un message simple"
      title: "Resultat avec une image"
    - url: /assets/images/notif_freeboxpop/pipup_original.png
      image_path: /assets/images/notif_freeboxpop/pipup_original.png
      alt: "Resultat notification"
      title: "Resultat avec une vidéo"

gallery1:
    - url: /assets/images/discordlink_battery_state/01_programmation_scenario.png
      image_path: /assets/images/discordlink_battery_state/01_programmation_scenario.png
      alt: "Programmation cron tab"

gallery2:
    - url: /assets/images/discordlink_battery_state/02_bloc_sialorsaction.png
      image_path: /assets/images/discordlink_battery_state/02_bloc_sialorsaction.png
      alt: "Programmation cron tab"  
---


Je vais vous partager petit a petit une suite d'article sur la mise en place de mon centre de notification. Ca ne sera peut être pas le top du top, mais il va correspondre à mes besoins. Bien sur avec la montée en puissance de la domotique chez moi, cela évoluera peut être au fur et a mesure. Mon but va surtout être de faire des modules dédié à un type de notification avant de mettre en place le centre de notification qui utilisera les différents modules. Nous allons aujourd'hui faire un petit scénario en mode bloc code pour envoyer une notification améliorée à la freebox pop, afin de pouvoir y afficher juste un message texte, une image, une video voir une page web.


## Exemple de ce que l'on peut avoir

Pour ceux qui voudrait voir avant d'en lire plus voila un petit exemple de ce que l'on peut obtenir

{% include gallery caption="Résultat de la notification via PiPup." %}

## Les prérequis

Pour mettre tout cela en place nous allons avoir besoin de deux choses importante :
1. L'application que vous devez installer sur la freebox pop s'appelle [PiPup](https://play.google.com/store/apps/details?id=nl.rogro82.pipup&hl=fr&gl=US). 
1. Le player devra avoir une adresse Ip fixe, c'est très important.




## Le bloc code
Alors je vous le dis tout de suite je ne réinvente pas la roue, j'ai utilisé le code de **[Naboleo](https://community.jeedom.com/t/plugin-pipup-notifications-sur-android-tv/32929/6?u=tarlak)** qu'il à partagé sur le [community](https://community.jeedom.com/t/plugin-pipup-notifications-sur-android-tv/32929/6?u=tarlak) jeedom merci à lui pour le partage, j'ai juste eu besoin de l'adapter pour que ca colle mieux à mes besoins.



## Valeur possibles des tag

| Variable        | Type                                         |
| --------------- | -------------------------------------------- |
| duration        | Integer (default=30)                         |
| position        | Integer (0..4, default=0)                    |
| title           | String                                       |
| titleSize       | Integer (default=16)                         |
| titleColor      | string (default=#FFFFFF, format=[AA]RRGGBB   |
| message         | String                                       |
| messageSize     | Integer (default=12)                         |
| messageColor    | String (default=#FFFFFF, format=[AA]RRGGBB   |
| backgroundColor | String (default=#CC000000, format=[AA]RRGGBB |
| image           | File                                         |
| imageWidth      | Integer (default=480)                        |

`position` is an enum ranging from 0 to 4

|  | Position    |
| -----: | ----------- |
| 0     | TopRight    |
| 1     | TopLeft     |
| 2     | BottomRight |
| 3     | BottomLeft  |
| 4     | Center      |




{: .notice--success}