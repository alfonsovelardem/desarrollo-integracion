# desarrollo-integracion
Integración que permita la comunicación entre dos sistemas diferentes

### Diagrama de Flujo

El proceso se centraliza en la Capa de Integración, el cual maneja la seguridad, la trazabilidad y la traducción de protocolos y formatos.

[SISTEMA A]
  (Solicitud SOAP/XML)
        ↓
[ADAPTADOR DE SERVICIO]
        ↓
(1) 🔑  Validar Autenticación (EXTRAER_HEADER)
  → SI FALLA: Detener y responder con ERROR 401 (Fault SOAP)
        ↓
(2)  Extraer Cuerpo XML (EXTRAER_BODY_SOAP)
        ↓
(3) 🚨  Validar Esquema XSD
  → SI FALLA: Detener y responder con ERROR 400 (Fault SOAP)
        ↓
(4) 🔄  Parsear XML a Objeto (PARSE_XML_A_OBJETO)
        ↓
(5) 🔄  Transformación de Datos (Objeto → JSON Payload)
        ↓
(6) 🛣️  Serializar a JSON (SERIALIZAR_A_JSON)
        ↓
(7) 🛣️  Llamar Servicio REST (LLAMAR_SERVICIO_REST)
        ↓
[SISTEMA B]
  (Endpoint POST /api/pedidos - Respuesta JSON)
        ↓
[ADAPTADOR DE SERVICIO]
        ↓
(8) 🚨  Verificar Status de Respuesta (Status >= 400)
  → SI FALLA: Detener y responder con ERROR Propagado (Fault SOAP)
        ↓
(9) 🔄  Transformar Respuesta JSON a XML/SOAP (CREAR_RESPUESTA_SOAP_EXITOSA)
        ↓
(10) 🛣️  Responder
        ↓
[SISTEMA A]
  (Respuesta SOAP/XML Exitosa)

### Manejo de Errores

| Tipo de Error | Detección (Punto del Flujo) | Acción del Adaptador (Respuesta a Sistema A) |
| :--- | :--- | :--- |
| **Error de Autenticación** | Pre-procesamiento. | Registrar intento no autorizado. Devuelve error **SOAP 401 (Unauthorized)**. |
| **Error de Validación** | Transformación (Validación XML). | Registrar esquema inválido. Devuelve error **SOAP 400 (Bad Request)**. |
| **Error de Conexión** | Al intentar llamar a Sistemas A o B. | Registrar fallo de conexión. Devuelve error **SOAP 503 (Service Unavailable)**. |
| **Error de Lógica** | Post-procesamiento (Status $\ge 400$ de Sistema B). | Registrar el error de negocio. Devuelve un **SOAP 4xx o 5xx** mapeado con el mensaje de error de B. |

### Archivo del Pseudocodigo

    *`transformation.pseudocode`
    
