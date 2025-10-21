---
jupytext:
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

# collages

+++

pour réaliser ce TP localement sur votre ordi, {download}`commencez par télécharger le zip<./ARTEFACTS-collages.zip>`

```{code-cell} ipython3
import pandas as pd
```

## simple critère

+++

on a trois fichiers à recoller

```{code-cell} ipython3
:cell_style: split

df1 = pd.read_csv('data/collages1.csv')
df1
```

```{code-cell} ipython3
:cell_style: split

df2 = pd.read_csv('data/collages2.csv')
df2
```

```{code-cell} ipython3
df3 = pd.read_csv('data/collages3.csv')
df3
```

comment vous feriez pour recoller les morceaux ? il s'agit d'obtenir une dataframe de 5 élèves et 4 caractéristiques

+++

on peut envisager deux versions de l'exercice, selon qu'on choisit ou non d'indexer selon le prénom

+++

### sans index

```{code-cell} ipython3
# à vous
df4 = pd.merge(df1,df2)  # on = 'name'
df5 = pd.concat([df4,df3],ignore_index = True)
df5
```

### avec index

+++

dans un premier temps, pour chacune des trois tables, adoptez la colonne `name` comme index;

puis recollez les morceaux comme dans le premier exercice

```{code-cell} ipython3
# à vous
df1.set_index('name',inplace = True)
df2.set_index('name',inplace = True)
df3.set_index('name',inplace = True)

df4 = pd.merge(df1,df2, on = 'name')
df5 = pd.concat([df4,df3])
df5
```

## multiples critères

même idée, mais on n'a plus unicité des prénoms

```{code-cell} ipython3
m1 = pd.read_csv("data/multi1.csv")
m1
```

```{code-cell} ipython3
m2 = pd.read_csv("data/multi2.csv")
m2
```

```{code-cell} ipython3
m3 = pd.read_csv("data/multi3.csv")
m3
```

```{code-cell} ipython3
# à vous - c'est vous qui décidez comment gérer les index
# juste, à la fin on voudrait un index "raisonnable"
m12 = pd.merge(m1,m2,left_on = ['nom','prenom'], right_on = ['last_name','first_name'],how = 'right')
print(m12)

m123 = pd.concat([m12,m3])
m123
```
