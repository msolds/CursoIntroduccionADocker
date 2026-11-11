# UD03. Principales acciones con Docker

## 1. Imágenes y contenedores

### Imagen (Image)

Una imagen Docker es una plantilla de solo lectura que contiene todo lo necesario para crear contenedores:

- Sistema de archivos.
- Aplicaciones.
- Librerías.
- Dependencias.
- Variables de entorno.
- Configuración y comandos de inicio.

### Características

- Está formada por capas (_layers_).
- Las capas pueden reutilizarse en otras imágenes.
- Una imagen puede crear muchos contenedores.
- Las imágenes no se modifican al ejecutar contenedores.

---

### Contenedor (Container)

Un contenedor es una instancia ejecutable de una imagen.

### Características

- Puede crearse, arrancarse, detenerse y eliminarse.
- Dispone de procesos y red aislados.
- Añade una capa de escritura sobre la imagen.
- Tiene un identificador único (_Container ID_).

Un contenedor puede identificarse mediante:

- ID completo.
- ID abreviado (si es único).
- Nombre.

Si no se especifica un nombre, Docker genera uno automáticamente.

---

## Imagen vs Contenedor

| Imagen                            | Contenedor                    |
| --------------------------------- | ----------------------------- |
| Plantilla                         | Instancia                     |
| Solo lectura                      | Tiene capa de escritura       |
| Puede generar varios contenedores | Se crea desde una imagen      |
| Permanente                        | Puede estar activo o detenido |

---

## 2. Capas y almacenamiento

### ¿Por qué Docker utiliza capas?

Docker almacena cada capa una sola vez.

Esto permite:

- Ahorrar espacio.
- Reducir descargas.
- Reutilizar componentes comunes.
- Aprovechar la caché durante la construcción de imágenes.

### Consultar la configuración:

```bash
docker info
```

Puedes ampliar la tabla con los campos más importantes que muestra `docker info`:

Información importante:

| Campo                | Significado                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| Server Version       | Versión del motor Docker instalado.                                                                    |
| Storage Driver       | Driver de almacenamiento utilizado para gestionar las capas de imágenes y contenedores.                |
| Docker Root Dir      | Directorio donde Docker almacena sus datos, como imágenes, contenedores y volúmenes.                   |
| Kernel Version       | Versión del núcleo del sistema operativo.                                                              |
| Operating System     | Sistema operativo sobre el que se ejecuta el motor Docker.                                             |
| Architecture         | Arquitectura del procesador (por ejemplo, x86_64 o aarch64).                                           |
| CPUs                 | Número de procesadores lógicos disponibles para Docker.                                                |
| Total Memory         | Memoria RAM total disponible para Docker.                                                              |
| Containers           | Número total de contenedores existentes.                                                               |
| Running              | Número de contenedores que están en ejecución.                                                         |
| Paused               | Número de contenedores pausados.                                                                       |
| Stopped              | Número de contenedores detenidos.                                                                      |
| Images               | Número de imágenes almacenadas localmente.                                                             |
| Registry             | Registro de imágenes utilizado por defecto (normalmente Docker Hub).                                   |
| Live Restore Enabled | Indica si los contenedores pueden seguir ejecutándose cuando se reinicia o detiene el servicio Docker. |

## 3. Tipos de almacenamiento

Docker dispone de tres mecanismos principales.

### Capa de escritura del contenedor

- Se crea al arrancar el contenedor.
- Almacena los cambios realizados.
- Se pierde al eliminar el contenedor.

---

### Volúmenes (Volumes)

Almacenamiento persistente gestionado por Docker.

#### Ventajas

- Conservan los datos.
- Pueden compartirse.
- Son independientes del contenedor.

#### Crear un volumen

```bash
docker volume create mis_datos
```

#### Listar volúmenes

```bash
docker volume ls
```

#### Inspeccionar un volumen

```bash
docker volume inspect mis_datos
```

---

### Bind Mount

Permite compartir directorios entre el anfitrión y el contenedor.

Ejemplo:

```bash
docker run -v ./documentos:/datos ubuntu
```

---

## 4. Docker Hub

Docker Hub es el registro público oficial de Docker.

Permite:

- Buscar imágenes.
- Descargar imágenes.
- Almacenar imágenes.
- Publicar imágenes propias.

---

### Conceptos importantes

#### Registry

Servidor que almacena imágenes.

#### Repository

Colección de imágenes relacionadas.

Ejemplo:

```text
nginx
```

#### Tag

Etiqueta que identifica una versión.

Ejemplos:

```text
ubuntu:24.04
nginx:alpine
hello-world:latest
```

Si no se indica una etiqueta, Docker utiliza:

```text
latest
```

---

### Comandos básicos

#### Buscar imágenes

```bash
docker search ubuntu
```

#### Descargar una imagen

```bash
docker pull ubuntu:24.04
```

#### Ver imágenes locales

```bash
docker images
```

---

## 5. Crear contenedores

### docker run

```bash
docker run imagen
```

Realiza automáticamente:

1. Comprueba si existe la imagen.
2. La descarga si es necesario.
3. Crea un contenedor.
4. Lo arranca.
5. Ejecuta el comando principal.

> Cada ejecución de `docker run` crea un contenedor nuevo.

---

### docker create

Crea un contenedor sin arrancarlo.

```bash
docker create ubuntu
docker start ID

# docker run equivalente a docker create + docker start
```

Estado:

```text
Created
```

## 6. Caso práctico: Hello World

```bash
docker run hello-world
```

Proceso:

1. Descarga la imagen si es necesario.
2. Crea el contenedor.
3. Ejecuta el programa.
4. Muestra el mensaje.
5. Finaliza.
6. El contenedor queda detenido.

---

## 7. Listar contenedores

```bash
# Contenedores activos
docker ps

#Todos los contenedores
docker ps -a

# Último contenedor creado
docker ps -l

# Solo IDs, es util para usarlo en scripts
docker ps -q

# borrar todos los contenedores
docker rm $(docker ps -aq)

```

### Estados habituales

| Estado     | Significado   |
| ---------- | ------------- |
| Created    | Creado        |
| Running    | En ejecución  |
| Exited     | Detenido      |
| Paused     | Pausado       |
| Restarting | Reiniciándose |

---

## 8. Control de contenedores

```bash
# Arrancar un contenedor detenido
docker start ID

# Detener un contenedor de forma ordenada
docker stop ID

# Reiniciar un contenedor
docker restart ID

# Pausar los procesos de un contenedor
docker pause ID

# Reanudar un contenedor pausado
docker unpause ID

# Forzar la detención inmediata de un contenedor
docker kill ID
```

---

## 9. Inspeccionar contenedores

```bash
docker inspect ID
```

Permite consultar:

- Configuración.
- Estado.
- Imagen utilizada.
- Redes.
- Dirección IP.
- Volúmenes.
- Variables de entorno.

La salida se muestra en formato JSON.

---

## 10. Ejecutar comandos en un contenedor

### docker exec

Permite ejecutar procesos en contenedores en funcionamiento.

```bash
# Abrir una terminal Bash interactiva dentro del contenedor
docker exec -it contenedor bash

# Abrir una terminal Shell interactiva dentro del contenedor
docker exec -it contenedor sh

# Ejecutar un comando dentro del contenedor (listar el directorio raíz)
docker exec contenedor ls /
```

### Opciones más utilizadas

| Opción | Función                                                                                            |
| ------ | -------------------------------------------------------------------------------------------------- |
| `-i`   | Mantiene abierta la entrada estándar (`stdin`), permitiendo introducir datos al proceso.           |
| `-t`   | Asigna una terminal virtual (TTY) al proceso, proporcionando una interfaz de terminal interactiva. |
| `-d`   | Ejecuta el contenedor en segundo plano, sin bloquear la terminal actual.                           |
| `-e`   | Define variables de entorno dentro del contenedor.                                                 |
| `-u`   | Especifica el usuario o UID con el que se ejecutará el proceso dentro del contenedor.              |
| `-w`   | Establece el directorio de trabajo desde el que se ejecutará el proceso dentro del contenedor.     |

---

## 11. Copiar archivos

En `docker cp`, `contenedor:` identifica el contenedor Docker desde el que se copia o al que se copia un archivo.

Tiene el formato:

```bash
contenedor:ruta
```

- `contenedor`: nombre o ID del contenedor.
- `:`: separador entre el contenedor y la ruta.
- `ruta`: ubicación del archivo o directorio dentro del contenedor.


```bash
# Copiar un archivo o directorio del contenedor al anfitrión
docker cp contenedor:/tmp/prueba .

# Copiar un archivo o directorio del anfitrión al contenedor
docker cp archivo.txt contenedor:/tmp/
```
Funciona con contenedores activos o detenidos.

## 12. Attach vs Exec

```bash
# docker attach: conecta la terminal al proceso principal del contenedor
docker attach contenedor

# docker exec: crea un proceso adicional dentro de un contenedor en ejecución
docker exec -it contenedor bash
```

### Diferencia

| attach                          | exec                      |
| ------------------------------- | ------------------------- |
| Se conecta al proceso principal | Ejecuta un nuevo proceso  |
| No crea procesos nuevos         | Crea procesos adicionales |
| Más limitado                    | Más flexible              |

---

## 13. Logs



```bash
# Mostrar registros
docker logs contenedor

# Seguir registros en tiempo real
docker logs -f contenedor

# Últimas líneas
docker logs --tail 10 contenedor
```

---

## 14. Renombrar contenedores

```bash
docker rename nombreViejo nombreNuevo
```

No:

- modifica la imagen,
- elimina el contenedor,
- interrumpe su ejecución.

---

## 15. Comandos fundamentales

| Acción             | Comando                |
| ------------------ | ---------------------- |
| Buscar imágenes    | `docker search`        |
| Descargar imagen   | `docker pull`          |
| Ver imágenes       | `docker images`        |
| Crear y arrancar   | `docker run`           |
| Crear sin arrancar | `docker create`        |
| Arrancar existente | `docker start`         |
| Detener            | `docker stop`          |
| Reiniciar          | `docker restart`       |
| Ver contenedores   | `docker ps -a`         |
| Inspeccionar       | `docker inspect`       |
| Ejecutar comandos  | `docker exec`          |
| Ver logs           | `docker logs`          |
| Copiar archivos    | `docker cp`            |
| Renombrar          | `docker rename`        |
| Crear volumen      | `docker volume create` |

# 16. ¿Por qué unos contenedores finalizan y otros siguen funcionando?

Una idea fundamental de Docker es que:

> Un contenedor permanece en ejecución mientras su proceso principal siga ejecutándose.

Cuando ese proceso termina, Docker detiene automáticamente el contenedor.

## Ejemplo 1: Un contenedor que termina automáticamente

```bash
docker run hello-world
```

¿Qué ocurre?

1. Docker descarga la imagen si es necesario.
2. Crea el contenedor.
3. Ejecuta el programa `hello`.
4. Muestra un mensaje por pantalla.
5. El programa termina.

Como ya no existe ningún proceso principal ejecutándose:

```text
Proceso principal → termina
         ↓
Contenedor → Exited
```

## Ejemplo 2: Un contenedor que sigue funcionando

```bash
docker run nginx
```

El servidor web Nginx queda esperando peticiones continuamente.

```text
Proceso principal → sigue activo
         ↓
Contenedor → Running
```

## Ejemplo 3: Ubuntu con terminal interactiva

```bash
docker run -it ubuntu bash
```

El proceso principal es:

```text
bash
```

Mientras la terminal permanezca abierta:

```text
bash → ejecutándose
```

el contenedor sigue activo.

Si escribimos:

```bash
exit
```

ocurre:

```text
bash → termina
     ↓
Contenedor → Exited
```

## Ejemplo 4: Mantener un contenedor activo

Podemos arrancar un contenedor ejecutando un proceso que nunca finalice:

```bash
# Mantener un contenedor activo con sleep, que espera un tiempo indefinido
docker run ubuntu sleep infinity

# Mantener un contenedor activo con tail, que espera indefinidamente a que se escriban datos en un archivo
docker run ubuntu tail -f /dev/null
```

En ambos casos:

```text
Proceso principal → nunca termina
         ↓
Contenedor → Running
```

# 17 ¿Qué ocurre si se apaga el anfitrión?

Si se apaga o reinicia el anfitrión (la máquina donde se está ejecutando Docker), todos los contenedores se detienen porque sus procesos dejan de ejecutarse. Sin embargo, los contenedores no se eliminan y continúan almacenados en el sistema, normalmente en estado `Exited` cuando el equipo vuelve a arrancar. Los datos también se conservan, especialmente si están almacenados en volúmenes. Para que un contenedor vuelva a iniciarse automáticamente tras reiniciar el sistema, puede configurarse una política de reinicio mediante el parámetro `--restart`.

## Políticas de reinicio (`--restart`)

Docker permite configurar el comportamiento de un contenedor cuando se detiene o cuando se reinicia el anfitrión.

```bash
docker run -d --restart unless-stopped --name web nginx
```

Las políticas más habituales son:

| Política         | Comportamiento                                                   |
| ---------------- | ---------------------------------------------------------------- |
| `no`             | No reinicia automáticamente (valor por defecto).                 |
| `always`         | Reinicia siempre el contenedor.                                  |
| `unless-stopped` | Reinicia automáticamente salvo que se haya detenido manualmente. |
| `on-failure`     | Reinicia únicamente si el proceso termina con error.             |

La política de reinicio también puede modificarse en contenedores ya existentes mediante:

```bash
docker update --restart unless-stopped nombreContenedor
```
