# Especificación: Login de Usuario en App Móvil

---

## 1. Descripción General

| Campo | Detalle |
|-------|---------|
| **Historia de Usuario** | Como usuario quiero ingresar a una app móvil por medio de login para poder utilizar los servicios de la app |
| **Funcionalidad** | Autenticación de usuario mediante credenciales (email y contraseña) |
| **Módulo** | Auth / Login |
| **Prioridad** | Alta |

---

## 2. Pantalla de Login

### 2.1 Elementos de la Interfaz

| Elemento | Descripción | Tipo |
|----------|-------------|------|
| Campo Email | Input para correo electrónico del usuario | TextField |
| Campo Contraseña | Input para contraseña (oculta por defecto) | TextField (Password) |
| Checkbox "Recordar usuario" | Mantiene la sesión iniciada | Checkbox |
| Botón "Iniciar Sesión" | Ejecuta la autenticación | Button |
| Enlace "¿Olvidaste tu contraseña?" | Redirige a recuperación | Link |
| Enlace "Crear cuenta" | Redirige a registro | Link |
| Indicador de carga | Muestra estado de procesamiento | Spinner/Loader |
| Mensaje de error | Muestra errores de validación/autenticación | Alert/Banner |

### 2.2 Estados de la Pantalla

| Estado | Descripción |
|--------|-------------|
| **Idle** | Campos vacíos, listos para ingreso |
| **Ingresando datos** | Usuario escribe en campos (validación en tiempo real) |
| **Cargando** | Spinner visible, campos deshabilitados |
| **Error** | Mensaje de error visible |
| **Éxito** | Transición hacia dashboard |

---

## 3. Validaciones

### 3.1 Validación en Tiempo Real

| Campo | Regla | Mensaje de Error |
|-------|-------|------------------|
| Email | Formato válido (contiene @ y dominio) | "Ingresa un correo electrónico válido" |
| Contraseña | Mínimo 8 caracteres | "La contraseña debe tener al menos 8 caracteres" |

### 3.2 Validación en Envío

| Condición | Mensaje de Error |
|-----------|------------------|
| Campos vacíos | "Por favor completa todos los campos" |
| Email mal formateado | "Ingresa un correo electrónico válido" |
| Contraseña < 8 caracteres | "La contraseña debe tener al menos 8 caracteres" |

---

## 4. Happy Path (Flujo Exitoso)

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO EXITOSO                             │
└─────────────────────────────────────────────────────────────┘

1.  Actor abre la app
    → Pantalla de Login displayed

2.  Actor ingresa email "usuario@example.com"
    → Campo se llena, validación en tiempo real: ✓

3.  Actor ingresa contraseña "MiPassword123"
    → Campo se llena, caracteres ocultos (••••••••), validación: ✓

4.  Actor marca checkbox "Recordar usuario"
    → Checkbox seleccionado

5.  Actor presionar botón "Iniciar Sesión"
    → Spinner aparece, campos se deshabilitan, botón muestra "Ingresando..."

6.  Sistema valida credenciales contra backend
    → 200 OK recibido

7.  Sistema almacena sesión (token/session)
    → Sesión persistente activada

8.  Sistema redirige a Dashboard
    → Pantalla principal de la app displayed

─────────────────────────────────────────────────────────────
```

---

## 5. Sad Path (Flujos de Error)

### 5.1 Credenciales Inválidas

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa email                   Campo se llena
2.    Usuario ingresa contraseña incorrecta    Campo se llena
3.    Usuario presiona "Iniciar Sesión"       Spinner aparece
4.    Backend responde 401 Unauthorized        Spinner desaparece
5.    Mensaje error aparece: "Credenciales    Banner rojo visible
      incorrectas. Por favor verifica tu        Campo email mantiene
      email y contraseña"                      valor
                                              Campo contraseña se limpia
6.    Usuario puede reintentar                Ciclo vuelve al paso 1
```

### 5.2 Campos Vacíos en Envío

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario deja campos vacíos              -
2.    Usuario presiona "Iniciar Sesión"       Validación executed
3.    Mensaje error aparece: "Por favor       Campos se remarcan
      completa todos los campos"              como requeridos
```

### 5.3 Formato de Email Inválido

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "correoinvalido"         -
2.    Validación en tiempo real ejecuta       Línea roja bajo campo
3.    Mensaje: "Ingresa un correo             Icono X aparece
      electrónico válido"
4.    Botón "Iniciar Sesión" deshabilitado    hasta validación OK
```

### 5.4 Contraseña Muy Corta

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "pass" (4 chars)        -
2.    Validación en tiempo real ejecuta       Línea roja bajo campo
3.    Mensaje: "La contraseña debe tener      Icono X aparece
      al menos 8 caracteres"
4.    Botón "Iniciar Sesión" deshabilitado    hasta validación OK
```

### 5.5 Error de Conexión / Timeout

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa credenciales válidas    -
2.    Usuario presiona "Iniciar Sesión"       Spinner aparece
3.    Timeout 30s sin respuesta                Spinner desaparece
4.    Mensaje: "Error de conexión.            Banner visible
      Verifica tu conexión a internet"         
5.    Usuario puede reintentar                -
```

---

## 6. BDD - Behavior Driven Development

### Feature: Login de Usuario

---

#### Scenario: Login exitoso con credenciales válidas

**Given** el usuario está en la pantalla de Login
**And** no existe una sesión activa
**When** el usuario ingresa "usuario@example.com" en el campo email
**And** el usuario ingresa "MiPassword123" en el campo contraseña
**And** el usuario marca la opción "Recordar usuario"
**And** el usuario presiona el botón "Iniciar Sesión"
**Then** el sistema muestra un indicador de carga
**And** el sistema valida las credenciales contra el backend
**And** el sistema almacena el token de sesión
**And** el sistema redirige al usuario a la pantalla Dashboard

---

#### Scenario: Login fallido con credenciales incorrectas

**Given** el usuario está en la pantalla de Login
**When** el usuario ingresa "usuario@example.com" en el campo email
**And** el usuario ingresa "passwordIncorrecto" en el campo contraseña
**And** el usuario presiona el botón "Iniciar Sesión"
**Then** el sistema muestra un indicador de carga
**And** el backend retorna error 401
**And** el sistema muestra el mensaje "Credenciales incorrectas. Por favor verifica tu email y contraseña"
**And** el campo email mantiene el valor ingresado
**And** el campo contraseña se limpia
**And** el usuario puede intentar nuevamente

---

#### Scenario: Login con email mal formateado

**Given** el usuario está en la pantalla de Login
**When** el usuario ingresa "correoinvalido" en el campo email
**Then** el sistema muestra validación en tiempo real fallida
**And** el sistema muestra el mensaje "Ingresa un correo electrónico válido"
**And** el botón "Iniciar Sesión" permanece deshabilitado

---

#### Scenario: Login con contraseña muy corta

**Given** el usuario está en la pantalla de Login
**When** el usuario ingresa "pass" en el campo contraseña
**Then** el sistema muestra validación en tiempo real fallida
**And** el sistema muestra el mensaje "La contraseña debe tener al menos 8 caracteres"
**And** el botón "Iniciar Sesión" permanece deshabilitado

---

#### Scenario: Login con campos vacíos

**Given** el usuario está en la pantalla de Login
**When** el usuario deja los campos vacíos
**And** el usuario presiona el botón "Iniciar Sesión"
**Then** el sistema muestra el mensaje "Por favor completa todos los campos"

---

#### Scenario: Login con error de conexión

**Given** el usuario está en la pantalla de Login
**And** el dispositivo tiene conexión a internet fallida
**When** el usuario ingresa "usuario@example.com" en el campo email
**And** el usuario ingresa "MiPassword123" en el campo contraseña
**And** el usuario presiona el botón "Iniciar Sesión"
**Then** el sistema muestra un indicador de carga
**And** el sistema agota el tiempo de espera (timeout 30s)
**And** el sistema muestra el mensaje "Error de conexión. Verifica tu conexión a internet"
**And** el usuario puede intentar nuevamente

---

#### Scenario: Usuario olvida su contraseña

**Given** el usuario está en la pantalla de Login
**When** el usuario presiona el enlace "¿Olvidaste tu contraseña?"
**Then** el sistema redirige a la pantalla de recuperación de contraseña

---

#### Scenario: Nuevo usuario desea crear cuenta

**Given** el usuario está en la pantalla de Login
**When** el usuario presiona el enlace "Crear cuenta"
**Then** el sistema redirige a la pantalla de registro

---

#### Scenario: Usuario pide ser lembrado

**Given** el usuario está en la pantalla de Login
**When** el usuario marca el checkbox "Recordar usuario"
**Then** el sistema almacena la preferencia de sesión persistente
**And** en próximos accesos, el usuario permanecerá logueado automáticamente

---

## 7. Requisitos Técnicos

| Requisito | Detalle |
|-----------|---------|
| **Tiempo de espera (timeout)** | 30 segundos máximo |
| **Persistencia de sesión** | Token almacenado en lugar seguro del dispositivo |
| **Seguridad** | Contraseña nunca almacenada en texto plano |
| **UX** | Validación en tiempo real para mejor retroalimentación |

---

## 8. Diagrama de Estados

```
        ┌──────────┐
        │   IDLE   │
        └────┬─────┘
             │ usuario ingresa datos
             ▼
        ┌──────────┐
        │ INPUT    │◄─────────┐
        └────┬─────┘          │
             │ presiona login │
             ▼                │
        ┌──────────┐          │
        │ LOADING  │          │
        └────┬─────┘          │
             │                │
       ┌─────┴─────┐          │
       │           │          │
       ▼           ▼          │
  ┌────────┐  ┌────────┐      │
  │ SUCCESS│  │ ERROR  │──────┘
  └────────┘  └────────┘  reintentar
```

---

*Documento generado: Especificación de Login - App Móvil*