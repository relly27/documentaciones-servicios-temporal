# Requerimientos del Sistema y Despliegue

Este documento describe tanto los requisitos del sistema como el proceso de despliegue para el servicio.

---

## 1. Requisitos del Sistema

Esta sección detalla las especificaciones de hardware y software necesarias para un funcionamiento estable del servicio.

### 1.1. Arquitectura Recomendada

La arquitectura recomendada consta de dos componentes principales:

- **Servidor de Aplicación:** Responsable de ejecutar la aplicación Node.js, gestionar las solicitudes API y la lógica de negocio.
- **Servidor de Base de Datos:** Dedicado a ejecutar la base de datos PostgreSQL, asegurando la persistencia y gestión de los datos.

### 1.2. Especificaciones de Hardware

| Entorno            | CPU       | RAM  | Almacenamiento | Notas                                |
| ------------------ | --------- | ---- | -------------- | ------------------------------------ |
| **Desarrollo**     | 2 núcleos | 4 GB | 50 GB SSD      | Mínimo para un rendimiento aceptable.  |
| **Producción**     | 4 núcleos | 8 GB | 100 GB SSD     | Incluye espacio para copias de seguridad. |

### 1.3. Especificaciones de Software

| Componente     | Servidor          | Software       | Versión Recomendada | Notas                                  |
| -------------- | ----------------- | -------------- | ------------------- | -------------------------------------- |
| **Sistema Operativo** | Ambos             | Ubuntu         | 20.04 LTS o superior | Una distribución de Linux estable.     |
| **Aplicación** | Servidor de App   | Node.js        | 24.x LTS            | O una versión LTS más reciente.        |
| **Gestor de Paquetes** | Servidor de App   | npm            | 11.x                | Se instala junto con Node.js.          |
| **Gestor de Procesos** | Servidor de App   | PM2            | 6.x                 | Para gestionar el ciclo de vida de la app. |
| **Base de Datos**    | Servidor de BD    | PostgreSQL     | 17.x                | O una versión estable más reciente.    |
| **Proxy Inverso**    | Servidor de App   | Nginx          | Última estable      | Para gestionar el tráfico y SSL.       |

---

## 2. Requisitos de Despliegue del Servicio

Esta sección explica cómo desplegar la aplicación en un entorno de producción.

### 2.1. Proceso de Despliegue en Producción

El despliegue en producción implica los siguientes pasos clave:

1.  **Configuración del Servidor:** Preparar el servidor con todo el software necesario (Node.js, PostgreSQL, Nginx, PM2).
2.  **Clonación del Repositorio:** Descargar el código fuente desde el repositorio de Git.
3.  **Instalación de Dependencias:** Instalar todas las dependencias del proyecto utilizando `npm install`.
4.  **Configuración del Entorno:** Crear y configurar el archivo `.env` con las variables de entorno para producción (credenciales de base de datos, secretos JWT, etc.).
5.  **Ejecución de Migraciones:** Aplicar las migraciones de la base de datos para configurar el esquema.
6.  **Inicio de la Aplicación con PM2:** Iniciar la aplicación utilizando un gestor de procesos como PM2 para asegurar que se ejecute de forma continua.
7.  **Configuración de Nginx:** Configurar Nginx como un proxy inverso para dirigir el tráfico a la aplicación Node.js y gestionar los certificados SSL.

### 2.2. Configuración de Nginx

Nginx actúa como proxy inverso, redirigiendo las solicitudes al puerto donde se ejecuta la aplicación Node.js.

**Instalación en Ubuntu:**
```bash
sudo apt update && sudo apt upgrade
sudo apt install nginx
sudo ufw allow 'Nginx Full' # Habilitar si UFW está activo
```

**Ejemplo de Configuración (`/etc/nginx/sites-available/your-domain`):**
```nginx
server {
    listen 80;
    server_name su-dominio.com www.su-dominio.com;

    location / {
        proxy_pass http://localhost:3000; # Asegúrese de que el puerto coincida
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Para activar la configuración, cree un enlace simbólico:
```bash
sudo ln -s /etc/nginx/sites-available/your-domain /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx
```

### 2.3. Configuración de PM2

PM2 es un gestor de procesos que mantiene la aplicación en ejecución de forma continua.

**Instalación y Configuración:**
```bash
npm install pm2 -g
pm2 startup
# Siga las instrucciones para configurar el inicio automático
```

**Archivo de Ecosistema (`ecosystem.config.js`):**
Se recomienda usar un archivo de ecosistema para definir la configuración de la aplicación.
```javascript
module.exports = {
  apps : [{
    name        : 'api-service',
    script      : 'src/app.js',
    instances   : 'max', // O un número específico de instancias
    exec_mode   : 'cluster',
    autorestart : true,
    watch       : false,
    max_memory_restart: '1G',
    env_production: {
      NODE_ENV: 'production',
      // Aquí se pueden añadir más variables de entorno
    }
  }]
};
```

**Gestión de la Aplicación:**
```bash
pm2 start ecosystem.config.js --env production # Iniciar la aplicación
pm2 list                                     # Listar procesos
pm2 stop api-service                         # Detener la aplicación
pm2 restart api-service                      # Reiniciar la aplicación
pm2 logs api-service                         # Ver logs
```

---

## 3. Guía de Instalación y Configuración

Esta sección proporciona una guía detallada para configurar el entorno de desarrollo y producción.

### 3.1. Ubicación del Código Fuente

El código fuente del proyecto debe ser ubicado en el directorio `/var/www/`. Esto se puede lograr clonando el repositorio directamente en esta ubicación:
```bash
git clone <url-del-repositorio> /var/www/nombre-del-proyecto
```

### 3.2. Instalación de Dependencias

Todas las dependencias del proyecto se gestionan a través de npm. Para instalarlas, ejecute el siguiente comando desde el directorio raíz del proyecto:
```bash
npm install
```

### 3.3. Configuración del Archivo de Entorno

El proyecto requiere un archivo `.env` en el directorio raíz para gestionar las variables de entorno. Se proporciona un archivo de ejemplo llamado `env.example` que puede ser copiado y modificado:
```bash
cp env.example .env
```
Asegúrese de configurar las variables en el archivo `.env` según las necesidades de su entorno (por ejemplo, credenciales de la base de datos, secretos JWT).

### 3.4. Comandos Útiles del Proyecto

El archivo `package.json` incluye varios scripts útiles para la gestión del proyecto:

- **`npm start`**: Inicia la aplicación en modo de producción.
- **`npm run dev`**: Inicia la aplicación en modo de desarrollo con reinicio automático (utiliza `nodemon`).
- **`npm test`**: Ejecuta el conjunto de pruebas del proyecto (utiliza `jest`).
- **`npm run swagger-autogen`**: Genera o actualiza la documentación de la API (utiliza `swagger-autogen`).
- **`npm run migrate`**: Aplica las migraciones de la base de datos.
- **`npm run db:dump`**: Realiza un volcado de la base de datos (schema-only).

### 3.5. Script de Configuración del Servidor

El proyecto incluye un script de bash llamado `setup_server.sh` que automatiza la configuración del servidor y del proyecto. Este script está diseñado para ser ejecutado en un entorno compatible (por ejemplo, Ubuntu) y facilita la instalación de dependencias y la configuración inicial.

### 3.6. Configuración de Docker

El proyecto también incluye un archivo `Dockerfile` y `docker-compose.yml` para facilitar la contenedorización. Esto permite un despliegue y desarrollo consistentes a través de diferentes entornos. Para levantar el entorno con Docker, puede utilizar:
```bash
docker-compose up --build
```
