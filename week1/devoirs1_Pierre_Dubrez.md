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

```{code-cell} ipython3
import numpy as np
from matplotlib import pyplot as plt
```

\
**Les rayures**

```{code-cell} ipython3
def zebre(n):
    return np.indices((n,n))[1]%2

zebre(5)
```

\
**Le damier**

```{code-cell} ipython3
def checkers(n,boo):
    lignes,colonnes = np.indices((n,n)) 
    return (lignes + colonnes + boo)%2

print(checkers(5,True))
print(checkers(5,False))
    
```

\
**Le super damier par blocs**

```{code-cell} ipython3
def block_checkers(n, k):
    lignes,colonnes = np.indices((n*k,n*k)) 
    return (lignes//k + colonnes//k) % 2

block_checkers(4, 3)
```

\
**L'escalier**

```{code-cell} ipython3
def stairs(n):
    temp = np.array([i for i in range(n+1)]+[i for i in range(n)][::-1])
    return temp + np.reshape(temp,(2*n+1,1))

stairs(5)
```
