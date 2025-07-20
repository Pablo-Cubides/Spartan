# MercadoPago Integration for Spartan Market API

## ✅ Status: Working Correctly

La integración con MercadoPago está funcionando perfectamente. Todas las pruebas han pasado exitosamente.

## 🚀 Quick Start

### 1. Iniciar el servidor
```bash
cd backend
python start_server.py
```

### 2. Probar la integración
```bash
python test_mercadopago.py
```

### 3. Probar los endpoints de la API
```bash
python test_api_endpoints.py
```

## 📋 Endpoints Disponibles

### Health Check
- **GET** `/health` - Verificar estado del servidor

### MercadoPago Endpoints
- **GET** `/payments/test-preference` - Crear preferencia de prueba
- **POST** `/payments/create-preference` - Crear preferencia personalizada
- **GET** `/payments/payment/{payment_id}` - Obtener información de pago
- **POST** `/payments/webhook` - Procesar webhooks de MercadoPago

## 🔧 Configuración

### Access Token
El access token está configurado en `app/payments/mercadopago_client.py`:
```
APP_USR-663505429919307-071920-990d4c44c11d5465eeb7d7e11f9997d1-2553283846
```

### URLs de Retorno
- **Success**: `http://localhost:3000/payment/success`
- **Failure**: `http://localhost:3000/payment/failure`
- **Pending**: `http://localhost:3000/payment/pending`

## 📝 Ejemplos de Uso

### Crear una Preferencia de Pago
```python
import requests

preference_data = {
    "items": [
        {
            "title": "Producto de Prueba",
            "quantity": 1,
            "unit_price": 100.0,
            "currency_id": "ARS"
        }
    ],
    "back_urls": {
        "success": "http://localhost:3000/payment/success",
        "failure": "http://localhost:3000/payment/failure",
        "pending": "http://localhost:3000/payment/pending"
    }
}

response = requests.post(
    "http://localhost:8000/payments/create-preference",
    json=preference_data
)

preference = response.json()
print(f"Preference ID: {preference['id']}")
print(f"Payment URL: {preference['init_point']}")
```

### Procesar Webhook
```python
# MercadoPago enviará POST requests a este endpoint
# cuando haya actualizaciones de pago
```

## 🧪 Pruebas Realizadas

### ✅ Pruebas Exitosas
1. **Importación de MercadoPago** - ✅ Funcionando
2. **Inicialización del Cliente** - ✅ Funcionando
3. **Creación de Preferencias** - ✅ Funcionando
4. **Conexión con API** - ✅ Funcionando
5. **Endpoints de FastAPI** - ✅ Funcionando
6. **Esquemas de Pydantic** - ✅ Funcionando

### 📊 Resultados de las Pruebas
- **Preference ID generado**: `2553283846-3f712483-240c-4c3d-a921-9af32ae9b8da`
- **Init Point**: `https://www.mercadopago.com.co/checkout/v1/redirect?...`
- **Sandbox Init Point**: `https://sandbox.mercadopago.com.co/checkout/v1/redirect?...`

## 🔍 Troubleshooting

### Si las pruebas fallan:

#### 1. Verificar Dependencias
```bash
pip install -r requirements.txt
```

#### 2. Verificar Access Token
- Asegúrate de que el access token sea válido
- Verifica que tenga los permisos necesarios

#### 3. Verificar Conexión a Internet
```bash
ping mercadopago.com
```

#### 4. Verificar Puerto del Servidor
```bash
netstat -an | findstr :8000
```

#### 5. Revisar Logs del Servidor
```bash
python start_server.py
```

### Errores Comunes y Soluciones

#### Error: "Could not import module main"
**Solución**: Asegúrate de estar en el directorio `backend` y que `main.py` exista.

#### Error: "Connection refused"
**Solución**: Verifica que el puerto 8000 esté disponible y no esté siendo usado por otro proceso.

#### Error: "Access token invalid"
**Solución**: Verifica que el access token sea correcto y tenga los permisos necesarios.

#### Error: "MercadoPago API not accessible"
**Solución**: Verifica tu conexión a internet y que MercadoPago esté disponible.

## 📚 Documentación Adicional

### API Documentation
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

### MercadoPago Documentation
- [MercadoPago API Documentation](https://www.mercadopago.com.ar/developers/es/docs)
- [Python SDK Documentation](https://github.com/mercadopago/sdk-python)

## 🚀 Próximos Pasos

1. **Integrar con Frontend**: Conectar los endpoints con tu aplicación React/Next.js
2. **Configurar Webhooks**: Configurar URLs de webhook para notificaciones de pago
3. **Implementar Base de Datos**: Guardar información de pagos en SQLAlchemy
4. **Testing en Producción**: Probar con pagos reales en modo sandbox
5. **Monitoreo**: Implementar logging y monitoreo de transacciones

## 📞 Soporte

Si encuentras problemas:
1. Revisa los logs del servidor
2. Ejecuta las pruebas de diagnóstico
3. Verifica la documentación de MercadoPago
4. Revisa los errores comunes en esta guía

---

**Estado**: ✅ **FUNCIONANDO CORRECTAMENTE**
**Última prueba**: Todas las pruebas pasaron exitosamente
**Access Token**: Válido y funcionando
**API Endpoints**: Todos operativos 