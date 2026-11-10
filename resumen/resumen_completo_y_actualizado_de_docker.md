# Apuntes: Introducción a los contenedores y a Docker

**Unidad:** UD 01.01 — Introducción a los contenedores y a Docker
**Autor del material original:** Sergi García Barea
**Fecha:** Enero de 2026
**Licencia:** Reconocimiento–NoComercial–CompartirIgual (BY-NC-SA)

## 1. Virtualización y máquinas virtuales

La **virtualización** abstrae recursos físicos para que puedan administrarse como recursos virtuales. Se utiliza, entre otros fines, para desplegar sistemas, desarrollar y probar software, analizar programas y escalar servicios.

Una máquina virtual permite ejecutar programas como si dispusieran de una máquina independiente. Varias tecnologías relacionadas:

* **Máquinas virtuales de proceso:** Ejecutan programas mediante un entorno compatible para una arquitectura concreta (ej. Máquina Virtual de Java - JVM, o la plataforma .NET).
* **Emuladores:** Reproducen un hardware o una API determinados (ej. emuladores de consolas o Wine para Windows en Linux).
* **Hipervisores:** Virtualizan total o parcialmente el hardware y permiten ejecutar sistemas operativos completos (máquinas virtuales tradicionales), como hacen VirtualBox y VMware.
* **Contenedores:** Aíslan entornos sin virtualizar todo el hardware (el modelo de Docker).

## 2. Qué es un contenedor

Un contenedor es un entorno privado y aislado (a nivel de procesos, memoria, sistema de archivos y red) que **comparte el sistema base y el núcleo (kernel) de la máquina anfitriona**. A diferencia de una máquina virtual tradicional, cada contenedor no necesita incluir un sistema operativo completo.

Por depender del núcleo anfitrión, un contenedor no ejecuta de forma nativa un sistema operativo distinto.

**La analogía del transporte marítimo:**
Al igual que los contenedores de mercancías estandarizaron el tamaño para poder llevar cualquier carga en cualquier barco, los contenedores de software estandarizan el código. Si se respeta un estándar común, el contenedor puede trasladarse entre plataformas compatibles y su contenido puede ser independiente de la máquina concreta.

## 3. Usos, ventajas y limitaciones

### Usos habituales

* Preparar entornos reproducibles de desarrollo, compilación y depuración.
* Probar una aplicación con distintas versiones, configuraciones o límites de recursos.
* Distribuir y desplegar aplicaciones o servicios (web, correo, bases de datos, DNS) con el mismo entorno usado durante las pruebas (evitando el "en mi máquina funciona").
* Automatizar pruebas y despliegues mediante procesos de integración y entrega continua (**CI/CD**).
* Ejecutar varias copias de un servicio para escalarlo horizontalmente, normalmente con ayuda de herramientas de orquestación.
* Probar software rápidamente usando imágenes públicas (ej. Docker Hub).

### Ventajas

* Menor consumo de espacio que las máquinas virtuales completas, al compartir el núcleo del anfitrión.
* Rendimiento cercano al nativo y, generalmente, muy superior al de una virtualización completa por hipervisor.
* Entornos coherentes entre desarrollo, pruebas y despliegue, reduciendo incompatibilidades.
* Amplio soporte de empresas y proyectos que publican imágenes listas para usar.

### Limitaciones

* El aislamiento tiene un coste: el rendimiento no suele superar al de ejecutar de forma *bare metal* (directamente sobre el sistema físico).
* La gestión de datos persistentes y su acceso entre contenedores requiere planificación.
* Los contenedores se usan principalmente desde la línea de comandos (CLI); proporcionarles un entorno gráfico es más tedioso.
* La compatibilidad depende del sistema operativo y del núcleo subyacente.

## 4. Contenedores en Linux

Los entornos aislados no son nuevos; tienen antecedentes como **chroot** (1982) y **FreeBSD jail** (1999). En Linux, los contenedores modernos (como **LXC**) se desarrollaron a partir de herramientas de aislamiento que ya venían integradas dentro del núcleo (kernel),.

Dos mecanismos clave que utilizan son:

* **Namespaces:** Aíslan lo que los procesos pueden ver (procesos, red, usuarios). Así, por ejemplo, un proceso puede aparecer como `root` dentro del contenedor sin tener esos privilegios en el anfitrión real.
* **Cgroups (Control Groups):** Permiten organizar y limitar el uso de recursos físicos, como cuánta memoria, CPU o entrada/salida puede usar un contenedor.

En conjunto, estos mecanismos proporcionan aislamiento y control de recursos. Aunque es posible construir un contenedor manualmente usando scripts, herramientas como Docker simplifican enormemente su creación y gestión.

## 5. Docker: Conceptos y Arquitectura

**Docker** es una tecnología de contenedores Linux que aprovecha las capacidades del núcleo para desarrollar y desplegar aplicaciones. El ecosistema se basa en un motor de código abierto y ofrece diversas suscripciones comerciales para entornos de escritorio (Personal, Pro, Team y Business).

La arquitectura básica de Docker consta de tres partes:

1. **Cliente:** El software o interfaz de línea de comandos (CLI) que envía solicitudes al servidor Docker (ej. `docker build`, `docker run`).
2. **Servidor o Docker Host:** Ejecuta el servicio (*Daemon*) que atiende las solicitudes del cliente y gestiona internamente las imágenes y los contenedores.
3. **Registro (Registry):** Almacena imágenes públicas o privadas y sus versiones. Docker Hub es el registro más conocido y el configurado por defecto.

## 6. Docker en distintos Sistemas Operativos

En Linux, los contenedores se apoyan directamente en el núcleo y pueden alcanzar un rendimiento casi nativo. Para ejecutar contenedores Linux en Windows o macOS se necesita una capa de compatibilidad (Docker Desktop):

* **En Windows:** Docker Desktop utiliza **WSL 2 (Windows Subsystem for Linux 2)** como estándar en prácticamente todas las versiones modernas de Windows, ofreciendo un rendimiento excelente. El uso de Hyper-V tradicional se reserva para sistemas muy antiguos o para ejecutar nativamente *contenedores Windows Server Core*.
* **En macOS:** Docker Desktop se integra con el **Virtualization Framework nativo de Apple**, el cual está completamente optimizado para los procesadores Apple Silicon

## 7. Idea principal

Los contenedores ofrecen entornos aislados, ligeros y reproducibles que facilitan desarrollar, probar, distribuir y desplegar aplicaciones. No sustituyen a las máquinas virtuales tradicionales en todos los casos: su ventaja principal brilla cuando interesa compartir el núcleo del anfitrión para ahorrar recursos y mantener entornos consistentes. Docker es la herramienta que proporciona el ecosistema para gestionar fácilmente esos contenedores e imágenes.