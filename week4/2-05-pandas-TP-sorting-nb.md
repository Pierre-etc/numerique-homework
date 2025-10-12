---
jupytext:
  cell_metadata_json: true
  encoding: '# -*- coding: utf-8 -*-'
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# TP sur le tri d'une dataframe

+++

**Notions intervenant dans ce TP**

* affichage des données par `plot`
* tri de `pandas.DataFrame` par ligne, par colonne et par index

**N'oubliez pas d'utiliser le help en cas de problème.**

+++

## tri et affichage

+++

1. importez les librairies `numpy` et `pandas`

```{code-cell} ipython3
# votre code
import numpy as np
import pandas as pd
```

2. importez la librairie `matplotlib.pyplot` avec le nom `plt`

```{code-cell} ipython3
# votre code
import matplotlib.pyplot as plt
```

3. lors de la lecture du fichier de données `data/titanic.csv`  
   1. gardez uniquement les colonnes `cols` suivantes `'PassengerId'`, `'Survived'`, `'Pclass'`, `'Name'`, `'Sex'`, `'Age'` et `'Fare'`

   1. mettez la colonne `PassengerId` comme index des lignes
   1. besoin d'aide ? faites `pd.read_csv?`

```{code-cell} ipython3
# votre code
df = pd.read_csv("titanic.csv",usecols = ['PassengerId', 'Survived', 'Pclass', 'Name', 'Sex', 'Age', 'Fare'],index_col='PassengerId')
```

4. en utilisant la méthode `df.plot()`  
   plottez la dataframe (pas la série) réduite à la colonne des ages  
   utilisez le paramètre de `style` `'rv'` (`r` pour rouge et `v` pour le style: points triangulaires)

   vous allez voir les points *en vrac*; dans la suite on va s'efforcer de les trier, pour mieux
   voir la distribution des âges dans la population concernée

```{code-cell} ipython3
# votre code
df['Age'].plot(style = 'rv')
```

5. pour commencer on va trier - i.e. mettre les lignes de la  dataframe suivant l'ordre d'une colonne    
   en utilisant la méthode `df.sort_values()`:
   1. créez une nouvelle dataframe  dont les lignes sont triées  
      dans l'ordre croissant des `'Age'` des passagers
   2. pour constater qu'elles sont triées, affichez les 4 premières lignes de la dataframe  
      la colonne des `Age` est triée  
      les lignes ont changé de place dans la table
   3. remarquez que l'indexation a été naturellement conservée

```{code-cell} ipython3
# votre code
df_triee = df.sort_values('Age')
df_triee.head(4)
```

6. 1. plottez la colonne des ages de la dataframe triée  
      pour changer un peu on va mettre un style `'b.'`
   1. Que constatez-vous ?

```{code-cell} ipython3
# votre code
df_triee['Age'].plot(style = 'b.')

#plot a affiché les âges dans l'ordre de PassengerId donc n'a pas tenu compte du tri
```

7. 1. la logique de `df.plot()` consiste

      * à **utiliser comme abscisses** l'index de la dataframe
      * et accessoirement à faire autant de plots que de colonnes - ici on n'en a qu'une
    vous tracez donc le point $(804, 0.42)$ puis le point $(756, 0.67)$ ...  
    alors que vous voudriez tracer le point $(0, 0.42)$ puis le point $(1, 0.67)$ ...  
    c'est à dire: le fait d'utiliser le 'PassengerId' n'est pas bon, on voudrait que les abscisses soient les indices de lignes
   1. une solution: voyez la méthode `reset_index()`
      qui permet de transformer l'index en une colonne normale  
   1. utiliser cette méthode et regardez ce que vous avex dans l'index ensuite
   1. plottez le résultat  
      normalement à ce stade vous obtenez la visualisation qu'on cherche

```{code-cell} ipython3
# votre code
df_triee.reset_index(inplace = True)

df_triee['Age'].plot(style = 'b.')
```

## tri des lignes selon plusieurs critères

quand on trie, que faire en cas d'égalité ?  
en général on choisit plusieurs critères, on trie selon le premier, puis en cas d'égalité selon le second, etc..

*note*: on appelle cela un ordre lexicographique, car c'est - un peu - comme dans un dictionnaire

+++

0. rechargez la dataframe

```{code-cell} ipython3
# votre code
df = pd.read_csv('titanic.csv',usecols = ['PassengerId', 'Survived', 'Pclass', 'Name', 'Sex', 'Age', 'Fare'],index_col='PassengerId')
```

2. utilisez `df.sort_values()` pour trier la dataframe suivant la colonne (`'Pclass'`)  
   et trier les lignes identiques (passagers de même classe) suivant la colonne (`'Age'`)

```{code-cell} ipython3
# votre code
df_triee = df.sort_values(['Pclass','Age'])
df_triee.head(10)
```

3. sélectionnez, dans la nouvelle dataframe, la sous-dataframe des gens dont les ages ne sont pas définis  

*hint*: utiliser la méthode `isna` sur une série, pour créer un masque de booléens, et appliquer ce masque à la dataframe

```{code-cell} ipython3
# votre code
mask = df_triee['Age'].isna()
df_triee_age_non_def = df_triee[mask]
```

4. combien nous manque-t-il d'ages ?

```{code-cell} ipython3
# votre code
df_triee_age_non_def.shape[0]
```

5. où sont placés ces passagers dans la data-frame globale triée ?
   - [ ] en début (voir avec `head`)
   - [ ] ou en fin (voir avec `tail`)
   - [x] ou c'est plus compliqué que ça ?

````{admonition} *hint*
:class: dropdown

la façon standard d'afficher un dataframe consiste à montrer le début et la fin  
il y a des situations, comme celle-ci, où on veut avoir une *vision globale* des données,
et pour cela le bon réflexe consiste à se ramener à un tableau numpy  
pour cela voyez par exemple `df.no_numpy()`
````

```{code-cell} ipython3
# votre code
df_triee.to_numpy()
```

6. trouvez le paramètre de `sort_values()`  
qui permet de mettre ces lignes en début de dataframe lors du tri

```{code-cell} ipython3
# votre code
df_triee2 = df.sort_values(['Pclass','Age'],na_position = 'first')
df_triee2.head(220)
```

7. produire une nouvelle dataframe en ne gardant que les ages connus,
   et triée selon les ages, puis les prix de billet

```{code-cell} ipython3
df[~(df['Age'].isna())].sort_values(['Age','Fare'])
```

## tri d'une dataframe selon l'index

en utilisant `df.sort_index()` il est possible de trier une dataframe  
dans l'axe de ses index de ligne (ou même de colonnes)

+++ {"cell_style": "center"}

1. reprenez la dataframe du Titanic, en choisissant toujours comme index `PassengerId`  
   utilisez la méthode des dataframe `sort_index` pour la trier dans l'ordre des index

```{code-cell} ipython3
# votre code
df = pd.read_csv('titanic.csv',usecols = ['PassengerId', 'Survived', 'Pclass', 'Name', 'Sex', 'Age', 'Fare'],index_col='PassengerId')
df.sort_index()
```

***
