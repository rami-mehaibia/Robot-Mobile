# Présentation de notre calibrage et distance qu'on sélectionne nous même

Alors... La première fois qu'on a écrit ceci, ça n'a pas marché car la connexion internet s'est interrompue... Donc on va retenter.
Notre robot en employant le code dans le dossier de Calibrage est capable de faire les choses suivantes :

- Il est capable de lire le temps sur une distance de calibrage qu'on peut sélectionner au préalable
- Il est capable d'aller à une distance donné en utilisant le calibrage établi précédemment.

Pour cela, notre robot a 4 états différents (le 5ème étant juste une action spéciale) :
1. Lecture de la distance de calibrage et de la vitesse qu'on donne au robot
2. Lecture du temps dès qu'il se trouve sur la piste noire de calibrage
3. Lecture de la distance qu'on veut lui faire faire au robot.
4. Action du robot pour faire la distance séléctionné avec la vitesse séléctionnée au début
5. Arrêt Mannuel

## Voici les détails :
Les états se changent en appuyant sur le bouton central
### État 1: 

Il y a deux états de choix qu'on change à chaque bouton haut:
- le choix de la vitesse de calibrage (comprise entre 0 et 500 unités avec un incrément de 25 unités) qui change de couleur (en allant vers le rouge)
- le choix de la distance de calibrage (comprise entre 0 et 40 cm) qui change de couleur (en allant vers le vert) tout les 2 incréments de 1 cm.

## État 2:

Le robot tant qu'il ne détecte pas de noir, il avance à une vitesse constante (avec une couleur bleu)
dès qu'il détecte du noir, il met sa vitesse séléctionnée et lit le temps tout les 0,1 s (couleur vert)
dès qu'il redétecte du blanc il s'arrête (et du coup le temps s'arrête) (couleur cyan)

## État 3:

Le robot maintenant lit la distance à effectuer entre 0 et 40 cm pour la distance à effectuer (change couleur de plus en plus magenta)

## État 4:

Le robot exécute la distance en question

## Spécial:

En appuyant sur le bouton de bas, on enclenche l'arrêt manuel qui cette fois arrête tout et remet tout à 0. (signalé par la couleur jaune)



# Quelques détails sur les mesures qu'on a pris sur notre simulateur :
| Vitesse en u | Temps produit en s | Vitesse en cm/s |
| ------ | ------ | ----- |
|0	|infinity| 0
|25	|35,2|	1,136363636|
|50	|17,5|	2,285714286|
|75	|11,8|	3,389830508|
|100	|8,9|	4,494382022|
|125	|7,1|	5,633802817|
|150	|6|	6,666666667|
|175	|5|	8|
|200	|4,4|	9,090909091|
|225	|3,9|	10,25641026|
|250	|3,6|	11,11111111|
|275	|3,2|	12,5|
|300	|3|	13,33333333|
|325	|2,7|	14,81481481|
|350	|2,5|	16|
|375	|2,4|	16,66666667|
|400	|2,3|	17,39130435|
|425	|2,1|	19,04761905|
|450	|2|	20|
|475	|1,9|	21,05263158|
|500	|1,8|	22,22222222|
