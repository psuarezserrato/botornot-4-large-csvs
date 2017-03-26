# botornot-4-large-csvs
This is a simple way to time Twitter's API so that a large file of accounts can be checked with the BotOrNor API. 

__Tutorial en Español__

Esta es una manera de revisar cuentas de Twitter usando BotOrNot, para darles probabilidades en 6 categorías que evalúan la posibilidad de que las cuentas sean bots. 

En una libreta de Jupyter vamos primero a cargar los módulos que vamos a usar para evaluar las cuentas.

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
