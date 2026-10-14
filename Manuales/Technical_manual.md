# Manual Técnico - AlertaMujer

**Versión:** 1.0  
**Fecha:** 5 de octubre de 2026  
**Audiencia:** Desarrolladores, Arquitectos de Software, Ingenieros de DevOps  
**Nivel Técnico:** Profesional

---

## Tabla de Contenidos

1. [Arquitectura del Sistema](#1-arquitectura-del-sistema)
2. [Estructura del Código](#2-estructura-del-código)
3. [Diagramas de Secuencia](#3-diagramas-de-secuencia)
4. [Documentación de API Endpoints](#4-documentación-de-api-endpoints)
5. [Guía de Desarrollo](#5-guía-de-desarrollo)
6. [Base de Datos](#6-base-de-datos)
7. [Seguridad](#7-seguridad)
8. [Despliegue](#8-despliegue)

---

## 1. Arquitectura del Sistema

### 1.1 Visión General

AlertaMujer es una aplicación de seguridad personal compuesta por tres componentes principales:

- **Backend:** API RESTful monolítica modular construida con Spring Boot 3.2.0 y Java 17
- **Frontend Web:** Aplicación SPA construida con Angular 21.2.0 y TypeScript 5.9.2
- **Frontend Móvil:** Aplicación React Native con Expo SDK ~54.0.33

### 1.2 Arquitectura Monolítica Modular

Actualmente, el backend está implementado como un monolito modular organizado por dominios de negocio. Esta es una arquitectura temporal diseñada para evolucionar hacia microservicios.

```
┌─────────────────────────────────────────────────────────────┐
│                    AlertaMujer Backend                         │
│                  (Spring Boot Monolith)                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Identity    │  │   Alerts     │  │   Devices    │      │
│  │  Module      │  │   Module     │  │   Module     │      │
│  │              │  │              │  │              │      │
│  │ - Auth       │  │ - Alerts     │  │ - Device     │      │
│  │ - Users      │  │ - Contacts   │  │ - Sessions   │      │
│  │ - Account    │  │ - Evidence   │  │ - Perms     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Zones     │  │  Resources  │  │    Admin     │      │
│  │   Module     │  │   Module     │  │   Module     │      │
│  │              │  │              │  │              │      │
│  │ - Zones      │  │ - Resources │  │ - Audit      │      │
│  │ - Reports    │  │ - Calls      │  │ - Reports    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                  Shared Infrastructure                         │
│  - Security (JWT, BCrypt)                                     │
│  - Database (PostgreSQL + JPA)                               │
│  - Configuration (Environment Variables)                       │
│  - Logging                                                    │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Arquitectura de Dominios (DDD)

Cada módulo sigue los principios de Domain-Driven Design (DDD):

```
com.alertamujer.{domain}
├── application
│   ├── service          # Lógica de negocio
│   └── dto              # Data Transfer Objects
├── domain
│   ├── model            # Entidades del dominio
│   └── repository       # Interfaces de repositorio
└── infrastructure
    ├── controller       # Controladores REST
    ├── repository       # Implementaciones JPA
    └── config           # Configuración específica
```

### 1.4 Arquitectura de Frontend Web

El frontend web sigue una arquitectura basada en componentes con Angular:

```
src/app/
├── core/                    # Módulo principal
│   ├── auth/               # Autenticación
│   ├── guards/             # Guards de ruta
│   ├── interceptors/       # Interceptores HTTP
│   ├── layouts/            # Layouts reutilizables
│   ├── models/             # Modelos de datos
│   ├── services/           # Servicios API
│   └── settings/           # Configuración
├── features/               # Funcionalidades específicas
│   ├── authentication/     # Login, registro
│   ├── dashboard/          # Panel de usuario
│   ├── admin/              # Panel de administrador
│   └── ...
└── shared/                  # Componentes compartidos
    ├── components/         # Componentes UI
    ├── pipes/              # Pipes personalizados
    └── directives/         # Directivas
```

### 1.5 Arquitectura de Frontend Móvil

El frontend móvil usa React Native con Expo:

```
src/
├── features/               # Funcionalidades
│   ├── authentication/    # Autenticación
│   ├── BotonActivacion/    # Botón de pánico
│   ├── contacts/           # Contactos
│   ├── map/                # Mapa
│   └── ...
├── navigation/             # Navegación
│   ├── AppNavigator.tsx    # Navegador principal
│   ├── AuthNavigator.tsx   # Navegación auth
│   └── MainNavigator.tsx   # Navegación principal
├── services/               # Servicios
│   ├── api.ts              # Cliente Axios
│   ├── auth.service.ts     # Servicio de auth
│   └── ...
├── context/                # Contextos React
│   ├── AuthContext.tsx     # Contexto de auth
│   └── ...
└── components/             # Componentes UI
```

### 1.6 Flujo de Arquitectura Completa

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Web App   │     │ Mobile App  │     │   Admin     │
│  (Angular)  │     │ (React Nat) │     │  (Angular)  │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │ HTTP/REST         │ HTTP/REST         │ HTTP/REST
       │ JWT Auth          │ JWT Auth          │ JWT Auth
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Backend   │
                    │ (Spring Boot)│
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ PostgreSQL  │
                    │  Database   │
                    └─────────────┘
```

---

## 2. Estructura del Código

### 2.1 Estructura del Backend

```
Backend-ALERTA-MUJER/
├── backend-monolith/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/alertamujer/
│   │       │       ├── AlertaMujerBackendApplication.java
│   │       │       ├── identity/
│   │       │       │   ├── application/
│   │       │       │   │   └── service/
│   │       │       │   │       └── AuthService.java
│   │       │       │   ├── domain/
│   │       │       │   │   ├── model/
│   │       │       │   │   │   ├── User.java
│   │       │       │   │   │   ├── Account.java
│   │       │       │   │   │   └── UserProfile.java
│   │       │       │   │   └── repository/
│   │       │       │   │       ├── UserRepository.java
│   │       │       │   │       └── AccountRepository.java
│   │       │       │   └── infrastructure/
│   │       │       │       ├── controller/
│   │       │       │       │   └── AuthController.java
│   │       │       │       ├── repository/
│   │       │       │       │   ├── UserRepositoryImpl.java
│   │       │       │       │   └── AccountRepositoryImpl.java
│   │       │       │       └── config/
│   │       │       │           ├── SecurityConfig.java
│   │       │       │           └── JwtAuthenticationFilter.java
│   │       │       ├── alerts/
│   │       │       │   ├── application/
│   │       │       │   │   └── service/
│   │       │       │   │       ├── AlertService.java
│   │       │       │   │       └── EmergencyContactService.java
│   │       │       │   ├── domain/
│   │       │       │   │   ├── model/
│   │       │       │   │   │   ├── Alert.java
│   │       │       │   │   │   ├── EmergencyContact.java
│   │       │       │   │   │   ├── LocationLog.java
│   │       │       │   │   │   └── Evidence.java
│   │       │       │   │   └── repository/
│   │       │       │   │       ├── AlertRepository.java
│   │       │       │   │       └── EmergencyContactRepository.java
│   │       │       │   └── infrastructure/
│   │       │       │       ├── controller/
│   │       │       │       │   ├── AlertController.java
│   │       │       │       │   └── EmergencyContactController.java
│   │       │       │       └── repository/
│   │       │       │           └── ...
│   │       │       ├── devices/
│   │       │       ├── zones/
│   │       │       ├── resources/
│   │       │       └── admin/
│   │       └── resources/
│   │           ├── application.properties
│   │           └── application-dev.properties
│   ├── pom.xml
│   └── docker/
│       └── Dockerfile
├── db/
│   └── init.sql
├── docker-compose.yml
├── .env.example
└── README.md
```

### 2.2 Estructura del Frontend Web

```
Frond-end-Web/alerta-mujer-web/
├── src/
│   ├── app/
│   │   ├── app.config.ts
│   │   ├── app.routes.ts
│   │   ├── app.component.ts
│   │   ├── core/
│   │   │   ├── auth/
│   │   │   │   ├── auth.guard.ts
│   │   │   │   ├── auth.service.ts
│   │   │   │   └── auth.interceptor.ts
│   │   │   ├── guards/
│   │   │   │   ├── auth.guard.ts
│   │   │   │   ├── user.guard.ts
│   │   │   │   └── admin.guard.ts
│   │   │   ├── interceptors/
│   │   │   │   └── jwt.interceptor.ts
│   │   │   ├── layouts/
│   │   │   │   ├── public-layout/
│   │   │   │   ├── dashboard-layout/
│   │   │   │   └── admin-layout/
│   │   │   ├── models/
│   │   │   │   ├── user.model.ts
│   │   │   │   ├── alert.model.ts
│   │   │   │   └── ...
│   │   │   └── services/
│   │   │       ├── alerts.services.ts
│   │   │       ├── emergency-contact.service.ts
│   │   │       └── ...
│   │   ├── features/
│   │   │   ├── authentication/
│   │   │   ├── dashboard/
│   │   │   └── admin/
│   │   └── shared/
│   │       ├── components/
│   │       └── pipes/
│   ├── environments/
│   │   ├── environment.ts
│   │   └── environment.prod.ts
│   └── index.html
├── package.json
├── angular.json
├── tsconfig.json
└── .env.example
```

### 2.3 Estructura del Frontend Móvil

```
AlertaMujer/Frontend-movil/
├── src/
│   ├── App.tsx
│   ├── features/
│   │   ├── authentication/
│   │   │   ├── login.tsx
│   │   │   ├── register.tsx
│   │   │   └── forgot-password.tsx
│   │   ├── BotonActivacion/
│   │   │   ├── view/
│   │   │   │   └── activacionScreen.tsx
│   │   │   └── activacion.tsx
│   │   ├── contacts/
│   │   ├── map/
│   │   ├── resources/
│   │   └── settings/
│   ├── navigation/
│   │   ├── AppNavigator.tsx
│   │   ├── AuthNavigator.tsx
│   │   └── MainNavigator.tsx
│   ├── services/
│   │   ├── api.ts
│   │   ├── auth.service.ts
│   │   └── location.service.ts
│   ├── context/
│   │   └── AuthContext.tsx
│   └── components/
├── app.json
├── package.json
├── tsconfig.json
└── .env.example
```

### 2.4 Patrones de Diseño Implementados

**Backend:**
- **Repository Pattern:** Abstracción del acceso a datos
- **Service Layer:** Lógica de negocio separada de controladores
- **DTO Pattern:** Transferencia de datos entre capas
- **Dependency Injection:** Inyección de dependencias con Spring

**Frontend Web:**
- **Component Pattern:** Componentes reutilizables
- **Service Pattern:** Servicios para lógica de negocio y API
- **Guard Pattern:** Protección de rutas
- **Interceptor Pattern:** Interceptores HTTP para JWT

**Frontend Móvil:**
- **Context Pattern:** Estado global con React Context
- **Navigator Pattern:** Navegación con React Navigation
- **Service Pattern:** Servicios para API y lógica

---

## 3. Diagramas de Secuencia

### 3.1 Diagrama de Secuencia: Registro de Usuario

```
Usuario          Frontend          Backend           Database
  │                 │                 │                 │
  │-- Registrar --->│                 │                 │
  │                 │-- POST /api/auth/register --->│
  │                 │                 │                 │
  │                 │                 |-- Check email exists -->│
  │                 │                 │<-- NO ----------│
  │                 │                 │                 │
  │                 │                 |-- Create User -->│
  │                 │                 │<-- User ID -----│
  │                 │                 │                 │
  │                 │                 |-- Hash password ->│
  │                 │                 │                 │
  │                 │                 |-- Create Account ->│
  │                 │                 │<-- Account -----│
  │                 │                 │                 │
  │                 │                 |-- Generate JWT ->│
  │                 │                 │                 │
  │                 │<-- Token + User --│                 │
  │<-- Success -----│                 │                 │
```

### 3.2 Diagrama de Secuencia: Login

```
Usuario          Frontend          Backend           Database
  │                 │                 │                 │
  │-- Login ------>│                 │                 │
  │                 │-- POST /api/auth/login --->│
  │                 │                 │                 │
  │                 │                 |-- Find User -->│
  │                 │                 │<-- User -------│
  │                 │                 │                 │
  │                 │                 |-- Find Account ->│
  │                 │                 │<-- Account ----│
  │                 │                 │                 │
  │                 │                 |-- Check status ->│
  │                 │                 │<-- active -----│
  │                 │                 │                 │
  │                 │                 |-- Verify BCrypt ->│
  │                 │                 │<-- match ------│
  │                 │                 │                 │
  │                 │                 |-- Update last access ->│
  │                 │                 │                 │
  │                 │                 |-- Generate JWT ->│
  │                 │                 │                 │
  │                 │<-- Token + User --│                 │
  │<-- Success -----│                 │                 │
```

### 3.3 Diagrama de Secuencia: Activación de Alerta

```
Usuario          Mobile App        Backend           Database      Contacts
  │                 │                 │                 │              │
  │-- Long press --->│                 │                 │              │
  │                 │-- Confirm ----->│                 │              │
  │                 │                 │                 │              │
  │                 │-- POST /api/alerts --->│                 │              │
  │                 │                 │                 │              │
  │                 │                 |-- Create Alert -->│              │
  │                 │                 │<-- Alert ID -----│              │
  │                 │                 │                 │              │
  │                 │                 |-- Get GPS ----->│              │
  │                 │                 │<-- Location ----│              │
  │                 │                 │                 │              │
  │                 │                 |-- Log location ->│              │
  │                 │                 │                 │              │
  │                 │                 |-- Get contacts ->│              │
  │                 │                 │<-- Contacts ----│              │
  │                 │                 │                 │              │
  │                 │                 |-- Notify ----->│              │
  │                 │                 │                 │-- Push/SMS -->│
  │                 │                 │                 │              │
  │                 │                 |-- Start tracking interval ->│
  │                 │<-- Alert active --│                 │              │
  │<-- UI update ---│                 │                 │              │
  │                 │-- GPS every 30s -->│                 │              │
  │                 │                 |-- Log location ->│              │
```

### 3.4 Diagrama de Secuencia: Solicitud de Recuperación de Contraseña

```
Usuario          Frontend          Backend           Database      Email
  │                 │                 │                 │           │
  │-- Forgot pwd --->│                 │                 │           │
  │                 │-- POST /api/auth/forgot-password --->│           │
  │                 │                 │                 │           │
  │                 │                 |-- Find User -->│           │
  │                 │                 │<-- User -------│           │
  │                 │                 │                 │           │
  │                 │                 |-- Generate token ->│          │
  │                 │                 │                 │           │
  │                 │                 |-- Create recovery request ->│
  │                 │                 │                 │           │
  │                 │                 |-- Send email ------------->│
  │                 │                 │                 │           │
  │                 │<-- Message -------│                 │           │
  │<-- Check email --│                 │                 │           │
  │                 │                 │                 │           │
  │-- Click link --->│                 │                 │           │
  │                 │-- POST /api/auth/reset-password --->│           │
  │                 │                 |-- Validate token ->│          │
  │                 │                 │<-- valid -------│           │
  │                 │                 |-- Update password ->│         │
  │                 │                 │                 │           │
  │                 │<-- Success ------│                 │           │
  │<-- Success -----│                 │                 │           │
```

### 3.5 Diagrama de Secuencia: Carga de Zonas de Riesgo

```
Usuario          Frontend          Backend           Database
  │                 │                 │                 │
  │-- Open map ---->│                 │                 │
  │                 │-- GET /api/zones/city/{city} --->│
  │                 │                 │                 │
  │                 │                 |-- Query zones -->│
  │                 │                 │<-- Zones ------│
  │                 │                 │                 │
  │                 │<-- Zones --------│                 │
  │<-- Render map --│                 │                 │
  │                 │                 │                 │
  │-- Tap zone ---->│                 │                 │
  │                 │-- GET /api/zones/{id} --->│
  │                 │                 │                 │
  │                 │                 |-- Get details ->│
  │                 │                 │<-- Details ----│
  │                 │                 │                 │
  │                 │<-- Details ------│                 │
  │<-- Show details -│                 │                 │
```

---

## 4. Documentación de API Endpoints

### 4.1 Autenticación

#### POST /api/auth/register
**Descripción:** Registro de nuevo usuario

**Request Body:**
```json
{
  "nombre": "Maria Garcia",
  "email": "maria.garcia@example.com",
  "password": "SecurePass123",
  "telefono": "+573001234567"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "firstName": "Maria",
    "lastName": "Garcia",
    "email": "maria.garcia@example.com",
    "telephone": "+573001234567",
    "roleId": 1
  }
}
```

**Error (409 Conflict):**
```json
{
  "message": "Email already exists"
}
```

---

#### POST /api/auth/login
**Descripción:** Inicio de sesión

**Request Body:**
```json
{
  "email": "maria.garcia@example.com",
  "password": "SecurePass123"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "firstName": "Maria",
    "lastName": "Garcia",
    "email": "maria.garcia@example.com",
    "roleId": 1
  }
}
```

**Error (401 Unauthorized):**
```json
{
  "message": "Invalid credentials"
}
```

**Error (403 Forbidden):**
```json
{
  "message": "Account blocked"
}
```

---

#### POST /api/auth/forgot-password
**Descripción:** Solicitud de recuperación de contraseña

**Request Body:**
```json
{
  "email": "maria.garcia@example.com"
}
```

**Response (200 OK):**
```json
{
  "message": "If the email exists, a reset link will be sent"
}
```

---

#### POST /api/auth/reset-password
**Descripción:** Restablecimiento de contraseña

**Request Body:**
```json
{
  "token": "reset_token_here",
  "newPassword": "NewSecurePass123"
}
```

**Response (200 OK):**
```json
{
  "message": "Password updated successfully"
}
```

---

### 4.2 Alertas

#### GET /api/alerts
**Descripción:** Obtener todas las alertas (requiere JWT, admin)

**Headers:**
```
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "userProfileId": 1,
    "deviceId": 1,
    "alertType": "main",
    "activationMethod": "panic_button",
    "status": "active",
    "message": "Pánico activado",
    "startedAt": "2026-10-05T10:30:00",
    "endedAt": null,
    "cancelledAt": null
  }
]
```

---

#### GET /api/alerts/{id}
**Descripción:** Obtener alerta por ID

**Response (200 OK):**
```json
{
  "id": 1,
  "userProfileId": 1,
  "deviceId": 1,
  "alertType": "main",
  "activationMethod": "panic_button",
  "status": "active",
  "message": "Pánico activado",
  "startedAt": "2026-10-05T10:30:00",
  "endedAt": null,
  "cancelledAt": null
}
```

---

#### GET /api/alerts/user/{userProfileId}
**Descripción:** Obtener alertas de un usuario

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "userProfileId": 1,
    "status": "active",
    "startedAt": "2026-10-05T10:30:00"
  }
]
```

---

#### POST /api/alerts
**Descripción:** Crear nueva alerta

**Request Body:**
```json
{
  "userProfileId": 1,
  "deviceId": 1,
  "alertType": "main",
  "activationMethod": "panic_button",
  "status": "active",
  "message": "Pánico activado",
  "latitude": 4.7110,
  "longitude": -74.0721
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "userProfileId": 1,
  "deviceId": 1,
  "status": "active",
  "startedAt": "2026-10-05T10:30:00"
}
```

---

#### PUT /api/alerts/{id}
**Descripción:** Actualizar alerta

**Request Body:**
```json
{
  "status": "resolved",
  "endedAt": "2026-10-05T11:00:00"
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "status": "resolved",
  "endedAt": "2026-10-05T11:00:00"
}
```

---

#### DELETE /api/alerts/{id}
**Descripción:** Eliminar alerta

**Response (204 No Content)**

---

### 4.3 Contactos de Emergencia

#### GET /api/contacts
**Descripción:** Obtener todos los contactos

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "userProfileId": 1,
    "contactName": "Juan Pérez",
    "telephone": "3111234567",
    "email": "juan@example.com",
    "relationship": "Amigo"
  }
]
```

---

#### GET /api/contacts/user/{userProfileId}
**Descripción:** Obtener contactos de un usuario

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "userProfileId": 1,
    "contactName": "Juan Pérez",
    "telephone": "3111234567",
    "relationship": "Amigo"
  }
]
```

---

#### POST /api/contacts
**Descripción:** Crear contacto de emergencia

**Request Body:**
```json
{
  "userProfileId": 1,
  "contactName": "Juan Pérez",
  "telephone": "3111234567",
  "email": "juan@example.com",
  "relationship": "Amigo"
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "userProfileId": 1,
  "contactName": "Juan Pérez",
  "telephone": "3111234567",
  "email": "juan@example.com",
  "relationship": "Amigo"
}
```

---

#### PUT /api/contacts/{id}
**Descripción:** Actualizar contacto

**Request Body:**
```json
{
  "contactName": "Juan Pérez",
  "telephone": "3111234567",
  "relationship": "Hermano"
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "contactName": "Juan Pérez",
  "telephone": "3111234567",
  "relationship": "Hermano"
}
```

---

#### DELETE /api/contacts/{id}
**Descripción:** Eliminar contacto

**Response (204 No Content)**

---

### 4.4 Zonas de Riesgo

#### GET /api/zones
**Descripción:** Obtener todas las zonas (público)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Parque Santander",
    "zoneType": "risk",
    "riskLevel": "high",
    "description": "Zona de alto riesgo",
    "city": "Neiva",
    "latitude": 4.7110,
    "longitude": -74.0721,
    "radiusMeters": 500.0,
    "isActive": true
  }
]
```

---

#### GET /api/zones/city/{city}
**Descripción:** Filtrar zonas por ciudad

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Parque Santander",
    "city": "Neiva",
    "riskLevel": "high"
  }
]
```

---

#### GET /api/zones/type/{type}
**Descripción:** Filtrar zonas por tipo

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Parque Santander",
    "zoneType": "risk",
    "riskLevel": "high"
  }
]
```

---

#### POST /api/zones
**Descripción:** Crear zona (admin)

**Request Body:**
```json
{
  "name": "Parque Central",
  "zoneType": "risk",
  "riskLevel": "medium",
  "description": "Zona de riesgo medio",
  "city": "Bogotá",
  "latitude": 4.7110,
  "longitude": -74.0721,
  "radiusMeters": 300.0
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "name": "Parque Central",
  "zoneType": "risk",
  "riskLevel": "medium",
  "city": "Bogotá"
}
```

---

### 4.5 Recursos de Emergencia

#### GET /api/resources
**Descripción:** Obtener todos los recursos (público)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Línea de Emergencias 123",
    "resourceType": "emergency_line",
    "telephone": "123",
    "city": "Bogotá",
    "description": "Atención inmediata para cualquier emergencia",
    "isActive": true
  }
]
```

---

#### GET /api/resources/city/{city}
**Descripción:** Filtrar recursos por ciudad

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Línea de Emergencias 123",
    "resourceType": "emergency_line",
    "telephone": "123",
    "city": "Bogotá"
  }
]
```

---

#### GET /api/resources/type/{type}
**Descripción:** Filtrar recursos por tipo

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Línea de Emergencias 123",
    "resourceType": "emergency_line",
    "telephone": "123"
  }
]
```

---

### 4.6 Administración

#### GET /api/admin/users
**Descripción:** Obtener todos los usuarios (admin)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "firstName": "Maria",
    "lastName": "Garcia",
    "email": "maria.garcia@example.com",
    "roleId": 1
  }
]
```

---

#### GET /api/admin/alerts
**Descripción:** Obtener todas las alertas (admin)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "userProfileId": 1,
    "status": "active",
    "startedAt": "2026-10-05T10:30:00"
  }
]
```

---

#### GET /api/admin/audit-logs
**Descripción:** Obtener logs de auditoría

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "accountId": 1,
    "action": "UPDATE_USER",
    "entityType": "User",
    "entityId": 1,
    "description": "Updated user profile",
    "ipAddress": "192.168.1.1",
    "createdAt": "2026-10-05T10:30:00"
  }
]
```

---

#### POST /api/admin/audit-logs
**Descripción:** Crear log de auditoría

**Request Body:**
```json
{
  "accountId": 1,
  "action": "UPDATE_USER",
  "entityType": "User",
  "entityId": 1,
  "description": "Updated user profile",
  "ipAddress": "192.168.1.1"
}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "accountId": 1,
  "action": "UPDATE_USER",
  "entityType": "User",
  "createdAt": "2026-10-05T10:30:00"
}
```

---

### 4.7 Health Check

#### GET /health
**Descripción:** Verificar estado del backend (público)

**Response (200 OK):**
```json
{
  "status": "ok",
  "service": "alerta-mujer-backend",
  "type": "monolithic"
}
```

---

## 5. Guía de Desarrollo

### 5.1 Configuración del Entorno de Desarrollo

#### Requisitos del Backend

- Java 17 o superior
- Maven 3.9+
- Docker y Docker Compose (opcional, recomendado)
- PostgreSQL 16 (si no se usa Docker)
- IDE: IntelliJ IDEA, Eclipse, o VS Code

#### Requisitos del Frontend Web

- Node.js 18+
- npm 9+
- Angular CLI 21.2.11
- IDE: VS Code, WebStorm

#### Requisitos del Frontend Móvil

- Node.js 18+
- npm 9+
- Expo CLI
- Android Studio (para emulador Android)
- Xcode (para simulador iOS, solo macOS)
- Expo Go app (para dispositivo físico)

### 5.2 Configuración del Backend

#### Paso 1: Clonar el Repositorio

```bash
git clone <repository-url>
cd Backend-ALERTA-MUJER
```

#### Paso 2: Configurar Variables de Entorno

```bash
cp .env.example .env
```

Editar `.env`:
```env
POSTGRES_USER=alerta_mujer
POSTGRES_PASSWORD=alerta_mujer_2026
POSTGRES_DB=alerta_mujer
JWT_SECRET=alerta_mujer_jwt_secret_key_2026_xK9mP2qR5sT8uV1wX4yZ7
JWT_EXPIRES_IN=24h
CORS_ORIGIN=*
```

#### Paso 3: Iniciar con Docker (Recomendado)

```bash
docker compose up --build
```

#### Paso 4: Verificar Instalación

```bash
curl http://localhost:8080/health
```

Respuesta esperada:
```json
{
  "status": "ok",
  "service": "alerta-mujer-backend",
  "type": "monolithic"
}
```

### 5.3 Configuración del Frontend Web

#### Paso 1: Navegar al Directorio

```bash
cd Frond-end-Web/alerta-mujer-web
```

#### Paso 2: Instalar Dependencias

```bash
npm install
```

#### Paso 3: Configurar URL del Backend

Crear `.env`:
```env
NG_APP_API_URL=http://localhost:8080
```

#### Paso 4: Iniciar Servidor de Desarrollo

```bash
ng serve
```

El frontend estará disponible en `http://localhost:4200`

### 5.4 Configuración del Frontend Móvil

#### Paso 1: Navegar al Directorio

```bash
cd AlertaMujer/Frontend-movil
```

#### Paso 2: Instalar Dependencias

```bash
npm install
```

#### Paso 3: Configurar URL del Backend

Crear `.env`:
```env
EXPO_PUBLIC_API_URL=http://localhost:8080
```

Para emulador Android:
```env
EXPO_PUBLIC_API_URL=http://10.0.2.2:8080
```

Para dispositivo físico:
```env
EXPO_PUBLIC_API_URL=http://192.168.1.X:8080
```

#### Paso 4: Iniciar Expo

```bash
npx expo start
```

Opciones:
- `a` - Abrir en emulador Android
- `i` - Abrir en simulador iOS (macOS)
- `w` - Abrir en navegador web
- Escanear código QR con Expo Go

### 5.5 Estructura de un Nuevo Módulo

#### Backend: Crear Nuevo Módulo

1. Crear estructura de paquetes:
```
com.alertamujer.{newmodule}/
├── application/
│   └── service/
│       └── NewModuleService.java
├── domain/
│   ├── model/
│   │   └── NewEntity.java
│   └── repository/
│       └── NewEntityRepository.java
└── infrastructure/
    ├── controller/
    │   └── NewEntityController.java
    └── repository/
        └── NewEntityRepositoryImpl.java
```

2. Crear entidad del dominio:
```java
@Entity
@Table(name = "new_entity", schema = "new_schema")
public class NewEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false)
    private String name;
    
    // Getters and Setters
}
```

3. Crear repositorio:
```java
public interface NewEntityRepository extends JpaRepository<NewEntity, Long> {
    Optional<NewEntity> findByName(String name);
}
```

4. Crear servicio:
```java
@Service
public class NewEntityService {
    private final NewEntityRepository repository;
    
    public NewEntityService(NewEntityRepository repository) {
        this.repository = repository;
    }
    
    public NewEntity create(NewEntity entity) {
        return repository.save(entity);
    }
}
```

5. Crear controlador:
```java
@RestController
@RequestMapping("/api/new-entities")
public class NewEntityController {
    private final NewEntityService service;
    
    public NewEntityController(NewEntityService service) {
        this.service = service;
    }
    
    @PostMapping
    public ResponseEntity<NewEntity> create(@RequestBody NewEntity entity) {
        return ResponseEntity.ok(service.create(entity));
    }
}
```

#### Frontend Web: Crear Nuevo Componente

1. Generar componente:
```bash
ng generate component features/new-feature
```

2. Crear servicio:
```bash
ng generate service core/services
```

3. Agregar ruta en `app.routes.ts`:
```typescript
{
  path: 'new-feature',
  loadComponent: () => import('./features/new-feature/new-feature.component')
}
```

#### Frontend Móvil: Crear Nueva Pantalla

1. Crear componente en `src/features/`:
```typescript
// src/features/new-feature/NewFeatureScreen.tsx
import React from 'react';
import { View, Text } from 'react-native';

export default function NewFeatureScreen() {
  return (
    <View>
      <Text>New Feature</Text>
    </View>
  );
}
```

2. Agregar ruta en `MainNavigator.tsx`:
```typescript
<Stack.Screen 
  name="NewFeature" 
  component={NewFeatureScreen} 
  options={{ title: 'New Feature' }}
/>
```

### 5.6 Convenciones de Código

#### Backend (Java)

- **Naming:** CamelCase para variables y métodos, PascalCase para clases
- **Paquetes:** Todo en minúsculas
- **Constantes:** UPPER_SNAKE_CASE
- **Comentarios:** Javadoc para clases y métodos públicos

Ejemplo:
```java
/**
 * Service for managing user authentication
 */
@Service
public class AuthService {
    private static final int TOKEN_EXPIRATION_MS = 86400000;
    
    /**
     * Registers a new user
     * @param nombre User's full name
     * @param email User's email
     * @param password User's password
     * @return Map with token and user data
     */
    public Map<String, Object> register(String nombre, String email, String password) {
        // Implementation
    }
}
```

#### Frontend Web (TypeScript)

- **Naming:** CamelCase para variables y métodos, PascalCase para clases y componentes
- **Archivos:** kebab-case para archivos de componentes
- **Interfaces:** PascalCase con prefijo 'I' opcional
- **Constantes:** UPPER_SNAKE_CASE

Ejemplo:
```typescript
// user.model.ts
export interface User {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
}

// user.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly API_URL = '/api/users';
  
  constructor(private http: HttpClient) {}
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.API_URL}/${id}`);
  }
}
```

#### Frontend Móvil (TypeScript)

- **Naming:** CamelCase para variables y métodos, PascalCase para componentes
- **Archivos:** kebab-case o camelCase
- **Hooks:** Prefijo 'use' para hooks personalizados

Ejemplo:
```typescript
// src/services/auth.service.ts
export const authService = {
  async login(email: string, password: string) {
    const response = await api.post('/api/auth/login', { email, password });
    return response.data;
  }
};

// src/hooks/useAuth.ts
export function useAuth() {
  const [user, setUser] = useState(null);
  
  const login = async (email: string, password: string) => {
    const data = await authService.login(email, password);
    setUser(data.user);
  };
  
  return { user, login };
}
```

### 5.7 Pruebas

#### Backend: Pruebas Unitarias

```java
@SpringBootTest
class AuthServiceTest {
    
    @Autowired
    private AuthService authService;
    
    @MockBean
    private UserRepository userRepository;
    
    @Test
    void testRegister() {
        // Arrange
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        
        // Act
        Map<String, Object> result = authService.register("Test User", "test@test.com", "password", "123456789");
        
        // Assert
        assertNotNull(result.get("token"));
        assertNotNull(result.get("user"));
    }
}
```

#### Frontend Web: Pruebas Unitarias

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  it('should get user', () => {
    service.getUser(1).subscribe(user => {
      expect(user.id).toBe(1);
    });
    
    const req = httpMock.expectOne('/api/users/1');
    req.flush({ id: 1, name: 'Test' });
    httpMock.verify();
  });
});
```

### 5.8 Git Workflow

#### Ramas

- `main`: Rama principal de producción
- `develop`: Rama de desarrollo
- `feature/*`: Ramas de nuevas funcionalidades
- `bugfix/*`: Ramas de corrección de bugs
- `hotfix/*`: Ramas de correcciones urgentes en producción

#### Flujo de Trabajo

1. Crear rama desde `develop`:
```bash
git checkout develop
git pull origin develop
git checkout -b feature/new-feature
```

2. Realizar cambios y commits:
```bash
git add .
git commit -m "feat: add new feature"
```

3. Push y crear Pull Request:
```bash
git push origin feature/new-feature
```

4. Revisión y merge a `develop`

5. Release a `main`:
```bash
git checkout main
git merge develop
git tag v1.0.0
git push origin main --tags
```

---

## 6. Base de Datos

### 6.1 Esquema de Base de Datos

El sistema usa PostgreSQL 16 con 4 esquemas:

- **identity:** Autenticación y usuarios
- **alert:** Alertas y contactos
- **resource:** Zonas y recursos
- **admin:** Administración y auditoría

### 6.2 Tablas Principales

#### Esquema identity

**role**
```sql
CREATE TABLE identity.role (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(30) NOT NULL UNIQUE,
    description VARCHAR(255)
);
```

**users**
```sql
CREATE TABLE identity.users (
    id BIGSERIAL PRIMARY KEY,
    role_id BIGINT NOT NULL DEFAULT 1 REFERENCES identity.role(id),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    telephone VARCHAR(20) UNIQUE,
    email VARCHAR(150) NOT NULL UNIQUE,
    document_number VARCHAR(20) UNIQUE,
    document_type VARCHAR(20),
    birthdate DATE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**account**
```sql
CREATE TABLE identity.account (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE REFERENCES identity.users(id) ON DELETE CASCADE,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    last_access TIMESTAMP
);
```

**device**
```sql
CREATE TABLE identity.device (
    id BIGSERIAL PRIMARY KEY,
    account_id BIGINT NOT NULL REFERENCES identity.account(id) ON DELETE CASCADE,
    device_uuid VARCHAR(255) NOT NULL UNIQUE,
    brand VARCHAR(50),
    model VARCHAR(100),
    os_name VARCHAR(30) NOT NULL,
    os_version VARCHAR(30),
    app_version VARCHAR(30),
    gps_status VARCHAR(20) NOT NULL DEFAULT 'unknown',
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    last_access TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

#### Esquema alert

**emergency_contact**
```sql
CREATE TABLE alert.emergency_contact (
    id BIGSERIAL PRIMARY KEY,
    user_profile_id BIGINT NOT NULL REFERENCES identity.user_profile(id) ON DELETE CASCADE,
    contact_name VARCHAR(100) NOT NULL,
    telephone VARCHAR(20) NOT NULL,
    email VARCHAR(150),
    relationship VARCHAR(30),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);
```

**alert**
```sql
CREATE TABLE alert.alert (
    id BIGSERIAL PRIMARY KEY,
    user_profile_id BIGINT NOT NULL REFERENCES identity.user_profile(id) ON DELETE CASCADE,
    device_id BIGINT NOT NULL REFERENCES identity.device(id) ON DELETE CASCADE,
    alert_type VARCHAR(20) NOT NULL DEFAULT 'main',
    activation_method VARCHAR(30) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    message VARCHAR(500),
    started_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ended_at TIMESTAMP,
    cancelled_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**location_log**
```sql
CREATE TABLE alert.location_log (
    id BIGSERIAL PRIMARY KEY,
    user_profile_id BIGINT NOT NULL REFERENCES identity.user_profile(id) ON DELETE CASCADE,
    alert_id INTEGER REFERENCES alert.alert(id) ON DELETE SET NULL,
    latitude DECIMAL(9,6) NOT NULL,
    longitude DECIMAL(9,6) NOT NULL,
    accuracy DECIMAL(6,2),
    recorded_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

#### Esquema resource

**zone**
```sql
CREATE TABLE resource.zone (
    id BIGSERIAL PRIMARY KEY,
    created_by_admin_id INTEGER,
    name VARCHAR(150) NOT NULL,
    zone_type VARCHAR(20) NOT NULL,
    risk_level VARCHAR(20) NOT NULL DEFAULT 'medium',
    description VARCHAR(500),
    address VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    latitude DECIMAL(9,6) NOT NULL,
    longitude DECIMAL(9,6) NOT NULL,
    radius_meters DECIMAL(8,2) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);
```

**emergency_resource**
```sql
CREATE TABLE resource.emergency_resource (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    resource_type VARCHAR(40) NOT NULL,
    telephone VARCHAR(30),
    secondary_telephone VARCHAR(30),
    email VARCHAR(150),
    address VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6),
    description VARCHAR(500),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);
```

#### Esquema admin

**audit_log**
```sql
CREATE TABLE admin.audit_log (
    id BIGSERIAL PRIMARY KEY,
    account_id BIGINT NOT NULL REFERENCES identity.account(id) ON DELETE CASCADE,
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50) NOT NULL,
    entity_id INTEGER,
    description VARCHAR(500),
    ip_address VARCHAR(45),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 6.3 Índices

Índices recomendados para optimización:

```sql
-- identity.users
CREATE INDEX idx_users_email ON identity.users(email);
CREATE INDEX idx_users_telephone ON identity.users(telephone);

-- alert.alert
CREATE INDEX idx_alert_user ON alert.alert(user_profile_id);
CREATE INDEX idx_alert_status ON alert.alert(status);
CREATE INDEX idx_alert_started ON alert.alert(started_at);

-- alert.location_log
CREATE INDEX idx_location_user ON alert.location_log(user_profile_id);
CREATE INDEX idx_location_alert ON alert.location_log(alert_id);
CREATE INDEX idx_location_recorded ON alert.location_log(recorded_at);

-- resource.zone
CREATE INDEX idx_zone_city ON resource.zone(city);
CREATE INDEX idx_zone_type ON resource.zone(zone_type);
```

### 6.4 Migraciones

Para cambios en la base de datos, crear scripts de migración:

```sql
-- migrations/V1.1__add_new_column.sql
ALTER TABLE identity.users ADD COLUMN middle_name VARCHAR(100);
```

---

## 7. Seguridad

### 7.1 Autenticación

#### JWT (JSON Web Tokens)

El sistema usa JWT para autenticación stateless:

```java
private String generateToken(Long userId, String role) {
    return Jwts.builder()
            .subject(userId.toString())
            .claim("role", role)
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + 86400000)) // 24 horas
            .signWith(jwtKey)
            .compact();
}
```

**Payload del Token:**
```json
{
  "sub": "1",
  "role": "user",
  "iat": 1696500000,
  "exp": 1696586400
}
```

#### BCrypt para Contraseñas

Las contraseñas se hashean con BCrypt:

```java
BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
String hashedPassword = passwordEncoder.encode(rawPassword);
```

### 7.2 Autorización

#### Roles

- **user (role_id: 1):** Usuario estándar
- **administrator (role_id: 2):** Administrador del sistema

#### Guards de Ruta (Frontend Web)

```typescript
@Injectable({
  providedIn: 'root'
})
export class AdminGuard implements CanActivate {
  constructor(private authService: AuthService) {}
  
  canActivate(): Observable<boolean> {
    return this.authService.getUserRole().pipe(
      map(role => role === 'administrator')
    );
  }
}
```

### 7.3 CORS

Configuración de CORS en el backend:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.cors(cors -> cors.configurationSource(corsConfigurationSource()))
           // ... other configuration
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

### 7.4 Validación de Input

#### Backend

```java
public class User {
    @NotBlank(message = "Name is required")
    @Size(max = 100, message = "Name must be less than 100 characters")
    private String firstName;
    
    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;
    
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}
```

#### Frontend Web

```typescript
this.registerForm = this.fb.group({
  firstName: ['', [Validators.required, Validators.maxLength(100)]],
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]]
});
```

### 7.5 Protección contra SQL Injection

El uso de JPA con parámetros nombrados previene SQL injection:

```java
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);
```

---

## 8. Despliegue

### 8.1 Despliegue con Docker

#### Dockerfile del Backend

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/backend-monolith-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: alerta-mujer-db
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend-monolith
    container_name: alerta-mujer-backend
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: ${JWT_EXPIRES_IN}
      CORS_ORIGIN: ${CORS_ORIGIN}
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

volumes:
  postgres_data:
```

### 8.2 Despliegue del Frontend Web

#### Build de Producción

```bash
ng build --configuration production
```

#### Dockerfile del Frontend Web

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN ng build --configuration production

FROM nginx:alpine
COPY --from=build /app/dist/alerta-mujer-web /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g"]
```

### 8.3 Despliegue del Frontend Móvil

#### Build de APK

```bash
# Para desarrollo
eas build --profile development --platform android

# Para producción
eas build --profile production --platform android
```

#### Configuración en app.json

```json
{
  "expo": {
    "android": {
      "package": "com.alertamujer.app",
      "versionCode": 1,
      "permissions": [
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION",
        "CAMERA",
        "RECORD_AUDIO",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE"
      ]
    }
  }
}
```

### 8.4 Variables de Entorno de Producción

```env
# Database
POSTGRES_USER=alerta_mujer_prod
POSTGRES_PASSWORD=<strong_password>
POSTGRES_DB=alerta_mujer_prod

# JWT
JWT_SECRET=<strong_random_secret>
JWT_EXPIRES_IN=24h

# CORS
CORS_ORIGIN=https://alertamujer.com

# Frontend
NG_APP_API_URL=https://api.alertamujer.com
EXPO_PUBLIC_API_URL=https://api.alertamujer.com
```

### 8.5 Monitoreo

#### Health Check

```bash
curl https://api.alertamujer.com/health
```

#### Logs

```bash
# Docker logs
docker logs alerta-mujer-backend -f

# Application logs (si se usa archivo de logs)
tail -f /var/log/alertamujer/application.log
```

---

## 9. Apéndice

### 9.1 Códigos de Error

| Código | Descripción |
|--------|-------------|
| AUTH_001 | Credenciales inválidas |
| AUTH_002 | Token expirado |
| AUTH_003 | Token inválido |
| AUTH_004 | Email ya existe |
| USER_001 | Usuario no encontrado |
| USER_002 | Cuenta bloqueada |
| ALERT_001 | Alerta no encontrada |
| ALERT_002 | Alerta ya activa |

### 9.2 Estados del Sistema

**Estados de Alerta:**
- `active` - Alerta activa
- `resolved` - Alerta resuelta
- `cancelled` - Alerta cancelada
- `expired` - Alerta expirada

**Estados de Cuenta:**
- `active` - Cuenta activa
- `inactive` - Cuenta inactiva
- `blocked` - Cuenta bloqueada

**Estados de Reporte de Zona:**
- `pending` - Pendiente de revisión
- `approved` - Aprobado
- `rejected` - Rechazado

### 9.3 Recursos Adicionales

- [Documentación de Spring Boot](https://spring.io/projects/spring-boot)
- [Documentación de Angular](https://docs.angular.io)
- [Documentación de React Native](https://reactnative.dev)
- [Documentación de Expo](https://docs.expo.dev)
- [Documentación de PostgreSQL](https://www.postgresql.org/docs)

---

**Fin del Manual Técnico**

*Este manual está diseñado para desarrolladores y arquitectos de software que trabajan en el proyecto AlertaMujer. Para información sobre el uso de la aplicación, consulte el Manual de Usuario.*
