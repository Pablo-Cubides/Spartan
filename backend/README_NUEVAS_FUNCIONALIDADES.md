# 🚀 Nuevas Funcionalidades - Spartan Market API

## 📋 Resumen de Implementaciones

### ✅ 1. Prefijo `/api/v1` y Documentación OpenAPI

- **Rutas actualizadas**: Todas las rutas ahora comienzan con `/api/v1/`
- **Documentación disponible**:
  - Swagger UI: `/api/v1/docs`
  - Redoc UI: `/api/v1/redoc`
  - OpenAPI JSON: `/api/v1/openapi.json`

### ✅ 2. Endpoints de Health Check

- **`/api/v1/healthz`**: Verifica que la aplicación esté ejecutándose
- **`/api/v1/readyz`**: Verifica PostgreSQL, Redis y R2
  - Retorna HTTP 200 si todos los servicios están OK
  - Retorna HTTP 503 si algún servicio falla

### ✅ 3. Tabla `credit_packages`

Nueva tabla con los siguientes campos:
- `id`: Primary Key
- `name`: Nombre del paquete
- `credits`: Número de créditos
- `price`: Precio en moneda local
- `created_at`: Timestamp de creación
- `is_active`: Si el paquete está disponible

**Paquetes iniciales**:
- 100 créditos - $5.00
- 500 créditos - $20.00
- 1000 créditos - $35.00
- 2000 créditos - $60.00

### ✅ 4. URLs Firmadas para Avatares (R2)

**Endpoints implementados**:
- `POST /api/v1/users/avatar/presign-upload` → Genera URL para subir avatar
- `GET /api/v1/users/avatar/presign-download/{avatar_id}` → Genera URL para descargar avatar
- `POST /api/v1/users/avatar/confirm-upload` → Confirma subida exitosa
- `DELETE /api/v1/users/avatar/{avatar_id}` → Elimina avatar

**Características**:
- URLs válidas por 1 hora
- Compatible con Cloudflare R2
- Soporte para múltiples formatos de imagen

### ✅ 5. Script de Backup Diario

**Ubicación**: `scripts/backup_daily.py`

**Funcionalidades**:
- Backup de PostgreSQL usando `pg_dump`
- Backup de Redis (RDB snapshot)
- Compresión automática (.tar.gz)
- Subida a Cloudflare R2
- Limpieza automática (7 días)

### ✅ 6. Panel de Administración

**Frontend**: `/admin` (Next.js/React)
**Backend**: Endpoints `/api/v1/admin/*`

**Características**:
- SSO con Google (Firebase)
- Listado de usuarios con perfiles
- Historial de compras
- Gestión de paquetes de créditos
- Estadísticas generales

## 🔧 Configuración Requerida

### Variables de Entorno

```bash
# Cloudflare R2
R2_ENDPOINT_URL=https://your-account-id.r2.cloudflarestorage.com
R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET_NAME=spartan-avatars

# Base de Datos
DB_HOST=localhost
DB_PORT=5432
DB_NAME=spartan_db
DB_USER=postgres
DB_PASSWORD=password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# Firebase (para el panel admin)
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_PRIVATE_KEY=your-private-key
FIREBASE_CLIENT_EMAIL=your-client-email
```

### Instalación de Dependencias

```bash
pip install -r requirements.txt
```

### Migraciones de Base de Datos

```bash
# Crear migración inicial
alembic revision --autogenerate -m "Add credit_packages and purchases tables"

# Aplicar migraciones
alembic upgrade head
```

### Inicializar Paquetes de Créditos

```bash
python scripts/init_credit_packages.py
```

### Configurar Backup Diario

```bash
# Hacer ejecutable el script
chmod +x scripts/backup_daily.py

# Ejecutar manualmente para probar
python scripts/backup_daily.py

# Configurar cron job (ejecutar diariamente a las 2 AM)
0 2 * * * /path/to/backend/scripts/backup_daily.py
```

## 🚀 Endpoints Principales

### Health Checks
```bash
curl http://localhost:8000/api/v1/healthz
curl http://localhost:8000/api/v1/readyz
```

### Paquetes de Créditos
```bash
# Listar paquetes
curl http://localhost:8000/api/v1/credits/packages

# Comprar créditos
curl -X POST http://localhost:8000/api/v1/credits/buy \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"package_id": 1}'
```

### Avatares (URLs Firmadas)
```bash
# Generar URL de subida
curl -X POST http://localhost:8000/api/v1/users/avatar/presign-upload \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file_extension": ".jpg"}'

# Generar URL de descarga
curl http://localhost:8000/api/v1/users/avatar/presign-download/avatar_id \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Panel de Administración
```bash
# Obtener usuarios (solo admin)
curl http://localhost:8000/api/v1/admin/users \
  -H "Authorization: Bearer ADMIN_TOKEN"

# Obtener compras (solo admin)
curl http://localhost:8000/api/v1/admin/purchases \
  -H "Authorization: Bearer ADMIN_TOKEN"

# Obtener estadísticas (solo admin)
curl http://localhost:8000/api/v1/admin/stats \
  -H "Authorization: Bearer ADMIN_TOKEN"
```

## 📊 Panel de Administración

**URL**: `http://localhost:3000/admin`

**Acceso**:
- Solo usuarios con email `@spartan.com`
- SSO con Google obligatorio
- Interfaz de solo lectura

**Funcionalidades**:
- Dashboard con estadísticas
- Listado de usuarios registrados
- Historial de compras
- Gestión de paquetes de créditos

## 🔒 Seguridad

- Todas las rutas protegidas con autenticación Firebase
- Panel admin con verificación de permisos
- URLs firmadas con expiración automática
- Rate limiting en endpoints críticos

## 📈 Monitoreo

- Métricas Prometheus disponibles
- Health checks automáticos
- Logs estructurados
- Integración con Sentry

## 🐛 Troubleshooting

### Error de conexión a R2
```bash
# Verificar variables de entorno
echo $R2_ENDPOINT_URL
echo $R2_ACCESS_KEY_ID
echo $R2_SECRET_ACCESS_KEY
```

### Error de migración
```bash
# Verificar conexión a PostgreSQL
psql -h localhost -U postgres -d spartan_db

# Recrear migraciones
alembic stamp head
alembic revision --autogenerate -m "Recreate tables"
alembic upgrade head
```

### Error de backup
```bash
# Verificar permisos
ls -la scripts/backup_daily.py

# Ejecutar con debug
python -u scripts/backup_daily.py
```

## 📝 Notas de Desarrollo

- Todas las nuevas funcionalidades están documentadas en OpenAPI
- Los scripts incluyen logging detallado
- El panel admin es responsive y accesible
- Los backups se comprimen automáticamente
- Las URLs firmadas expiran en 1 hora por seguridad 