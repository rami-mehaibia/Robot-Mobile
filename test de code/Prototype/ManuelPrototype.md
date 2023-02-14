#### Bonjour, aujourd'hui on va vous présenter comment notre prototype de robot golfeur fonctionne :

Pour cela on va vous décrire les états, et ce que le robot fait dans chacune de ces étapes.

#### Tout d'abord nous allons parler des états du robot.

### État 1 : Le robot attend que vous lui faites enregistrer une vitesse en cm/s avec les boutons droite et gauche du robot.
### État 2 : Le robot va suivre la ligne jusqu'à ce qu'il puisse détecter une amorce et va ainsi lire 2 codes barres.
### État 3 : Le robot une fois qu'il est lu les 2 codes barres, il va se mettre à tourner sur lui même en fonction de ce que le deuxième code barre lui a dit puis il va avancer à la distance que le premier code barre lui avait dit.
### État spécial : Arrêt manuel

## État 1 : Lecture de la vitesse...

Voici le code qui nous permet de lire la vitesse dans aseba

```
onevent button.right #incrémente en cm/s
if  switch==0 then
	if button.right==1 then
			if impaire==1 then
			speedcms+=23
			impaire=0
			colour++
			else
			speedcms+=22
			impaire=1
			colour++
			end
			if speedcms>500 then #revient à 0 si la vitesse surpasse 500 u
				speedcms=0
				impaire=1
				colour=10
			end
			call leds.top(colour,0,0)
	end
end

onevent button.left #décremente en cm/s
if  switch==0 then
	if button.left==1 then
			if impaire==1 then
			speedcms-=22
			impaire=0
			colour--
			else
			speedcms-=23
			impaire=1
			colour--
			end
			if speedcms<0 then #revient à 495 si la vitesse est en dessouds de 0 u
				speedcms=495
				impaire=1
				colour=32
			end
			call leds.top(colour,0,0)
	end
end
```

La variable `switch` correspond au mode principal auquel on se situe. Ici on lit les vitesse quand `switch==0`.

Pour ce qui est de la vitesse, la variable `speedcms` s'occupe de la vitesse en unités du robot

La variable `colour` a pour rôle d'à la fois faire office d'information visuelle sur quelle vitesse on a à chaque fois  qu'on appuye sur gauche ou droite, mais détient aussi l'information du nombre de cm/s (avec `colour-10` étant le nombre de cm/s et le robot ayant une limite de 22cm/s)

Pour obtenir l'information/conversion entre les unités du robot (qui ont pour maximum 500u) et la vitesse en cm/s, on a créé un base de donnée pour une piste de 40 cm  avec le temps qu'a mis le robot de la faire :

| Vitesse en u |	Temps produit en s	| Vitesse en cm/s |
| ------------ | ---------------------- | --------------- |
| 0 |	infinity |	0 |
| 25 |	35,2 |	1,136363636 |
| 50 |	17,5 |	2,285714286 |
| 75 |	11,8 |	3,389830508 |
| 100 |	8,9 |	4,494382022 |
| 125 |	7,1 |	5,633802817 |
| 150 |	6 |	6,666666667 |
| 175 |	5 |	8 |
| 200 |	4,4 |	9,090909091 |
| 225 |	3,9 |	10,25641026 |
| 250 |	3,6 |	11,11111111 |
| 275 |	3,2 |	12,5 |
| 300 |	3 |	13,33333333 |
| 325 |	2,7 |	14,81481481 |
| 350 |	2,5 |	16 |
| 375 |	2,4 |	16,66666667 |
| 400 |	2,3 |	17,39130435 |
| 425 |	2,1 |	19,04761905 |
| 450 |	2 |	20 |
| 475 |	1,9 |	21,05263158 |
| 500 |	1,8 |	22,22222222 |

Après cela on a fait une régression linéaire pour obtenir cette formule : x = y/0,0444 - 0,1061 avec x étant égal au nombre d'unités du robot et y le nombre de cm/s, et on a observé que lorsqu'on passe d'un nombre impaire à un nombre impaire, la différence est de 23u, tandis que quand on passe d'un nombre paire à un nombre impaire, la différence est de 22u.

On peut voir cela dans cette partie du code :
```
			if impaire==1 then
				speedcms-=22
				impaire=0
				colour--
			else
				speedcms-=23
				impaire=1
				colour--
			end
```
La variable `impaire` correspond à une sorte de booléen, à chaque fois que l'on appuye sur les boutons de droite ou de gauche, impaire va changer en fonction de la valeur en cm/s (au début, impaire est automatiquement égal à 1).

#### Ainsi en appuyant sur le bouton de droite on augmente le nombre de cm/s et avec le bouton de gauche on diminue ce nombre.

Cela dit il faut savoir que dans ce code prototype, cette vitesse correspond à quand le robot fait sa rotation et sa distance après la lecture de données. Car pour ce qui est de la lecture des données et de la suivie de ligne, on reste constamment à 1 cm/s.

Aussi au tout début, le robot s'illumine sur le cercle de manière moyenne afin de dire qu'il est prêt à lire, et des lumières sur le côtés indique qu'on est dans le premier état.

## État 2: Suivi de ligne et saisie des données

Voici le code qui nous fait à la fois suivre une ligne et saisir des données :

```
if  switch==1 then
	if (etatg<gris-100) then #si gauche voit du noir, alors il vire à droite
		motor.left.target=23 + ((gris-etatg)%23*2)
		motor.right.target=23 - ((gris-etatg)%23*2)
else if (etatg>gris+100) then #si gauche voit du blanc, alors il vire à gauche	
		motor.left.target=23 + ((gris-etatg)%23*2)
		motor.right.target=23 - ((gris-etatg)%23*2)
	else #sinon il va tout droit
		motor.left.target=23
		motor.right.target=23
		end
	end
	if etatd<500 and amorce==0 then #dès que le détecteur droit détecte du noir alors tmp1 décrémente 1 fois et amorce est vrai
		tmp1--
		amorce=1
		if  mode==0 then			
			L[0]=32
			callsub verif
		end
		if  mode==2 then			
			L[4]=32
			callsub verif
		end
	end
	if amorce==1 and tmp1>0 then #si l'amorce est vrai et tmp1 est  supérieur à 0 alors on continue de décrémenter tmp1
		tmp1--
	end
	if tmp1==0 and tmp2>0 then #dès que tmp1 est égal à 0, ça veut dire que l'amorce est finie, et on passe à la lecture tant que tmp 2 est supérieur à0
		if tmp2==60 then # si le robot se situe dans la première case, il lit le premier bit
			if etatd<500 then #comparaison pour déterminer si c'est 1 ou 0
				bit[0]=1
			else 
				bit[0]=0
			end
			if  mode==0 then
				if  bit[0]==1 then					
					L[1]=32
					callsub verif
				else
					L[1]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[0]==1 then					
					L[5]=32
					callsub verif
				else
					L[5]=0
					callsub verif
				end
			end		
		end
		if tmp2==37 then # si le robot se situe dans la deuxième case, il lit le deuxième bit
			if etatd<500 then
				bit[1]=1
			else 
				bit[1]=0
			end
			if  mode==0 then
				if  bit[1]==1 then					
					L[2]=32
					callsub verif
				else
					L[2]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[1]==1 then					
					L[6]=32
					callsub verif
				else
					L[6]=0
					callsub verif
				end
			end
		end
		if tmp2==12 then # si le robot se situe dans la troisième case, il lit le troisième bit
			if etatd<500 then
				bit[2]=1
			else 
				bit[2]=0
			end
			if  mode==0 then
				if  bit[2]==1 then					
					L[3]=32
					callsub verif
				else
					L[3]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[2]==1 then					
					L[7]=32
					callsub verif
				else
					L[7]=0
					callsub verif
				end
			end
		end
		tmp2-- #décrémentation de tmp2
		if  tmp2==0 then #dès que tmp2 est égal à 0 on affiche la couleur correspondant au bit lu
			amorce=0 # on remet amorce à 0 pour qu'on puisse relire un autre code barre en chemin
			tmp1=incamorce #on remet tmp1 à la valeur incamorce
			if bit[0]==0 then
				if bit[1]==0 then
					if bit[2]==0 then
						call leds.top(16,16,16) #si 000 alors blanc
					else
						call leds.top(32,0,0)#si 001 alors rouge
					end
				else 
					if bit[2]==0 then
						call leds.top(0,32,0)#si 010 alors vert
					else
						call leds.top(32,32,0)#si 011 alors jaune
					end
				end
			else 
				if bit[1]==0 then
					if bit[2]==0 then
						call leds.top(0,0,32)#si 100 alors bleu
					else
						call leds.top(32,0,32)#si 101 alors magenta
					end
				else 
					if bit[2]==0 then
						call leds.top(0,32,32)#si 110 alors cyan
					else
						call leds.top(0,32,16)#si 111 alors vert bleuté
					end
				end
			end
			mode++
			tmp2=inclect#tmp2 est remis à la valeur de inclect
		end
	end
	if  mode==1 then
		dist=bit[0]*10+bit[1]*20+bit[2]*40
		if  colour==10 then
			timefordist=0
		else
			timefordist=dist*10/(colour-10)
		end
		mode++
	end
	if  mode==3 then
		if  bit[0]==0 then #les ifs qui permettent de savoir dans quel rotamode on se situe
			if bit[1]==0 then
				if  bit[2]==0 then
					rotamode=0
				else
					rotamode=4
				end
			else
				if  bit[2]==0 then
					rotamode=2
				else
					rotamode=6
				end
			end
		else
			if bit[1]==0 then
				if  bit[2]==0 then
					rotamode=1
				else
					rotamode=5
				end
			else
				if  bit[2]==0 then
					rotamode=3
				else
					rotamode=7
				end
			end
		end
		if  rotamode==0 or rotamode==7 then #si rotamode est égal à 0 ou 7 alors le robot ne fait rien 
			timeforrota=0 #pas de temps pour tourner donc
		else
			if  colour==10 then #si la vitesse séléctionné au début est 0 alors il reste sur place
				timeforrota=0
			else
				if  rotamode==1 or rotamode==2 then #si rotamode est égal à 1 ou 2 alors il tourne 45°
					timeforrota=190/(colour-10)/4#formule pour le temps de faire 45°
				end
				if rotamode==3 or rotamode==4 then #si rotamode est égal à 3 ou 4 alors il tourne 90°
					timeforrota=190/(colour-10)/2
				end
				if rotamode==5 or rotamode==6 then #si rotamode est égal à 5 ou 6 alors il tourne 120°
					timeforrota=190/(colour-10)/2 + 190/(colour-10)/6
				end
			end
		end
		mode=0
		switch++
	end
end
```
### État 2.1 : Suivre la ligne

`switch=1` ne se passe que quand on appuye sur le bouton central (donc tant qu'on a pas appuyé sur le bouton central, le code ne s'éxécute pas),
La partie qui s'occupe de suivre la ligne est celle-ci :
```
if (etatg<gris-100) then #si gauche voit du noir, alors il vire à droite
		motor.left.target=23 + ((gris-etatg)%23*2)
		motor.right.target=23 - ((gris-etatg)%23*2)
else if (etatg>gris+100) then #si gauche voit du blanc, alors il vire à gauche	
		motor.left.target=23 + ((gris-etatg)%23*2)
		motor.right.target=23 - ((gris-etatg)%23*2)
	else #sinon il va tout droit
		motor.left.target=23
		motor.right.target=23
		end
	end
```
23 u correspond à 1cm/s dans le simulateur; `gris` correspond à une valeur comparative afin que le robot puisse la suivre dans ses capteurs (capteurs qui détectent la "luminosité" sous eux avec des valeurs allant  de 0 à 999).
`etatg` correspond à la valeur du capteur gauche qui se met à jour tout les 0,1 secondes (et pareil pour `etatd` qui correspond à la droite)
On peut voir ça dans ce code :

```
timer.period[1]=100 #100ms =0,1 secondes
onevent timer1
etatg=prox.ground.delta[0] #lecture gauche
etatd=prox.ground.delta[1] #lecture droite
```

Dans notre code, afin que le robot puisse bien lire les codes barres avec son capteur droit et suivre la ligne avec son capteur gauche, nous avons mis `gris=700`.

### État 2.2: Lecture de code barre

Maintenant pour ce qui est de la lecture de code barre...

C'est la partie compliqué...

On a plusieurs variables à tenir compte
`inclect` et `incamorce` correspondent au temps en 0,1 secondes qu'il faut pour traverser l'amorce et le code barre...

`tmp1` lit `incamorce` et comme son nom l'indique, c'est une valeur temporaire car en fait il va servir d'horloge pour quand `etatd` déétecte du noir, il puisse décrémenter pour dire "on a passé l'amorce", et une fois qu'on a lu le premier code barre, tmp1 va reprendre la valeur d'incamorce pour pouvoir lire le deuxième code barre...

idem pour `tmp2` qui lit `inclect`.

`amorce` indique une sorte de booléen pour dire que "oui... on est sur un code barre".

`bit[]` est un tableau de taille 3 qui prend les données des codes barres.

Maintenant le code pour lire les codes barres...
```
if etatd<500 and amorce==0 then #dès que le détecteur droit détecte du noir alors tmp1 décrémente 1 fois et amorce est vrai
		tmp1--
		amorce=1
		if  mode==0 then			
			L[0]=32
			callsub verif
		end
		if  mode==2 then			
			L[4]=32
			callsub verif
		end
	end
	if amorce==1 and tmp1>0 then #si l'amorce est vrai et tmp1 est  supérieur à 0 alors on continue de décrémenter tmp1
		tmp1--
	end
	if tmp1==0 and tmp2>0 then #dès que tmp1 est égal à 0, ça veut dire que l'amorce est finie, et on passe à la lecture tant que tmp 2 est supérieur à0
		if tmp2==60 then # si le robot se situe dans la première case, il lit le premier bit
			if etatd<500 then #comparaison pour déterminer si c'est 1 ou 0
				bit[0]=1
			else 
				bit[0]=0
			end
			if  mode==0 then
				if  bit[0]==1 then					
					L[1]=32
					callsub verif
				else
					L[1]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[0]==1 then					
					L[5]=32
					callsub verif
				else
					L[5]=0
					callsub verif
				end
			end		
		end
		if tmp2==37 then # si le robot se situe dans la deuxième case, il lit le deuxième bit
			if etatd<500 then
				bit[1]=1
			else 
				bit[1]=0
			end
			if  mode==0 then
				if  bit[1]==1 then					
					L[2]=32
					callsub verif
				else
					L[2]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[1]==1 then					
					L[6]=32
					callsub verif
				else
					L[6]=0
					callsub verif
				end
			end
		end
		if tmp2==12 then # si le robot se situe dans la troisième case, il lit le troisième bit
			if etatd<500 then
				bit[2]=1
			else 
				bit[2]=0
			end
			if  mode==0 then
				if  bit[2]==1 then					
					L[3]=32
					callsub verif
				else
					L[3]=0
					callsub verif
				end
			end
			if  mode==2 then
				if  bit[2]==1 then					
					L[7]=32
					callsub verif
				else
					L[7]=0
					callsub verif
				end
			end
		end
		tmp2-- #décrémentation de tmp2
```
On va y revenir à la variable `mode` plus tard...

Le tableau `L` de taille 8 prend en données les couleurs à montrer sur le cercle.