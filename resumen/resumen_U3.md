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

- Está formada por capas (*layers*).
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
- Tiene un identificador único (*Container ID*).

Un contenedor puede identificarse mediante:

- ID completo.
- ID abreviado (si es único).
- Nombre.

Si no se especifica un nombre, Docker genera uno automáticamente.

---

## Imagen vs Contenedor

| Imagen | Contenedor |
|----------|----------|
| Plantilla | Instancia |
| Solo lectura | Tiene capa de escritura |
| Puede generar varios contenedores | Se crea desde una imagen |
| Permanente | Puede estar activo o detenido |

---

## 2. Capas y almacenamiento

### ¿Por qué Docker utiliza capas?

Docker almacena cada capa una sola vez.

Esto permite:

- Ahorrar espacio.
- Reducir descargas.
- Reutilizar componentes comunes.
- Aprovechar la caché durante la construcción de imágenes.

---

### OverlayFS y overlay2

#### OverlayFS

Sistema de archivos por capas integrado en Linux.

#### overlay2

Driver de almacenamiento utilizado por Docker para gestionar imágenes y contenedores.

Consultar la configuración:

```bash
docker info
```

Información importante:

| Campo | Significado |
|---------|-------------|
| Storage Driver | Driver de almacenamiento utilizado |
| Docker Root Dir | Directorio donde Docker almacena sus datos |

Salida habitual:

```text
Storage Driver: overlay2
Docker Root Dir: /var/lib/docker
```

---

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
```

Estado:

```text
Created
```

---

### docker start

Arranca un contenedor existente.

```bash
docker start contenedor
```

---

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

#### Contenedores activos

```bash
docker ps
```

#### Todos los contenedores

```bash
docker ps -a
```

#### Solo IDs

```bash
docker ps -q
```

#### Último contenedor creado

```bash
docker ps -l
```

### Estados habituales

| Estado | Significado |
|----------|----------|
| Created | Creado |
| Running | En ejecución |
| Exited | Detenido |
| Paused | Pausado |
| Restarting | Reiniciándose |

---

## 8. Control de contenedores

#### Arrancar

```bash
docker start ID
```

#### Detener

```bash
docker stop ID
```

#### Reiniciar

```bash
docker restart ID
```

#### Pausar

```bash
docker pause ID
```

#### Reanudar

```bash
docker unpause ID
```

#### Forzar parada

```bash
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

#### Abrir Bash

```bash
docker exec -it contenedor bash
```

#### Abrir Shell

```bash
docker exec -it contenedor sh
```

#### Ejecutar un comando

```bash
docker exec contenedor ls /
```

### Opciones más utilizadas

| Opción | Función |
|----------|----------|
| `-i` | Mantiene la entrada estándar |
| `-t` | Crea una terminal virtual |
| `-d` | Ejecuta en segundo plano |
| `-e` | Define variables de entorno |
| `-u` | Usuario de ejecución |
| `-w` | Directorio de trabajo |

---

## 11. Copiar archivos

#### Del contenedor al anfitrión

```bash
docker cp contenedor:/tmp/prueba .
```

#### Del anfitrión al contenedor

```bash
docker cp archivo.txt contenedor:/tmp/
```

Funciona con contenedores activos o detenidos.

---

## 12. Attach vs Exec

### docker attach

Conecta la terminal al proceso principal del contenedor.

```bash
docker attach contenedor
```

### docker exec

Crea un proceso adicional dentro del contenedor.

```bash
docker exec -it contenedor bash
```

### Diferencia

| attach | exec |
|----------|----------|
| Se conecta al proceso principal | Ejecuta un nuevo proceso |
| No crea procesos nuevos | Crea procesos adicionales |
| Más limitado | Más flexible |

---

## 13. Logs

#### Mostrar registros

```bash
docker logs contenedor
```

#### Seguir registros en tiempo real

```bash
docker logs -f contenedor
```

#### Últimas líneas

```bash
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

| Acción | Comando |
|----------|----------|
| Buscar imágenes | `docker search` |
| Descargar imagen | `docker pull` |
| Ver imágenes | `docker images` |
| Crear y arrancar | `docker run` |
| Crear sin arrancar | `docker create` |
| Arrancar existente | `docker start` |
| Detener | `docker stop` |
| Reiniciar | `docker restart` |
| Ver contenedores | `docker ps -a` |
| Inspeccionar | `docker inspect` |
| Ejecutar comandos | `docker exec` |
| Ver logs | `docker logs` |
| Copiar archivos | `docker cp` |
| Renombrar | `docker rename` |
| Crear volumen | `docker volume create` |

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
docker run ubuntu sleep infinity
```

o

```bash
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

| Política | Comportamiento |
|-----------|-----------|
| `no` | No reinicia automáticamente (valor por defecto). |
| `always` | Reinicia siempre el contenedor. |
| `unless-stopped` | Reinicia automáticamente salvo que se haya detenido manualmente. |
| `on-failure` | Reinicia únicamente si el proceso termina con error. |

La política de reinicio también puede modificarse en contenedores ya existentes mediante:

```bash
docker update --restart unless-stopped nombreContenedor
```

