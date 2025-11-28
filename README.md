# **Despliegue de un Servidor de Minecraft (Java Edition) con Docker**

## **1. Introducción**

Este es el procedimiento para desplegar, configurar y administrar un servidor de Minecraft utilizando **Docker Compose**. La configuración incluye persistencia de datos, limitación de recursos de hardware (RAM) y gestión de privilegios de administrador. Esta práctica se basa en la imagen oficial mantenida por la comunidad ([itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server)).

## **2. Requisitos Previos**

* **Docker Engine** instalado y en ejecución.
* **Docker Compose** (incluido en las versiones modernas de Docker Desktop/Engine).
* Conexión a internet para la descarga de la imagen base.

## **3. Configuración del Servicio**

El núcleo de la configuración reside en el archivo docker-compose.yml. A continuación se presenta la configuración unificada que incluye la aceptación del EULA (Acuerdo de Licencia de Usuario Final) y la limitación de memoria RAM para optimizar el rendimiento del host.

### **3.1. Creación del Fichero**

Cree un directorio para el proyecto y dentro genere un archivo llamado docker-compose.yml con el siguiente contenido:

```yaml
services:
  minecraft-server:
    image: itzg/minecraft-server
    container_name: mc-server
    ports:
      - "25565:25565"
    environment:
      # Aceptación obligatoria del Acuerdo de Licencia de Usuario Final
      - EULA=TRUE
      # Límite de memoria asignada a la JVM (Java Virtual Machine)
      - MEMORY=2G
    volumes:
      # Persistencia de datos (mundos, configuraciones, inventarios)
      - mc-data:/data
    restart: unless-stopped

volumes:
  mc-data:
```

### **3.2. Desglose de Parámetros Técnicos**

| Parámetro | Descripción |
| :---- | :---- |
| **image** | Utiliza itzg/minecraft-server, que automatiza la instalación de Java y los binarios del servidor. |
| **ports** | Mapea el puerto 25565 del contenedor al host, permitiendo conexiones externas. |
| **EULA=TRUE** | Variable de entorno crítica. Sin ella, el servidor abortará el inicio inmediatamente. |
| **MEMORY=2G** | Gestiona el *Heap Size* de Java. Evita que el proceso consuma toda la RAM del sistema anfitrión, previniendo errores de "Out of Memory". |
| **volumes** | El volumen mc-data garantiza que, si el contenedor es eliminado, los datos del mundo persistan en el sistema. |

---

## **4. Ejecución y Despliegue**

Para iniciar el servicio, abra una terminal en el directorio del archivo y ejecute el siguiente comando. El parámetro `-d` permite la ejecución en segundo plano (modo *detached*).

```bash
docker compose up -d
```

### **Verificación del Estado**

Puede supervisar el proceso de inicialización y generación del mundo mediante la lectura de logs:

```bash
docker logs -f mc-server
```

*El servidor estará operativo cuando aparezca el mensaje: Done (X.Xs)! For help, type "help".*

---

## **5. Gestión de Permisos (Operador/Admin)**

Dado que el servidor se ejecuta en un entorno aislado, la consola estándar no es accesible directamente. Para otorgar permisos de administrador (OP) a un usuario, utilizaremos la herramienta rcon-cli inyectada a través de docker exec. Con dicha modifiación podríamos cambiar el modo de juego a creativo, teletransportarte, cambiar la hora del día o expulsar jugadores.

Ejecute el siguiente comando en su terminal, sustituyendo `<NOMBRE_USUARIO>` por el nickname exacto del jugador:

```bash
docker exec mc-server rcon-cli op <NOMBRE_USUARIO>
```

**Resultado esperado:** El servidor confirmará la acción con el mensaje Made `<NOMBRE_USUARIO>` a server operator.

---

## **6. Ciclo de Vida y Mantenimiento**

A continuación se listan los comandos esenciales para la administración del ciclo de vida del contenedor:

* **Detener el servidor (Graceful shutdown):**

  ```bash
  docker compose down
  ```

* **Reiniciar el servidor (Aplicar cambios de configuración):**

  ```bash
  docker compose up -d
  ```

* **Acceso interactivo a la consola del servidor (RCON):**

  ```bash
  docker exec -it mc-server rcon-cli
  ```
