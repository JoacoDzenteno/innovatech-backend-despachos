# Innovatech Backend Despachos

Repositorio del microservicio de despachos de la tienda Innovatech, desarrollado con Spring Boot y Java.

## Tecnologías
- Java 17
- Spring Boot
- Maven
- MySQL 8.0
- Docker

## Estructura del proyecto
- `Springboot-API-REST-DESPACHO/` — código fuente del microservicio
- `Dockerfile` — imagen multi-stage para producción
- `docker-compose.yml` — stack del servicio backend + base de datos
- `.github/workflows/deploy.yml` — pipeline CI/CD

## Contenedorización

### Dockerfile
Se utilizó un Dockerfile multi-stage:
- **Stage 1 (builder):** compila el proyecto con Maven omitiendo tests
- **Stage 2 (production):** copia el `.jar` generado, crea usuario sin privilegios root y expone el puerto 8080

### Docker Compose
Levanta tres servicios:
- `db-innovatech` — base de datos MySQL 8.0 con volumen de persistencia
- `back-despachos` — microservicio de despachos conectado a la BD
- `back-ventas` — microservicio de ventas conectado a la BD

## Variables de entorno

### Base de datos (db-innovatech)
| Variable | Valor | Descripción |
|----------|-------|-------------|
| `MYSQL_ROOT_PASSWORD` | root | Contraseña del usuario root de MySQL |
| `MYSQL_DATABASE` | innovadb | Nombre de la base de datos creada al iniciar |

### Backend Despachos
| Variable | Valor | Descripción |
|----------|-------|-------------|
| `SPRING_DATASOURCE_URL` | jdbc:mysql://db-innovatech:3306/innovadb?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true | URL de conexión a MySQL dentro de la red Docker |
| `SPRING_DATASOURCE_USERNAME` | root | Usuario de la base de datos |
| `SPRING_DATASOURCE_PASSWORD` | root | Contraseña de la base de datos |

## Puertos expuestos

| Servicio | Puerto host | Puerto contenedor |
|----------|-------------|-------------------|
| MySQL | 3315 | 3306 |
| Backend Despachos | 8091 | 8080 |
| Backend Ventas | 8092 | 8080 |

## Dependencias entre servicios
Los backends dependen de la base de datos mediante `depends_on`, lo que garantiza que MySQL esté levantado antes de que los microservicios intenten conectarse.

## Persistencia de datos

Se utilizó un **named volume** (`db_data_innovatech`) para la base de datos MySQL. Este volumen asegura que toda la información crítica almacenada en la base de datos, como registros de despachos y datos operativos, no se pierda al reiniciar o detener los contenedores.

### ¿Por qué named volume y no bind mount?

| Criterio | Named Volume | Bind Mount |
|----------|-------------|------------|
| Portabilidad | ✅ Gestionado por Docker, funciona en cualquier entorno | ❌ Depende de la ruta del host |
| Seguridad | ✅ Aislado del sistema de archivos del host | ❌ Acceso directo al sistema de archivos |
| Persistencia | ✅ Los datos sobreviven al reinicio de contenedores | ✅ También persiste |
| Facilidad | ✅ No requiere configurar rutas | ❌ Requiere gestionar permisos y rutas |

Se eligió **named volume** porque:
1. El entorno de despliegue es un servidor compartido donde no se tiene control total sobre las rutas del sistema de archivos del host.
2. Docker gestiona completamente el ciclo de vida del volumen, reduciendo errores de configuración.
3. Garantiza que los datos de la base de datos persistan independientemente de reinicios, actualizaciones o recreación de contenedores.
4. Es la práctica recomendada para bases de datos en entornos de producción contenerizados.

## Cómo ejecutar

### Con Docker Compose
```bash
docker-compose up -d 
