# botornot-4-large-csvs
This is a simple way to time Twitter's API so that a large file of accounts can be checked with the BotOrNor API. 

__Tutorial en Español__

Esta es una manera de revisar cuentas de Twitter usando BotOrNot, para darles probabilidades en 6 categorías que evalúan la posibilidad de que las cuentas sean bots. 

En una libreta de Jupyter vamos primero a cargar los módulos que vamos a usar para evaluar las cuentas:

```python

from __future__ import print_function, unicode_literals

import datetime
import json
import re
import time
import traceback
import botornot
import pandas as pd
import requests
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
```

Habiendo guardado un csv simple con las cuentas a evaluar, digamos ```python 'Cuentas.csv' ```, con dos columnas, el índice y ```python screen_name```. Cargamos este archivo:

```python
file1 = 'Cuentas.csv'
```

Usando Pandas lo convertimos en un «data frame»:

```python
df = pd.read_csv(file1, low_memory = False, encoding='latin1')
```

Separamos la columna de los nombres de kas cuentas:
```python
df0 = df['screen_name']
```

Para darnos una idea rápida de las cuentas que tenemos la miramos en corto:
```python
df0.describe()
```

Creamos una lista que tenga solo una menci
```python
df3 = df0.unique()
```



```python

```








