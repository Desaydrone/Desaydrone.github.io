---
title: "Améliorer les notifications Discordlink pour les deamons"
date: 2020-08-29 17:30
excerpt: "Dans cet article nous allons voir comment avoir une notification des deamons arrêté dans jeedom sur discord"
read_time: true
toc: true
toc_label: "Sommaire"
toc_icon: "file-alt"
toc_sticky: true

gallery:
    - url: /assets/images/discordlink_deamon/00_apercu_notification.png
      image_path: /assets/images/discordlink_deamon/00_apercu_notification.png
      alt: "Resultat notification"


gallery1:
    - url: /assets/images/discordlink_deamon/01_scenario_programmation.png
      image_path: /assets/images/discordlink_deamon/01_scenario_programmation.png
      alt: "Programmation cron tab"

gallery2:
    - url: /assets/images/discordlink_deamon/02_scenario_notification.png
      image_path: /assets/images/discordlink_deamon/02_scenario_notification.png
      alt: "Bloc Si alors et action"

gallery3:
    - url: /assets/images/discordlink_deamon/03_totalite_scenario.png
      image_path: /assets/images/discordlink_deamon/03_totalite_scenario.png
      alt: "Bloc Si alors et action"


---
**Mise à jour 28 Décembre 2020** : Ajout de l'exclusion, afin de permettre de ne pas prendre en compte certain deamon, par exemple dans mon cas j'ai blea d'installé sur jeeom, mais le deamon n'est pas activé. Je n'utilise que des antennes.

Petit article rapide pour mettre en place, une notification pour les daemons jeedom ayant un problème, via le plugin discordlink et obtenir quelque chose comme ça :

{% include gallery layout="half" caption="Résultat de la notification Discord." %}

Je suis du genre à vouloir savoir ce qui ne fonctionne pas, ce qui est fonctionnel ne m'intéresse pas, car je n'ai pas besoin de le savoir. Ça engendrerait trop de notification. Du coup; la notification de base du plugin ne me convenait pas.

Donc pour mettre cela en place il va nous falloir le plugin [Discord Link](https://market.jeedom.com/index.php?v=d&p=market_display&id=3938), disponible sur le market de jeedom. Je vous laisse lire la documentation du plugin pour le mettre en place.

Et en deuxième lieu il nous faudra un scénario permettant de faire ce qu'on veut.

## Côté scénario

Première chose à faire, la création d’un scénario, pour ma part il est programmé pour s’exécuter toutes les 6 minutes

{% include gallery id="gallery1" caption="Programmation du scénario." %}



### Bloc code
Ensuite, dans un bloc code je saisis ce qui suit  :

```php

//Partie 1
$daemon = 0;
$ok = 0;
$nok = 0 ;
$ListeDaemonKO = '';
//Permet l'exclusion de certain deamon qui ne sont pas a surveiller, ici dans l'exemple Blea qui ne tournerais pas sur jeedom mais sur des antennes
$excludeEq = array("Bluetooth Advertisement" => 1);

//Partie 2
foreach (plugin::listPlugin(true) as $plugin) {
	if ($plugin->getHasOwnDeamon() == 1) {
		$daemon ++;
		if($plugin->deamon_info()['state'] == 'ok') {
			$ok++;
		} else {
			//Test les exclusions
			if ($excludeEq[$plugin->getName()]!=1){
				$nok++;
				$ListeDaemonKO .= ":red_circle: ".$plugin->getName() . "\n";
			}
		}
	}
}

//Partie 3
$scenario->setData('okD',$ok);
$scenario->setData('koD',$nok);
$scenario->setData('listKoD',$ListeDaemonKO);
```

Non, je ne vous laisse pas dans l’ignorance, voilà une petite explication simple

### Partie 1 du bloc code

```php
//Partie 1
$daemon = 0;
$ok = 0;
$nok = 0 ;
$ListeDaemonKO = '';
//Permet l'exclusion de certain deamon qui ne sont pas a surveiller, ici dans l'exemple Blea qui ne tournerais pas sur jeedom mais sur des antennes
$excludeEq = array("Bluetooth Advertisement" => 1);
```

Phase d'initilisation du bloc code, je définis des variables qui vont récupérer certaines infos :

* $deamon : compteur qui me retournera à la fin le nombre de daemons existant sur mon jeedom
* $ok : compteur qui me retournera le nombre de daemons qui ne sont pas en erreur
* $nok : compteur qui me retournera le nombre de daemons en erreur
* $ListeDeamonKO = variable qui contiendra la liste nominative des déamons en erreur (ex : Bluetooth Advertissement)

### Partie 2 du bloc code

```php
//Partie 2
foreach (plugin::listPlugin(true) as $plugin) {
	if ($plugin->getHasOwnDeamon() == 1) {
		$daemon ++;
		if($plugin->deamon_info()['state'] == 'ok') {
			$ok++;
		} else {
			//Test les exclusions
			if ($excludeEq[$plugin->getName()]!=1){
				$nok++;
				$ListeDaemonKO .= ":red_circle: ".$plugin->getName() . "\n";
			}
		}
	}
}
```

Ce bloc de code va parcourir tous les plugins installés sur mon jeedom et regardé a chaque plugin si un daemon existe, si un daemon existe alors j’augmente mon compteur `$daemon` de 1. Ensuite, je teste l’état du daemon si il est égal à OK alors j’incrémente ma variable `$ok` de 1, dans le cas contraire alors j’incrémente la variable `$nok` de 1 et en plus je concatène à ma variable `$ListeDaemonKO` le nom du daemon (avec devant une emoticon cercle rouge) 

### Partie 3 du bloc code

```php
//Partie 3
$scenario->setData('okD',$ok);
$scenario->setData('koD',$nok);
$scenario->setData('listKoD',$ListeDaemonKO);
```

Ici, rien de bien compliqué, pour chaque variable de mon bloc code je crée une variable **Jeedom**, elles vont alors me permettre de passer les valeurs au plugin **Discord Link**.

## Suite du scénario bloc Si alors...

On ajoute un bloc **si alors sinon** et ensuite dans celui-ci un bloc action

{% include gallery id="gallery2" caption="Bloc Si alors et action." %}

1. On test si la variable koD est différente de 0 et si c'est le cas on envoi notre notification
2. On utilise l’envoi de message évolué d’une commande du plugin **Discord Link**, dans le titre je récupère mes différents compteurs `$okD` (nombre de daemon fonctionnel) et `$koD` (nombre de deamon non fonctionnel). Dans la description je récupère le contenu de la variable `$listKoD` (nom des daemons en erreur)

Et c’est tout, avec ceci j’ai juste la liste des daemons qui ont un souci, avec quand même cette petite info du nombre de daemon fonctionnel dans le titre de la notification.

{% include gallery id="gallery3" caption="Le scénario au complet." %}

Si vous avez été attentif vous allez me dire que je n'ai pas utilisé le contenu de ma variable : `$daemon` et vous avez bien raison. Je pourrais la rajouter, dans le titre de ma notification ou dans le footer, pour le moment le résultat me convient. Je la garde quand même de côté au cas où.


On pourrait améliorer le scénario également en prévoyant des exceptions aussi pour ne pas tester l'état de certains daemon. Je pense par exemple au Daemon Bluetooth Advertisement qui ne tourne pas forcement sur jeedom mais sur des antennes. Ce serait dommage que toutes les 6 minutes on reçoive une notification d'un daemon que nous avons volontairement arrêté.



Je tiens à remercier **Thibaut Black** pour son plugin **Discord Link** qui vraiment me permet de bien mieux gérer mes notifications qu'avec télégram.
{: .notice--success}