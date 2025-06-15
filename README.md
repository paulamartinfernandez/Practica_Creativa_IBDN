# Practica_Creativa_IBDN

## Descripción

Este proyecto consiste en el diseño y despliegue de una arquitectura Big Data para predecir retrasos en vuelos. A partir de datos históricos de vuelos publicados por el Bureau of Transportation Statistics (EE.UU.), se entrena un modelo de aprendizaje automático con **RandomForest**. El sistema permite hacer predicciones en tiempo real mediante una interfaz web conectada con Kafka, Spark Streaming y MongoDB.

---

## Arquitectura

1. **Interfaz Web (Flask):** permite introducir los datos de un vuelo.
2. **Kafka:** se encarga de transmitir los datos del vuelo al sistema de predicción.
3. **Spark Streaming:** recibe los datos y predice si habrá retraso.
4. **MongoDB:** guarda la predicción.
5. **Polling desde Flask:** para recuperar y mostrar el resultado.
6. **Apache NiFi:** lee las predicciones cada 10 segundos y las guarda en un fichero `.txt`.

---

## Requisitos previos

- Docker y Docker Compose
- Git

---

## Instrucciones de despliegue

### 1. Clonar el repositorio

```
git clone https://github.com/paulamartinfernandez/Practica_Creativa_IBDN.git
cd practica_creativa
```

### 2. Levantar todos los servicios
```
docker compose build
docker compose up -d
```
```
cd flight_predictions
sbt clean compile package
```
### 3. Cargar los datos en el contenedor de mongo 
Para que funcione correctamente el programa lo primero es importar los datos al contenedor de mongo, que se llama mongo_dockerizado. 
```
docker exec -it mongo_dockerizado bash -c "mongoimport --username admin --password password --authenticationDatabase admin --db agile_data_science --collection origin_dest_distances --file /resources/data/origin_dest_distances.jsonl"
```
### 4. Habilitar consumidor de kafka. 
Cuando estamos en este punto se hace lo primero un submit dentro de la interfaz web implementada con flask.
```
http://localhost:5003/flights/delays/predict_kafka
```
![Imagen de WhatsApp 2025-06-15 a las 11 36 55_ed5cc5d9](https://github.com/user-attachments/assets/e27a0762-720c-410f-be75-627e1abc9692)

Cuando hacemos esto sin haber entrado en el contenedor de kafka inicialmente se cae el contenedor de spark-submit. Accedemos al directorio de kafka dentro de prácticas creativas y después accedemos al contenedor. En primer lugar listamos los tópicos creados.
```
docker exec -it kafka kafka-topics.sh --bootstrap-server localhost:9092 --list
```
![Imagen de WhatsApp 2025-06-15 a las 11 37 52_40546ffb](https://github.com/user-attachments/assets/a1e3dc2e-68bd-43d2-a542-9f2660128f61)

Comprobamos que están los dos y a partir de ahí accedemos al consumidor del tópico de las predicciones:
```
docker exec -it kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic flight-delay-ml-response \
  --from-beginning
```
Una vez esto está hecho volvemos a lanzar el contenedor de spark-submit:
```
docker compose up -d spark-submit
```
Ahora, al dar al botón de submit en la interfaz web observamos dos cosas:


1. En el consumidor de kafka se muestra en tiempo real la predicción.
   
   ![Imagen de WhatsApp 2025-06-15 a las 11 38 15_39e01a3b](https://github.com/user-attachments/assets/f12bc064-7a9b-4acb-8cc4-b0f956e7323e)

3. Si dentro de la interfaz le damos a inspeccionar y mostramos la consola vemos como la predicción también se muestra ahí.
   
   ![Imagen de WhatsApp 2025-06-15 a las 11 37 07_e9d171f6](https://github.com/user-attachments/assets/62824fcf-6ded-464a-a914-d93635d4617c)


### 5. Comprobar en mongo: 
Una vez el sistema está funcionando comprobamos que las predicciones se están guardando también en la base de datos de mongo:
```
docker exec -it mongo_dockerizado mongo -u admin -p password --authenticationDatabase admin
```
```
show databases
```
```
use agile_data_science
```
```
show collections
```
```
db.flight_delay_ml_response.find()
```
![Imagen de WhatsApp 2025-06-15 a las 11 29 02_9db926d5](https://github.com/user-attachments/assets/1e5a8952-897a-4dc5-9f0e-0dec10eeb8ea)

### 6. Desplegar Apache NiFi:

1. Se accede a NiFi en ```http://localhost:8080/nifi/```
2. Definimos el flujo que consume los datos de Kafka
3. Definimos un putfile que exporta las predicciones como archivos .txt en la ruta /opt/nifi/nifi-current/datos_nifi
   
   ![Imagen de WhatsApp 2025-06-15 a las 11 38 31_c8095445](https://github.com/user-attachments/assets/30cbc9ae-899f-4d67-8838-2456bfd36e5a)

5. Comprobamos que se está haciendo correctamente acccediendo al contenedor:
```
docker exec –it nifi bash
```
```
cd datos_nifi
```
```
ls
```
```
cat id
```


![Imagen de WhatsApp 2025-06-15 a las 11 38 56_f2d9f768](https://github.com/user-attachments/assets/57a92d54-dab1-487a-8ce9-144047b3ac0d)




