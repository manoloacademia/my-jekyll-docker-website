---
layout: post
title: "A challenge from Alkemy: Data Engineering first approach"
categories: personal, data_engineering
---

## How to get into the Data Engineering world?

This challenge was my first approach to the Data Engineering side.
The idea was to develop a project that extracts data from three sources and populate a SQL database.

The sources are:
- [Argentinean Museums Data](https://datos.cultura.gob.ar/dataset/espacios-culturales-argentina-sinca/archivo/4207def0-2ff7-41d5-9095-d42ae8207a5d)
- [Argentinean Movie Theaters Data](https://datos.cultura.gob.ar/dataset/espacios-culturales-argentina-sinca/archivo/f7a8edb8-9208-41b0-8f19-d72811dcea97)
- [Argentinean Public Libraries Data](https://datos.cultura.gob.ar/dataset/espacios-culturales-argentina-sinca/archivo/01c6c048-dbeb-44e0-8efa-6944f73715d7)

Using [Requests](https://pypi.org/project/requests/) library, you can get the data to save locally.

The files must be saved under the file structure: "category/year-month/category-day-month-year.csv".
If the file is already created, it must be replaced.

The creation of the tables in the database must use [PostgreSQL](https://www.postgresql.org/) kind.
All the creation scripts must be saved under the `.sql` extension.

## `Main.py` file

The main.py performs the following actions:
- Download of the data from the 3 sources in .csv files.
- Generates specific directories for each data-source file.
- It transforms the source data into general tables with normalized columns.
- Generates normalized general table with the data from the 3 sources.
- Send these tables (pandas Data Frames) to the corresponding files.
- Generates the engine and the connection to the PostgreSQL database.
- Generates extra analysis tables from data-source files.
- Feed the database.
- Generates log file and error handling.

## Download the data from the sources

Using sqlalchemy and sqlalchemy_utils, we can write a function to get the engine:
``` python
from sqlalchemy import create_engine
from sqlalchemy_utils import database_exists, create_database

def get_engine(user, passwd, host, port, db):
    url = f'postgresql://{user}:{passwd}@{host}:{port}/{db}'
    if not database_exists(url):
        create_database(url)
    engine = create_engine(url, pool_size=50, echo=False)
    return engine
```

With this engine, we can connect with the postgresql database using [Python Decouple](https://pypi.org/project/python-decouple/) for security matters (saving all the connection data in a `.env` file that can be ignored).
Also, generate the function to feed the database:
``` python
from sqlalchemy_conn import get_engine
import pandas as pd
import logging
import datetime
from decouple import config

user = config('DB_USER')
passwd = config('DB_PASSWORD')
host = config('DB_HOST')
port = config('DB_PORT')
db = config('DB_NAME')

# Seteo de logging
logging.basicConfig(level=logging.DEBUG, filename='datos_generados.log', filemode='a', format='%(asctime)s:%(levelname)s:%(message)s')

# Importar los módulos de los archivos correspondientes
from gen_csv_files import gen_tabla_normalizada
from otras_tablas import analisis_datos_cines, tabla_categoria, tabla_fuente, tabla_prov_cat

# Generar la constante del día en el que realizamos la descarga
FECHA_DESCARGA = datetime.date.today().strftime('%d-%m-%Y')

# Generar tabla donde se inyectarán los valores de los data frames
def alimentar_db(tabla: pd.DataFrame, tabla_sql: str, user, passwd, host, port, db):
    """ Esta función genera una tabla en la base de datos PostgreSQL
    y la alimenta con el data frame que se ingresa como argumento.
    """
    engine = get_engine(user, passwd, host, port, db)
    tabla['fecha_descarga'] = FECHA_DESCARGA
    tabla.to_sql(f'{tabla_sql}', con=engine, if_exists='replace')
```

## Generate directories and transforme data into `.csv` files

To generate the directories, we use two python libraries called [os](https://docs.python.org/3/library/os.html) and [pathlib](https://docs.python.org/3/library/pathlib.html).
``` python
# Definir el directorio base para hacer la generación
BASE_DIR = Path(os.path.abspath(os.path.dirname(__file__))).absolute()

# Definir la función de generación de los directorios
def crear_directorios(categoria:str):
    """ Esta función, realiza la creación de los directorios en donde se 
    guardarán los archivos del análisis de los datos.
    """
    try:
        dir_cat = os.path.join('data', categoria)
        dir_cat = os.path.join(BASE_DIR, dir_cat) 
        os.mkdir(dir_cat)
        dir_fecha = os.path.join(dir_cat, f'{ANIO_MES}')
        path_nuevo = os.path.join(BASE_DIR, dir_fecha)
        os.mkdir(path_nuevo)
        logging.info("Se crean los directorios de categorías y los específicos al mes y año.")
    except:
        dir_fecha = os.path.join(dir_cat, f'{ANIO_MES}')
        path_nuevo = os.path.join(BASE_DIR, dir_fecha)
        os.mkdir(path_nuevo)
        logging.info("Se crean solamente los directorios de mes y año ya que los de categoría existen.")
```

Once the folders are generated, using [Pandas](https://pandas.pydata.org/) and [csv](https://docs.python.org/3/library/csv.html) libraries it is possible
to build the functions to populate the required standard tables:

`csv from sources`
``` python
def csv_datos_fuente():
    """ Esta función genera una lista con los archivos correspondientes
    a las fuentes:
        - Museos: index = 0
        - Cines: index = 1
        - Bibliotecas: index = 2
    """
    archivos_csv = []
    for item in LISTA_FUENTES:
        r = requests.get(item['url'])
        download = r.content.decode('utf-8')
        csv_file = csv.reader(download.splitlines(), delimiter=',')
        csv_creator = list(csv_file)
        raw_data = pd.DataFrame(csv_creator[1:], columns=csv_creator[0])
        archivos_csv.append(raw_data)
    return archivos_csv
```

`example: data from museums`
``` python
def datos_museos(df_museos: pd.DataFrame):
    """ Esta funcion recibe como parámetro el data frame de datos-fuente
    de museos y genera un data frame con los datos normalizados.
    """
    df_museos_csv = pd.DataFrame(columns=columnas)
    df_museos_csv['cod_localidad'] = df_museos['Cod_Loc']
    df_museos_csv['id_provincia'] = df_museos['IdProvincia']
    df_museos_csv['id_departamento'] = df_museos['IdDepartamento']
    df_museos_csv['categoria'] = df_museos['categoria']
    df_museos_csv['provincia'] = df_museos['provincia']
    df_museos_csv['localidad'] = df_museos['localidad']
    df_museos_csv['nombre'] = df_museos['nombre']
    df_museos_csv['domicilio'] = df_museos['direccion']
    df_museos_csv['código postal'] = df_museos['CP']
    df_museos_csv['número de teléfono'] = df_museos['cod_area'] + df_museos['telefono']
    df_museos_csv['mail'] = df_museos['Mail']
    df_museos_csv['web'] = df_museos['Web']
    logging.info("Se crea dataframe de museos.")
    return df_museos_csv
```

## Functionality

``` python
if __name__ == '__main__':
  # Crear directorio data
    os.mkdir('data')
    logging.info("Se crea directorio data.")

    # Generar los directorios correspondientes
    try: 
        categorias = ['cines', 'bibliotecas', 'museos']
        for categoria in categorias:
            crear_directorios(categoria)
        logging.info("Directorios generados.")
    except Exception as e:
        logging.warning(f'Los directorios ya se encuentran creados. El detalle de la excepcion es: {e}')

    # Generar los archivos .csv    
    try:
        gen_tabla_normalizada()
        generar_csv('museos')
        generar_csv('cines')
        generar_csv('bibliotecas')
        logging.info('Archivos csv generados correctamente.')
    except:
        logging.warning('Los archivos ya se encuentran generados.')

    # Generar las otras tablas
    try:
        analisis_datos_cines()
        tabla_categoria()
        tabla_fuente()
        tabla_prov_cat()
        logging.info("Se crean correctamente las tablas de análisis.")
    except:
        logging.warning("Chequear que se hayan importado correctamente los archivos csv.")
    
    # Generar conexión a base de datos y alimentar las tablas
    try:
        alimentar_db(gen_tabla_normalizada(), 'tabla_general', user, passwd, host, port, db)
        alimentar_db(analisis_datos_cines(), 'tabla_cines', user, passwd, host, port, db)
        alimentar_db(tabla_categoria(), 'tabla_por_categoria', user, passwd, host, port, db)
        alimentar_db(tabla_fuente(), 'tabla_por_fuente', user, passwd, host, port, db)
        alimentar_db(tabla_prov_cat(), 'tabla_por_provincia_categoria', user, passwd, host, port, db)
        logging.info('Se crean las tablas correspondientes en la base de datos.')
    except Exception as e:
        logging.warning(f'Revisar los datos de conexión a los archivos anteriores. Revisar excepción: {e}.')
```

## Logging

It was necessary to log all the requirements and save it using certain configuration.
A snippet for the data could be read as:
`datos_generados.log`
``` log
2022-07-08 11:52:02,332:INFO:Se crea archivo .csv con los datos-fuente de categoría bibliotecas.
2022-07-08 11:52:02,332:INFO:Archivos csv generados correctamente.
2022-07-08 11:52:02,348:INFO:Se crea tabla de análisis de cines.
2022-07-08 11:52:02,348:INFO:Se crea tabla de análisis por categoria.
2022-07-08 11:52:02,354:INFO:Se crea tabla de análisis por fuente.
2022-07-08 11:52:02,354:INFO:Se crea tabla de análisis por provincia y categoría.
2022-07-08 11:52:02,354:INFO:Se crean correctamente las tablas de análisis.
```

## Where can I get the repo?

In order to get all details and the files in more details, please check my repo in github.
[Alkemy Data Challenge](https://github.com/manoloacademia/alkemy_data_challenge).

## *__Cheers, Pablo.__*

