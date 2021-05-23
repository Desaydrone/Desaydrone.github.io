---
title: "Améliorer les notifications Discordlink pour les batteries"
date: 2020-08-29 17:30
last_modified_at: 2020-11-28 12:30
excerpt: "Dans cet article nous allons voir comment avoir une notification des batteries en warning ou en état critique de jeedom sur discord"
read_time: true
toc: true
toc_label: "Sommaire"
toc_icon: "file-alt"
toc_sticky: true

published: true

gallery:
    - url: /assets/images/discordlink_battery_state/00_visuel_discord.png
      image_path: /assets/images/discordlink_battery_state/00_visuel_discord.png
      alt: "Resultat notification"

gallery1:
    - url: /assets/images/discordlink_battery_state/01_programmation_scenario.png
      image_path: /assets/images/discordlink_battery_state/01_programmation_scenario.png
      alt: "Programmation cron tab"

gallery2:
    - url: /assets/images/discordlink_battery_state/02_bloc_sialorsaction.png
      image_path: /assets/images/discordlink_battery_state/02_bloc_sialorsaction.png
      alt: "Programmation cron tab"  
---

**Mise à jour 28 Novembre 2020** : Modification du script pour prendre en compte l'exclusion de certain équipements (exemple les virtuels), merci à Sebfar sur discord de m'avoir fait remarqué cela.

Un nouvel article rapide pour mettre en place une notification, pour les équipements munis de batteries dans jeedom ayant un status **warning** ou **danger**, via le plugin discordlink et obtenir quelque chose comme ça :

{% include gallery layout="half" caption="Résultat de la notification Discord." %}

Comme pour la notification pour les daemons, je veux juste l’info de ce qui ne va pas, tant que ça va je n'ai pas besoin d’avoir de notification, c’est une façon de voir qui me suis depuis plusieurs années, ça vient d’une déformation professionnelle où _trop de notifications tue la notification_. Donc je veux la bonne information au bon moment.

## Les prérequis
Pour mettre en place ce qui suit, il nous faudra comme pour l'article sur les notifications des daemons : 

* le plugin [Discord Link](https://market.jeedom.com/index.php?v=d&p=market_display&id=3938)
* un scénario dédié.

Je pars du principe que vous avez réussi à installer le plugin, sinon allez lire la documentation.

Pour le scénario on va y aller par étape.

## Le scénario

### La pragrammation
Pour la programmation, je le fais se déclencher toutes les 12h, ce ne sont que des batteries, sauf si vous avez un gros souci sur un équipement il est peut probable que passiez d’une batterie à 80% et dès le lendemain à 5% donc la pas vraiment besoin d’un état des lieux trop fréquent.

{% include gallery id="gallery1" caption="Programmation du scénario." %}

Quel va être le principe de ce code, il va parcourir tous les équipements qui existent sur mon jeedom, pour chaque équipement je vais tester si il existe une commande contenant le mot **_Batterie_**, une fois que j'ai trouvé cette information je vais récupérer son état si elle est en **_danger_** ou en **_warning_**. Et bien sûr pour chaque état j'alimenterais un compteur correspondant et également j'ajouterais un message qui sera ensuite envoyé à discord.

### Le bloc Code
```php

//Partie 1
$batterie = "Batterie"; // Nom de la commande à rechercher
//$excludeEq = array("[Domotique][Mo]"=>1,"[Appartement][Batteries]" => 1); // Liste des équipements à ignorer (qui contiennent la commande "$batterie"), ceci est un exemple de syntaxe
$excludeEq = array();
$msgA = ""; //Message a envoyer sur l'état des batteries en danger
$msgW = ""; //Message a envoyer sur l'état des batteries en warning

$errEqLogics = array();
$etatBatterie = array(); //Tableau d'information sur la batterie consultée
$cpt_ok=0;
$cpt_w=0;
$cpt_a=0;

$eqLogics = eqLogic::all();


foreach($eqLogics as $eqLogic)
{
  if ($excludeEq[$eqLogic->getHumanName()] == 1){
    continue;
  }
  try{
    //Partie 2
    if (isset($batterie))
    {
      	// si la commande n'existe pas, une exception est levée
    	$cmd = cmd::byString('#' . $eqLogic->getHumanName() . '['. $batterie .']#');
    }

    $allCmds = $eqLogic->getCmd();
    if (count($allCmds) > 0)
    {
      //Partie 3
      foreach($allCmds as $cmd)
      {  
        if (strpos($cmd->getHumanName(),'['.$batterie.']') !== false)
        {
          $lenNom = ($lenNom < strlen($eqLogic->getName())) ? strlen($eqLogic->getName())+1 : $lenNom;
        }
      }

      //Partie 4
      foreach($allCmds as $cmd)
      {  
        if (strpos($cmd->getHumanName(),'['.$batterie.']') !== false)
        {
          $cmd->execCmd();
          
          $nbspace=str_repeat(' ',($lenNom - strlen($eqLogic->getName())));
          $etatBatterie=$eqLogic->getStatus();
          $nom=$eqLogic->getName();
          if($etatBatterie['batterydanger']==1)
          {
            $msgA .= ":red_circle: `".$eqLogic->getName() . $nbspace ." ` - `". $etatBatterie['battery']." %`\n";
            $cpt_a+=1;
          }
          else if($etatBatterie['batterywarning']==1)
          {

            $msgW .= ":orange_circle: `".$eqLogic->getName(). $nbspace ." ` - `".$etatBatterie['battery']." %`\n";
            $cpt_w+=1;
          }else
          {
           $cpt_ok+=1; 
          }
        }
      }
    }
  }catch (Exception $e)
  {
    // pas de commande
  }
  
}

//Partie 5
$scenario->setData('messageA', $msgA);
$scenario->setData('messageW', $msgW);
$scenario->setData('cpt_ok', $cpt_ok);
$scenario->setData('cpt_w', $cpt_w);
$scenario->setData('cpt_a', $cpt_a);

//Partie 6
unset($msgA);
unset($msgW);
unset($cpt_ok);
unset($cpt_w);
unset($cpt_a);
```
{: .full}



#### Explication partie 1

Dans cette partie je vais définir plusieurs variables qui me serviront soit de compteurs soit pour stocker une phrase, un mot. Je vous explique tout de suite les plus importantes pour notre scénario, celles qui vont se retrouver dans notre notification finale.

* `$batterie= "Batterie"` me permet de stocker le mot que je vais rechercher dans le nom des commandes que je vais parcourir pour un équipement,
* `$excludeEq = array();` va me permettre d'ajouter dans le tableau des exclusions, imaginons que votre aspiro autonome passe au moment du check et que batterie est dans un état _Warning_ ou _Danger_ ce n'est peut être pas utile d'avoir une notification,
* `$msgA = "";` va contenir le message pour les batteries en _Danger_, à savoir une émoticon cercle rouge et le nom de l'équipement concerné,
* `$msgW = "";` va contenir le message pour les batteries en _Warning_, à savoir une émoticon cercle orange et le nom de l'équipement concerné,
* `$etatBatterie = array();` à chaque fois que je vais trouver une batterie sur un équipement je vais stocker toutes les informations dans un tableau afin de le consulter et faire certaines intéractions ensuite,
* `$cpt_ok=0;` un simple compteur qui va me permettre de comptabiliser le nombre de batteries n'étant ni en _Danger_ ou en _Warning_,
* `$cpt_w=0;` un deuxième compteur qui va me permettre de comptabiliser le nombre de batteries en _Warning_,
* `$cpt_a=0;` un troisième compteur qui va me permettre de comptabiliser le nombre de batteries en _Danger_,
* `$eqLogics = eqLogic::all();` me permet de récupérer tout les équipements de Jeedom

#### Explication partie 2
A partir de maintenant accrochez-vous on complique les choses. Ici je teste dans mon `if` si la variable `$batterie` contient quelque chose et on essai de voir s'il existe dans l'équipement une commande concernant la batterie. Et j'en profite pour récupérer ensuite toutes les commandes de cet équipement et je teste malgrè tout qu'il y a bien des commandes existants pour cet équipement (on est jamais trop prudent)

#### Explication partie 3
Dans cette partie on va parcourir tous les équipements ayant une commande batterie, et récupérer le nombre de caractères de ce nom, et a chaque fois le comparer à la longueur du nom de l'équipement précédent. Pourquoi donc, parce que je suis un peu maniaque et que je veux une belle présentation et en ayant cette longueur de chaîne de caractères dans `$lenNom` je pourrais faire en sorte de bien aligner les valeurs de mes batteries. Mais je vous en reparle plus tard.

#### Explication partie 4
Cette partie est la plus importante, elle va permettre de tester si une batterie est _OK_, en _Warning_ ou en _Danger_:
* je récupère dans la variable `$nbspace` le nombre d'espace que je devrais rajouter à la fin du nom de l'équipement pour que je puisse créer mon fameux alignement des valeurs de batteries,
* je récupère dans la variable `$etatBatterie` toutes les informations concernant la batterie de l'équipement,
* je récupère dans la variable `$nom` le nom de l'équipement actuellement consulté,
* je teste avec `$etatBatterie['batterydanger']` si la valeur retournée est égale à 1 si c'est le cas ma batterie est en danger et alors j'assigne à ma variable `$msgA` le message que je veux afficher, ici un émoticon cercle rouge, suivi du nom de l'équipement, du nombre d'espaces dont j'ai besoin pour aligner avec le nom d'équipement le plus long. J'en profite ensuite pour incrémenter mon compteur (`$cpt_a`) de batterie en _Danger_,
* je teste avec `$etatBatterie['batterywarning']` si la valeur retournée est égale à 1 et même principe que précédemment je crée mon message pour les batteries en _Warning_ et incrémente le compteur concerné (`$cpt_w`),
* si mes deux tests précédents ont échoué alors j'incrémente mon compteur (`$cpt_ok`) de batterie en état _OK_


#### Explication partie 5
Ici comme pour le scénario avec les daemon je créer des variables Jeedom que je vais pouvoir utiliser dans mon bloc action ensuite, qui enverra ma notification sur discord

#### Explication partie 6
Rien de bien impressionnant ici, je supprime juste les variables php qui me servait de compteur et pour stocker le nom des équipements dont la batterie avait un état _Warning_ ou _Danger_


### Le bloc Si alors et Action

{% include gallery id="gallery2" caption="Bloc Si alors et action." %}

Ici c'est assez simple

1. Je teste si la variable contenant le nombre de batteries en _Warning_  est supérieure à zéro **ou** si le compteur pour les batteries en _Danger_ est supérieur à 0 c'est uniquement dans ce cas-là que je fais la suite,
2. J'envoie le message évolué via un équipement Discorlink, en titre j'affiche mes différents compteurs avec l'emoticon qui va bien, en descriptions je mets les deux messages que j'ai générés pour le _Warning_ et le _Danger_

Voilà avec tout ceci vous avez maintenant une jolie notification sur l'état de vos batteries uniquement quand celles-ci sont en _Warning_ ou _Danger_. 

Je tiens à remercier **Thibaut Black** pour son plugin **Discord Link** qui vraiment me permet de bien mieux gérer mes notifications qu'avec télégram.
{: .notice--success}