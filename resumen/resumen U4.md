# Guía Práctica de Gestión de Imágenes en Docker (UD 04)

## 1. Listado y búsqueda de imágenes

### Imágenes locales

Muestra las imágenes almacenadas en el equipo, indicando su repositorio, etiqueta, identificador y tamaño.

Se pueden utilizar indistintamente los dos comandos:

```bash
docker images
docker image ls
```

Ejemplos de filtros:

```bash
# Buscar una imagen con una etiqueta concreta
docker images ubuntu:22.04

# Buscar imágenes cuyo nombre coincida con el patrón
docker images --filter "reference=ubuntu:*"
```

### Imágenes remotas

Permite buscar imágenes disponibles en Docker Hub.

```bash
docker search ubuntu
```

Este comando muestra imágenes públicas disponibles en el registro, junto con información como su descripción, popularidad y si son oficiales.

### Etiquetas de una imagen

Las etiquetas (tags) de las imágenes Docker son identificadores alfanuméricos (como `ubuntu:22.04` o `nginx:latest`) esenciales para distinguir y gestionar las diferentes versiones de una imagen.

Permiten rastrear cambios, revertir a versiones anteriores y asegurar que los despliegues utilicen exactamente las mismas dependencias y configuraciones.

**Convenciones de nomenclatura comunes:**

- **Versionado Semántico (SemVer):** Uso de versiones estructuradas (ej. `1.19.0`).
- **Etiquetas descriptivas y de arquitectura:** Indicadores de estado o entorno (ej. `latest`, `stable`, `alpine`, `amd64`).
- **Metadatos de compilación:** Inclusión de hashes de commits o identificadores únicos de construcción (ej. `v1.2.3-abc123`).

## 2. Descarga e inspección de imágenes

### Descargar una imagen sin ejecutarla

Descarga una imagen del registro y la almacena localmente.

```bash
docker pull alpine:3.22
```

Si no se especifica una etiqueta, Docker utiliza `latest` por defecto.

### Consultar el historial de una imagen

Muestra las capas y las instrucciones que han intervenido en la construcción de una imagen.

```bash
docker history nginx
```

Permite consultar información como los comandos ejecutados durante su construcción y el tamaño de las diferentes capas.

### Inspeccionar una imagen

Muestra información detallada sobre una imagen en formato JSON. Para verlo más claro un JSON en linux podemos uar `jq` y en Windows podemos usar `ConvertFrom-Json`

```bash
docker image inspect ubuntu:22.04 | jq
```

```powershell
docker image inspect ubuntu:22.04 | ConvertFrom-Json
```

Permite consultar aspectos como la configuración, las variables de entorno, los puertos declarados y el sistema operativo de la imagen.

## 3. Eliminación y limpieza de imágenes

### Eliminar una imagen concreta

Elimina una imagen del almacenamiento local.

```bash
docker rmi ubuntu:22.04
```

También se puede utilizar:

```bash
docker image rm ubuntu:22.04
```

Docker no permite eliminar normalmente una imagen que esté siendo utilizada por algún contenedor, incluso si este se encuentra detenido.

### Localizar los contenedores que utilizan una imagen

Si Docker devuelve un error indicando que una imagen está siendo utilizada por un contenedor, podemos localizar los contenedores asociados mediante el filtro `ancestor`.

```bash
docker ps -a --filter ancestor=ubuntu:22.04
```

También se puede utilizar su forma moderna:

```bash
docker container ls -a --filter ancestor=ubuntu:22.04
```

Una vez localizados, podemos eliminar los contenedores que ya no necesitemos y después borrar la imagen.

### Eliminar todas las imágenes locales

El siguiente comando intenta eliminar todas las imágenes almacenadas localmente:

```bash
docker rmi $(docker images -q)
```

No podrá eliminar las imágenes que estén siendo utilizadas por contenedores.

### Forzar la eliminación de una imagen

La opción `-f` permite forzar la eliminación de una imagen y de sus referencias locales.

```bash
docker rmi -f ubuntu:22.04
```

**Importante:** forzar la eliminación puede dejar contenedores existentes utilizando una imagen que ya no está disponible localmente. No es recomendable utilizar esta opción como método habitual de limpieza.

### Detener y eliminar todos los contenedores

Primero detenemos todos los contenedores existentes:

```bash
docker stop $(docker ps -a -q)
```

Después eliminamos todos los contenedores:

```bash
docker rm $(docker ps -a -q)
```

### Limpieza general del sistema

Docker dispone de un comando para eliminar recursos que no están siendo utilizados.

```bash
docker system prune
# elimina:
# Todos los contenedores detenidos.
# Todas las redes no utilizadas por al menos un contenedor.
# Todas las imágenes huérfanas (no usadas y sin etiqueta).
# La caché de construcción no utilizada.
```

Para realizar una limpieza más profunda:

```bash
docker system prune -a
```

La opción `-a` también elimina las imágenes que no están asociadas a ningún contenedor existente, aunque no sean imágenes intermedias.

**Importante:** estos comandos pueden eliminar recursos que queramos conservar. Conviene revisar el resumen que muestra Docker antes de confirmar la operación.

Docker prune por defecto no elimina volúmenes, ya que pueden contener datos importantes


## 4. Creación y etiquetado de imágenes

### Crear una imagen desde un contenedor modificado

El comando `docker commit` permite crear una nueva imagen a partir de los cambios realizados en un contenedor.

```bash
docker commit -a "Marisol" -m "Ubuntu modificado" ID_CONTENEDOR usuariodockerhub/ubuntumod:2026
```

- `-a`: indica el autor de los cambios.
- `-m`: añade un mensaje descriptivo.
- `ID_CONTENEDOR`: identificador o nombre del contenedor.
- Último argumento: nombre y etiqueta de la nueva imagen.

**Importante:** `docker commit` no es el método habitual para construir imágenes. Para proyectos reproducibles se recomienda utilizar un Dockerfile.

### Añadir etiquetas adicionales

El comando `docker tag` permite asignar otra etiqueta a una imagen sin duplicar su contenido.

```bash
docker tag usuariodockerhub/ubuntumod:2026 usuariodockerhub/ubuntumod:latest
```

De esta forma, la misma imagen (id) tendrá dos etiquetas (tags) diferentes:

- `2026`
- `latest`

## 5. Exportación e importación de imágenes

### Exportar una imagen

El comando `docker save` permite guardar una imagen en un archivo TAR para realizar copias de seguridad o trasladarla a otro equipo.

```bash
docker save -o copiaSeguridad.tar usuariodockerhub/ubuntumod:2026
```

El archivo conserva las capas, la configuración y las etiquetas de la imagen.

### Importar una imagen

El comando `docker load` permite recuperar imágenes almacenadas previamente con `docker save`.

```bash
docker load -i copiaSeguridad.tar
```

Una vez importada, la imagen estará disponible localmente.

**Importante: diferencia entre save/load y export/import**

- `docker save`: guarda una o varias imágenes, conservando sus capas y etiquetas.
- `docker export`: exporta el sistema de archivos de un contenedor, pero no conserva la estructura de capas ni la configuración original de la imagen.
- `docker import`: crea una nueva imagen a partir de un archivo generado, por ejemplo, con `docker export`.

| Característica                      | `save` / `load`                       | `export` / `import`                              |
| ----------------------------------- | ------------------------------------- | ------------------------------------------------ |
| **Objetivo inicial**                | Imágenes                              | Contenedores                                     |
| **¿Conserva el historial/capas?**   | Sí, las mantiene todas separadas.     | No, aplasta todo en una sola capa plana.         |
| **¿Conserva metadatos (CMD, ENV)?** | Sí.                                   | No, se pierden al exportar.                      |
| **Tamaño del archivo resultante**   | Más grande (incluye historial).       | Más pequeño (solo archivos finales).             |
| **Propósito principal**             | Migrar imágenes exactas sin internet. | Crear imágenes base "planas" o extraer archivos. |

## 6. Subida de imágenes a registros remotos (Docker Hub)

Docker Hub permite almacenar y compartir imágenes para que otros usuarios puedan descargarlas.

### 1. Crear un repositorio

Acceder a Docker Hub y crear un repositorio con un nombre.

La estructura habitual de una imagen es:

```text
usuario/nombre_imagen:etiqueta
```

Por ejemplo:

```text
usuariodockerhub/prueba:latest
```

### 2. Autenticarse

Desde la terminal, iniciar sesión en Docker Hub:

```bash
docker login
```

Docker solicitará las credenciales o el método de autenticación correspondiente.

### 3. Etiquetar la imagen

La imagen local debe tener el nombre del repositorio remoto al que queremos subirla.

```bash
docker tag ubuntunano:latest usuariodockerhub/prueba:latest
```

### 4. Subir la imagen

Una vez etiquetada y autenticados, podemos subirla:

```bash
docker push usuariodockerhub/prueba:latest
```

La imagen quedará almacenada en el repositorio remoto y podrá descargarse desde otros equipos con `docker pull`

## 7. Automatización de imágenes mediante Dockerfile

Un Dockerfile es un archivo de texto que contiene las instrucciones necesarias para construir una imagen Docker.

Su nombre habitual es exactamente `Dockerfile`, sin extensión.

Permite automatizar la construcción de imágenes, documentar su configuración y reproducir el mismo entorno en diferentes equipos. Es la forma recomendada de crear imágenes, en lugar de utilizar `docker commit`.

### Ejemplo de Dockerfile básico

```dockerfile
# Imagen base
FROM ubuntu:24.04

# Actualizar los repositorios e instalar nano
RUN apt-get update && apt-get install -y nano

# Comando que se ejecutará por defecto al iniciar el contenedor
CMD ["/bin/bash"]
```

### Construir una imagen desde un Dockerfile

Situándonos en el directorio donde se encuentra el Dockerfile, ejecutamos:

```bash
docker build -t ubuntunano .
```

Donde:

- `docker build`: construye una imagen.
- `-t`: asigna un nombre y una etiqueta a la imagen.
- `.`: indica que el contexto de construcción es el directorio actual.

### Instrucciones principales de Dockerfile

Un **Dockerfile** es un archivo de texto que contiene instrucciones para construir una imagen Docker. Cada instrucción permite definir la imagen base, instalar programas, copiar archivos, configurar el entorno y establecer cómo se ejecutará el contenedor.

Estas son las instrucciones más utilizadas:

#### 1. FROM

Indica la imagen base sobre la que se construirá la nueva imagen.

Suele ser la primera instrucción de un Dockerfile, salvo excepciones relacionadas con directivas especiales.

```
FROM ubuntu:24.04
```

En este ejemplo, se utiliza Ubuntu 24.04 como imagen base.

#### 2. RUN

Ejecuta comandos durante la construcción de la imagen. Se utiliza principalmente para instalar programas, crear directorios o configurar el sistema.

```
RUN apt-get update && apt-get install -y nano
```

**Importante:** Es recomendable agrupar los comandos relacionados en una misma instrucción `RUN` para reducir el número de capas de la imagen. En imágenes Debian y Ubuntu también conviene limpiar la caché de paquetes cuando ya no sea necesaria.

```
RUN apt-get update && \
    apt-get install -y --no-install-recommends nano && \
    rm -rf /var/lib/apt/lists/*
```

#### 3. EXPOSE

Documenta los puertos en los que la aplicación del contenedor espera recibir conexiones.

```
EXPOSE 80 443 8080
```

**Importante:** `EXPOSE` no publica los puertos en el equipo anfitrión. Para permitir el acceso desde el exterior se utiliza la opción `-p` al crear el contenedor.

```bash
docker run -d -p 8080:80 mi-imagen
```

En este caso, el puerto 8080 del equipo anfitrión se comunica con el puerto 80 del contenedor.

#### 4. COPY

Copia archivos o directorios desde el contexto de construcción al interior de la imagen.

```
COPY ./mifichero.zip /var/www/html/
```

También permite copiar archivos utilizando varias fuentes:

```
COPY archivo1.txt archivo2.txt /app/
```

#### 5. ADD

Permite copiar archivos y directorios, pero incorpora funcionalidades adicionales respecto a `COPY`:

- Extrae automáticamente determinados archivos comprimidos, como los archivos `.tar`.
- Permite obtener archivos desde URL y utilizar otras fuentes.

```
ADD mifichero.tar.gz /var/www/html/
ADD https://example.com/archivo.txt /app/
```

En este ejemplo, Docker extrae automáticamente el contenido del archivo comprimido en el directorio indicado.

**Importante:** Se recomienda utilizar `COPY` para copiar archivos locales y reservar `ADD` para los casos en los que se necesiten sus funcionalidades adicionales.

#### 6. ENTRYPOINT

Define el ejecutable principal que se iniciará cuando arranque el contenedor.

```
ENTRYPOINT ["cat"]
```

En este ejemplo, el proceso principal del contenedor será el comando `cat`.

Se recomienda utilizar la forma ejecutable (_exec form_), que emplea corchetes y comillas, para que el proceso se ejecute directamente y pueda recibir correctamente las señales del sistema.

#### 7. CMD

Indica el comando o los argumentos predeterminados que se utilizarán al iniciar el contenedor.

```
CMD ["/etc/passwd"]
```

Cuando se combinan `ENTRYPOINT` y `CMD`, el contenido de `CMD` se utiliza como argumentos predeterminados de `ENTRYPOINT`.

Por ejemplo:

```
ENTRYPOINT ["cat"]
CMD ["/etc/passwd"]
```

Al iniciar el contenedor, se ejecutaría:

```
cat /etc/passwd
```

**Importante:** Si al ejecutar `docker run` se proporcionan argumentos adicionales, estos **sustituyen** los valores definidos en `CMD`, manteniendo el `ENTRYPOINT`.

```
docker run mi-imagen /etc/hosts
```

En este caso, se ejecutaría:

```
cat /etc/hosts
```

Si no se define `ENTRYPOINT`, `CMD` puede utilizarse para establecer el comando predeterminado del contenedor.

#### 8. USER

Indica el usuario con el que se ejecutarán las instrucciones posteriores y el proceso principal del contenedor.

```
USER marisol
```

El usuario debe existir previamente en la imagen.

Si no se especifica, normalmente se utiliza `root` (administrador), lo que implica que los procesos del contenedor se ejecutarán con privilegios elevados dentro de su entorno.

**Buena práctica:** Siempre que sea posible, es recomendable ejecutar las aplicaciones con un usuario sin privilegios de administrador para mejorar la seguridad.

#### 9. WORKDIR

Establece el directorio de trabajo para las instrucciones posteriores y para el proceso principal del contenedor.

```
WORKDIR /var/www/html
```

Si el directorio no existe, Docker lo crea automáticamente.

Las rutas relativas de instrucciones posteriores se interpretarán respecto a este directorio.

Por ejemplo:

```
WORKDIR /app
COPY . .
```

Los archivos del contexto de construcción se copiarán al directorio `/app`.

#### 10. ENV

Define variables de entorno que estarán disponibles durante las instrucciones de construcción posteriores y en los contenedores creados a partir de la imagen.

```
ENV V1="valor1" V2="valor2"
```

También se puede definir cada variable por separado:

```
ENV APP_ENV=production
ENV APP_PORT=8080
```

Estas variables pueden utilizarse en instrucciones posteriores y dentro de la aplicación que se ejecuta en el contenedor.

**Importante:** No se deben utilizar variables `ENV` para almacenar contraseñas, claves privadas o información sensible, ya que pueden quedar expuestas en la configuración o en el historial de la imagen.

#### 11. ARG

Define variables que se pueden proporcionar durante la **construcción** de la imagen.

```
ARG VERSION=1.0
```

Se pueden utilizar en instrucciones posteriores del Dockerfile:

```
RUN echo "Construyendo versión $VERSION"
```

Para proporcionar otro valor durante la construcción:

```
docker build --build-arg VERSION=2.0 -t mi-imagen .
```

**Diferencia entre `ARG` y `ENV`:**

- `ARG`: se utiliza principalmente durante la construcción de la imagen y no está disponible automáticamente en el contenedor.
- `ENV`: establece variables de entorno que permanecen disponibles en los contenedores creados a partir de la imagen.

#### 12. LABEL

Permite añadir metadatos a una imagen, como el autor, la versión o una descripción.

```
LABEL maintainer="Marisol"
LABEL version="1.0"
LABEL description="Servidor web basado en Ubuntu"
```

Estos datos permiten identificar y organizar mejor las imágenes.

#### 13. VOLUME

Indica que una determinada ruta del contenedor está destinada a almacenar datos persistentes.

```
VOLUME ["/datos"]
```

Docker puede crear un volumen anónimo para esa ruta cuando se inicia un contenedor, si no se proporciona otro almacenamiento.

Los volúmenes permiten conservar datos aunque se elimine el contenedor.

#### 14. HEALTHCHECK

Permite definir una comprobación para que Docker pueda determinar si el contenedor está funcionando correctamente.

```
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
```

En este ejemplo:

- `--interval=30s`: realiza una comprobación cada 30 segundos.
- `--timeout=5s`: establece un tiempo máximo de espera de 5 segundos.
- `--retries=3`: considera que el contenedor no está saludable después de tres comprobaciones fallidas consecutivas.
- `CMD`: ejecuta el comando que comprueba el estado de la aplicación.

**Importante:** El programa utilizado para realizar la comprobación, en este caso `curl`, debe estar instalado en la imagen.

### Resumen de las instrucciones

| Instrucción   | Función principal                                      |
| ------------- | ------------------------------------------------------ |
| `FROM`        | Selecciona la imagen base.                             |
| `RUN`         | Ejecuta comandos durante la construcción.              |
| `EXPOSE`      | Documenta los puertos utilizados.                      |
| `COPY`        | Copia archivos del contexto de construcción.           |
| `ADD`         | Copia archivos y ofrece funcionalidades adicionales.   |
| `ENTRYPOINT`  | Define el ejecutable principal.                        |
| `CMD`         | Establece el comando o los argumentos predeterminados. |
| `USER`        | Selecciona el usuario de ejecución.                    |
| `WORKDIR`     | Establece el directorio de trabajo.                    |
| `ENV`         | Define variables de entorno.                           |
| `ARG`         | Define variables para la construcción.                 |
| `LABEL`       | Añade metadatos a la imagen.                           |
| `VOLUME`      | Declara puntos de almacenamiento persistente.          |
| `HEALTHCHECK` | Define comprobaciones del estado del contenedor.       |

**Nota:** Estas son las instrucciones principales que conviene conocer para empezar a crear Dockerfiles. Existen otras, como `SHELL`, `STOPSIGNAL`, `ONBUILD` ... que permiten configuraciones más específicas.

Podemos ver las capas de una imagen construida con un Dockerfile utilizando el comando `docker history`:

```bash
docker history nginx
# el orden de las capas es de arriba a abajo, siendo la capa superior la más reciente.

docker history --no-trunc nginx
# permite ver el contenido completo de las capas, incluyendo los comandos ejecutados durante la construcción.
```

Además de utilizar Dockerfile, existen otras herramientas y procedimientos para construir imágenes de contenedores.

| Método o herramienta | Descripción                                                                            |
| -------------------- | -------------------------------------------------------------------------------------- |
| **Dockerfile**       | Método habitual. Define mediante instrucciones cómo construir una imagen.              |
| **Buildah**          | Permite construir imágenes mediante comandos, sin necesidad de utilizar un Dockerfile. |
| **Rockcraft**        | Herramienta de Canonical para construir imágenes de contenedores llamadas _rocks_.     |
| **umoci**            | Permite crear, modificar y configurar imágenes OCI.                                    |
| **Kaniko**           | Permite construir imágenes a partir de Dockerfiles sin utilizar un demonio Docker.     |
| **BuildKit**         | Motor avanzado de construcción de imágenes, utilizado también por Docker.              |

> OCI (Open Container Initiative) es un proyecto que define estándares abiertos para contenedores y sus imágenes.

## 8. Optimización de imágenes: buenas prácticas

El objetivo es construir imágenes más pequeñas, eficientes y fáciles de mantener.

### Utilizar imágenes base ligeras

Siempre que sea posible, utilizar imágenes base que incluyan únicamente los componentes necesarios.

Por ejemplo, Alpine Linux es una distribución ligera que dispone de imágenes oficiales para Docker.

```dockerfile
FROM alpine:3.22
```

Hay que tener en cuenta que una imagen más pequeña no siempre es la mejor opción: también debemos comprobar la compatibilidad de los programas y las dependencias.

### Agrupar instrucciones RUN

Es recomendable agrupar los comandos relacionados en una misma instrucción `RUN`.

Cada instrucción `RUN` genera una nueva capa en la imagen.

Por ejemplo, al instalar paquetes en Ubuntu, podemos actualizar los repositorios, instalar los programas y eliminar los archivos temporales en una sola instrucción:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends nano \
    && rm -rf /var/lib/apt/lists/*
```

De esta manera:

- Se actualizan los repositorios.
- Se instala el programa sin paquetes recomendados innecesarios.
- Se elimina la información temporal de los repositorios.
- Se evita almacenar esa información en una capa independiente.

### Evitar instalar programas innecesarios

Cuantos más programas y dependencias incluya una imagen, mayor será su tamaño.

Conviene instalar únicamente los paquetes que realmente necesite la aplicación.

### Utilizar etiquetas concretas

Es recomendable especificar versiones concretas en las imágenes base:

```dockerfile
FROM ubuntu:24.04
```

En lugar de:

```dockerfile
FROM ubuntu:latest
```

Así se reduce el riesgo de que una futura actualización de la imagen base cambie el entorno de construcción.

### Evitar ejecutar aplicaciones como root. 

Siempre que sea posible, utilizar la instrucción USER para ejecutar los procesos con un usuario sin privilegios de administrador.

### Utilizar .dockerignore

El archivo `.dockerignore` permite excluir archivos y directorios del contexto de construcción.

Por ejemplo:

```text
.git
.env
*.log
__pycache__
```

Esto evita enviar archivos innecesarios al motor de Docker y ayuda a **reducir** el contexto de construcción. Ademas Docker no podrá acceder a ellos mediante instrucciones como COPY o ADD desde ese contexto por lo que mejorará la **seguridad** de la imagen.
