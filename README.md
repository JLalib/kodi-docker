# 🎬 Kodi Docker - Servidor de Media Center Headless

**Descripción**: Servidor de medios completo y autohospedado sin necesidad de pantalla, accesible mediante interfaz web y API.

**Kodi Docker** es la implementación en contenedores de **Kodi Headless**, un centro de medios open source que organiza y reproduce películas, series, música y fotos. La versión "headless" permite ejecutar Kodi en un servidor sin monitor ni periféricos, proporcionando un acceso remoto total a través de noVNC, VNC y una API REST, ideal para centralizar tu biblioteca multimedia en un servidor doméstico o NAS.

---

## ✨ Características principales

- **Interfaz Web (noVNC)**: Acceso completo a la interfaz gráfica de Kodi directamente desde cualquier navegador web sin instalar clientes adicionales.
- **VNC Nativo**: Soporte para clientes VNC tradicionales en el puerto 5900 para un control remoto fluido.
- **API REST**: Permite controlar Kodi mediante solicitudes HTTP, facilitando la integración con aplicaciones externas y scripts.
- **Sincronización vía MySQL**: Capacidad de compartir la base de datos entre múltiples dispositivos Kodi para tener una biblioteca unificada y sincronizada.
- **Gestión Multimedia Avanzada**: Organización automática de películas, series y música con descarga de metadatos, carátulas y descripciones.
- **Soporte Multiformato**: Reproducción nativa de decenas de formatos populares como MKV, MP4, FLAC, MP3 y más.
- **Ecosistema de Add-ons**: Extensible mediante complementos para subtítulos, scrapers y skins personalizadas.
- **EventServer**: Comunicación bidireccional optimizada para el uso de mandos a distancia y aplicaciones de control.

---

## ⚠️ Requisitos previos

- **Docker** y **Docker Compose** (versión 2.x o superior).
- **2-4 GB RAM** (dependiendo del volumen de la biblioteca).
- **10+ GB espacio en disco** para la base de datos y caché de metadatos.
- **Puertos disponibles**:
  - `8000`: Interfaz Web noVNC.
  - `5900`: Acceso VNC.
  - `8080`: API HTTP.
  - `9090`: WebSockets.
  - `9777/UDP`: EventServer.
- **Base de datos**: MySQL 5.7+ o MariaDB (recomendado para sincronización multi-dispositivo).

---

## ⚙️ Configuración con Docker Compose

### 1. Clona este repositorio y accede al directorio:
```bash
git clone https://github.com/JLalib/kodi-docker.git
cd kodi-docker
```

### 2. Archivo `docker-compose.yml`
Crea el archivo `docker-compose.yml` con la siguiente configuración (incluyendo MySQL para sincronización):

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: kodi-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: kodivideodb110
      MYSQL_USER: kodi
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysql-data:/var/lib/mysql

  kodi:
    image: fhriley/kodi-headless-novnc:latest
    container_name: kodi-headless
    restart: unless-stopped
    depends_on:
      - mysql
    ports:
      - "8000:8000" # Web noVNC
      - "5900:5900" # VNC
      - "8080:8080" # API HTTP
      - "9090:9090" # WebSocket
      - "9777:9777/udp" # EventServer
    volumes:
      - ./kodi-data:/data
      - /mnt/media:/media # Cambia /mnt/media por la ruta de tus archivos
    environment:
      - KODI_DB_HOST=mysql
      - KODI_DB_PORT=3306
      - KODI_DB_USER=kodi
      - KODI_DB_PASS=${DB_PASSWORD}
      - KODI_DB_NAME=kodivideodb110
      - TZ=Europe/Madrid
      - KODI_UMASK=002

volumes:
  mysql-data:
```

### 3. Configura las variables de entorno
Crea un archivo `.env` en la raíz:
```env
DB_ROOT_PASSWORD=tu_password_root_seguro
DB_PASSWORD=tu_password_kodi_seguro
```

### 4. Despliega la aplicación
```bash
docker compose up -d
```

### 5. Accede a Kodi
Abre tu navegador en:
🔗 [http://localhost:8000](http://localhost:8000)

---

## 🛠️ Primeros pasos

### 1. Configurar fuentes de medios
1. Accede a la interfaz web en el puerto 8000.
2. Ve a **Settings $\rightarrow$ Media $\rightarrow$ Libraries $\rightarrow$ Videos**.
3. Haz clic en **"Add videos..."** y selecciona la ruta `/media` (donde montaste tus archivos).
4. Define el contenido como "Movies" o "TV Shows" para que Kodi descargue los metadatos automáticamente.

### 2. Sincronización MySQL
Si has usado la configuración de MySQL, cualquier otro dispositivo Kodi en tu red puede conectarse a este servidor de base de datos para compartir la misma biblioteca y el estado de "visto/no visto".

### 3. Instalación de Add-ons
Navega a **Add-ons $\rightarrow$ Install from repository** para añadir funcionalidades como subtítulos automáticos o nuevos scrapers de información.

---

## 🔒 Seguridad y recomendaciones

### 1. Optimización del Escaneo (Path Substitution)
Para acelerar drásticamente el escaneo de bibliotecas grandes en red, puedes usar el mapeo de rutas locales en el archivo `advancedsettings.xml` de Kodi, sustituyendo la ruta SMB por la ruta local `/media`.

### 2. Acceso Remoto Seguro (HTTPS)
Para acceder a tu centro de medios desde fuera de casa, utiliza **Caddy** como reverse proxy:

**`Caddyfile`**:
```
kodi.tudominio.com {
    reverse_proxy localhost:8000
}
```

### 3. Montaje de Medios
Asegúrate de que el usuario que ejecuta Docker tenga permisos de lectura sobre la carpeta de medios en el host para evitar errores de "Permiso denegado" en Kodi.

---

## 📂 Estructura del proyecto

```
./
├── docker-compose.yml    # Orquestación de Kodi y MySQL
├── .env                  # Credenciales de base de datos
├── kodi-data/            # Configuración y perfiles de Kodi
└── README-Kodi-Docker.md  # Este archivo
```

---

## 🔄 Actualización y mantenimiento

### Actualizar Kodi
```bash
docker compose pull
docker compose up -d
```

### Comandos útiles
| Comando                          | Descripción                                  |
|----------------------------------|----------------------------------------------|
| `docker compose logs -f kodi`     | Ver logs del servidor de medios.              |
| `docker exec -it kodi-headless bash` | Entrar en la terminal del contenedor.      |
| `docker compose restart kodi`     | Reiniciar el servicio de Kodi.                |

---

## 📊 Comparativa con alternativas

| Característica               | Kodi Docker           | Netflix / Spotify  | Plex / Emby       |
|------------------------------|-----------------------|-------------------|-------------------|
| **Autohospedado**            | ✅ Sí                 | ❌ No             | ✅ Sí             |
| **Control de Datos**         | ✅ Total              | ❌ Nulo           | ✅ Total           |
| **Costo**                   | ✅ Gratis / Open Source| ❌ Suscripción     | ⚠️ Freemium       |
| **Sincronización DB**        | ✅ MySQL Compartida    | ✅ Nube           | ✅ Nube/Local     |
| **Privacidad**               | ✅ Máxima             | ❌ Telemetría     | ⚠️ Parcial         |

---

## 📚 Referencias

- [Kodi Official Website](https://kodi.tv/)
- [Kodi Headless noVNC - GitHub](https://github.com/fhriley/kodi-headless-novnc)
- [Kodi Wiki](https://kodi.wiki/)
- [Docker Hub Image](https://hub.docker.com/r/fhriley/kodi-headless-novnc)

---
