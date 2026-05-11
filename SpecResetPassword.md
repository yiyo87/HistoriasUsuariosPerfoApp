# Especificación: Restablecimiento de Contraseña

---

## 1. Descripción General

| Campo | Detalle |
|-------|---------|
| **Historia de Usuario** | Como usuario (admin, supervisor, operador, ayudante) quiero solicitar el restablecimiento de mi contraseña mediante mi correo electrónico para poder generar una nueva clave de acceso en caso de olvido y volver a utilizar los servicios de la app |
| **Funcionalidad** | Restablecimiento de contraseña olvidado mediante enlace enviado por email |
| **Módulo** | Auth / Restablecer Contraseña |
| **Prioridad** | Alta |

---

## 2. Flujo General

```
┌─────────────────────────────────────────────────────────────┐
│                  FLUJO DE RESTABLECIMIENTO                  │
└─────────────────────────────────────────────────────────────┘

PASO 1: Solicitud                        PASO 2: Email        PASO 3: Nueva Contraseña
┌──────────────────────┐                 ┌───────────┐        ┌──────────────────────┐
│  Pantalla "Olvidé mi  │ ───email──────▶ │   Email   │ ──────▶ │  Pantalla "Nueva     │
│  contraseña"         │                 │ recibido  │  clic   │  Contraseña"         │
│  Campo: Email        │                 └───────────┘         │  Campo: Nueva pass   │
│  Botón: Enviar       │                                         │  Campo: Confirmar     │
└──────────────────────┘                                         │  Botón: Guardar       │
                                                                  └──────────────────────┘
```

---

## 3. Pantalla de Solicitud de Restablecimiento

### 3.1 Elementos de la Interfaz

| Elemento | Descripción | Tipo |
|----------|-------------|------|
| Campo Email | Input para correo electrónico | TextField |
| Botón "Enviar enlace" | Solicita el envío del email | Button |
| Enlace "Volver al login" | Redirige a pantalla de login | Link |
| Indicador de carga | Muestra procesamiento | Spinner/Loader |
| Mensaje informativo | Estado de la solicitud | Banner/Alert |
| Mensaje error | Errores de validación | Alert/Banner |

### 3.2 Estados

| Estado | Descripción |
|--------|-------------|
| **Idle** | Campo vacío, botón habilitado |
| **Cargando** | Spinner visible, botón deshabilitado |
| **Email enviado** | Mensaje de confirmación mostrado |
| **Error** | Mensaje de error visible |

---

## 4. Pantalla de Nueva Contraseña (Post-clic en enlace)

### 4.1 Elementos de la Interfaz

| Elemento | Descripción | Tipo |
|----------|-------------|------|
| Campo Nueva Contraseña | Input para nueva contraseña | Password |
| Campo Confirmar Contraseña | Repetir nueva contraseña | Password |
| Indicador de requisitos | Muestra si cumple requisitos | Text/Help |
| Botón "Guardar nueva contraseña" | Confirma el cambio | Button |
| Indicador de carga | Muestra procesamiento | Spinner/Loader |
| Mensaje error | Errores de validación | Alert/Banner |

### 4.2 Estados

| Estado | Descripción |
|--------|-------------|
| **Idle** | Campos vacíos, botón deshabilitado hasta cumplir requisitos |
| **Validando** | Validación en tiempo real mientras escribe |
| **Cargando** | Spinner visible, campos deshabilitados |
| **Éxito** | Contraseña cambiada, redirige a login |
| **Error** | Mensaje de error visible |

---

## 5. Validaciones

### 5.1 Pantalla de Solicitud

| Campo | Regla | Mensaje Error |
|-------|-------|---------------|
| Email | Formato válido | "Ingresa un correo electrónico válido" |
| Email | No vacío | "El correo electrónico es obligatorio" |

### 5.2 Pantalla de Nueva Contraseña

| Campo | Regla | Mensaje Error |
|-------|-------|---------------|
| Nueva Contraseña | Mínimo 8 caracteres | "La contraseña debe tener mínimo 8 caracteres" |
| Nueva Contraseña | Al menos 1 mayúscula | "La contraseña debe tener al menos 1 mayúscula" |
| Nueva Contraseña | Al menos 1 número | "La contraseña debe tener al menos 1 número" |
| Confirmar | Debe coincidir con nueva contraseña | "Las contraseñas no coinciden" |

---

## 6. Happy Path (Flujo Exitoso)

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO EXITOSO                             │
└─────────────────────────────────────────────────────────────┘

1.  Actor abre la app
    → Pantalla de Login displayed

2.  Actor presiona enlace "¿Olvidaste tu contraseña?"
    → Pantalla de Restablecimiento displayed

3.  Actor ingresa email "usuario@example.com"

4.  Actor presiona botón "Enviar enlace"
    → Spinner aparece
    → Sistema verifica si el email existe (no revela al usuario)

5.  Sistema genera token único de restablecimiento
    → Token almacenado con fecha de expiración (24h)

6.  Sistema envía email con enlace:
    → Asunto: "Restablece tu contraseña"
    → Enlace: https://app.com/reset-password?token=XYZ123

7.  Sistema muestra mensaje:
    → "Si el email está registrado, recibirás un enlace para
       restablecer tu contraseña. Revisa tu bandeja de entrada."

8.  Actor abre su email y presiona el enlace
    → Navegador abre: app.com/reset-password?token=XYZ123

9.  Sistema valida token:
    → Token existe ✓
    → Token no expirado ✓
    → Token no usado ✓

10. Sistema muestra pantalla "Nueva Contraseña"

11. Actor ingresa nueva contraseña "MiPass456"
    → Validación en tiempo real: ✓

12. Actor ingresa confirmar contraseña "MiPass456"
    → Validación: ✓

13. Actor presiona botón "Guardar nueva contraseña"
    → Spinner aparece

14. Sistema actualiza contraseña del usuario
    → Contraseña hasheada y almacenada

15. Sistema marca token como usado
    → Token ya no es válido

16. Sistema envía email de notificación:
    → Asunto: "Tu contraseña ha sido cambiada"
    → Cuerpo: "Se ha modificado tu contraseña. Si no fuiste tú,
              contacta al soporte inmediatamente."

17. Sistema redirige a Login
    → Mensaje: "Contraseña actualizada. Ahora puedes iniciar sesión."

18. Actor ingresa credenciales:
    - Email: "usuario@example.com"
    - Nueva contraseña: "MiPass456"

19. Actor presiona "Iniciar Sesión"
    → Acceso exitoso a la app

─────────────────────────────────────────────────────────────
```

---

## 7. Sad Path (Flujos de Error)

### 7.1 Email con Formato Inválido

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "correoinvalido"        -
2.    Validación en tiempo real falla         Línea roja bajo campo
3.    Mensaje: "Ingresa un correo             Icono X aparece
      electrónico válido"
4.    Botón "Enviar enlace" deshabilitado    -
```

### 7.2 Límite de Solicitudes Excedido

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario solicita restablecimiento       Solicitud 1: ✓ enqueued
2.    ... solicita nuevamente                 Solicitud 2: ✓ enqueued
3.    ... solicita por 3ra vez                Solicitud 3: ✓ enqueued
4.    ... solicita por 4ta vez                Solicitud 4: BLOQUEADO
5.    Mensaje: "Has superado el límite       Banner informativo
      de solicitudes. Intenta nuevamente      
      en 1 hora."
```

### 7.3 Enlace Expirado

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario abre enlace de restablecimiento -
      con token expirado (>24h)                
2.    Sistema valida token                     Token expirado ✗
3.    Sistema muestra pantalla de error       -
4.    Mensaje: "Este enlace ha expirado.      Banner de error
      Solicita uno nuevo."
5.    Enlace a "Solicitar nuevo enlace"       -
```

### 7.4 Enlace Ya Utilizado

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario intenta usar token ya usado    Token usado ✗
2.    Sistema muestra pantalla de error       -
3.    Mensaje: "Este enlace ya fue utilizado. Solicita uno nuevo."
4.    Enlace a "Solicitar nuevo enlace"       -
```

### 7.5 Contraseña No Cumple Requisitos

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "pass" (4 chars)        -
2.    Validación en tiempo real falla         Línea roja bajo campo
3.    Mensaje: "La contraseña debe tener      -
      mínimo 8 caracteres, una mayúscula       
      y un número"
4.    Indicador muestra qué falta:            "✗ Mínimo 8 caracteres"
                                             "✗ Al menos 1 mayúscula"
                                             "✗ Al menos 1 número"
5.    Botón deshabilitado                     -
```

### 7.6 Contraseñas No Coinciden

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "MiPass123"            -
2.    Usuario ingresa "MiPass456" en          -
      confirmar                                
3.    Validación falla                        Línea roja
4.    Mensaje: "Las contraseñas no            Icono X
      coinciden"
5.    Botón deshabilitado                     -
```

### 7.7 Error de Conexión al Enviar Solicitud

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa email válido            -
2.    Usuario presiona "Enviar enlace"       Spinner aparece
3.    Timeout 30s sin respuesta               Spinner desaparece
4.    Mensaje: "Error de conexión.            Banner de error
      Verifica tu conexión a internet"
5.    Usuario puede reintentar                 -
```

### 7.8 Error de Conexión al Guardar Contraseña

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario completa campos correctamente    -
2.    Usuario presiona "Guardar nueva         Spinner aparece
      contraseña"                              
3.    Timeout 30s sin respuesta               Spinner desaparece
4.    Mensaje: "Error de conexión.            Banner de error
      Tu contraseña no fue cambiada.          
      Intenta nuevamente."
5.    Usuario puede reintentar                 -
```

---

## 8. BDD - Behavior Driven Development

### Feature: Restablecimiento de Contraseña

---

#### Scenario: Solicitud de restablecimiento exitosa

**Given** el usuario está en la pantalla de Login
**And** el usuario recuerda su email "usuario@example.com"
**When** el usuario presiona el enlace "¿Olvidaste tu contraseña?"
**Then** el sistema muestra la pantalla de restablecimiento
**When** el usuario ingresa "usuario@example.com" en el campo email
**And** el usuario presiona el botón "Enviar enlace"
**Then** el sistema muestra un indicador de carga
**And** el sistema genera un token único de restablecimiento
**And** el sistema envía un email con el enlace de restablecimiento
**And** el sistema muestra el mensaje "Si el email está registrado, recibirás un enlace para restablecer tu contraseña"

---

#### Scenario: Solicitud con email no registrado en el sistema

**Given** el usuario está en la pantalla de Restablecimiento
**And** el usuario ingresa un email que no existe en el sistema
**When** el usuario presiona el botón "Enviar enlace"
**Then** el sistema muestra un indicador de carga
**And** el sistema NO genera token
**And** el sistema NO envía email
**And** el sistema muestra mensaje genérico "Si el email está registrado, recibirás un enlace para restablecer tu contraseña"
**And** el sistema NO revela si el email existe o no

---

#### Scenario: Restablecimiento con enlace válido

**Given** el usuario ha recibido un email de restablecimiento
**And** el token en el enlace es válido
**And** el token no ha expirado
**And** el token no ha sido usado
**When** el usuario abre el enlace en el navegador
**Then** el sistema valida el token correctamente
**And** el sistema muestra la pantalla "Nueva Contraseña"
**When** el usuario ingresa "MiPass456" en el campo nueva contraseña
**And** el usuario ingresa "MiPass456" en el campo confirmar contraseña
**And** el usuario presiona el botón "Guardar nueva contraseña"
**Then** el sistema muestra un indicador de carga
**And** el sistema actualiza la contraseña del usuario
**And** el sistema marca el token como usado
**And** el sistema envía email de notificación de cambio
**And** el sistema redirige a Login con mensaje de éxito

---

#### Scenario: Intento de uso con enlace expirado

**Given** el usuario recibió un email de restablecimiento hace más de 24 horas
**When** el usuario abre el enlace
**Then** el sistema detecta que el token ha expirado
**And** el sistema muestra el mensaje "Este enlace ha expirado. Solicita uno nuevo"
**And** el sistema proporciona enlace para solicitar nuevo restablecimiento

---

#### Scenario: Intento de uso con enlace ya utilizado

**Given** el usuario ya usó un enlace de restablecimiento anteriormente
**When** el usuario intenta abrir el mismo enlace nuevamente
**Then** el sistema detecta que el token ya fue usado
**And** el sistema muestra el mensaje "Este enlace ya fue utilizado. Solicita uno nuevo"
**And** el sistema proporciona enlace para solicitar nuevo restablecimiento

---

#### Scenario: Nueva contraseña no cumple requisitos

**Given** el usuario está en la pantalla "Nueva Contraseña"
**When** el usuario ingresa "password" en el campo nueva contraseña
**Then** el sistema muestra validación fallida
**And** el sistema muestra el mensaje "La contraseña debe tener mínimo 8 caracteres, una mayúscula y un número"
**And** el botón "Guardar nueva contraseña" permanece deshabilitado

---

#### Scenario: Confirmación de contraseña no coincide

**Given** el usuario está en la pantalla "Nueva Contraseña"
**When** el usuario ingresa "MiPass123" en el campo nueva contraseña
**And** el usuario ingresa "MiPass456" en el campo confirmar contraseña
**Then** el sistema muestra validación fallida
**And** el sistema muestra el mensaje "Las contraseñas no coinciden"
**And** el botón permanece deshabilitado

---

#### Scenario: Límite de solicitudes excedido

**Given** el usuario ha solicitado 3 restablecimientos en la última hora
**When** el usuario intenta solicitar un cuarto restablecimiento
**Then** el sistema muestra el mensaje "Has superado el límite de solicitudes. Intenta nuevamente en 1 hora"
**And** el botón "Enviar enlace" permanece deshabilitado

---

#### Scenario: Restablecimiento exitoso e inicio de sesión

**Given** el usuario ha completado el proceso de restablecimiento
**And** la nueva contraseña ha sido guardada exitosamente
**When** el usuario es redirigido a la pantalla de Login
**And** el usuario ingresa su email "usuario@example.com"
**And** el usuario ingresa la nueva contraseña "MiPass456"
**And** el usuario presiona el botón "Iniciar Sesión"
**Then** el sistema autentica al usuario correctamente
**And** el sistema redirige al usuario al dashboard de la app

---

## 9. Requisitos Técnicos

| Requisito | Detalle |
|-----------|---------|
| **Expiración del token** | 24 horas desde su creación |
| **Límite de solicitudes** | Máximo 3 solicitudes por hora por email |
| **Longitud mínima token** | 32 caracteres (UUID v4) |
| **Unicidad del token** | Nunca se reutiliza un token |
| **Validez contraseña** | Min 8 chars, 1 mayúscula, 1 número |
| **Notificación post-cambio** | Email automático de confirmación |

---

## 10. Diagrama de Estados - Solicitud

```
        ┌──────────┐
        │   IDLE   │
        └────┬─────┘
             │ ingresa email
             ▼
        ┌──────────┐
        │ VALID    │◄────────────┐
        └────┬─────┘             │
             │ presiona enviar    │
             ▼                    │
        ┌──────────┐              │
        │ LOADING  │              │
        └────┬─────┘              │
             │                    │
       ┌─────┴─────┐              │
       │           │              │
       ▼           ▼              │
  ┌────────┐  ┌────────┐          │
  │  SENT  │  │ ERROR  │──────────┘
  └────────┘  └────────┘    reintentar
```

---

## 11. Diagrama de Estados - Nueva Contraseña

```
        ┌──────────┐
        │   IDLE   │
        └────┬─────┘
             │ingresa password
             ▼
        ┌──────────┐
        │ VALIDATE │◄────────────┐
        └────┬─────┘             │
             │ campos válidos    │
             │ presiona guardar  │
             ▼                    │
        ┌──────────┐              │
        │ LOADING  │              │
        └────┬─────┘              │
             │                    │
       ┌─────┴─────┐              │
       │           │              │
       ▼           ▼              │
  ┌────────┐  ┌────────┐          │
  │ SUCCESS│  │ ERROR  │──────────┘
  └────────┘  └────────┘    reintentar
```

---

## 12. Plantilla de Email - Restablecimiento

```
Para: usuario@example.com
Asunto: Restablece tu contraseña

Estimado usuario,

Hemos recibido una solicitud para restablecer la contraseña de tu cuenta.

Para crear una nueva contraseña, haz clic en el siguiente enlace:

[Restablecer contraseña]

Este enlace expirará en 24 horas.

Si no solicitaste este cambio, puedes ignorar este email. Tu contraseña
permanecerá sin cambios.

Saludos,
El equipo de [Nombre App]
```

---

## 13. Plantilla de Email - Confirmación de Cambio

```
Para: usuario@example.com
Asunto: Tu contraseña ha sido cambiada

Estimado usuario,

Te confirmamos que la contraseña de tu cuenta ha sido modificada exitosamente.

Detalles del cambio:
- Fecha: [fecha y hora]
- IP: [dirección IP]

Si no fuiste tú quien realizó este cambio, por favor contacta al soporte
inmediatamente.

Saludos,
El equipo de [Nombre App]
```

---

*Documento generado: Especificación de Restablecimiento de Contraseña - App Móvil*