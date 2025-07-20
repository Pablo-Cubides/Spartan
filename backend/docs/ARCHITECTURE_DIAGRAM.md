# Diagrama de Arquitectura - Módulo de Perfiles de Usuario

## 🏗️ **Arquitectura General**

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Next.js)                      │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │   Profile   │  │   Avatar    │  │  Privacy    │           │
│  │   Editor    │  │  Selector   │  │  Settings   │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ Public      │  │  Purchase   │  │  Complete   │           │
│  │ Profile     │  │  History    │  │   Modal     │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND (FastAPI)                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │   User      │  │   Upload    │  │   Privacy   │           │
│  │  Routes     │  │  Handler    │  │   Logic     │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │   User      │  │  Storage    │  │  Firebase   │           │
│  │ Services    │  │ Interface   │  │   Auth      │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DATABASE (PostgreSQL)                   │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ users       │  │users_profiles│  │user_privacy │           │
│  │             │  │             │  │ _settings   │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │user_purchases│  │  Indexes    │  │ Constraints │           │
│  │             │  │             │  │             │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL SERVICES                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │  Firebase   │  │ MercadoPago │  │   Storage   │           │
│  │    Auth     │  │             │  │    (S3)     │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

## 🔄 **Flujo de Datos**

### **1. Registro y Autenticación**

```mermaid
sequenceDiagram
    participant U as Usuario
    participant F as Frontend
    participant B as Backend
    participant FA as Firebase Auth
    participant DB as Database

    U->>F: Registro/Login
    F->>FA: Autenticación
    FA->>F: Token JWT
    F->>B: Validar Token
    B->>FA: Verificar Token
    FA->>B: Información del Usuario
    B->>DB: Buscar Perfil
    DB->>B: Perfil (si existe)
    B->>F: Respuesta + Estado del Perfil
    F->>U: Redirigir según Estado
```

### **2. Completar Perfil**

```mermaid
sequenceDiagram
    participant U as Usuario
    participant F as Frontend
    participant B as Backend
    participant DB as Database

    U->>F: Llenar Formulario
    F->>B: POST /profile/complete
    B->>B: Validar Datos
    B->>B: Verificar Alias Único
    B->>DB: Crear Perfil
    DB->>B: Confirmación
    B->>F: Perfil Creado
    F->>U: Redirigir a Dashboard
```

### **3. Subida de Avatar**

```mermaid
sequenceDiagram
    participant U as Usuario
    participant F as Frontend
    participant B as Backend
    participant S as Storage
    participant DB as Database

    U->>F: Seleccionar Imagen
    F->>B: POST /profile/avatar
    B->>B: Validar Imagen
    B->>S: Subir Archivo
    S->>B: URL del Archivo
    B->>DB: Actualizar Avatar
    DB->>B: Confirmación
    B->>F: Avatar Actualizado
    F->>U: Mostrar Nuevo Avatar
```

### **4. Configuración de Privacidad**

```mermaid
sequenceDiagram
    participant U as Usuario
    participant F as Frontend
    participant B as Backend
    participant DB as Database

    U->>F: Cambiar Configuración
    F->>B: PUT /profile/privacy
    B->>B: Validar Configuración
    B->>DB: Actualizar Privacidad
    DB->>B: Confirmación
    B->>F: Configuración Actualizada
    F->>U: Mostrar Cambios
```

## 📊 **Modelo de Datos Detallado**

### **Relaciones entre Tablas**

```
users (Firebase UID)
├── users_profiles (1:1)
│   ├── user_privacy_settings (1:1)
│   └── user_purchases (1:N)
└── credits (1:1)
```

### **Campos por Tabla**

#### **users_profiles**
- `id` (PK)
- `uid` (FK a users, único)
- `alias` (único, 3-30 caracteres)
- `name` (2-100 caracteres)
- `age` (13-120 años)
- `gender` (enum: Masculino, Femenino, Otro)
- `location` (enum: Colombia, España, Otro)
- `phrase` (opcional, 200 caracteres)
- `about_me` (opcional, 1000 caracteres)
- `interests` (opcional, 500 caracteres)
- `avatar_url` (opcional, 500 caracteres)
- `avatar_type` (enum: icon, uploaded)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### **user_privacy_settings**
- `id` (PK)
- `user_id` (FK a users_profiles)
- `name_public` (boolean, default: false)
- `age_public` (boolean, default: false)
- `gender_public` (boolean, default: false)
- `location_public` (boolean, default: false)
- `phrase_public` (boolean, default: false)
- `about_me_public` (boolean, default: false)
- `interests_public` (boolean, default: false)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### **user_purchases**
- `id` (PK)
- `user_id` (FK a users_profiles)
- `purchase_type` (enum: credits, product)
- `amount` (decimal, 10,2)
- `currency` (varchar, 3, default: ARS)
- `status` (enum: pending, approved, rejected, cancelled)
- `payment_method` (varchar, 50)
- `mercadopago_payment_id` (varchar, 100, nullable)
- `created_at` (timestamp)

## 🔐 **Seguridad y Autenticación**

### **Flujo de Autenticación**

```
1. Usuario se registra/login en Firebase
2. Firebase retorna JWT token
3. Frontend envía token en header Authorization
4. Backend valida token con Firebase Admin SDK
5. Si válido, extrae UID y email del token
6. Busca usuario en base de datos local
7. Si no existe, crea registro en tabla users
8. Retorna información del usuario autenticado
```

### **Validaciones de Seguridad**

#### **Rate Limiting**
```python
# Máximo 100 requests por minuto por usuario
RATE_LIMIT = {
    "requests": 100,
    "window": 60  # segundos
}
```

#### **Validación de Archivos**
```python
ALLOWED_TYPES = ['image/jpeg', 'image/png']
MAX_SIZE = 2 * 1024 * 1024  # 2MB
MAX_DIMENSIONS = (1024, 1024)  # 1024x1024px
```

#### **Sanitización de Datos**
```python
def sanitize_text(text: str) -> str:
    """Elimina caracteres peligrosos del texto"""
    import html
    return html.escape(text.strip())
```

## 🚀 **Performance y Optimización**

### **Índices de Base de Datos**

```sql
-- Índices para consultas frecuentes
CREATE INDEX idx_users_profiles_alias ON users_profiles(alias);
CREATE INDEX idx_users_profiles_location ON users_profiles(location);
CREATE INDEX idx_users_profiles_gender ON users_profiles(gender);
CREATE INDEX idx_users_profiles_created_at ON users_profiles(created_at);

-- Índices para historial de compras
CREATE INDEX idx_user_purchases_user_id ON user_purchases(user_id);
CREATE INDEX idx_user_purchases_status ON user_purchases(status);
CREATE INDEX idx_user_purchases_created_at ON user_purchases(created_at);

-- Índices compuestos para consultas complejas
CREATE INDEX idx_user_purchases_user_status ON user_purchases(user_id, status);
CREATE INDEX idx_users_profiles_location_gender ON users_profiles(location, gender);
```

### **Caching Strategy**

```python
# Cache para perfiles públicos (30 minutos)
PROFILE_CACHE_TTL = 1800

# Cache para opciones de avatar (1 hora)
AVATAR_OPTIONS_CACHE_TTL = 3600

# Cache para estadísticas (1 día)
STATS_CACHE_TTL = 86400
```

### **Optimización de Consultas**

```python
# Consulta optimizada para perfil público
async def get_public_profile_optimized(alias: str) -> dict:
    query = """
    SELECT 
        up.alias,
        up.name,
        up.age,
        up.gender,
        up.location,
        up.phrase,
        up.about_me,
        up.interests,
        up.avatar_url,
        up.avatar_type,
        ups.name_public,
        ups.age_public,
        ups.gender_public,
        ups.location_public,
        ups.phrase_public,
        ups.about_me_public,
        ups.interests_public
    FROM users_profiles up
    LEFT JOIN user_privacy_settings ups ON up.id = ups.user_id
    WHERE up.alias = :alias
    """
    
    result = await db.execute(query, {"alias": alias})
    return result.fetchone()
```

## 📱 **Frontend Architecture**

### **Componentes React**

```
src/
├── components/
│   ├── ProfileCompleteModal/
│   │   ├── index.tsx
│   │   ├── ProfileForm.tsx
│   │   └── styles.module.css
│   ├── ProfileEdit/
│   │   ├── index.tsx
│   │   ├── AvatarSelector.tsx
│   │   ├── PrivacySettings.tsx
│   │   └── styles.module.css
│   ├── PublicProfile/
│   │   ├── index.tsx
│   │   ├── ProfileHeader.tsx
│   │   ├── ProfileInfo.tsx
│   │   └── styles.module.css
│   └── PurchaseHistory/
│       ├── index.tsx
│       ├── PurchaseCard.tsx
│       └── styles.module.css
├── hooks/
│   ├── useAuth.ts
│   ├── useProfile.ts
│   └── usePurchases.ts
├── lib/
│   ├── api/
│   │   ├── users.ts
│   │   └── purchases.ts
│   └── firebase/
│       └── auth.ts
└── types/
    ├── profile.ts
    ├── purchase.ts
    └── privacy.ts
```

### **Estado Global**

```typescript
// Context para autenticación
interface AuthContext {
  user: User | null;
  loading: boolean;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
}

// Context para perfil
interface ProfileContext {
  profile: UserProfile | null;
  loading: boolean;
  updateProfile: (data: UpdateProfileData) => Promise<void>;
  refreshProfile: () => Promise<void>;
}
```

## 🔄 **Deployment y CI/CD**

### **Pipeline de Deployment**

```yaml
# .github/workflows/deploy.yml
name: Deploy User Profiles Module

on:
  push:
    branches: [main]
    paths: ['backend/app/users/**', 'backend/app/uploads/**']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Tests
        run: |
          cd backend
          pip install -r requirements.txt
          pytest tests/test_user_profiles.py -v

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        run: |
          # Scripts de deployment
          echo "Deploying user profiles module..."
```

### **Variables de Entorno**

```bash
# .env.production
FIREBASE_PROJECT_ID=spartan-market-prod
FIREBASE_PRIVATE_KEY=your-private-key
MERCADOPAGO_ACCESS_TOKEN=your-access-token
DATABASE_URL=postgresql://user:pass@host:port/db
STORAGE_BUCKET=spartan-market-avatars
REDIS_URL=redis://localhost:6379
```

## 📊 **Monitoreo y Analytics**

### **Métricas Clave**

```python
# Métricas a monitorear
METRICS = {
    "profile_completion_rate": "Tasa de completado de perfiles",
    "avatar_upload_rate": "Tasa de subida de avatares",
    "privacy_settings_usage": "Uso de configuraciones de privacidad",
    "purchase_conversion_rate": "Tasa de conversión de compras",
    "api_response_time": "Tiempo de respuesta de API",
    "error_rate": "Tasa de errores"
}
```

### **Logging Structure**

```python
# Estructura de logs
LOG_STRUCTURE = {
    "timestamp": "ISO 8601",
    "level": "INFO|WARN|ERROR",
    "user_id": "Firebase UID",
    "action": "profile_create|profile_update|avatar_upload",
    "details": {
        "endpoint": "/api/v1/users/profile/complete",
        "method": "POST",
        "status_code": 200,
        "response_time": 150  # ms
    }
}
```

---

**Estado**: ✅ **Documentación Completa**
**Próximo Paso**: Implementación de Fase 1 - Base de Datos y Modelos 