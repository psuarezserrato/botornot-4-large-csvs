# Detectando Bots
This is a simple way to time Twitter's API so that a large file of accounts can be checked with the BotOrNor API. It's a tutorial in Spanish. Please read [Clayton Davis BotOrNot API page](https://github.com/truthy/botornot-python) first!
%% 

__Tutorial en Español__

Esta es una manera de revisar cuentas de Twitter usando _BotOrNot_, para darles probabilidades en 6 categorías que evalúan la posibilidad de que las cuentas sean bots. Vamos a asumir que tenemos ya una aplicación de Twitter inicializada, con claves, secretos y pases listos.

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

Habiendo guardado un csv simple con las cuentas a evaluar, digamos ``` 'Cuentas.csv' ```, con dos columnas, el índice y ```'screen_name'```. Cargamos este archivo:

```python
file1 = 'Cuentas.csv'
```

Usando Pandas lo convertimos en un «data frame»:

```python
df = pd.read_csv(file1, low_memory = False, encoding='latin1')
```

Separamos la columna de los nombres de las cuentas:
```python
df0 = df['screen_name']
```

Para darnos una idea rápida de las cuentas que tenemos la miramos en corto:

```python
df0.describe()
```

Creamos una lista que tenga solo una mención de cada cuenta a evaluar (para no repetir):
```python
df3 = df0.unique()
```

Incluimos las llaves, secretos y pase:

```python
twitter_app_auth = {
    'consumer_key': 'XXXXXXXXXXXXXXX',
    'consumer_secret': 'XXXXXXXXXXXXXXX',
    'access_token': 'XXXXXXXXXXXXXXX',
    'access_token_secret': 'XXXXXXXXXXXXXXX',
  }

for api_key in twitter_app_auth.values(): 
    assert not api_key.startswith('COPY_')
```

Ahora inicializamos el API de _BorOrNot_:

```python
bon = botornot.BotOrNot(**twitter_app_auth)
api = bon.twitter_api
```

Ya estamos listos para el paso importante, llamar a _BorOrNot_ usando nuesta lista de cuentas:

```python
import time 

#time.time() # el tiempo ahora en segundos
#time.sleep(10) # se espera durante 10 segundos

import pickle

retry = 0

time_at_block_start = time.time()
calls_in_block = 0
results = {}    
for user_id in df3[:4999]:
    try:
        print('Getting result for ' + user_id)
        bon_result = bon.check_account(user_id)
        results[user_id] = bon_result
        print('    ...success!')
    except:
        print('    ...fail.')
        
    calls_in_block += 1
    now = time.time()
        
    if calls_in_block == 179 and now - time_at_block_start <= 15 * 60:
        time.sleep(15 * 60 - now + time_at_block_start)
        
    if now - time_at_block_start >= 15 * 60:
        time_at_block_start = time.time()
        calls_in_block = 0

print("Done.")
```

Cuando el proceso anterior haya terminado (esto puede durar mucho si tenemos muchas cuentas) guardamos los resultados (sí, es un «hack» barato):

```python
import pickle
sresults = pickle.dumps(results)    
pickle.dump(sresults, open('Cuentas.pkl', 'wb')) 
```

Con este archivo podemos pasar a la fase de visualización simple. Abrimos el archivo que creamos con los puntajes:

```python
import pickle

sresults = pickle.loads(pickle.load(open('Cuentas.pkl', 'rb')))
```

Preparamos una nueva «data frame» que vamos a usar para analizar los puntajes de _BotOrNot_:

```python
users = []; content = []; friend = []; network = []; sentiment = []; temporal = []; userc = []
for user, user_dict in sresults.items() :
    users.append(user)
    content.append(user_dict['categories']['content_classification'])
    friend.append(user_dict['categories']['friend_classification'])
    network.append(user_dict['categories']['network_classification'])
    sentiment.append(user_dict['categories']['sentiment_classification'])
    temporal.append(user_dict['categories']['temporal_classification'])
    userc.append(user_dict['categories']['user_classification'])
```

Ahora creamos esta nueva «data frame»:

```python
import pandas
df = pandas.DataFrame({'user': users, 'content_classification': content, 'friend_classification': friend, 'network_classification': network,
  'sentiment_classification': sentiment,
  'temporal_classification': temporal,
  'user_classification': userc})
```

En este momento ya somos capaces de empezar a visualizar los puntajes obtenidos, por ejemplo en la categoría de «Network», usando un estimado de la densidad de núcleo, conocido como «kernel density estimate»:

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

scores = df[df['network_classification'].notnull()]['network_classification']
sns.set(rc={"figure.figsize": (12, 6)})
ax = sns.distplot(scores)
ax.set_title('Distribución de puntajes de Red, «Network», obtenidos con BotOrNot en Cuentas.csv')
ax.set_xlim([0,1])
ax.set_xlabel('Content classification')
ax.yaxis.set_visible(True)
ax.patch.set_visible(True)

filename = 'Network-Cuentas'
plt.savefig(filename)
```
Este es un ejemplo de lo que se obtiene, en este caso con tuits que usaron la etiqueta #Tanhuato:

![Ejemplo de distribución de puntajes de Red](https://github.com/psuarezserrato/botornot-4-large-csvs/blob/master/Network-dist-Tanhuato_19-20_8_16.png)

Vemos una señal prominente en .8, lo que indica que estas cuentas son muy probablemente bots y que ademas tienen las mismas características, lo cuál sucedería si todas fueron automatizadas (programadas) con los mismos parámetros de comportamiento.

Distintas cuentas se comportan de distintas maneras, por lo que es conveniente comparar varios puntajes a la vez. Esto es sencillo en 2D. Por ejemplo, comparemos puntajes de Red y Amigos:

```python
sns.set(style="white")

x1 = df[df['friend_classification'].notnull()]['friend_classification']
x2 = df[df['network_classification'].notnull()]['network_classification']
x1 = pd.Series(x1, name="Friend class")
x2 = pd.Series(x2, name="Net class")

g = sns.jointplot(x1, x2, kind="kde", n_levels=60, shade=True, size=12, space=0)

filename = 'KDE-Friend-Temporal-Cuentas'
plt.savefig(filename)
```
Como antes, va un ejemplo de #Tanhuato :

![Ejemplo de KDE en 2D combinando Red y Amigos](https://github.com/psuarezserrato/botornot-4-large-csvs/blob/master/KDE-Friend-Net-Tanhuato_19-20_8_16.png)

Se aprecia un cúmulo de cuentas con una distribución muy uniforme en la esquina superior derecha, la zona de los bots.

De esta manera podemos identificar cúmulos de cuentas que son potencialmente bots dejando que los datos guíen el análisis. 

Estos métodos fueron desarrollados para [éste artículo](https://link.springer.com/chapter/10.1007/978-3-319-47874-6_19), disponible con [libre acceso aquí](https://arxiv.org/abs/1609.08239).

__Suárez-Serrato, Pablo, Margaret E. Roberts, Clayton Davis, and Filippo Menczer. "On the influence of social bots in online protests." In International Conference on Social Informatics, pp. 269-278. Springer International Publishing, 2016.__

Agradezco que [se cite](http://dblp.uni-trier.de/rec/bibtex/journals/corr/Suarez-SerratoR16) este trabajo si se usan estos métodos o códigos.

También agradezco apoyos de CONACyT, PAPIIT-UNAM y al IPAM en UCLA.

Este tutorial está siendo distribuido bajo una licencia Creative Commons Attribution-NonCommercial-ShareAlike 
CC BY-NC-SA. ;)

![CC license](https://github.com/psuarezserrato/botornot-4-large-csvs/blob/master/CC-bon.png)
