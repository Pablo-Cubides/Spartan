# =============================================================================
# SPARTAN MARKET API - DOCKER DEPLOYMENT
# =============================================================================

## 🐳 Configuración de Docker

### 📋 Prerrequisitos

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB RAM mínimo
- 10GB espacio en disco

### 🚀 Despliegue Rápido

```bash
# 1. Clonar repositorio
git clone <repository-url>
cd webpage/backend

# 2. Configurar variables de entorno
cp env.example .env
# Editar .env con tus configuraciones

# 3. Desplegar con Docker Compose
docker-compose up -d

# 4. Verificar servicios
docker-compose ps
```

### 🔧 Configuraciones Disponibles

#### **Despliegue Básico**
```bash
# Solo API, Database y Redis
docker-compose up -d
```

#### **Despliegue con Monitoreo**
```bash
# Incluye Prometheus y Grafana
docker-compose --profile monitoring up -d
```

#### **Despliegue de Producción**
```bash
# Incluye Nginx como reverse proxy
docker-compose --profile production up -d
```

### 📊 Servicios Disponibles

| Servicio | Puerto | Descripción |
|----------|--------|-------------|
| API | 8000 | FastAPI Application |
| Database | 5432 | PostgreSQL Database |
| Redis | 6379 | Cache & Sessions |
| Prometheus | 9090 | Metrics Collection |
| Grafana | 3000 | Metrics Dashboard |
| Nginx | 80/443 | Reverse Proxy |

### 🔍 Health Checks

```bash
# Verificar estado de servicios
docker-compose ps

# Ver logs de la API
docker-compose logs api

# Ver logs de base de datos
docker-compose logs db

# Ver logs de Redis
docker-compose logs redis
```

### 📈 Monitoreo

#### **Prometheus**
- URL: http://localhost:9090
- Métricas: http://localhost:8000/metrics

#### **Grafana**
- URL: http://localhost:3000
- Usuario: admin
- Contraseña: admin

### 🔧 Comandos Útiles

```bash
# Reconstruir imagen
docker-compose build --no-cache

# Reiniciar servicios
docker-compose restart

# Ver logs en tiempo real
docker-compose logs -f

# Ejecutar migraciones
docker-compose exec api alembic upgrade head

# Ejecutar tests
docker-compose exec api python test_api_endpoints.py

# Acceder a base de datos
docker-compose exec db psql -U spartan_user -d spartan_market

# Acceder a Redis
docker-compose exec redis redis-cli
```

### 🛠️ Troubleshooting

#### **Problema: API no inicia**
```bash
# Verificar logs
docker-compose logs api

# Verificar variables de entorno
docker-compose exec api env | grep DATABASE_URL
```

#### **Problema: Base de datos no conecta**
```bash
# Verificar estado de PostgreSQL
docker-compose exec db pg_isready

# Verificar logs de base de datos
docker-compose logs db
```

#### **Problema: Redis no conecta**
```bash
# Verificar estado de Redis
docker-compose exec redis redis-cli ping

# Verificar logs de Redis
docker-compose logs redis
```

### 🔒 Seguridad

#### **Variables de Entorno Críticas**
```bash
# Base de datos
POSTGRES_USER=spartan_user
POSTGRES_PASSWORD=strong_password_here
POSTGRES_DB=spartan_market

# API Keys
SENTRY_DSN=your_sentry_dsn
BREVO_API_KEY=your_brevo_key
FIREBASE_PROJECT_ID=your_firebase_project

# JWT Secret
JWT_SECRET_KEY=your_jwt_secret
```

#### **Firewall**
```bash
# Solo exponer puertos necesarios
# 8000 - API
# 5432 - Database (solo local)
# 6379 - Redis (solo local)
```

### 📝 Logs y Debugging

#### **Estructura de Logs**
```
/app/logs/
├── api.log          # Logs de la aplicación
├── access.log       # Logs de acceso
├── error.log        # Logs de errores
└── metrics.log      # Logs de métricas
```

#### **Niveles de Log**
- `DEBUG`: Información detallada
- `INFO`: Información general
- `WARNING`: Advertencias
- `ERROR`: Errores
- `CRITICAL`: Errores críticos

### 🚀 Producción

#### **Optimizaciones**
```bash
# Usar múltiples workers
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]

# Configurar límites de memoria
docker-compose up -d --scale api=3
```

#### **Backup**
```bash
# Backup de base de datos
docker-compose exec db pg_dump -U spartan_user spartan_market > backup.sql

# Restaurar backup
docker-compose exec -T db psql -U spartan_user spartan_market < backup.sql
```

### 📚 Recursos Adicionales

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Documentation](https://redis.io/documentation) 