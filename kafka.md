# Kafka

### Arquitectura

- ZooKeeper
	- Dependencia, es un servicio de coordinación centralizada para aplicaciones distribuidas
	- Estructura similar a un sistema de ficheros, cada elemento de la estructura es conocido como **ZNode** pudiendo tener información en su interior (por ejemplo json string)
	- Elige el *líder*

- Productores y consumidores
	- Modelo de colas
		Los mensajes son repartidos
	- Modelo publicador/subscriptor
		Los mensajes se difunden en broadcast

- Broker
	- Servicio principal de kafka
	- Almacena las distintas colas de mensajes (topics)
	- Utilizados para crear clusters y que kafka escale
	- Sincronización entre los broker utilizando *ZooKeeper*

- Topics
	- Son las colas de mensajes
	- Están divididas en particiones, y dentro de estas particiones es donde encontraríamos los mensajes

- Partición
	- Cada partición es un fichero que se encuentra en el disco denominados **logs**

- Leaders
	- Cada partición tiene un *leader* y si hay replicas tendríamos *followers*. *ZooKeeper* se encarga de ir copiando los mensajes del *leader* al *follower*, y también si el *broker* se cae convierte el *follower* en *leader* automáticamente

- Mensajes
	- Se pueden identificar por el *topic* en el que está, la partición (log) en la que está de ese *topic*, y cada mensaje es identificado por un **offset**
	- Estructura
		- headers
		- timestamp
		- clave
			Se suele usar para decidir a que partición se envía cada mensaje
		- valor

- Escalabilidad
	- Podemos escalar añadiendo *brokers* o *replicas*
	- Replicas
		- Podremos poner tantas replicas como número de *brokers* tengamos, no más
		- Si sólo tenemos para un *topic* una réplica, 3 particiones y 3 *brokers* por ejemplo, si perdemos un *broker* perderíamos el servicio porque el *leader* de que se encuentre en la partición del *broker* que se ha caído no tiene ningún *follower* al que convertir en *leader*. Si en cambio tuviesemos replicas puede reconvertir un *follower* en *leader*
		- El problema de las replicas sería que estamos sobrecargando el sistema, porque estamos necesitamos más espacio en disco, y más procesado al tener que ir copiando datos de los *leaders* a los *followers*

- Productores
	- Encargados de enviar los mensajes a los distintos *topics*
	- Los mensajes se envían directamente al *broker* que tiene el *leader* de la partición
		1. El productor pregunta a cualquier *broker*
		2. El broker devuelve el *leader* de la partición
	- Los mensajes se pueden producir utilizando un particionado o de manera aleatoria
	- Normalmente se utiliza una clave de los mensajes para realizar el particionado
	- Los mensajes se pueden enviar por lotes de manera asíncrona
		- Basado en tiempo
		- Basado en número

- Consumidores
	- Son los encargados de leer los mensajes desde los distintos *topics*
	- Utiliza un *topic* especial denominado **__consumer_offsets** para obtener dónde se encuentran las particiones y para guardar el estado de sus *offsets*. Va guardando por dónde va leyendo, por lo que si se cayese otro podría seguir desde el punto en el que iba leyendo
	- Los consumidores se gestionan utilizando los **consumers groups**


### Configuración

#### 1. Server

- Fichero de configuración básico `server.properties`

- Server Basics
	`broker.id=0` -> Cada *broker* tiene un id diferente

- Socket Server Settings
	`listeners=PLAINTEXT://0.0.0.0:9092` -> Dirección de la máquina para escuchar
	`advertised.listeners=PLAINTEXT://localhost:9092` -> Dirección pública para publicar

- Log Basics
	`log.dirs=/tmp/kafka-logs`

- Log Retention Policy
	`log.retention.hours=168`
	`log.segment.bytes=1073741824`
	`log.retention.check.interval.ms=300000`
	`log.cleaner.enable=true`

- ZooKeeper
	`zookeeper.connect=localhost:2181`
	`zookeeper.connection.timeout.ms=6000`

#### 2. Productor

- Principales propiedades
	- `bootstrap.servers` -> Lista de *brokers* separados con comas indicando sus puertos
	- `key.serializer` -> Clase utilizada para serializar las claves de los mensajes
	- `value.serializer` -> Clase utilizada para serializar el payload del mensaje
	- `retries` -> Número de intentos al enviar un mensaje si se produce un fallo (puede probocar duplicidad)
	- `ack` -> Asentimiento
		- 0 : El p roductor no espera ningún asentimiento
		- 1 : El productor espera asentimiento por la partición líder
		- all : El productor espera el asentimiento por la partición líder y por las réplicas
	- Enviar mensaje
		- Partición
			- Indicar la partición a la hora de enviar cada mensaje
			- Utilizar un sistema de particionado, utilizando la propiedad `partitioner.class`
		- Lotes
			- `batch.size` -> Número de mensajes que conforma un lote antes de enviarlo
			- `linger.ms` -> Tiempo máximo de espera mientras se forma el lote antes de enviarlo

#### 3. Consumer

- Principales propiedades
	- `bootstrap.servers` -> Lista de *brokers* separados con comas indicando sus puertos
	- `key.deserializer` -> Clase utilizada para deserializar las claves de los mensajes
	- `value.deserializer` -> Clase utilizada para deserializar el payload del mensaje
	- `group.id` -> Se utiliza para indicar que un consumidor pertenece a un grupo de consumidores
	- **Offset Control**
		- Se puede configurar de forma automática o manual
			- `enable.auto.commit` -> Activación del control automático
			- `auto.commit.interval.ms` -> Cada cuanto tiempo se actualizan los *offsets* leídos
		- `auto.offset.reset` -> Nos permite configurar dónde queremos comenzar a leer del *topic*
			- `earliest` -> Desde el origen del *topic*
			- `latest` -> Desde el final del *topic*
