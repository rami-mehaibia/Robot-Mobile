

# Projet golf :

Comment on est censé faire que notre robot mette des trous ?

Le robot de base, il est censé mettre une balle dans le trou, mais comment faire ça ?

## Premier élément :

Il faut que notre robot puisse d'abord avancer à une vitesse constante, 125 unités (=u) soit 5 cm/s afin qu'il n'aille pas trop vite).

## Deuximème élément :

Notre robot va devoir lire des codes bar pour la distance et pour les angles
Avec un bit d'amorce et ensuite 3 bits de données.
Notre robot va tout d'abord lire l'amorce et va laisser une pause de 0,75 secondes (car c'est la distance de 1,5 case)
Après notre robot va lire les 3 autres cases à des intervalles de 0,5 secondes (intervalle d'une case)

## Comment un code bar fonctionne :

Les cases noires sont codées 1 et les cases blanches sont codées 0

x= 0 ou 1

Premier bit = 2^0*x = x

Deuximème bit = 2^1*x =2*x

Troisième bit = 2^2*x = 4*x

### Premier code bar :

Distance :

|Lecture de gauche à droite avec amorce|Signification |Lecture de droite à gauche sans amorce|Le temps qu'il faut pour parcourir à une vitesse de 125 u (5cm/s)|
|:------------------------------------:|:------------:|:------------------------------------:|:-:|
|1000|0 dm|000|0 s|
|1100|1 dm|001|2 s|
|1010|2 dm|010|4 s|
|1110|3 dm|011|6 s|
|1001|4 dm|100|8 s|
|1101|5 dm|101|10 s|
|1011|6 dm|110|12 s|
|1111|7 dm|111|14 s|


### Deuximème code bar :

Angle de "frappe" :

|Lecture de gauche à droite avec amorce|Valeur entière en codage défini|Signification |Lecture de droite à gauche sans amorce|
|:------------------------------------:|:-----------------------------:|:------------:|:-------------------------------------:|
|1000|0|0°|000|
|1100|1|+45°|001|
|1010|2|-45°|010|
|1110|3|+90°|011|
|1001|4|-90°|100|
|1101|5|+120°|101|
|1011|6|-120°|110|
|1111|7|0°|111|

## Les actions du robot :

#### 1. Notre robot quand on appuye sur le bouton central, il se met en route en avançant tout droit à la vitesse de 125 u
#### 2. Tant qu'il ne capte pas de tâche/case noire au sol, il avance
#### 3. Lorsqu'il détecte une case noire, il considère que c'est le bit d'amorce pour le premier code bar et il va marquer un temps de pause à la détection pendant 0,75 secondes pour arriver au premier bit.
#### 4. Après ce temps il va lire ce qu'il y a au sol et ensuite il va marquer un temps de pause de 0,5 secondes pour pouvoir arriver au deuxième bit.
#### 5. On répète le processus du 4. encore une fois.
#### 6. Après avoir lu les 3 bits il se remet en état 2.
#### 7. Il va procéder de la même manière que les états 3., 4. et 5. mais cette fois-ci pour faire l'angle de "frappe"
#### 8. Après lecture du dernier bit, il va s'arrêter après 0,25 secondes pour qu'il puisse arriver au bord de la dernière case et qu'il puisse ensuite exécuter les ordres donnés.
#### 9. Tout d'abord il va tourner sur lui même en fonction de la deuxième lecture
#### 10. Une fois qu'il est en place il va avancer à la vitesse de 125 u pour un temps de x secondes en fonction du résultat de la première lecture
#### Bis : Faire que le robot suive la ligne

Tout ce qu'on a écrit au-dessus, on y a pensé nous deux depuis un même PC

# Log 2 de notre aventure robot :

On a réussi à mettre un incrément à notre robot, on va upload cela...

Mais on doit également repenser à nos étapes pour que ça soit plus basique :

## Étapes à suivre (update) :

#### 1. Le robot avance tant qu'il y a du blanc (et est à la recherche d'une trace noir)
#### 2. Dès qu'il détecte une ligne noir, le robot la suit.
#### 3. S'il détecte avec son autre capteur de proximité un premier code barre, il va interpréter comme étant celui de la distance.
#### 4. S'il détecte avec son autre capteur de proximité un deuxième code barre, il va interpréter comme étant celui de l'angle de visée.
#### 5. Dès qu'il finit de lire le deuxième code barre, il va s'arrêter pour exécuter les commandes reçues.
#### 6. Dès qu'il réussit son strike il s'arrête.
