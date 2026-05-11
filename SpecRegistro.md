# Especificación: Registro de Usuario en App Móvil

---

## 1. Descripción General

| Campo | Detalle |
|-------|---------|
| **Historia de Usuario** | Como nuevo usuario necesito crear una cuenta en la app móvil para tener mis propias credenciales y comenzar a trabajar dentro de la app |
| **Funcionalidad** | Registro de nuevos usuarios con datos personales y validación de RUT chileno |
| **Módulo** | Auth / Registro |
| **Prioridad** | Alta |

---

## 2. Pantalla de Registro

### 2.1 Elementos de la Interfaz

| Elemento | Descripción | Tipo |
|----------|-------------|------|
| Campo Nombre | Primer nombre del usuario | TextField |
| Campo Apellido | Apellido Paterno | TextField |
| Campo RUT | RUN chileno con dígito verificador | TextField |
| Campo Ciudad | Ciudad de residencia con autocomplete | Autocomplete |
| Campo Fecha de Nacimiento | Selector de fecha | Date Picker |
| Campo AFP | Fondo de pensiones | Dropdown |
| Campo Previsión de Salud | Sistema de salud | Dropdown (Fonasa/Isapre/Otro) |
| Campo Isapre (secundario) | Nombre de isapre | Dropdown (solo si Prev=Isapre) |
| Campo Estado Civil | Estado civil del usuario | Dropdown |
| Campo Email | Correo electrónico | TextField |
| Campo Contraseña | Contraseña de acceso | Password |
| Campo Confirmar Contraseña | Repetir contraseña | Password |
| Checkbox Términos | Aceptación de términos | Checkbox |
| Botón "Crear Cuenta" | Envía el formulario | Button |
| Enlace "Ya tengo cuenta" | Redirige a login | Link |
| Indicador de carga | Muestra procesamiento | Spinner/Loader |
| Mensajes de error | Validación de campos | Alert/Banner |

### 2.2 Estados de la Pantalla

| Estado | Descripción |
|--------|-------------|
| **Idle** | Formulario vacío, listo para ingreso |
| **Ingresando datos** | Usuario llena campos (validación en tiempo real) |
| **Cargando** | Spinner visible, campos deshabilitados |
| **Error** | Mensaje de error visible |
| **Éxito** | Pantalla de confirmación mostrada |

---

## 3. Campos del Formulario - Especificaciones Detalladas

### 3.1 Datos Personales

| Campo | Obligatorio | Tipo Input | Validación | Mensaje Error |
|-------|-------------|------------|------------|---------------|
| Nombre | Sí | TextField | No vacío | "El nombre es obligatorio" |
| Apellido | Sí | TextField | No vacío | "El apellido es obligatorio" |
| RUT | Sí | TextField | Algoritmo módulo 11 | "El RUT no es válido" |
| Fecha de Nacimiento | Sí | Date Picker | Mayor de 18 años | "Debes ser mayor de 18 años para registrarte" |
| Estado Civil | Sí | Dropdown | Selección válida | "Selecciona tu estado civil" |

### 3.2 Ubicación y Afiliación

| Campo | Obligatorio | Tipo Input | Opciones/Validación |
|-------|-------------|------------|---------------------|
| Ciudad | Sí | Autocomplete | Lista de ciudades chilenas |
| AFP | No | Dropdown | Capital, Cuprum, Habitat, Prima, Prognus, Modelo, Uno, otra |
| Previsión de Salud | Sí | Dropdown | Fonasa, Isapre, Ninguno |
| Isapre | Condicional | Dropdown | Solo visible si Prev=Isapre. Lista de isapres chilenas |

### 3.3 Credenciales

| Campo | Obligatorio | Tipo Input | Validación | Mensaje Error |
|-------|-------------|------------|------------|---------------|
| Email | Sí | TextField | Formato válido + único | "El email no es válido" / "Este email ya está registrado" |
| Contraseña | Sí | Password | Min 8 chars, 1 mayúscula, 1 número | "La contraseña debe tener mínimo 8 caracteres, una mayúscula y un número" |
| Confirmar Contraseña | Sí | Password | Debe coincidir con contraseña | "Las contraseñas no coinciden" |

### 3.4 Términos y Condiciones

| Campo | Obligatorio | Tipo Input | Mensaje |
|-------|-------------|------------|---------|
| Acepto Términos | Sí | Checkbox | "Debes aceptar los términos y condiciones" |

---

## 4. Validaciones en Tiempo Real

### 4.1 RUT - Algoritmo Módulo 11

```
Pasos de validación:
1. Separar cuerpo (sin puntos ni guión) y dígito verificador
2. El cuerpo debe tener 7-8 dígitos
3. Multiplicar cada dígito por la secuencia 2,3,4,5,6,7,2,3,4,5... (de derecha a izquierda)
4. Sumar los productos
5. Calcular: 11 - (suma mod 11)
6. Si resultado = 10 → digit = "K"
7. Si resultado = 11 → digit = "0"
8. Comparar con dígito verificador ingresado
```

### 4.2 Email

- Debe contener exactamente una @
- Debe tener dominio con al menos un punto
- Dominio debe tener al menos 2 caracteres después del último punto

### 4.3 Contraseña

- Mínimo 8 caracteres
- Al menos 1 letra mayúscula (A-Z)
- Al menos 1 número (0-9)

### 4.4 Fecha de Nacimiento

- No puede ser futura
- Debe ser al menos 18 años antes de la fecha actual

---

## 5. Happy Path (Flujo Exitoso)

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO EXITOSO                             │
└─────────────────────────────────────────────────────────────┘

1.  Actor abre la app
    → Pantalla de Login displayed
    → Actor presiona enlace "Crear cuenta"

2.  Actor presiona "Crear cuenta"
    → Pantalla de Registro displayed

3.  Actor completa campos obligatorios:
    - Nombre: "Juan"
    - Apellido: "Pérez"
    - RUT: "12.345.678-5"
    - Ciudad: "Santiago" (autocomplete)
    - Fecha Nacimiento: "15/03/1990"
    - AFP: "Capital"
    - Previsión: "Fonasa"
    - Estado Civil: "Soltero"
    - Email: "juan.perez@gmail.com"
    - Contraseña: "MiPass123"
    - Confirmar: "MiPass123"

4.  Actor acepta términos checkbox

5.  Actor presiona botón "Crear Cuenta"
    → Spinner aparece, campos se deshabilitan

6.  Sistema valida todos los campos
    → Validación exitosa

7.  Sistema envía email de verificación
    → Email enviado a juan.perez@gmail.com

8.  Sistema muestra pantalla de éxito:
    → "Tu cuenta ha sido creada"
    → "Te enviamos un enlace de verificación a tu email"
    → "Revisa tu bandeja de entrada y activa tu cuenta"

9.  Actor revisa email y presiona enlace de verificación
    → Sistema valida token del enlace

10. Sistema activa la cuenta
    → Redirige a Login

11. Actor ingresa credenciales:
    - Email: "juan.perez@gmail.com"
    - Contraseña: "MiPass123"

12. Actor presiona "Iniciar Sesión"
    → Acceso exitoso a la app

─────────────────────────────────────────────────────────────
```

---

## 6. Sad Path (Flujos de Error)

### 6.1 RUT Inválido

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa RUT "12.345.678-9"     -
2.    Validación módulo 11 falla              Línea roja bajo campo
3.    Mensaje: "El RUT no es válido"         Icono X aparece
4.    Botón "Crear Cuenta" deshabilitado     hasta validación OK
```

### 6.2 Email Ya Registrado

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa email                   -
      "usuario@ejemplo.com"                   
2.    Sistema verifica uniqueness            ✓ (no existe)
3.    Validación exitosa                      ✓
4.    ... posterior al envío ...
5.    Backend retorna 409 Conflict            Mensaje error displayed
6.    Mensaje: "Este email ya está           
      registrado. ¿Ya tienes una cuenta?      
      Inicia sesión"                          
7.    Enlace "Inicia sesión" visible          -
```

### 6.3 Contraseña No Cumple Requisitos

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "password" (sin        -
      mayúscula ni número)                   
2.    Validación en tiempo real ejecuta       Línea roja bajo campo
3.    Mensaje: "La contraseña debe tener     Icono X aparece
      mínimo 8 caracteres, una mayúscula      
      y un número"
4.    Botón deshabilitado                     -
```

### 6.4 Contraseñas No Coinciden

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario ingresa "MiPass123"            Campo confirmado vacío
2.    Usuario ingresa "MiPass456"            -
3.    Validación ejecuta al escribir          Línea roja bajo campo
4.    Mensaje: "Las contraseñas no            Icono X aparece
      coinciden"
5.    Botón deshabilitado                     -
```

### 6.5 Menor de 18 Años

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario selecciona fecha                -
      "15/03/2015" (tiene 11 años)            
2.    Validación de edad falla                Línea roja bajo campo
3.    Mensaje: "Debes ser mayor de 18        Icono X aparece
      años para registrarte"
4.    Botón deshabilitado                     -
```

### 6.6 Previsión Isapre Sin Selección

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario selecciona "Isapre"            Dropdown Isapre aparece
2.    Usuario deja dropdown Isapre vacío      -
3.    Al intentar enviar                      Validación falla
4.    Mensaje: "Selecciona tu isapre"        Campo se marca rojo
5.    Botón deshabilitado                     -
```

### 6.7 Términos No Aceptados

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario completa todos los campos       -
2.    Usuario NO marca checkbox términos       -
3.    Usuario presiona "Crear Cuenta"         Spinner no aparece
4.    Mensaje: "Debes aceptar los términos    Checkbox marcado en rojo
      y condiciones"
```

### 6.8 Error de Conexión en Registro

```
PASO  ACCIÓN                                  RESULTADO
─────────────────────────────────────────────────────────────
1.    Usuario completa todos los campos       -
2.    Usuario presiona "Crear Cuenta"         Spinner aparece
3.    Timeout 30s sin respuesta                Spinner desaparece
4.    Mensaje: "Error de conexión.            Banner visible
      Verifica tu conexión a internet"
5.    Usuario puede reintentar                 -
```

---

## 7. BDD - Behavior Driven Development

### Feature: Registro de Usuario

---

#### Scenario: Registro exitoso con todos los campos

**Given** el usuario está en la pantalla de Registro
**And** no existe una cuenta con email "juan.perez@gmail.com"
**When** el usuario ingresa "Juan" en el campo nombre
**And** el usuario ingresa "Pérez" en el campo apellido
**And** el usuario ingresa "12.345.678-5" en el campo RUT
**And** el usuario selecciona "Santiago" en el campo ciudad (autocomplete)
**And** el usuario selecciona "15/03/1990" en el campo fecha de nacimiento
**And** el usuario selecciona "Capital" en el campo AFP
**And** el usuario selecciona "Fonasa" en el campo previsión de salud
**And** el usuario selecciona "Soltero" en el campo estado civil
**And** el usuario ingresa "juan.perez@gmail.com" en el campo email
**And** el usuario ingresa "MiPass123" en el campo contraseña
**And** el usuario ingresa "MiPass123" en el campo confirmar contraseña
**And** el usuario marca el checkbox de términos y condiciones
**And** el usuario presiona el botón "Crear Cuenta"
**Then** el sistema muestra un indicador de carga
**And** el sistema valida todos los campos correctamente
**And** el sistema crea la cuenta en el backend
**And** el sistema envía un email de verificación a "juan.perez@gmail.com"
**And** el sistema muestra la pantalla de éxito con mensaje de verificación

---

#### Scenario: Registro con Isapre y campos secundarios

**Given** el usuario está en la pantalla de Registro
**When** el usuario completa todos los campos obligatorios
**And** el usuario selecciona "Isapre" en el campo previsión de salud
**Then** el sistema muestra el dropdown de isapre
**When** el usuario selecciona "Consalud" en el campo isapre
**And** el usuario completa los campos de credenciales
**And** el usuario acepta los términos
**And** el usuario presiona el botón "Crear Cuenta"
**Then** el sistema procesa el registro correctamente
**And** la cuenta se crea con Isapre "Consalud"

---

#### Scenario: Registro fallido por RUT inválido

**Given** el usuario está en la pantalla de Registro
**When** el usuario ingresa "12.345.678-9" en el campo RUT
**Then** el sistema ejecuta validación módulo 11
**And** la validación falla
**And** el sistema muestra el mensaje "El RUT no es válido"
**And** el botón "Crear Cuenta" permanece deshabilitado

---

#### Scenario: Registro fallido por email duplicado

**Given** el usuario está en la pantalla de Registro
**And** ya existe una cuenta con email "usuario@ejemplo.com"
**When** el usuario ingresa "usuario@ejemplo.com" en el campo email
**And** el usuario completa todos los demás campos
**And** el usuario presiona el botón "Crear Cuenta"
**Then** el sistema muestra un indicador de carga
**And** el backend retorna error 409 Conflict
**And** el sistema muestra el mensaje "Este email ya está registrado. ¿Ya tienes una cuenta? Inicia sesión"

---

#### Scenario: Registro fallido por contraseña débil

**Given** el usuario está en la pantalla de Registro
**When** el usuario ingresa "password" en el campo contraseña
**Then** el sistema valida en tiempo real
**And** la validación falla (falta mayúscula y número)
**And** el sistema muestra el mensaje "La contraseña debe tener mínimo 8 caracteres, una mayúscula y un número"
**And** el botón "Crear Cuenta" permanece deshabilitado

---

#### Scenario: Registro fallido por contraseñas no coinciden

**Given** el usuario está en la pantalla de Registro
**When** el usuario ingresa "MiPass123" en el campo contraseña
**And** el usuario ingresa "MiPass456" en el campo confirmar contraseña
**Then** el sistema muestra validación fallida
**And** el sistema muestra el mensaje "Las contraseñas no coinciden"
**And** el botón "Crear Cuenta" permanece deshabilitado

---

#### Scenario: Registro fallido por menor de edad

**Given** el usuario está en la pantalla de Registro
**When** el usuario selecciona fecha de nacimiento "15/03/2015"
**Then** el sistema calcula la edad: 11 años
**And** la validación falla
**And** el sistema muestra el mensaje "Debes ser mayor de 18 años para registrarte"
**And** el botón "Crear Cuenta" permanece deshabilitado

---

#### Scenario: Registro fallido por términos no aceptados

**Given** el usuario está en la pantalla de Registro
**And** todos los campos están correctamente completados
**When** el usuario NO marca el checkbox de términos y condiciones
**And** el usuario presiona el botón "Crear Cuenta"
**Then** el sistema no envía el formulario
**And** el sistema muestra el mensaje "Debes aceptar los términos y condiciones"

---

#### Scenario: Registro fallido por isapre no seleccionada

**Given** el usuario está en la pantalla de Registro
**And** el usuario selecciona "Isapre" en el campo previsión de salud
**And** el dropdown de isapre aparece
**When** el usuario deja el campo isapre vacío
**And** el usuario presiona el botón "Crear Cuenta"
**Then** el sistema muestra el mensaje "Selecciona tu isapre"
**And** el botón permanece deshabilitado

---

#### Scenario: Verificación exitosa de email

**Given** el usuario ha completado el registro
**And** el sistema ha enviado un email de verificación
**When** el usuario abre el email y presiona el enlace de verificación
**Then** el sistema valida el token del enlace
**And** el sistema activa la cuenta
**And** el sistema redirige al usuario a la pantalla de Login

---

#### Scenario: Link de verificación expirado

**Given** el usuario recibió un email de verificación
**And** el link ha expirado (más de 24 horas)
**When** el usuario presiona el enlace de verificación
**Then** el sistema muestra mensaje de error
**And** el sistema indica: "El enlace ha expirado. Solicita uno nuevo"

---

## 8. Requisitos Técnicos

| Requisito | Detalle |
|-----------|---------|
| **Tiempo de espera (timeout)** | 30 segundos máximo |
| **Validez del enlace de verificación** | 24 horas |
| **Seguridad** | Contraseña hasheada, nunca almacenada en texto plano |
| **RUT** | Formato con puntos y guión, validación algoritmo módulo 11 |
| **Edad mínima** | 18 años |
| **Isapres Chilenas** | Consalud, Banmédica, Vida Tres, Colmena, Masvida, Chuquicamata, Río Blanco |

---

## 9. Diagrama de Estados

```
        ┌──────────┐
        │   IDLE   │
        └────┬─────┘
             │ usuario llena campos
             ▼
        ┌──────────┐
        │  INPUT   │◄────────────┐
        └────┬─────┘             │
             │ presiona crear    │
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

## 10. Lista de Isapres Chilenas

1. Consalud
2. Banmédica
3. Vida Tres
4. Colmena Golden Cross
5. Masvida
6. Chuquicamata
7. Río Blanco
8. Magna
9. Fossal
10. San Alfonso

---

## 11. Lista de AFPs Chilenas

1. Capital
2. Cuprum
3. Habitat
4. Prima
5. ProVida
6. Modelo
7. UNO
8. Río

---

*Documento generado: Especificación de Registro - App Móvil*