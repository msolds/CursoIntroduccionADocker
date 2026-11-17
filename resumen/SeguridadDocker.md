# Seguridad de imágenes Docker: vulnerabilidades, Hardened Images y registros

## 1. Seguridad en las imágenes Docker

Las imágenes Docker contienen todo lo necesario para ejecutar una aplicación: sistema de archivos, bibliotecas, dependencias, herramientas y configuración.

Sin embargo, una imagen puede incorporar componentes con vulnerabilidades de seguridad que pueden ser aprovechadas por un atacante.

Una vulnerabilidad es un fallo en un componente software que puede comprometer la confidencialidad, integridad o disponibilidad de un sistema.

### 1.1. ¿De dónde proceden las vulnerabilidades?

Las vulnerabilidades pueden encontrarse en diferentes elementos de una imagen:

| Componente               | Ejemplo de vulnerabilidad                                       |
| ------------------------ | --------------------------------------------------------------- |
| Sistema operativo base   | Una versión antigua de Alpine o Debian con fallos de seguridad. |
| Bibliotecas              | Una versión vulnerable de OpenSSL.                              |
| Aplicaciones             | Una versión de Nginx con vulnerabilidades conocidas.            |
| Dependencias             | Paquetes de Python, Node.js o Java con fallos de seguridad.     |
| Herramientas adicionales | Utilidades instaladas que contienen vulnerabilidades conocidas. |

Las vulnerabilidades se identifican habitualmente mediante identificadores CVE (Common Vulnerabilities and Exposures).

Por ejemplo: [CVE-2026-109618](https://www.cve.org/CVERecord?id=CVE-2026-109618), vulnerabilidad crítica del núclero de php.

### 1.2. ¿Cómo se clasifican las vulnerabilidades?

Se utiliza habitualmente el sistema CVSS (Common Vulnerability Scoring System), que asigna una puntuación de 0 a 10 según la gravedad de la vulnerabilidad.

| Puntuación CVSS | Gravedad  |
| --------------- | --------- |
| 0.1 - 3.9       | Baja L    |
| 4.0 - 6.9       | Media M   |
| 7.0 - 8.9       | Alta H    |
| 9.0 - 10.0      | Crítica C |

Una vulnerabilidad crítica puede permitir, por ejemplo, ejecutar código de forma remota, obtener privilegios o acceder a información protegida.

Importante: que una imagen contenga una vulnerabilidad no significa necesariamente que la aplicación sea explotable. Hay que analizar si el componente vulnerable se utiliza, si está expuesto y si se cumplen las condiciones necesarias para explotar el fallo.

## 2. Análisis de vulnerabilidades en imágenes Docker

Antes de utilizar una imagen en producción es recomendable analizar sus componentes y detectar vulnerabilidades conocidas.

Este análisis se denomina Image Vulnerability Scanning.

### 2.1. Docker Scout

Docker Scout es una herramienta de Docker que analiza las imágenes y detecta vulnerabilidades conocidas en los paquetes que contienen.

Para utilizarlo se pueden emplear los siguientes comandos:

```bash
#Consultar un resumen de vulnerabilidades:
docker scout quickview nginx:alpine

#Mostrar los identificadores CVE, los paquetes afectados y la información disponible sobre las vulnerabilidades.
docker scout cves nginx:alpine

#Consultar recomendaciones de actualización:
docker scout recommendations nginx:alpine

#Consultar únicamente vulnerabilidades críticas:
docker scout cves --only-severity critical nginx:alpine
```

### 2.2. ¿Cómo se corrigen las vulnerabilidades?

Cuando un análisis detecta vulnerabilidades, se deben estudiar las medidas correctoras disponibles:

1. Actualizar la imagen base a una versión que incorpore las correcciones de seguridad.
2. Actualizar las bibliotecas y dependencias vulnerables.
3. Eliminar paquetes y herramientas que no sean necesarios.
4. Reconstruir la imagen después de realizar los cambios.
5. Volver a analizarla para comprobar si las vulnerabilidades se han corregido.

>Importante: descargar una imagen nueva no actualiza automáticamente los contenedores que ya están creados. Es necesario reconstruir o volver a crear los contenedores utilizando la imagen actualizada.

```bash
# 1. Descargar la versión más reciente de la imagen
docker pull nginx:latest

# 2. Detener el contenedor actual (el que usa la imagen desactualizada)
docker stop mi_contenedor_nginx

# 3. Eliminar el contenedor antiguo
docker rm mi_contenedor_nginx

# 4. Volver a crear y levantar el contenedor (usará automáticamente la nueva imagen descargada)
docker run -d -p 80:80 --name mi_contenedor_nginx nginx:latest

# 5. Borrar la imagen anterior que quedó huérfana para liberar espacio (-f evita que pida confirmación)
docker image prune -f
```
> Al ejecutar el comando docker rm (paso 3), cualquier archivo o configuración que se haya modificado o guardado dentro del contenedor se perderá para siempre. Para evitar esto y que los datos sobrevivan a las actualizaciones, siempre debes usar Volúmenes (Volumes) al crear contenedores que guarden información importante.

La forma recomendada es con compose
```bash
docker compose pull
docker compose up -d
```


## 3. Docker Hardened Images (DHI)

Docker Hardened Images son imágenes de contenedores diseñadas para reducir los riesgos de seguridad.

Se distribuyen desde un registro específico: [dhi.io](https://hub.docker.com/hardened-images/catalog)

A diferencia de las imágenes convencionales, las imágenes Hardened están preparadas aplicando medidas de seguridad desde su construcción. [docker.com](https://www.docker.com/blog/what-are-hardened-images/?utm_source=chatgpt.com)


Las principales medidas son:

| Medida                       | Descripción                                                                                     |
| ---------------------------- | ----------------------------------------------------------------------------------------------- |
| Imágenes mínimas             | Incluyen únicamente los componentes necesarios para ejecutar la aplicación.                     |
| Menos dependencias           | Reducen la cantidad de bibliotecas y paquetes que podrían contener vulnerabilidades.            |
| Ejecución sin privilegios    | Utilizan usuarios sin privilegios de administrador (`root`) siempre que sea posible.            |
| Eliminación de herramientas  | Pueden prescindir de shells, compiladores y gestores de paquetes en las variantes de ejecución. |
| Actualizaciones de seguridad | Incorporan correcciones de vulnerabilidades conocidas.                                          |
| Firma digital                | Permite verificar la autenticidad e integridad de las imágenes.                                 |
| SBOM                         | Proporcionan un inventario de los componentes software incluidos.                               |


Para utilizar estas imágenes es necesario autenticarse en el registro:

```bash
docker login dhi.io
```

Una vez autenticado, se puede descargar una imagen y ejecutarla

```bash
docker pull dhi.io/nginx:1-alpine
docker run -d -p 8080:8080 dhi.io/nginx:1-alpine
```

La referencia de una imagen sigue la estructura habitual:

```bash
dhi.io/nombre-imagen:etiqueta
```

Por ejemplo:

- `dhi.io`: registro de imágenes.
- `nginx`: nombre de la imagen.
- `1-alpine`: etiqueta o versión.

### 3.3. Variantes de Docker Hardened Images

Las imágenes Hardened pueden ofrecer diferentes variantes según su finalidad.

| Variante | Finalidad                                                                                              |
| -------- | ------------------------------------------------------------------------------------------------------ |
| Runtime  | Ejecución de aplicaciones. Incluye los componentes necesarios para funcionar.                          |
| Dev      | Desarrollo y construcción. Puede incluir herramientas adicionales, como shells y gestores de paquetes. |
| FIPS     | Variante preparada para cumplir requisitos relacionados con el estándar criptográfico FIPS.            |
| STIG     | Variante preparada para configuraciones de seguridad basadas en las guías STIG.                        |

Las variantes FIPS y STIG requieren una **suscripción** Docker Hardened Images Select o Enterprise.

>Importante: no todas las variantes están disponibles con todas las imágenes. 

## 4. Buenas prácticas de seguridad en imágenes Docker

Para reducir los riesgos de seguridad al trabajar con contenedores se recomienda:

1. Utilizar imágenes de confianza: priorizar imágenes oficiales, de publicadores verificados o imágenes Hardened.
2. Mantener las imágenes actualizadas: utilizar versiones que incorporen las últimas correcciones de seguridad.
3. Reducir el tamaño de las imágenes: evitar instalar herramientas y dependencias que no sean necesarias.
4. Evitar ejecutar como root: utilizar usuarios sin privilegios administrativos.
5. Analizar las imágenes: utilizar herramientas como Docker Scout para detectar vulnerabilidades.
6. Controlar las dependencias: comprobar las bibliotecas y paquetes incluidos en las imágenes.
7. Proteger los secretos: no incluir contraseñas, claves privadas ni tokens dentro de las imágenes.
8. Limitar los permisos: conceder a los contenedores únicamente los recursos y privilegios que necesitan.
9. Consultar los logs: revisar los registros para detectar errores y comportamientos sospechosos.
10. Verificar la procedencia: utilizar imágenes firmadas y comprobar su integridad cuando sea posible.

Hay que tener en cuenta que la seguridad de un contenedor no depende únicamente de la imagen, sino también de la configuración del entorno y de las medidas de seguridad aplicadas durante su ejecución.

Debemos aplicar medidas para:

- Analizar: detectar vulnerabilidades en una imagen mediante Docker Scout.
- Proteger: utilizar imágenes Hardened y aplicar medidas de seguridad.
- Supervisar: consultar los logs para conocer lo que sucede durante la ejecución de los contenedores.
