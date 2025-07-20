# 📋 **Documentación Completa de Endpoints - Spartan Market API**

## **Base URL**
```
https://api.spartanmarket.com/api/v1
```

## **Autenticación**
Todos los endpoints requieren autenticación mediante token JWT de Firebase:
```
Authorization: Bearer <firebase_jwt_token>
```

---

## **📊 Tabla Completa de Endpoints**

### **🔐 Autenticación y Seguridad**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `GET` | `/health` | Health check del sistema | ❌ | 100/min |
| `GET` | `/metrics/system` | Métricas del sistema | ❌ | 10/min |
| `GET` | `/metrics/requests` | Métricas de requests | ❌ | 10/min |

### **👤 Perfiles de Usuario**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `POST` | `/users/profile/complete` | Completar perfil inicial | ✅ | 10/min |
| `GET` | `/users/profile/me` | Obtener perfil propio | ✅ | 100/min |
| `PUT` | `/users/profile/update` | Actualizar perfil | ✅ | 10/min |
| `GET` | `/users/profile/{alias}` | Perfil público | ❌ | 100/min |
| `GET` | `/users/profile/completion` | Porcentaje completado | ✅ | 100/min |

### **🖼️ Avatares**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `POST` | `/users/avatar/upload` | Subir avatar | ✅ | 5/min |
| `DELETE` | `/users/avatar/remove` | Eliminar avatar | ✅ | 5/min |
| `GET` | `/users/avatar/options` | Opciones de avatar | ✅ | 100/min |

### **🔒 Privacidad**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `PUT` | `/users/privacy/update` | Actualizar privacidad | ✅ | 10/min |
| `GET` | `/users/privacy/settings` | Obtener configuración | ✅ | 100/min |

### **💰 Créditos**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `GET` | `/users/credits` | Obtener créditos | ✅ | 100/min |
| `GET` | `/advanced/credits/me` | Créditos con caché | ✅ | 100/min |
| `POST` | `/advanced/credits/buy` | Comprar créditos | ✅ | 10/min |
| `GET` | `/advanced/credits/packages` | Paquetes disponibles | ❌ | 100/min |

### **📦 Compras**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `GET` | `/users/purchases` | Historial de compras | ✅ | 100/min |
| `GET` | `/users/purchases/me` | Mis compras | ✅ | 100/min |

### **🔧 Funcionalidades Avanzadas**
| Método | Endpoint | Descripción | Auth | Rate Limit |
|--------|----------|-------------|------|------------|
| `POST` | `/advanced/profile/avatar/upload` | Avatar con procesamiento | ✅ | 5/min |
| `POST` | `/advanced/webhooks/mercadopago` | Webhook MercadoPago | ❌ | 100/min |
| `GET` | `/advanced/metrics/endpoints` | Métricas por endpoint | ❌ | 10/min |
| `GET` | `/advanced/health` | Health checks completos | ❌ | 10/min |
| `GET` | `/advanced/tasks/{task_id}` | Estado de tarea | ✅ | 100/min |
| `DELETE` | `/advanced/tasks/{task_id}` | Cancelar tarea | ✅ | 10/min |
| `GET` | `/advanced/tasks` | Tareas en ejecución | ✅ | 100/min |
| `DELETE` | `/advanced/cache/user/{user_id}` | Invalidar caché usuario | ✅ | 10/min |
| `DELETE` | `/advanced/cache/public-profiles` | Invalidar perfiles públicos | ✅ | 10/min |
| `GET` | `/advanced/analytics/user/{user_id}` | Analytics de usuario | ✅ | 10/min |

---

## **📝 Ejemplos Detallados por Endpoint**

### **1. POST /users/profile/complete**
**Completar perfil inicial del usuario**

#### **Request Body:**
```json
{
  "alias": "usuario123",
  "full_name": "Juan Pérez",
  "bio": "Desarrollador web apasionado por la tecnología",
  "location": "Colombia",
  "gender": "Masculino",
  "birth_date": "1990-05-15",
  "website": "https://juanperez.com",
  "social_media": {
    "instagram": "@juanperez",
    "twitter": "@juanperez",
    "linkedin": "juanperez"
  }
}
```

#### **Response (200):**
```json
{
  "id": 1,
  "uid": "firebase_uid_123",
  "alias": "usuario123",
  "full_name": "Juan Pérez",
  "bio": "Desarrollador web apasionado por la tecnología",
  "location": "Colombia",
  "gender": "Masculino",
  "birth_date": "1990-05-15",
  "website": "https://juanperez.com",
  "social_media": {
    "instagram": "@juanperez",
    "twitter": "@juanperez",
    "linkedin": "juanperez"
  },
  "avatar_url": null,
  "avatar_type": "default",
  "privacy_settings": {
    "full_name_public": true,
    "bio_public": true,
    "location_public": true,
    "gender_public": false,
    "birth_date_public": false,
    "website_public": true,
    "social_media_public": true
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### **Error Responses:**
```json
// 400 - Alias ya existe
{
  "detail": "El alias 'usuario123' ya está en uso"
}

// 400 - Datos inválidos
{
  "detail": "El campo 'full_name' es requerido"
}

// 401 - No autenticado
{
  "detail": "No autorizado"
}
```

### **2. GET /users/profile/me**
**Obtener perfil propio del usuario**

#### **Response (200):**
```json
{
  "id": 1,
  "uid": "firebase_uid_123",
  "alias": "usuario123",
  "full_name": "Juan Pérez",
  "bio": "Desarrollador web apasionado por la tecnología",
  "location": "Colombia",
  "gender": "Masculino",
  "birth_date": "1990-05-15",
  "website": "https://juanperez.com",
  "social_media": {
    "instagram": "@juanperez",
    "twitter": "@juanperez",
    "linkedin": "juanperez"
  },
  "avatar_url": "https://cdn.spartanmarket.com/avatars/avatar_123.jpg",
  "avatar_type": "uploaded",
  "privacy_settings": {
    "full_name_public": true,
    "bio_public": true,
    "location_public": true,
    "gender_public": false,
    "birth_date_public": false,
    "website_public": true,
    "social_media_public": true
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### **3. GET /users/profile/{alias}**
**Obtener perfil público por alias**

#### **Response (200):**
```json
{
  "alias": "usuario123",
  "full_name": "Juan Pérez",
  "bio": "Desarrollador web apasionado por la tecnología",
  "location": "Colombia",
  "website": "https://juanperez.com",
  "social_media": {
    "instagram": "@juanperez",
    "twitter": "@juanperez",
    "linkedin": "juanperez"
  },
  "avatar_url": "https://cdn.spartanmarket.com/avatars/avatar_123.jpg",
  "avatar_type": "uploaded"
}
```

#### **Error Responses:**
```json
// 404 - Perfil no encontrado
{
  "detail": "Perfil no encontrado"
}

// 403 - Perfil privado
{
  "detail": "Este perfil es privado"
}
```

### **4. POST /users/avatar/upload**
**Subir avatar del usuario**

#### **Request:**
```
Content-Type: multipart/form-data
Authorization: Bearer <token>

Form Data:
- image: [archivo de imagen]
```

#### **Response (200):**
```json
{
  "avatar_url": "https://cdn.spartanmarket.com/avatars/avatar_123.jpg",
  "avatar_type": "uploaded",
  "message": "Avatar subido exitosamente"
}
```

#### **Error Responses:**
```json
// 400 - Archivo inválido
{
  "detail": "Solo se permiten archivos JPG, PNG o GIF"
}

// 400 - Tamaño excedido
{
  "detail": "El archivo debe ser menor a 2MB"
}

// 413 - Archivo muy grande
{
  "detail": "El archivo es demasiado grande"
}
```

### **5. POST /advanced/credits/buy**
**Comprar créditos con MercadoPago**

#### **Request Body:**
```json
{
  "amount": 100
}
```

#### **Response (200):**
```json
{
  "purchase_id": 123,
  "credits": 100,
  "price_ars": 10.00,
  "payment_url": "https://www.mercadopago.com.ar/checkout/v1/redirect?pref_id=123456789",
  "is_sandbox": true,
  "external_reference": "credits_1_abc123"
}
```

#### **Error Responses:**
```json
// 400 - Cantidad inválida
{
  "detail": "Cantidad de créditos inválida. Opciones disponibles: [100, 500, 1000, 2000]"
}

// 404 - Perfil no encontrado
{
  "detail": "Perfil no encontrado. Complete su perfil primero."
}
```

### **6. GET /users/purchases/me**
**Obtener historial de compras del usuario**

#### **Query Parameters:**
```
?page=1&limit=10&status=approved
```

#### **Response (200):**
```json
{
  "purchases": [
    {
      "id": 1,
      "purchase_type": "credits",
      "amount": 100,
      "currency": "ARS",
      "status": "approved",
      "payment_method": "mercadopago",
      "mercadopago_payment_id": "123456789",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:35:00Z"
    },
    {
      "id": 2,
      "purchase_type": "product",
      "amount": -50,
      "currency": "ARS",
      "status": "approved",
      "payment_method": "credits",
      "created_at": "2024-01-14T15:20:00Z",
      "updated_at": "2024-01-14T15:20:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "pages": 3
  }
}
```

### **7. PUT /users/privacy/update**
**Actualizar configuración de privacidad**

#### **Request Body:**
```json
{
  "privacy_settings": {
    "full_name_public": true,
    "bio_public": true,
    "location_public": false,
    "gender_public": false,
    "birth_date_public": false,
    "website_public": true,
    "social_media_public": true
  }
}
```

#### **Response (200):**
```json
{
  "message": "Configuración de privacidad actualizada correctamente",
  "privacy_settings": {
    "full_name_public": true,
    "bio_public": true,
    "location_public": false,
    "gender_public": false,
    "birth_date_public": false,
    "website_public": true,
    "social_media_public": true
  }
}
```

### **8. GET /advanced/health**
**Health check completo del sistema**

#### **Response (200):**
```json
{
  "overall_status": "healthy",
  "database": {
    "status": "healthy",
    "response_time": 0.002,
    "checked_at": "2024-01-15T10:30:00Z"
  },
  "redis": {
    "status": "healthy",
    "response_time": 0.001,
    "checked_at": "2024-01-15T10:30:00Z"
  },
  "external_services": {
    "mercadopago": {
      "status": "healthy",
      "response_time": 0.150,
      "status_code": 200
    },
    "firebase": {
      "status": "healthy",
      "response_time": 0.080,
      "status_code": 200
    }
  },
  "system": {
    "status": "healthy",
    "cpu_percent": 25.5,
    "memory_percent": 45.2,
    "disk_percent": 60.1
  },
  "checked_at": "2024-01-15T10:30:00Z"
}
```

---

## **🔧 Códigos de Error Comunes**

### **4xx - Errores del Cliente**
| Código | Descripción | Ejemplo |
|--------|-------------|---------|
| `400` | Bad Request | Datos inválidos, validación fallida |
| `401` | Unauthorized | Token inválido o expirado |
| `403` | Forbidden | Sin permisos para el recurso |
| `404` | Not Found | Recurso no encontrado |
| `409` | Conflict | Alias ya existe |
| `413` | Payload Too Large | Archivo demasiado grande |
| `429` | Too Many Requests | Rate limit excedido |

### **5xx - Errores del Servidor**
| Código | Descripción | Ejemplo |
|--------|-------------|---------|
| `500` | Internal Server Error | Error interno del servidor |
| `502` | Bad Gateway | Error en servicio externo |
| `503` | Service Unavailable | Servicio temporalmente no disponible |

---

## **📊 Rate Limiting**

### **Límites por Endpoint**
- **Endpoints de lectura**: 100 requests/minuto
- **Endpoints de escritura**: 10 requests/minuto
- **Subida de archivos**: 5 requests/minuto
- **Health checks**: 10 requests/minuto
- **Métricas**: 10 requests/minuto

### **Headers de Rate Limiting**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642234567
```

---

## **🔐 Seguridad**

### **Validación de Tokens**
- Todos los endpoints protegidos requieren token JWT válido
- Tokens se validan con Firebase Authentication
- Tokens expiran según configuración de Firebase

### **Sanitización de Datos**
- Todos los inputs se sanitizan automáticamente
- Validación de tipos de archivo para uploads
- Límites de tamaño para archivos
- Validación de URLs y formatos

### **CORS**
```
Access-Control-Allow-Origin: https://spartanmarket.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

---

## **📝 Notas Importantes**

1. **Alias Único**: Los alias deben ser únicos en toda la plataforma
2. **Enums**: Ubicación solo permite "Colombia", "España", "Otro"
3. **Género**: Solo permite "Masculino", "Femenino", "Otro"
4. **Avatares**: Máximo 2MB, formatos JPG, PNG, GIF
5. **Créditos**: 4 paquetes predefinidos (100, 500, 1000, 2000)
6. **Privacidad**: Configuración granular por campo
7. **Caché**: Respuestas cacheadas por 5-30 minutos según endpoint

---

**📋 Documentación generada automáticamente con FastAPI** 