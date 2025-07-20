# Especificación Técnica - Módulo de Perfiles de Usuario

## 📋 **Resumen Ejecutivo**

Este documento especifica la implementación del módulo de perfiles de usuario para Spartan Market, incluyendo autenticación Firebase, gestión de perfiles, privacidad, avatares e historial de compras.

## 🏗️ **Arquitectura del Sistema**

### **Componentes Principales**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   External      │
│   (Next.js)     │◄──►│   (FastAPI)     │◄──►│   Services      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
│                        │                        │
│  • Profile UI         │  • User Routes         │  • Firebase Auth
│  • Avatar Selector    │  • Profile Services    │  • MercadoPago
│  • Privacy Settings   │  • Upload Handler      │  • Storage (S3)
│  • ISR Pages         │  • Database Models      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **Flujo de Datos**

1. **Registro/Login** → Firebase Auth → Token JWT
2. **Validación Token** → Middleware → User Context
3. **Perfil Incompleto** → Redirect → Complete Profile Form
4. **Guardar Perfil** → Validation → Database
5. **Avatar Upload** → Storage → URL
6. **Privacy Settings** → Database → Filtered Responses

## 📊 **Modelo de Datos**

### **Tabla: users_profiles**

```sql
CREATE TABLE users_profiles (
    id SERIAL PRIMARY KEY,
    uid VARCHAR(128) NOT NULL UNIQUE REFERENCES users(uid),
    alias VARCHAR(30) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    age INTEGER CHECK (age >= 13 AND age <= 120),
    gender VARCHAR(20) CHECK (gender IN ('Masculino', 'Femenino', 'Otro')),
    location VARCHAR(20) CHECK (location IN ('Colombia', 'España', 'Otro')),
    phrase VARCHAR(200),
    about_me TEXT,
    interests TEXT,
    avatar_url VARCHAR(500),
    avatar_type VARCHAR(10) CHECK (avatar_type IN ('icon', 'uploaded')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices para optimización
CREATE INDEX idx_users_profiles_alias ON users_profiles(alias);
CREATE INDEX idx_users_profiles_location ON users_profiles(location);
CREATE INDEX idx_users_profiles_gender ON users_profiles(gender);
CREATE INDEX idx_users_profiles_created_at ON users_profiles(created_at);
```

### **Tabla: user_privacy_settings**

```sql
CREATE TABLE user_privacy_settings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users_profiles(id),
    name_public BOOLEAN DEFAULT false,
    age_public BOOLEAN DEFAULT false,
    gender_public BOOLEAN DEFAULT false,
    location_public BOOLEAN DEFAULT false,
    phrase_public BOOLEAN DEFAULT false,
    about_me_public BOOLEAN DEFAULT false,
    interests_public BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Tabla: user_purchases**

```sql
CREATE TABLE user_purchases (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users_profiles(id),
    purchase_type VARCHAR(20) CHECK (purchase_type IN ('credits', 'product')),
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'ARS',
    status VARCHAR(20) CHECK (status IN ('pending', 'approved', 'rejected', 'cancelled')),
    payment_method VARCHAR(50),
    mercadopago_payment_id VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices para consultas de historial
CREATE INDEX idx_user_purchases_user_id ON user_purchases(user_id);
CREATE INDEX idx_user_purchases_status ON user_purchases(status);
CREATE INDEX idx_user_purchases_created_at ON user_purchases(created_at);
```

## 🔐 **Autenticación y Seguridad**

### **Firebase Authentication**

```python
# Middleware de autenticación
async def verify_firebase_token(token: str) -> dict:
    """
    Verifica el token de Firebase y retorna la información del usuario
    """
    try:
        decoded_token = auth.verify_id_token(token)
        return {
            'uid': decoded_token['uid'],
            'email': decoded_token['email'],
            'email_verified': decoded_token.get('email_verified', False)
        }
    except Exception as e:
        raise HTTPException(status_code=401, detail="Token inválido")

# Decorador para endpoints protegidos
def require_auth(func):
    async def wrapper(*args, **kwargs):
        # Lógica de verificación de token
        pass
    return wrapper
```

### **Validaciones de Seguridad**

1. **Rate Limiting**: Máximo 100 requests por minuto por usuario
2. **File Upload Security**: Validación de tipo y tamaño de archivo
3. **SQL Injection Protection**: Uso de ORM con parámetros
4. **XSS Protection**: Sanitización de datos de entrada
5. **CSRF Protection**: Tokens en formularios críticos

## 📝 **Validaciones de Datos**

### **Pydantic Schemas**

```python
from pydantic import BaseModel, Field, validator
from enum import Enum
import re

class Gender(str, Enum):
    MASCULINO = "Masculino"
    FEMENINO = "Femenino"
    OTRO = "Otro"

class Location(str, Enum):
    COLOMBIA = "Colombia"
    ESPANA = "España"
    OTRO = "Otro"

class CompleteProfileRequest(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    age: int = Field(..., ge=13, le=120)
    gender: Gender
    location: Location
    alias: str = Field(..., min_length=3, max_length=30)
    phrase: str = Field(None, max_length=200)
    about_me: str = Field(None, max_length=1000)
    interests: str = Field(None, max_length=500)

    @validator('alias')
    def validate_alias(cls, v):
        if not re.match(r'^[a-zA-Z0-9_-]+$', v):
            raise ValueError('Alias solo puede contener letras, números, guiones y guiones bajos')
        return v.lower()
```

### **Validación de Alias Único**

```python
async def validate_alias_unique(alias: str, user_id: int = None) -> bool:
    """
    Valida que el alias sea único en la base de datos
    Incluye manejo de concurrencia para evitar race conditions
    """
    from sqlalchemy import select
    from app.db.models import UserProfile
    
    # Usar lock para evitar race conditions
    async with db.transaction():
        query = select(UserProfile).where(UserProfile.alias == alias.lower())
        if user_id:
            query = query.where(UserProfile.id != user_id)
        
        result = await db.execute(query)
        existing = result.scalar_one_or_none()
        
        return existing is None
```

## 🖼️ **Sistema de Avatares**

### **Íconos Prediseñados**

```python
AVATAR_ICONS = [
    {
        "name": "user",
        "display_name": "Usuario Básico",
        "url": "/avatars/icons/user.svg",
        "category": "general"
    },
    {
        "name": "user-check",
        "display_name": "Usuario Verificado",
        "url": "/avatars/icons/user-check.svg",
        "category": "status"
    },
    {
        "name": "user-plus",
        "display_name": "Usuario Premium",
        "url": "/avatars/icons/user-plus.svg",
        "category": "premium"
    },
    {
        "name": "user-x",
        "display_name": "Usuario Experto",
        "url": "/avatars/icons/user-x.svg",
        "category": "expert"
    },
    {
        "name": "user-minus",
        "display_name": "Usuario Casual",
        "url": "/avatars/icons/user-minus.svg",
        "category": "casual"
    },
    {
        "name": "user-cog",
        "display_name": "Usuario Técnico",
        "url": "/avatars/icons/user-cog.svg",
        "category": "technical"
    },
    {
        "name": "user-edit",
        "display_name": "Usuario Creativo",
        "url": "/avatars/icons/user-edit.svg",
        "category": "creative"
    },
    {
        "name": "user-search",
        "display_name": "Usuario Explorador",
        "url": "/avatars/icons/user-search.svg",
        "category": "explorer"
    },
    {
        "name": "user-star",
        "display_name": "Usuario Destacado",
        "url": "/avatars/icons/user-star.svg",
        "category": "featured"
    },
    {
        "name": "user-heart",
        "display_name": "Usuario Amigable",
        "url": "/avatars/icons/user-heart.svg",
        "category": "friendly"
    }
]
```

### **Validación de Imágenes**

```python
ALLOWED_IMAGE_TYPES = ['image/jpeg', 'image/png']
MAX_IMAGE_SIZE = 2 * 1024 * 1024  # 2MB

async def validate_image_upload(file: UploadFile) -> bool:
    """
    Valida que el archivo sea una imagen válida
    """
    if file.content_type not in ALLOWED_IMAGE_TYPES:
        raise HTTPException(status_code=400, detail="Tipo de archivo no permitido")
    
    # Leer los primeros bytes para verificar el tipo real
    content = await file.read(1024)
    await file.seek(0)  # Resetear el cursor
    
    if not content.startswith(b'\xff\xd8') and not content.startswith(b'\x89PNG'):
        raise HTTPException(status_code=400, detail="Archivo no es una imagen válida")
    
    # Verificar tamaño
    file_size = 0
    while chunk := await file.read(8192):
        file_size += len(chunk)
        if file_size > MAX_IMAGE_SIZE:
            raise HTTPException(status_code=413, detail="Archivo demasiado grande")
    
    await file.seek(0)
    return True
```

## 🔒 **Sistema de Privacidad**

### **Lógica de Filtrado**

```python
async def get_public_profile(alias: str, viewer_type: str = "public") -> dict:
    """
    Obtiene el perfil público de un usuario, filtrando por configuración de privacidad
    """
    profile = await get_profile_by_alias(alias)
    privacy_settings = await get_privacy_settings(profile.id)
    
    public_data = {
        "alias": profile.alias,
        "avatar_url": profile.avatar_url,
        "avatar_type": profile.avatar_type
    }
    
    # Filtrar campos según configuración de privacidad
    if privacy_settings.name_public or viewer_type == "admin":
        public_data["name"] = profile.name
    
    if privacy_settings.age_public or viewer_type == "admin":
        public_data["age"] = profile.age
    
    if privacy_settings.gender_public or viewer_type == "admin":
        public_data["gender"] = profile.gender
    
    if privacy_settings.location_public or viewer_type == "admin":
        public_data["location"] = profile.location
    
    if privacy_settings.phrase_public or viewer_type == "admin":
        public_data["phrase"] = profile.phrase
    
    if privacy_settings.about_me_public or viewer_type == "admin":
        public_data["about_me"] = profile.about_me
    
    if privacy_settings.interests_public or viewer_type == "admin":
        public_data["interests"] = profile.interests
    
    return public_data
```

## 📈 **Estadísticas y Reportes**

### **Consultas Agregadas**

```sql
-- Usuarios por ubicación
SELECT location, COUNT(*) as count 
FROM users_profiles 
GROUP BY location 
ORDER BY count DESC;

-- Usuarios por género
SELECT gender, COUNT(*) as count 
FROM users_profiles 
GROUP BY gender 
ORDER BY count DESC;

-- Compras por período
SELECT 
    DATE(created_at) as date,
    COUNT(*) as purchases,
    SUM(amount) as total_amount
FROM user_purchases 
WHERE status = 'approved'
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- Top usuarios por compras
SELECT 
    up.alias,
    COUNT(upur.id) as total_purchases,
    SUM(upur.amount) as total_spent
FROM users_profiles up
JOIN user_purchases upur ON up.id = upur.user_id
WHERE upur.status = 'approved'
GROUP BY up.id, up.alias
ORDER BY total_spent DESC
LIMIT 10;
```

## 🚀 **Implementación Frontend**

### **Estructura de Componentes**

```
src/
├── app/
│   ├── u/
│   │   └── [alias]/
│   │       └── page.tsx          # Perfil público ISR
│   └── profile/
│       ├── edit/
│       │   └── page.tsx          # Edición de perfil
│       └── page.tsx              # Vista de perfil propio
├── components/
│   ├── ProfileCompleteModal.tsx  # Modal post-login
│   ├── ProfileEdit.tsx           # Formulario de edición
│   ├── AvatarSelector.tsx        # Selector de avatar
│   ├── PrivacySettings.tsx       # Configuración de privacidad
│   └── PurchaseHistory.tsx       # Historial de compras
└── lib/
    ├── api/
    │   └── users.ts              # Cliente API para usuarios
    └── firebase/
        └── auth.ts               # Autenticación Firebase
```

### **ISR para Perfiles Públicos**

```typescript
// app/u/[alias]/page.tsx
export async function generateStaticParams() {
  // Generar páginas estáticas para usuarios populares
  const popularUsers = await getPopularUsers();
  return popularUsers.map((user) => ({
    alias: user.alias,
  }));
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const profile = await getPublicProfile(params.alias);
  
  return {
    title: `${profile.name || profile.alias} - Spartan Market`,
    description: profile.about_me || `Perfil de ${profile.name || profile.alias}`,
    openGraph: {
      title: `${profile.name || profile.alias} - Spartan Market`,
      description: profile.about_me || `Perfil de ${profile.name || profile.alias}`,
      images: profile.avatar_url ? [profile.avatar_url] : [],
    },
  };
}

export default async function PublicProfilePage({ params }: Props) {
  const profile = await getPublicProfile(params.alias);
  
  return (
    <div>
      <h1>{profile.name || profile.alias}</h1>
      {profile.avatar_url && (
        <img src={profile.avatar_url} alt="Avatar" />
      )}
      {/* Resto del contenido del perfil */}
    </div>
  );
}
```

## 🧪 **Testing Strategy**

### **Tests Unitarios**

```python
# tests/test_user_profiles.py
import pytest
from app.users.services import validate_alias_unique

class TestUserProfiles:
    async def test_alias_validation(self):
        # Test alias válido
        assert await validate_alias_unique("test_user") == True
        
        # Test alias inválido
        with pytest.raises(ValueError):
            validate_alias("test@user")
    
    async def test_concurrent_alias_creation(self):
        # Test de concurrencia para alias único
        import asyncio
        
        async def create_profile(alias: str):
            return await create_user_profile({"alias": alias})
        
        # Intentar crear dos perfiles con el mismo alias simultáneamente
        tasks = [
            create_profile("test_alias"),
            create_profile("test_alias")
        ]
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Solo uno debe tener éxito
        success_count = sum(1 for r in results if not isinstance(r, Exception))
        assert success_count == 1
```

### **Tests de Integración**

```python
# tests/test_api_endpoints.py
class TestUserProfileEndpoints:
    async def test_complete_profile(self, client, auth_token):
        response = await client.post(
            "/api/v1/users/profile/complete",
            json={
                "name": "Test User",
                "age": 25,
                "gender": "Masculino",
                "location": "Colombia",
                "alias": "testuser"
            },
            headers={"Authorization": f"Bearer {auth_token}"}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert data["alias"] == "testuser"
    
    async def test_alias_uniqueness(self, client, auth_token):
        # Crear primer perfil
        await client.post(
            "/api/v1/users/profile/complete",
            json={
                "name": "User 1",
                "age": 25,
                "gender": "Masculino",
                "location": "Colombia",
                "alias": "testuser"
            },
            headers={"Authorization": f"Bearer {auth_token}"}
        )
        
        # Intentar crear segundo perfil con mismo alias
        response = await client.post(
            "/api/v1/users/profile/complete",
            json={
                "name": "User 2",
                "age": 30,
                "gender": "Femenino",
                "location": "España",
                "alias": "testuser"
            },
            headers={"Authorization": f"Bearer {auth_token}"}
        )
        
        assert response.status_code == 409
```

## 📊 **Métricas y Monitoreo**

### **KPIs a Monitorear**

1. **Registro de Usuarios**
   - Tasa de conversión de registro a perfil completo
   - Tiempo promedio para completar perfil
   - Abandono en formulario de perfil

2. **Engagement**
   - Usuarios activos por día/mes
   - Tasa de actualización de perfiles
   - Uso de funcionalidades de avatar

3. **Compras**
   - Tasa de conversión de compras
   - Valor promedio de compra
   - Frecuencia de compras por usuario

4. **Performance**
   - Tiempo de respuesta de endpoints
   - Tasa de errores
   - Uso de recursos del servidor

### **Logging**

```python
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

async def log_profile_action(user_id: int, action: str, details: dict = None):
    """
    Registra acciones de perfil para análisis
    """
    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "user_id": user_id,
        "action": action,
        "details": details or {}
    }
    
    logger.info(f"Profile action: {log_entry}")
    
    # Enviar a sistema de analytics si está configurado
    if analytics_enabled:
        await send_to_analytics(log_entry)
```

## 🔄 **Migración de Datos**

### **Script de Migración**

```python
# alembic/versions/001_add_user_profiles.py
"""Add user profiles tables

Revision ID: 001
Revises: 
Create Date: 2024-01-01 00:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

def upgrade():
    # Crear tabla users_profiles
    op.create_table(
        'users_profiles',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('uid', sa.String(length=128), nullable=False),
        sa.Column('alias', sa.String(length=30), nullable=False),
        sa.Column('name', sa.String(length=100), nullable=False),
        sa.Column('age', sa.Integer(), nullable=True),
        sa.Column('gender', sa.String(length=20), nullable=True),
        sa.Column('location', sa.String(length=20), nullable=True),
        sa.Column('phrase', sa.String(length=200), nullable=True),
        sa.Column('about_me', sa.Text(), nullable=True),
        sa.Column('interests', sa.Text(), nullable=True),
        sa.Column('avatar_url', sa.String(length=500), nullable=True),
        sa.Column('avatar_type', sa.String(length=10), nullable=True),
        sa.Column('created_at', sa.TIMESTAMP(), nullable=True),
        sa.Column('updated_at', sa.TIMESTAMP(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('uid'),
        sa.UniqueConstraint('alias')
    )
    
    # Crear índices
    op.create_index('idx_users_profiles_alias', 'users_profiles', ['alias'])
    op.create_index('idx_users_profiles_location', 'users_profiles', ['location'])
    op.create_index('idx_users_profiles_gender', 'users_profiles', ['gender'])
    
    # Crear tabla user_privacy_settings
    op.create_table(
        'user_privacy_settings',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('user_id', sa.Integer(), nullable=False),
        sa.Column('name_public', sa.Boolean(), nullable=True),
        sa.Column('age_public', sa.Boolean(), nullable=True),
        sa.Column('gender_public', sa.Boolean(), nullable=True),
        sa.Column('location_public', sa.Boolean(), nullable=True),
        sa.Column('phrase_public', sa.Boolean(), nullable=True),
        sa.Column('about_me_public', sa.Boolean(), nullable=True),
        sa.Column('interests_public', sa.Boolean(), nullable=True),
        sa.Column('created_at', sa.TIMESTAMP(), nullable=True),
        sa.Column('updated_at', sa.TIMESTAMP(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.ForeignKeyConstraint(['user_id'], ['users_profiles.id'], )
    )

def downgrade():
    op.drop_table('user_privacy_settings')
    op.drop_table('users_profiles')
```

## 📚 **Documentación Adicional**

### **README para Desarrolladores**

```markdown
# Módulo de Perfiles de Usuario

## Instalación

1. Instalar dependencias:
   ```bash
   pip install -r requirements.txt
   ```

2. Ejecutar migraciones:
   ```bash
   alembic upgrade head
   ```

3. Configurar variables de entorno:
   ```bash
   FIREBASE_PROJECT_ID=your-project-id
   MERCADOPAGO_ACCESS_TOKEN=your-access-token
   ```

## Uso

### Crear un perfil de usuario

```python
from app.users.services import create_user_profile

profile_data = {
    "name": "Juan Pérez",
    "age": 25,
    "gender": "Masculino",
    "location": "Colombia",
    "alias": "juanperez",
    "phrase": "¡Hola mundo!"
}

profile = await create_user_profile(profile_data, user_id=1)
```

### Obtener perfil público

```python
from app.users.services import get_public_profile

profile = await get_public_profile("juanperez")
```

## Testing

Ejecutar tests:

```bash
pytest tests/test_user_profiles.py -v
```

## Deployment

1. Configurar variables de entorno en producción
2. Ejecutar migraciones en la base de datos
3. Configurar storage para avatares (S3 recomendado)
4. Configurar Firebase Authentication
```

---

**Estado**: ✅ **Especificación Completa**
**Próximo Paso**: Implementación de Fase 1 - Base de Datos y Modelos 