# Manual de Administrador - AlertaMujer

**Versión:** 1.0  
**Fecha:** 5 de octubre de 2026  
**Audiencia:** Administradores del Sistema, Personal de Soporte  
**Nivel Técnico:** Intermedio

---

## Tabla de Contenidos

1. [Introducción al Panel de Administración](#1-introducción-al-panel-de-administración)
2. [Gestión de Usuarios](#2-gestión-de-usuarios)
3. [Gestión de Alertas](#3-gestión-de-alertas)
4. [Gestión de Zonas de Riesgo](#4-gestión-de-zonas-de-riesgo)
5. [Gestión de Recursos de Emergencia](#5-gestión-de-recursos-de-emergencia)
6. [Reportes y Estadísticas](#6-reportes-y-estadísticas)
7. [Auditoría y Logs](#7-auditoría-y-logs)
8. [Seguridad del Sistema](#8-seguridad-del-sistema)
9. [Seguimiento de Login de Administradores](#9-seguimiento-de-login-de-administradores)
10. [Solución de Problemas Administrativos](#10-solución-de-problemas-administrativos)

---

## 1. Introducción al Panel de Administración

### 1.1 Acceso al Panel de Administración

#### Paso 1: Iniciar Sesión

1. Navegar a la URL de la plataforma web: `https://alertamujer.com`
2. Hacer clic en **"Iniciar Sesión"**
3. Ingresar credenciales de administrador:
   - **Email:** `admin@test.com` (o el email configurado)
   - **Contraseña:** `Alerta@2024` (o la contraseña configurada)

![Pantalla: Pantalla de login con campos de email y contraseña]

#### Paso 2: Login con Contraseña Global

El sistema utiliza una **contraseña global de administrador** para acceder al panel. Después de ingresar las credenciales:

1. El sistema verifica que el usuario tenga rol de administrador
2. Se muestra una ventana solicitando el **nombre del administrador** que está accediendo
3. Ingresar el nombre completo del administrador
4. Este nombre se guarda en la base de datos para rastrear todas las acciones realizadas

![Pantalla: Ventana modal solicitando nombre del administrador]

**Importante:** Este paso es crucial para la auditoría. Todas las acciones realizadas en el panel quedarán asociadas al nombre del administrador que ingresó.

#### Paso 3: Acceder al Panel

Después del login exitoso, el sistema redirige automáticamente al panel de administración: `/admin`

### 1.2 Estructura del Panel de Administración

El panel de administración tiene la siguiente estructura:

```
┌─────────────────────────────────────────────────────────────┐
│  [Logo] AlertaMujer Admin    [Admin: Nombre] [Cerrar Sesión] │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Menú Lateral          Área Principal                        │
│  ┌─────────────────┐  ┌─────────────────────────────────┐  │
│  │ Dashboard       │  │                                 │  │
│  │ Usuarios        │  │   Contenido de la sección       │  │
│  │ Alertas         │  │                                 │  │
│  │ Contactos       │  │                                 │  │
│  │ Evidencias      │  │                                 │  │
│  │ Dispositivos    │  │                                 │  │
│  │ Zonas           │  │                                 │  │
│  │ Recursos        │  │                                 │  │
│  │ Reportes        │  │                                 │  │
│  │ Auditoría       │  │                                 │  │
│  │ Configuración   │  │                                 │  │
│  └─────────────────┘  └─────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Dashboard Principal

El dashboard muestra un resumen del sistema:

- **Total de usuarios:** Cantidad de usuarias registradas
- **Alertas activas:** Alertas actualmente en curso
- **Alertas hoy:** Alertas registradas en las últimas 24 horas
- **Usuarios nuevos:** Nuevos registros en la última semana
- **Gráficos de actividad:** Visualización de alertas por día/hora
- **Alertas recientes:** Lista de las últimas 5 alertas

![Pantalla: Dashboard con métricas y gráficos]

---

## 2. Gestión de Usuarios

### 2.1 Ver Lista de Usuarios

#### Paso 1: Navegar a Usuarios

1. En el menú lateral, hacer clic en **"Usuarios"**
2. Se muestra una tabla con todos los usuarios del sistema

#### Información Mostrada

La tabla de usuarios incluye:
- **ID:** Identificador único
- **Nombre:** Nombre completo
- **Email:** Correo electrónico
- **Teléfono:** Número de teléfono
- **Rol:** Usuario o Administrador
- **Estado:** Active, Inactive, Blocked
- **Fecha de registro:** Fecha de creación
- **Último acceso:** Último login

![Pantalla: Tabla de usuarios con información detallada]

### 2.2 Buscar y Filtrar Usuarios

#### Búsqueda por Nombre o Email

1. Usar el campo de búsqueda en la parte superior
2. Ingresar nombre o email
3. La lista se filtra automáticamente

#### Filtrar por Estado

1. Usar el filtro de estado
2. Seleccionar: Active, Inactive, o Blocked
3. La lista se actualiza

#### Filtrar por Rol

1. Usar el filtro de rol
2. Seleccionar: Usuario o Administrador
3. La lista se actualiza

### 2.3 Ver Detalles de Usuario

#### Paso 1: Seleccionar Usuario

1. Hacer clic en el usuario deseado
2. Se abre el panel de detalles del usuario

#### Información del Perfil

- **Información personal:** Nombre, email, teléfono, documento
- **Información de cuenta:** Estado, fecha de registro, último acceso
- **Dispositivos:** Lista de dispositivos registrados
- **Contactos de emergencia:** Contactos configurados
- **Historial de alertas:** Alertas del usuario
- **Evidencias:** Evidencias asociadas

![Pantalla: Panel de detalles de usuario]

### 2.4 Crear Nuevo Usuario

#### Paso 1: Iniciar Creación

1. Hacer clic en el botón **"Nuevo Usuario"**
2. Se abre el formulario de creación

#### Paso 2: Completar Formulario

**Información Requerida:**
- **Nombre completo**
- **Correo electrónico**
- **Contraseña** (mínimo 8 caracteres, mayúsculas, minúsculas y números)
- **Número de teléfono**
- **Rol:** Usuario o Administrador
- **Estado:** Active, Inactive, o Blocked

![Pantalla: Formulario de creación de usuario]

#### Paso 3: Guardar

1. Revisar la información
2. Hacer clic en **"Guardar"**
3. El usuario es creado y aparece en la lista

**Nota:** Esta acción queda registrada en el log de auditoría con el nombre del administrador que realizó la acción.

### 2.5 Editar Usuario

#### Paso 1: Seleccionar Usuario

1. Hacer clic en el usuario a editar
2. Hacer clic en el botón **"Editar"**

#### Paso 2: Modificar Información

Se pueden modificar:
- Nombre completo
- Email
- Teléfono
- Rol
- Estado

**Advertencia:** Cambiar el estado a "Blocked" impedirá que el usuario acceda al sistema.

#### Paso 3: Guardar Cambios

1. Hacer clic en **"Guardar"**
2. Los cambios se aplican inmediatamente
3. La acción queda registrada en auditoría

### 2.6 Bloquear/Desbloquear Usuario

#### Bloquear Usuario

1. Seleccionar el usuario
2. Cambiar estado a **"Blocked"**
3. Hacer clic en **"Guardar"**

**Efecto:** El usuario no podrá iniciar sesión hasta que sea desbloqueado.

#### Desbloquear Usuario

1. Seleccionar el usuario bloqueado
2. Cambiar estado a **"Active"**
3. Hacer clic en **"Guardar"**

**Efecto:** El usuario podrá iniciar sesión nuevamente.

### 2.7 Eliminar Usuario

**Advertencia:** Esta acción es irreversible. Eliminar un usuario también elimina:
- Su perfil
- Sus contactos de emergencia
- Su historial de alertas
- Sus evidencias
- Sus dispositivos

#### Paso 1: Seleccionar Usuario

1. Hacer clic en el usuario a eliminar
2. Hacer clic en el botón **"Eliminar"**

#### Paso 2: Confirmar Eliminación

1. Aparece una ventana de confirmación
2. Leer el mensaje de advertencia
3. Escribir **"CONFIRMAR"** para confirmar
4. Hacer clic en **"Eliminar"**

![Pantalla: Ventana de confirmación de eliminación]

**Nota:** Esta acción queda registrada en auditoría con el nombre del administrador.

---

## 3. Gestión de Alertas

### 3.1 Ver Lista de Alertas

#### Paso 1: Navegar a Alertas

1. En el menú lateral, hacer clic en **"Alertas"**
2. Se muestra una tabla con todas las alertas

#### Información Mostrada

- **ID:** Identificador de la alerta
- **Usuario:** Nombre de la usuaria
- **Estado:** Active, Resolved, Cancelled, Expired
- **Tipo:** Main, Medical, Other
- **Método de activación:** Panic button, Manual, Other
- **Fecha de inicio:** Cuándo se activó
- **Duración:** Tiempo activa
- **Ubicación:** Coordenadas GPS

![Pantalla: Tabla de alertas con información detallada]

### 3.2 Filtrar Alertas

#### Por Estado

1. Usar el filtro de estado
2. Seleccionar: Active, Resolved, Cancelled, Expired
3. La lista se actualiza

#### Por Fecha

1. Usar el selector de fechas
2. Seleccionar rango de fechas
3. La lista se actualiza

#### Por Usuario

1. Usar el buscador
2. Ingresar nombre o email del usuario
3. La lista se filtra

### 3.3 Ver Detalles de Alerta

#### Paso 1: Seleccionar Alerta

1. Hacer clic en la alerta deseada
2. Se abre el panel de detalles

#### Información Completa

- **Información de la alerta:** Estado, tipo, método, mensaje
- **Información del usuario:** Nombre, contacto
- **Ubicación:** Mapa con coordenadas
- **Timeline:** Cronología de eventos
- **Contactos notificados:** Lista y estado de notificación
- **Evidencias:** Fotos, videos, audio adjuntos
- **Logs de ubicación:** Historial de GPS durante la alerta

![Pantalla: Panel de detalles de alerta con mapa y timeline]

### 3.4 Gestionar Alerta Activa

#### Ver Alerta en Tiempo Real

Para alertas activas, el panel muestra:
- **Ubicación en tiempo real:** Mapa con posición actual
- **Duración:** Contador de tiempo activa
- **Estado de notificaciones:** Qué contactos fueron notificados
- **Logs de ubicación:** Historial de movimiento

#### Contactar Usuario

1. En el panel de la alerta, hacer clic en **"Contactar"**
2. Ver información de contacto del usuario
3. Llamar o enviar mensaje según procedimiento

#### Coordinar con Autoridades

Si la alerta es grave:
1. Verificar ubicación exacta
2. Contactar servicios de emergencia
3. Proporcionar información del usuario
4. Monitorear hasta resolución

### 3.5 Cerrar Alerta

#### Paso 1: Seleccionar Alerta Activa

1. Hacer clic en la alerta
2. Hacer clic en **"Cerrar Alerta"**

#### Paso 2: Agregar Notas

1. Ingresar notas sobre la resolución
2. Describir acciones tomadas
3. Hacer clic en **"Confirmar"**

**Efecto:** La alerta cambia a estado "Resolved".

**Nota:** Esta acción queda registrada en auditoría.

### 3.6 Exportar Reporte de Alertas

#### Paso 1: Aplicar Filtros

1. Aplicar los filtros deseados (fecha, estado, usuario)
2. Verificar que los resultados sean correctos

#### Paso 2: Exportar

1. Hacer clic en **"Exportar"**
2. Seleccionar formato: CSV, PDF, o Excel
3. El archivo se descarga automáticamente

---

## 4. Gestión de Zonas de Riesgo

### 4.1 Ver Lista de Zonas

#### Paso 1: Navegar a Zonas

1. En el menú lateral, hacer clic en **"Zonas"**
2. Se muestra una lista con todas las zonas

#### Información Mostrada

- **Nombre:** Nombre de la zona
- **Tipo:** Risk (riesgo) o Safe (segura)
- **Nivel de riesgo:** High, Medium, Low
- **Ciudad:** Ubicación
- **Reportes:** Cantidad de reportes
- **Estado:** Active o Inactive

![Pantalla: Lista de zonas con mapa]

### 4.2 Ver Mapa de Zonas

El panel incluye un mapa interactivo que muestra:
- **Zonas de riesgo:** Áreas coloreadas en rojo/naranja
- **Zonas seguras:** Áreas coloreadas en verde
- **Densidad de reportes:** Intensidad del color según cantidad

### 4.3 Crear Nueva Zona

#### Paso 1: Iniciar Creación

1. Hacer clic en **"Nueva Zona"**
2. Se abre el formulario

#### Paso 2: Completar Formulario

**Información Requerida:**
- **Nombre** de la zona
- **Tipo:** Risk o Safe
- **Nivel de riesgo:** High, Medium, Low
- **Descripción** del riesgo o seguridad
- **Dirección** o ubicación
- **Ciudad**
- **Coordenadas:** Latitud y longitud
- **Radio:** Radio en metros

![Pantalla: Formulario de creación de zona con mapa]

#### Paso 3: Definir Área en Mapa

1. Usar el mapa interactivo
2. Hacer clic en el centro de la zona
3. Ajustar el radio arrastrando el círculo
4. Las coordenadas se llenan automáticamente

#### Paso 4: Guardar

1. Hacer clic en **"Guardar"**
2. La zona aparece en el mapa y en la lista

**Nota:** Esta acción queda registrada en auditoría.

### 4.4 Editar Zona

#### Paso 1: Seleccionar Zona

1. Hacer clic en la zona a editar
2. Hacer clic en **"Editar"**

#### Paso 2: Modificar Información

Se pueden modificar:
- Nombre
- Tipo
- Nivel de riesgo
- Descripción
- Ubicación
- Radio

#### Paso 3: Guardar Cambios

1. Hacer clic en **"Guardar"**
2. Los cambios se aplican

### 4.5 Gestionar Reportes de Zonas

#### Ver Reportes Pendientes

1. En el panel de zonas, hacer clic en **"Reportes Pendientes"**
2. Se muestran reportes de usuarios sobre zonas

#### Aprobar Reporte

1. Seleccionar el reporte
2. Revisar la información
3. Hacer clic en **"Aprobar"**
4. La zona se actualiza con la información del reporte

#### Rechazar Reporte

1. Seleccionar el reporte
2. Hacer clic en **"Rechazar"**
3. Ingresar motivo del rechazo
4. Hacer clic en **"Confirmar"**

**Nota:** Estas acciones quedan registradas en auditoría.

### 4.6 Eliminar Zona

#### Paso 1: Seleccionar Zona

1. Hacer clic en la zona a eliminar
2. Hacer clic en **"Eliminar"**

#### Paso 2: Confirmar

1. Leer la advertencia
2. Escribir **"CONFIRMAR"**
3. Hacer clic en **"Eliminar"**

---

## 5. Gestión de Recursos de Emergencia

### 5.1 Ver Lista de Recursos

#### Paso 1: Navegar a Recursos

1. En el menú lateral, hacer clic en **"Recursos"**
2. Se muestra una lista con todos los recursos

#### Información Mostrada

- **Nombre:** Nombre del recurso
- **Tipo:** Police, Hospital, Emergency line, Support organization
- **Teléfono:** Número de contacto
- **Ciudad:** Ubicación
- **Estado:** Active o Inactive

![Pantalla: Lista de recursos de emergencia]

### 5.2 Crear Nuevo Recurso

#### Paso 1: Iniciar Creación

1. Hacer clic en **"Nuevo Recurso"**
2. Se abre el formulario

#### Paso 2: Completar Formulario

**Información Requerida:**
- **Nombre** del recurso
- **Tipo:** Police station, Hospital, Emergency line, Support organization, Health center, etc.
- **Teléfono principal**
- **Teléfono secundario** (opcional)
- **Email** (opcional)
- **Dirección**
- **Ciudad**
- **Coordenadas** (opcional)
- **Descripción** de servicios

![Pantalla: Formulario de creación de recurso]

#### Paso 3: Guardar

1. Hacer clic en **"Guardar"**
2. El recurso aparece en la lista

### 5.3 Editar Recurso

1. Seleccionar el recurso
2. Hacer clic en **"Editar"**
3. Modificar la información
4. Hacer clic en **"Guardar"**

### 5.4 Ver Estadísticas de Uso

#### Llamadas a Recursos

1. Seleccionar un recurso
2. Hacer clic en **"Estadísticas"**
3. Ver:
   - Cantidad de llamadas
   - Promedio de duración
   - Distribución por hora del día
   - Tendencia semanal

![Pantalla: Gráficos de estadísticas de uso de recursos]

---

## 6. Reportes y Estadísticas

### 6.1 Dashboard de Reportes

#### Acceder a Reportes

1. En el menú lateral, hacer clic en **"Reportes"**
2. Se muestra el dashboard de estadísticas

#### Métricas Disponibles

- **Total de usuarios:** Gráfico de crecimiento
- **Alertas por día:** Gráfico de barras
- **Alertas por hora:** Gráfico de líneas
- **Zonas más reportadas:** Lista ordenada
- **Tiempo de respuesta:** Promedio de atención
- **Tasa de resolución:** Porcentaje de alertas resueltas

![Pantalla: Dashboard de reportes con múltiples gráficos]

### 6.2 Reporte de Usuarios

#### Generar Reporte

1. Hacer clic en **"Reporte de Usuarios"**
2. Seleccionar rango de fechas
3. Hacer clic en **"Generar"**

#### Información Incluida

- Nuevos registros
- Usuarios activos
- Usuarios inactivos
- Usuarios bloqueados
- Distribución por ciudad

### 6.3 Reporte de Alertas

#### Generar Reporte

1. Hacer clic en **"Reporte de Alertas"**
2. Seleccionar rango de fechas
3. Seleccionar filtros opcionales (estado, tipo, ciudad)
4. Hacer clic en **"Generar"**

#### Información Incluida

- Total de alertas
- Alertas por estado
- Alertas por tipo
- Tiempo promedio de resolución
- Distribución geográfica

### 6.4 Reporte de Zonas

#### Generar Reporte

1. Hacer clic en **"Reporte de Zonas"**
2. Seleccionar rango de fechas
3. Hacer clic en **"Generar"**

#### Información Incluida

- Zonas más reportadas
- Nuevas zonas creadas
- Zonas modificadas
- Reportes aprobados/rechazados

### 6.5 Exportar Reportes

#### Formatos Disponibles

- **PDF:** Para presentación e impresión
- **Excel:** Para análisis de datos
- **CSV:** Para importación en otros sistemas

#### Paso a Paso

1. Generar el reporte deseado
2. Hacer clic en **"Exportar"**
3. Seleccionar formato
4. El archivo se descarga

---

## 7. Auditoría y Logs

### 7.1 Ver Logs de Auditoría

#### Acceder a Auditoría

1. En el menú lateral, hacer clic en **"Auditoría"**
2. Se muestra el log de todas las acciones administrativas

#### Información en el Log

- **Fecha y hora:** Cuándo se realizó la acción
- **Administrador:** Nombre del administrador (del login global)
- **Acción:** Tipo de acción realizada
- **Entidad:** Tipo de entidad afectada
- **ID:** Identificador de la entidad
- **Descripción:** Detalles de la acción
- **Dirección IP:** IP desde donde se realizó

![Pantalla: Tabla de logs de auditoría]

### 7.2 Filtrar Logs

#### Por Administrador

1. Usar el filtro de administrador
2. Seleccionar el nombre del administrador
3. Ver solo acciones de ese administrador

#### Por Tipo de Acción

1. Usar el filtro de acción
2. Seleccionar: CREATE, UPDATE, DELETE, LOGIN, etc.
3. Ver solo acciones de ese tipo

#### Por Fecha

1. Usar el selector de fechas
2. Seleccionar rango
3. Ver acciones en ese período

#### Por Entidad

1. Usar el filtro de entidad
2. Seleccionar: User, Alert, Zone, Resource, etc.
3. Ver solo acciones sobre esa entidad

### 7.3 Ver Detalles de Acción

1. Hacer clic en una entrada del log
2. Se abre el panel de detalles
3. Ver información completa de la acción

### 7.4 Exportar Logs

#### Paso 1: Aplicar Filtros

1. Aplicar los filtros deseados
2. Verificar resultados

#### Paso 2: Exportar

1. Hacer clic en **"Exportar"**
2. Seleccionar formato (CSV, PDF)
3. El archivo se descarga

**Uso:** Los logs de auditoría son útiles para:
- Investigar incidentes
- Cumplimiento de normativas
- Análisis de comportamiento
- Resolución de disputas

---

## 8. Seguridad del Sistema

### 8.1 Gestión de Administradores

#### Ver Administradores

1. En el menú lateral, hacer clic en **"Configuración"**
2. Seleccionar **"Administradores"**
3. Ver lista de todos los administradores

#### Crear Nuevo Administrador

1. Hacer clic en **"Nuevo Administrador"**
2. Completar formulario:
   - Nombre completo
   - Email
   - Contraseña
   - Permisos específicos (si aplica)
3. Hacer clic en **"Guardar"**

**Nota:** Solo los administradores existentes pueden crear nuevos administradores.

#### Eliminar Administrador

1. Seleccionar el administrador
2. Hacer clic en **"Eliminar"**
3. Confirmar

**Advertencia:** No puedes eliminar el último administrador activo.

### 8.2 Configuración de Contraseña Global

#### ¿Qué es la Contraseña Global?

La contraseña global es una contraseña compartida que todos los administradores usan para acceder al panel. Después de ingresar esta contraseña, se solicita el nombre individual del administrador para rastrear acciones.

#### Cambiar Contraseña Global

1. En **"Configuración"**, seleccionar **"Seguridad"**
2. Hacer clic en **"Cambiar Contraseña Global"**
3. Ingresar contraseña actual
4. Ingresar nueva contraseña
5. Confirmar nueva contraseña
6. Hacer clic en **"Guardar"**

**Importante:** Comunicar el cambio a todos los administradores.

### 8.3 Configuración de Políticas de Contraseña

#### Requisitos Actuales

- Mínimo 8 caracteres
- Al menos una mayúscula
- Al menos una minúscula
- Al menos un número

#### Modificar Políticas

1. En **"Configuración"**, seleccionar **"Seguridad"**
2. Modificar requisitos de contraseña
3. Hacer clic en **"Guardar"**

**Nota:** Los cambios aplican solo a nuevas contraseñas.

### 8.4 Configuración de Intentos de Login

#### Bloqueo por Intentos Fallidos

El sistema puede bloquear cuentas después de múltiples intentos fallidos.

#### Configurar

1. En **"Configuración"**, seleccionar **"Seguridad"**
2. Establecer número máximo de intentos
3. Establecer tiempo de bloqueo
4. Hacer clic en **"Guardar"**

### 8.5 Configuración de IP Permitidas

#### Restringir Acceso por IP

Para mayor seguridad, puedes restringir el acceso al panel a direcciones IP específicas.

#### Configurar

1. En **"Configuración"**, seleccionar **"Seguridad"**
2. Agregar direcciones IP permitidas
3. Hacer clic en **"Guardar"**

**Efecto:** Solo las IPs configuradas podrán acceder al panel.

---

## 9. Seguimiento de Login de Administradores

### 9.1 Sistema de Login con Nombre de Administrador

#### ¿Cómo Funciona?

El sistema implementa un mecanismo de seguimiento de acciones administrativas:

1. El administrador ingresa sus credenciales (email/contraseña global)
2. El sistema verifica que tenga rol de administrador
3. Se muestra una ventana modal solicitando el **nombre completo del administrador**
4. El administrador ingresa su nombre
5. Este nombre se guarda en la base de datos
6. Todas las acciones realizadas quedan asociadas a este nombre

![Pantalla: Flujo de login con solicitud de nombre]

#### ¿Por Qué es Importante?

- **Responsabilidad:** Saber exactamente quién realizó cada acción
- **Auditoría:** Trazabilidad completa de cambios
- **Seguridad:** Prevenir acciones no autorizadas
- **Cumplimiento:** Requisitos de normativas de seguridad

### 9.2 Ver Historial de Login

#### Acceder al Historial

1. En el menú lateral, hacer clic en **"Auditoría"**
2. Usar el filtro de acción: **"LOGIN"**
3. Ver todos los logins de administradores

#### Información Mostrada

- **Fecha y hora** del login
- **Nombre del administrador** (ingresado en el modal)
- **Dirección IP** desde donde se conectó
- **Navegador/Dispositivo** utilizado

### 9.3 Ver Acciones por Administrador

#### Filtrar por Nombre

1. En **"Auditoría"**, usar el filtro de administrador
2. Seleccionar el nombre deseado
3. Ver todas las acciones de ese administrador

#### Reporte Individual

1. Seleccionar un administrador
2. Hacer clic en **"Generar Reporte"**
3. Ver resumen de todas sus acciones

### 9.4 Configurar Alertas de Actividad

#### Alertas de Login

Puedes configurar alertas cuando:
- Un administrador hace login desde una IP nueva
- Un administrador hace login en horarios inusuales
- Hay múltiples intentos fallidos

#### Configurar

1. En **"Configuración"**, seleccionar **"Notificaciones"**
2. Habilitar alertas deseadas
3. Ingresar email de notificación
4. Hacer clic en **"Guardar"**

### 9.5 Ver Sesiones Activas

#### Lista de Sesiones

1. En **"Configuración"**, seleccionar **"Sesiones"**
2. Ver todas las sesiones activas de administradores

#### Información

- Nombre del administrador
- IP de conexión
- Hora de inicio
- Última actividad

#### Cerrar Sesión

1. Seleccionar una sesión
2. Hacer clic en **"Cerrar Sesión"**
3. El administrador deberá hacer login nuevamente

**Uso:** Útil si sospechas de una sesión comprometida.

---

## 10. Solución de Problemas Administrativos

### 10.1 Problemas de Acceso

#### No Puedo Acceder al Panel

**Posibles Causas:**
- Cuenta bloqueada
- Contraseña incorrecta
- IP no permitida
- Sesión expirada

**Soluciones:**

1. Verificar estado de la cuenta con otro administrador
2. Solicitar restablecimiento de contraseña
3. Verificar configuración de IP permitidas
4. Limpiar cookies y cache del navegador

#### Error "Administrador No Reconocido"

**Causa:** El nombre ingresado en el modal no coincide con un administrador registrado.

**Solución:**
1. Verificar que el nombre esté correctamente escrito
2. Contactar al administrador principal para verificar registro

### 10.2 Problemas con Usuarios

#### Usuario No Puede Iniciar Sesión

**Verificar:**
1. Estado de la cuenta (active/inactive/blocked)
2. Credenciales correctas
3. No hay bloqueo por intentos fallidos

**Acciones:**
1. Desbloquear cuenta si está bloqueada
2. Restablecer contraseña si el usuario la olvidó
3. Verificar que el email sea correcto

#### Usuario No Recibe Notificaciones

**Verificar:**
1. Contactos de emergencia configurados
2. Números de teléfono correctos
3. Servicio de notificaciones operativo

**Acciones:**
1. Verificar configuración de contactos del usuario
2. Probar envío de notificación de prueba
3. Contactar soporte si el problema persiste

### 10.3 Problemas con Alertas

#### Alerta No Se Cierra

**Causas:**
- Error en el sistema
- Usuario no tiene conexión
- Dispositivo sin GPS

**Acciones:**
1. Verificar estado del backend
2. Contactar al usuario por otros medios
3. Cerrar alerta manualmente desde el panel

#### Ubicación Incorrecta en Alerta

**Causas:**
- GPS del dispositivo descalibrado
- Problema de señal
- Error en el sistema

**Acciones:**
1. Verificar logs de ubicación
2. Contactar al usuario para verificar ubicación real
3. Reportar el problema a desarrollo si es recurrente

### 10.4 Problemas con Reportes

#### Reporte No Se Genera

**Causas:**
- Rango de fechas demasiado amplio
- Error en el servidor
- Falta de datos

**Soluciones:**
1. Reducir rango de fechas
2. Verificar estado del servidor
3. Intentar con filtros diferentes

#### Datos Incorrectos en Reporte

**Causas:**
- Error en la base de datos
- Problema de sincronización

**Soluciones:**
1. Verificar datos directamente en la base de datos
2. Contactar a desarrollo para revisión
3. Generar reporte manual si es urgente

### 10.5 Problemas de Auditoría

#### Logs No Aparecen

**Causas:**
- Servicio de auditoría detenido
- Error en la base de datos

**Soluciones:**
1. Verificar estado del servicio de auditoría
2. Revisar logs del servidor
3. Reiniciar servicio si es necesario

#### Nombre de Administrador Incorrecto en Logs

**Causa:** El administrador ingresó un nombre diferente en el modal de login.

**Solución:**
1. Educar a los administradores sobre la importancia de ingresar su nombre correcto
2. Si es un error recurrente, considerar implementar validación adicional

### 10.6 Contacto de Soporte

#### Cuándo Contactar Soporte

- Problemas que no pueden resolverse con las soluciones anteriores
- Errores del sistema no documentados
- Sospecha de seguridad comprometida
- Necesidad de funcionalidades no disponibles

#### Información a Proporcionar

1. Descripción detallada del problema
2. Pasos para reproducir el error
3. Capturas de pantalla si es posible
4. Logs relevantes
5. Hora aproximada del problema

**Soporte Técnico:**
- Email: soporte@alertamujer.com
- Teléfono: +57 1 123 4567
- Horario: Lunes a viernes, 8:00 AM - 6:00 PM

**Emergencias:**
- Línea 24/7: +57 1 800 123 4567

---

## 11. Mejores Prácticas

### 11.1 Seguridad

- **Nunca compartir** la contraseña global de administrador
- **Usar contraseñas fuertes** para todas las cuentas
- **Cambiar contraseñas** regularmente (cada 90 días)
- **Revisar logs de auditoría** semanalmente
- **Reportar actividades sospechosas** inmediatamente
- **Cerrar sesiones** cuando termine de trabajar
- **Usar VPN** cuando acceda desde redes públicas

### 11.2 Gestión de Usuarios

- **Verificar información** antes de aprobar usuarios
- **Documentar razones** al bloquear usuarios
- **Comunicar cambios** a los usuarios afectados
- **Revisar usuarios inactivos** mensualmente
- **Mantener registros** de acciones importantes

### 11.3 Gestión de Alertas

- **Monitorear alertas activas** constantemente
- **Responder rápidamente** a alertas críticas
- **Documentar resoluciones** de alertas
- **Analizar tendencias** de alertas
- **Coordinar con autoridades** cuando sea necesario

### 11.4 Auditoría

- **Revisar logs diariamente**
- **Investigar anomalías** inmediatamente
- **Mantener registros** por el tiempo requerido por ley
- **Exportar logs** regularmente como backup
- **Verificar nombres** de administradores en logs

---

## 12. Apéndice

### 12.1 Códigos de Estado

**Estados de Usuario:**
- `active` - Usuario activo y funcional
- `inactive` - Cuenta temporalmente inactiva
- `blocked` - Cuenta bloqueada por administrador

**Estados de Alerta:**
- `active` - Alerta actualmente activa
- `resolved` - Alerta resuelta
- `cancelled` - Alerta cancelada por usuario
- `expired` - Alerta expirada por tiempo

**Estados de Reporte de Zona:**
- `pending` - Reporte pendiente de revisión
- `approved` - Reporte aprobado
- `rejected` - Reporte rechazado

### 12.2 Permisos de Administrador

**Permisos Básicos:**
- Ver usuarios
- Ver alertas
- Ver zonas
- Ver recursos

**Permisos Avanzados:**
- Crear/Editar/Eliminar usuarios
- Crear/Editar/Eliminar zonas
- Crear/Editar/Eliminar recursos
- Cerrar alertas
- Ver auditoría completa

**Permisos de Super Administrador:**
- Todos los permisos anteriores
- Gestión de administradores
- Configuración del sistema
- Cambio de contraseña global

### 12.3 Recursos Adicionales

- **Manual de Usuario:** Para entender la perspectiva de las usuarias
- **Manual Técnico:** Para detalles técnicos del sistema
- **Manual de Instalación:** Para configuración del servidor

---

**Fin del Manual de Administrador**

*Este manual está diseñado para administradores del sistema AlertaMujer. Para soporte técnico adicional, contacte al equipo de soporte.*
