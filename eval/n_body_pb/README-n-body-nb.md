---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  name: python3
  display_name: Python 3 (ipykernel)
  language: python
---

# n-body problem

+++

pour faire cette activité sur votre ordi localement, {download}`commencez par télécharger le zip<ARTEFACTS-n-body.zip>`

dans ce TP on vous invite à écrire un simulateur de la trajectoire de n corps qui interagissent entre eux au travers de leurs masses, pour produire des sorties de ce genre

```{image} media/init3-1.png
:align: center
:width: 600px
```

+++

on suppose:

- on se place dans un monde en 2 dimensions
- on fixe au départ le nombre de corps N
- chacun est décrit par une masse constante
- et au début du monde chacun possède une position et une vitesse

```{admonition} la 3D
en option on vous proposera, une fois votre code fonctionnel en 2D, de passer à la 3D  
ça peut valoir le coup d'anticiper ça dès le premier jet, si vous vous sentez de le faire comme ça
```

+++

## imports

on pourra utiliser le mode `ipympl` de `matplotlib`

```{code-cell} ipython3
import numpy as np
import matplotlib.pyplot as plt
import random as rd

#%matplotlib ipympl
```

## initialisation aléatoire

en fixant arbitrairement des limites dans l'espace des positions, des vitesses et des masses, la fonction `init_problem()` tire au hasard une configuration de départ pour la simulation

```{code-cell} ipython3
# les bornes pour le tirage au sort initial
mass_max = 3.
    
x_min, x_max = -10., 10.
y_min, y_max = -10., 10.

speed_max = 1.
```

```{code-cell} ipython3
:tags: [level_basic]

# votre code

def init_problem(N):
    """
    retourne un tuple masses, positions, speeds
    de formes resp.   (N,)    (2, N)     (2, N)
    tiré au sort dans les bornes définies ci-dessus
    """
    return np.array([rd.uniform(0,mass_max) for _ in range(N)]), np.array([[rd.uniform(x_min,x_max) for _ in range(N)], [rd.uniform(y_min,y_max) for _ in range(N)]]) , np.array([[rd.uniform(-speed_max,speed_max) for _ in range(N)], [rd.uniform(-speed_max,speed_max) for _ in range(N)]])                 
```

```{code-cell} ipython3
:tags: [level_intermediate]

# pour tester

# normalement vous devez pouvoir faire ceci

masses, positions, speeds = init_problem(10)

# et ceci devrait afficher OK
try:
    masses.shape == (10,) and positions.shape == speeds.shape == (2, 10)
    print("OK")
except:
    print("KO")
```

## initialisation reproductible

par commodité on vous donne la fonction suivante qui crée 3 objets:

- le premier - pensez au soleil - de masse 3, an centre de la figure, de vitesse nulle
- et deux objets de masse 1, disposés symétriquement autour du soleil  
  - position initiale (5, 1) et vitesse initiale (-1, 0)
  - symétrique en     (-5, -1) et vitesse initiale (1, 0)

```{code-cell} ipython3
# for your convenience

def init3():
    # first element is sun-like: heavy, at the center, and no speed
    masses = np.array([3, 1, 1], dtype=float)
    positions = np.array([
        [0, 5, -5], 
        [0, 1, -1]], dtype=float)
    speeds = np.array([
        [0, -1, 1], 
        [0, 0, 0]], dtype=float)
    return masses, positions, speeds
```

## les forces

à présent, on va écrire une fonction qui va calculer les influences de toutes les particules entre elles, suivant la loi de Newton


$$
\vec{F}_i = \sum_{\substack{j=1 \\ j \neq i}}^N 
   G \, m_i m_j \, \frac{\vec{r}_j - \vec{r}_i}{\lvert \vec{r}_j - \vec{r}_i \rvert^3}
$$

pour cela on se propose d'écrire la fonction suivante

```{code-cell} ipython3
:tags: [level_basic]

# votre code
def forces(masses, positions, G=1.0):
    N = len(masses)
    """
    returns an array of shape (2,  N)
    that contains the force felt by each mass from all the others
    G is the Newton constant and by default is set to 1 
    (we have abstract units anyway)
    """
    return np.array([G*masses[k]*sum(masses[j]*(positions[:,j]-positions[:,k])/((positions[0,j]-positions[0,k])**2+(positions[1,j]-positions[1,k])**2)**(3/2) for j in range(N) if (j != k)) for k in range(N)]).transpose()
```

```{code-cell} ipython3
:tags: [level_intermediate]

# pour tester, voici les valeurs attendues avec la config prédéfinie

masses, positions, speeds = init3()

f = forces(masses, positions)
print(f)
# should be true
np.all(np.isclose(f, np.array([[ 0.        , -0.12257258,  0.12257258],[ 0.        , -0.02451452,  0.02451452]])))
```

## le simulateur

à présent il nous reste à utiliser cette brique de base pour "faire avancer" le modèle depuis son état initial et sur un nombre fixe d'itérations

cela pourrait se passer dans une fonction qui ressemblerait à ceci

```{code-cell} ipython3
:tags: [level_basic]

# votre code

def simulate(masses, positions, speeds, dt=0.1, nb_steps=100):
    """
    should return the positions across time
    so an array of shape (nb_steps, 2, N)
    optional dt is the time step
    """
    res = np.array([positions for _ in range(nb_steps)])
    speeds2 = np.copy(speeds)
    for t in range(1,nb_steps):
        f = forces(masses, res[t-1], 1.0)
        speeds2 += f/masses * dt
        res[t] = res[t-1] + speeds2*dt
    return res
```

```{code-cell} ipython3
:tags: [level_intermediate]

# pour tester

SMALL_STEPS = 4

s = simulate(masses, positions, speeds, nb_steps=SMALL_STEPS)

try:
    if s.shape == (SMALL_STEPS, 2, 3):
        print("shape OK")
except Exception as exc:
    print(f"OOPS {type(exc)} {exc}")
```

```{code-cell} ipython3
:tags: [level_intermediate]

# pour tester: should be true

# first step
positions1 = s[1]
print(s)
np.all(np.isclose(positions1, np.array([
     [ 0.        ,  4.89877427, -4.89877427],
     [ 0.        ,  0.99975485, -0.99975485]
 ])))
```

## dessiner

ne reste plus qu'à dessiner; quelques indices potentiels:

- 1. chaque corps a une couleur; l'appelant peut vous passer un jeu de couleurs, sinon en tirer un au hasard
- 2.a pour l'épaisseur de chaque point, on peut imaginer utiliser la masse de l'objet  
  2.b ou peut-être aussi, à tester, la vitesse de l'objet (plus c'est lent et plus on l'affiche en gros ?)

```{admonition} masses et vitesses ?
j'ai choisi de repasser à `draw()` le tableau des masses à cause de 2.a;  
si j'avais voulu implémenter 2.b il faudrait tripoter un peu plus nos interfaces - car en l'état on n'a pas accès aux vitesses pendant la simulation - mais n'hésitez pas à le faire si nécessaire..
```

```{code-cell} ipython3
:tags: [level_basic]

# votre code

def draw(simulation, masses, colors=None, scale=10.):
    N = len(masses)
    """
    takes as input the result of simulate() above,
    and draws the nb_steps positions of each of the N bodies
    ideally it should return a matplotlib Axes object

    one can provide a collection of N colors to use for each body
    if not provided this is randomized

    also the optional scale parameter is used as a constant
    multiplier to obtain the final size of each dot on the figure
    """
    for k in range(N):
        plt.plot(simulation[:,:,k].transpose()[0],simulation[:,:,k].transpose()[1],'.--', markersize = masses[k]*scale, color = colors[k])
    plt.show()
```

## un jeu de couleurs

```{code-cell} ipython3
# for convenience

colors3 = np.array([
    [32, 32, 32],
    (228, 90, 146),
    (111, 0, 255),
]) / 255
```

## on assemble le tout

pour commencer et tester, on se met dans l'état initial reproductible

```{code-cell} ipython3
:tags: [level_intermediate]

# décommentez ceci pour tester votre code

masses, positions, speeds = init3()
draw(simulate(masses, positions, speeds), masses, colors3)

# avec une initialisation aléatoire et un peu plus de planètes : 
#masses, positions, speeds = init_problem(5)
#draw(simulate(masses, positions, speeds), masses, ['b','g','r','c','m'])
```

et avec ces données vous devriez obtenir plus ou moins une sortie de ce genre  
mais [voyez aussi la discussion ci-dessous sur les diverses stratégies possibles](label-n-body-strategies)
```{image} media/init3-1.png
```

+++

`````{grid} 2 2 2 2 
````{card}
après vous avez le droit de vous enhardir avec des scénarii plus compliqués
par exemple avec ce code

```python
m5, p5, s5 = init_problem(5)
sim5 = simulate(m5, p5, s5, nb_steps=1000)
draw(sim5, m5, scale=3);
plt.savefig("random5.png")
```
````
````{card}
j'ai pu obtenir ceci
```{image} media/random5.png
```
````
`````

+++

***
***
***

+++

## partie optionnelle

+++

### option 1: la 3D

modifiez votre code pour passer à une simulation en 3D

```{code-cell} ipython3
####################################################################
# simulation du problème à n corps 3D + changement de point de vue #
####################################################################


%matplotlib ipympl 
# je n'active ipympl que maintenant car avant ça avait tendance à superposer les nouveaux graphiques sur les anciens
# ipympl va permettre d'ouvrir le graphique dans une fenêtre à part pour le rendre interactif (car sinon le rendu c'est juste une image png du graphique, qui est donc figée)



x_min, x_max = -10., 10.
y_min, y_max = -10., 10.
z_min, z_max = -10., 10.
    
def init_problem_3d(N):
    return np.array([rd.uniform(0,mass_max) for _ in range(N)]), np.array([[rd.uniform(x_min,x_max) for _ in range(N)], [rd.uniform(y_min,y_max) for _ in range(N)],[rd.uniform(z_min,z_max) for _ in range(N)]]) , np.array([[rd.uniform(-speed_max,speed_max) for _ in range(N)], [rd.uniform(-speed_max,speed_max) for _ in range(N)],[rd.uniform(-speed_max,speed_max) for _ in range(N)]])                 

# la fonction forces est toujours valable puisqu'on avait fait le calcul sur les vecteurs entiers (peu importe la dimension) et non pas composante par composante
def forces(masses, positions, G=1.0):
    N = len(masses)
    return np.array([G*masses[k]*sum(masses[j]*(positions[:,j]-positions[:,k])/((positions[0,j]-positions[0,k])**2+(positions[1,j]-positions[1,k])**2)**(3/2) for j in range(N) if (j != k)) for k in range(N)]).transpose()

# de même, la fonction simulate est toujours valable
def simulate(masses, positions, speeds, dt=0.1, nb_steps=1000):
    res = np.array([positions for _ in range(nb_steps)])
    speeds2 = np.copy(speeds)
    for t in range(1,nb_steps):
        f = forces(masses, res[t-1], 1.0)
        speeds2 += f/masses * dt
        res[t] = res[t-1] + speeds2*dt
    return res



# en revanche il faut modifier un peu draw pour inclure la 3e dimension et rendre le tout un peu interactif (possibilité de tourner autour du graphique)
def draw_3d(simulation, masses, colors=None, scale=10.):
    N = len(masses)
    fig = plt.figure("Pb à n corps 3D")
    axes = fig.add_subplot(projection="3d") # pour dire que l'on veut bien 3 axes et non plus 2
    for k in range(N):
        plt.plot(simulation[:,:,k].transpose()[0],simulation[:,:,k].transpose()[1],simulation[:,:,k].transpose()[2],'.', markersize = masses[k]*scale,color = colors[k])
    plt.show()


# enfin on teste : 

masses, positions, speeds = init_problem_3d(3)
draw_3d(simulate(masses, positions, speeds), masses, ['b','g','r','c','m','y','k'][:3])

#masses, positions, speeds = init_problem_3d(5)
#draw_3d(simulate(masses, positions, speeds), masses, ['b','g','c','m','y','k','r'][:5])
```

**On peut par exemple obtenir quelque chose comme ça :**
![Pb à n corps 3D.png](attachment:8dd93656-3e6f-4532-9755-351cc51413e8.png)

+++

### option 2: un rendu plus interactif

le rendu sous forme de multiples scatter plots donne une idée du résultat mais c'est très améliorable  
voyez un peu si vous arrivez à produire un outil un peu plus convivial pour explorer les résultats de manière interactive; avec genre

- une animation qui affiche les points au fur et à mesure du temps
- qu'on peut controler un peu comme une vidéo avec pause / backward / forward
- l'option de laisser la trace du passé
- et si vous avez un code 3d, la possibilité de changer le point de vue de la caméra sur le monde
- etc etc...

voici une possibilité avec matplotlib; mais cela dit ne vous sentez pas obligé de rester dans Jupyter Lab ou matplotlib, il y a plein de technos rigolotes qui savent se décliner sur le web, vous avez l'embarras du choix...

```{code-cell} ipython3
:tags: [prune-remove-input, remove-input]

# prune-remove-input

# credit: Damien Corral
# with good old matplotlib FuncAnimation

from matplotlib.animation import FuncAnimation
from IPython.display import HTML

def animate(simulation, masses, colors=None, scale=5., interval=50):
    nb_steps, _, N = simulation.shape
    colors = (colors if colors is not None
              else np.random.uniform(0.3, 1., size=(N, 3)))

    fig, ax = plt.subplots()
    ax.set_title(f"we have {N} bodies over {nb_steps} steps")

    ax.set_xlim(simulation[:, 0].min() - 1, simulation[:, 0].max() + 1)
    ax.set_ylim(simulation[:, 1].min() - 1, simulation[:, 1].max() + 1)

    scat = ax.scatter(np.zeros(N), np.zeros(N), c=colors, s=(masses*scale)**2)

    def init():
        scat.set_offsets(np.zeros((nb_steps, N)))
        return scat

    def update(step):
        x, y = simulation[step]
        scat.set_offsets(np.c_[x, y])
        return scat

    animation = FuncAnimation(
        fig, update, frames=nb_steps,
        init_func=init, blit=True, interval=interval
    )
    plt.close()
    return animation

def animate_from_file(filename):
    simulation = np.loadtxt(filename).reshape((100, 2, 3))
    animation = animate(simulation, masses, colors=colors3)
    return HTML(animation.to_jshtml())

animate_from_file("data/init3-simu-1.txt")
```

## Animations 2D
Pour les deux cellules suivantes, exécuter la cellule lance une animation.
(Attention, pour que les animations marchent bien il vaut mieux les lancer séparément, pas en même temps)

```{code-cell} ipython3
# On anime le graphe pour voir en direct l'évolution des planètes
# Commençons par 3 corps :

import matplotlib.animation as ani 


masses, positions, speeds = init3()
simulation = simulate(masses, positions, speeds)




fig, ax = plt.subplots(ncols=1, nrows=1) 
ax.set_xlim(-5,5)
ax.set_ylim(-1.5,1.5)
line = []
for k in range(3):
    line_k, = ax.plot(positions[0,k], positions[1,k], ".", color = colors3[k])
    line.append(line_k)

 
def animate(t):
    for k in range(3):
        x_k = simulation[:,:,k].transpose()[0][:t+1]
        y_k = simulation[:,:,k].transpose()[1][:t+1]
        line[k].set_data(x_k, y_k)
    return line

anim = ani.FuncAnimation(fig = fig, func = animate, frames = 99, interval = 50, blit = True, repeat = False) # pour plus de clarté j'ai choisi que l'animation ne se jouait qu'une seule fois.
                                                                                                            # Cependant, on peut aussi la faire jouer en boucle en mettant repeat = True

plt.show()
```

```{code-cell} ipython3
# tentons maintenant la généralisation à N corps :

colorsN = np.array([
    (179,226,221),
    (72,181,163),
    (148,168,208),
    (145,210,144),
    (191,228,118),
    (111,183,214),
    (251,182,209)
]) / 255


N = 7  # on initialise N comme variable globale pour simplifier

masses, positions, speeds = init_problem(N)
simulation = simulate(masses, positions, speeds, nb_steps = 1000, dt = 0.5)




fig, ax = plt.subplots(ncols=1, nrows=1) 
ax.set_xlim(-30,30)
ax.set_ylim(-30,30)
line = []
for k in range(N):
    line_k, = ax.plot(positions[0,k], positions[1,k], ".", color = colorsN[k])
    line.append(line_k)

 
def animate(t):
    for k in range(N):
        x_k = simulation[:,:,k].transpose()[0][:t+1]
        y_k = simulation[:,:,k].transpose()[1][:t+1]
        line[k].set_data(x_k, y_k)
    return line

anim = ani.FuncAnimation(fig = fig, func = animate, frames = 99, interval = 50, blit = True, repeat = False)

plt.show()
```

**On peut par exemple obtenir des motifs intéressants comme celui-ci, et on voit vraiment le déplacement des planètes grâce à l'animation.**
![image.png](attachment:f9d2edf5-7538-4dc0-826f-81cb7ada6132.png)

+++

(label-n-body-strategies)=
## plusieurs stratégies

Pour les curieux, vous avez sans doute observé qu'il y a plusieurs façons possibles d'écrire la fonction `simulate()`

dans ce qui suit, on note l'accélération $a$, la vitesse $s$ et la position $p$

````{admonition} Approche 1
:class: note
dans l'implémentation qui a servi à calculer l'illustration ci-dessus, on a écrit principalement ceci:
- on calcule l'accélération
- ce qui permet d'extrapoler les vitesses  
  $s = s + a.dt$
- et ensuite d'extrapoler les positions  
  $p = p + s.dt$
```{admonition} 1bis
:class: tip
une variante consiste à intervertir les deux, en arguant du fait que la vitesse à l'instant $t$ agit sur la position à l'instant $t$
- on extrapole d'abord les positions
- puis seulement on calcule l'accélération
- enfin on extrapole les vitesses
```
````
````{admonition} Approche 2
:class: tip
dans cette approche plus fine, on utiliserait deux versions de l'accélération (l'instant présent et l'instant suivant), et un dévelopmment du second ordre, ce qui conduirait à
- calculer la position comme $p = p + s.dt + \frac{a}{2}.dt^2$
- calculer les accélérations $a_+$ sur la base de cette nouvelle position
- estimer l'accélération sur l'intervalle comme la demie-somme entre les deux accélérations
  $a_m = (a+a_+)/2$
- mettre à jour les vitesses
  $s = s + a_m.dt$
- ranger $a_+$ dans $a$ pour le prochain instant
````

+++

Bref, vous voyez qu'il y a énormément de liberté sur la façon de s'y prendre  
Ce qui peut expliquer pourquoi vous n'obtenez pas la même chose que les illustrations avec pourtant les mêmes données initiales

D'autant que, c'est bien connu, ce problème des n-corps est l'exemple le plus célèbre de problème instable, et donc la moindre divergence entre deux méthodes de calcul peut entrainer de très sérieux écarts sur les résultats obtenus

+++

Voici d'ailleurs les résultats obtenus avec ces deux approches alternatives, et vous pouvez constater qu'effectivement les résultats sont tous très différents !

+++

### approche 1bis

+++

````{grid} 2 2 2 2 
```{image} media/init3-1bis.png
```
```{code-cell} python
:tags: [remove-input]
animate_from_file("data/init3-simu-1bis.txt")
```
````

+++

### approche 2

+++

````{grid} 2 2 2 2 
```{image} media/init3-2.png
```
```{code-cell} python
:tags: [remove-input]
animate_from_file("data/init3-simu-2.txt")
```
````
