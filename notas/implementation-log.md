# Bitacora de Implementacion - SESVIA

Este documento registra el avance tecnico del backend por fases para su uso como evidencia en el PI.

> Nota: la fecha de esta entrada corresponde a la actualización de la documentación, no necesariamente a la fecha exacta de implementación de los cambios. 

## Criterio de idioma y nomenclatura

1. La explicacion funcional y academica se redacta en espanol.
2. Los identificadores tecnicos se mantienen en ingles (tablas, columnas, endpoints y nombres de funciones).
3. Justificacion: estandar internacional, legibilidad tecnica y buenas practicas de mantenibilidad.

---

## 2026-02-16 - Fase 0 + Fase 1A

### Alcance entregado
1. Arranque del backend con FastAPI.
2. Base de identidad y acceso:
   - users
   - auth_accounts
   - roles
   - user_roles
3. Endpoints implementados:
   - POST /auth/register
   - POST /auth/login
   - POST /auth/logout
   - GET /auth/me
   - PATCH /auth/me

### Trabajo tecnico realizado
1. Estructura inicial del backend.
2. Configuracion de entorno y dependencias.
3. Conexion SQLAlchemy con esquema sesvia.
4. Modelos ORM de identidad.
5. JWT access/refresh y hashing con bcrypt.
6. Validaciones de unicidad (email y username).

### Validacion
1. Pruebas iniciales de seguridad y tokens.
2. Ajuste de importacion para ejecutar pytest desde raiz o backend.
3. Estado al cierre de fase: pruebas en verde.

### Decisiones cerradas
1. Auth local primero; OAuth diferido.
2. Auto-login en register activado.
3. context_role opcional en login; prioridad por defecto: professional, client, admin.

### Limitaciones conocidas
1. Envelope de error aun no unificado.
2. Revocacion refresh en memoria (sin persistencia).
3. Migraciones Alembic no generadas todavia.

### Siguiente fase
Fase 1B: configuracion profesional.

---

## 2026-02-21 - Fase 1B

### Alcance entregado
1. Entidades de configuracion profesional:
   - pro_profiles
   - billing_details
   - services
   - schedules
2. Endpoints implementados:
   - POST /pro-profile
   - GET /pro-profile/me
   - PATCH /pro-profile/me
   - POST /billing-details
   - GET /billing-details/me
   - PATCH /billing-details/{id}
   - POST /services
   - GET /services/me
   - PATCH /services/{id}
   - POST /schedules
   - GET /schedules/me
   - PATCH /schedules/{id}

### Trabajo tecnico realizado
1. Modelado ORM de Fase 1B.
2. Guarda de rol professional para rutas privadas.
3. Validaciones de propiedad/acceso por recurso.
4. Validaciones de agenda (rango horario y solapes).
5. Validaciones fiscales de servicios.
6. Soporte de desactivacion logica mediante el campo active.

### Validacion
1. Pruebas unitarias de validadores.
2. Estado tras fase: 8 pruebas superadas.

### Decisiones cerradas
1. El borrado logico no se aplica como eliminacion fisica; se usa active.
2. La parte publica se difiere a Fase 1C.

### Limitaciones conocidas
1. Faltan pruebas de integracion endpoint a endpoint.
2. Sin flujo de moderacion de perfil.

### Siguiente fase
Fase 1C: exploracion publica y disponibilidad.

---

## 2026-02-28 - Fase 1C

### Alcance entregado
1. Endpoints publicos:
   - GET /professionals
   - GET /professionals/{public_uuid}
   - GET /professionals/{public_uuid}/services
2. Endpoint de disponibilidad diaria:
   - GET /professionals/{public_uuid}/availability?serviceId=...&date=...

### Trabajo tecnico realizado
1. Motor de disponibilidad (agenda recurrente menos reservas ocupadas).
2. Filtros de visibilidad publica:
   - users.active = true
   - pro_profiles.visible = true
   - pro_profiles.status = active
   - services.active = true
3. Validacion de coherencia servicio-profesional.

### Validacion
1. Pruebas unitarias de generacion de huecos y resta de solapes.
2. Estado tras fase: 11 pruebas superadas.

### Decisiones cerradas
1. Disponibilidad calculada on-demand.
2. Solo disponibilidad diaria.
3. Manejo temporal en UTC en esta fase.

### Limitaciones conocidas
1. Faltan pruebas de integracion de endpoints publicos.
2. No hay calendario de excepciones (festivos/bloqueos manuales).

### Siguiente fase
Fase 1D: reservas y trazabilidad de estado.

---

## 2026-03-07 - Fase 1D

### Alcance entregado
1. Endpoints de reservas:
   - POST /bookings
   - GET /bookings/me
   - GET /bookings/{booking_id}
   - PATCH /bookings/{booking_id}/cancel
2. Registro de historial de estado en booking_history.

### Trabajo tecnico realizado
1. Ampliacion del modelo bookings:
   - price_total
   - currency_code
   - payment_deadline
   - notes
   - timestamps
2. Modelo booking_history.
3. Validaciones de negocio en creacion:
   - booking_start con zona horaria y en futuro
   - servicio activo y perteneciente al profesional
   - ajuste a agenda
   - rechazo de solapes
4. Calculo backend de precio y deadline de pago.
5. Visibilidad por rol activo en GET /bookings/me.

### Validacion
1. Pruebas unitarias de reglas de reserva.
2. Estado tras fase: 17 pruebas superadas.

### Decisiones cerradas
1. booking_id interno usado en rutas en MVP.
2. Cancelacion permitida desde pending o confirmed.

### Limitaciones conocidas
1. Faltan pruebas de integracion completas del API de bookings.
2. Estrategia de bloqueo transaccional avanzada pendiente.

### Siguiente fase
Fase 2: pagos y facturacion.

---

## Checklist - Fase 1B (Configuracion profesional)

### Alcance
- [x] pro_profiles
- [x] billing_details
- [x] services
- [x] schedules

### Endpoints
- [x] POST /pro-profile
- [x] GET /pro-profile/me
- [x] PATCH /pro-profile/me
- [x] POST /billing-details
- [x] GET /billing-details/me
- [x] PATCH /billing-details/{id}
- [x] POST /services
- [x] GET /services/me
- [x] PATCH /services/{id}
- [x] POST /schedules
- [x] GET /schedules/me
- [x] PATCH /schedules/{id}

### Criterios
- [x] Restriccion por rol professional.
- [x] Control de propiedad/acceso por recurso.
- [x] Un perfil profesional por usuario.
- [x] Validaciones de horario y solapes.
- [x] Validaciones fiscales de servicio.

### Pruebas pendientes
- [~] Casos endpoint de authz 401/403 — cubiertos transversalmente en fases posteriores (1D, pagos, mensajeria).
- [~] Casos endpoint de propiedad/acceso — validados en pruebas de integracion de fases 1D y 2.
- [~] Integracion completa del flujo correcto de Fase 1B — diferida; la cobertura funcional queda probada de forma indirecta en flujos de reserva y pagos.

---

## Checklist - Fase 1C (Publico y disponibilidad)

### Alcance
- [x] Listado publico de profesionales.
- [x] Detalle publico profesional.
- [x] Listado publico de servicios por profesional.
- [x] Endpoint de disponibilidad diaria.

### Criterios
- [x] Solo perfiles activos/visibles.
- [x] Solo servicios activos.
- [x] Validacion de propiedad del servicio respecto al profesional.
- [x] Resta de tramos ocupados en disponibilidad.

### Pruebas pendientes
- [~] Integracion de rutas publicas — diferida; rutas publicas ejercidas indirectamente en pruebas de reservas y disponibilidad.
- [~] Casos de error del endpoint de disponibilidad — diferida; logica del motor de disponibilidad cubierta por pruebas unitarias en Fase 1C.

---

## Checklist - Fase 1D (Reservas)

### Alcance
- [x] Crear reserva.
- [x] Listar mis reservas.
- [x] Ver detalle de reserva.
- [x] Cancelar reserva (MVP).
- [x] Escribir booking_history en create/cancel.

### Criterios
- [x] Validacion servicio-profesional.
- [x] Duracion derivada del servicio.
- [x] Ajuste a agenda.
- [x] Rechazo de solapes.
- [x] Calculo backend de precio total.
- [x] Calculo backend de payment_deadline.
- [x] Estado inicial pending y trazabilidad en historial.

### Pruebas pendientes
- [x] Integracion create con escenarios de rechazo.
- [x] Integracion visibilidad por rol en /bookings/me.
- [x] Integracion cancelacion y transiciones de estado.

---

## 2026-03-14 - Fase 2 (inicio)

### Alcance entregado
1. Endpoints de pagos implementados:
   - POST /payments
   - PATCH /payments/{payment_id}/status
   - GET /payments/me
2. Endpoints de facturas implementados:
   - POST /invoices/issue
   - GET /invoices/me

### Trabajo tecnico realizado
1. Modelado ORM de `payments` e `invoices`.
2. Matriz de transiciones de estado de pago.
3. Sincronizacion de reserva al cambiar estado de pago:
   - paid -> confirma booking pendiente
   - refunded/cancelled -> cancela booking pendiente/confirmada
4. Emision automatica de factura al marcar pago como paid.
5. Validaciones de coherencia:
   - acceso segun propiedad/rol
   - consistencia de importes booking-factura
   - control de metodos de pago permitidos

### Validacion
1. Pruebas unitarias de reglas de pagos/facturacion.
2. Estado tras fase: 22 pruebas superadas.

### Decisiones cerradas
1. En MVP se permite emision de factura simplificada y full segun importe.
2. Se mantiene control de flujo por transiciones explicitas en backend.

### Limitaciones conocidas
1. Faltan pruebas de integracion endpoint a endpoint para pagos/facturas.
2. Falta integrar webhooks reales de pasarela (modo sandbox/manual de estado).

### Siguiente fase
Fase 3: mensajeria y soporte.

---

## Checklist - Fase 2 (Pagos y facturacion)

### Alcance
- [x] Crear pago para reserva.
- [x] Actualizar estado de pago con matriz de transiciones.
- [x] Listar pagos por contexto de rol.
- [x] Emitir factura manual por endpoint.
- [x] Listar facturas por contexto de rol.

### Criterios
- [x] Validacion de metodo de pago permitido.
- [x] Rechazo de transiciones de estado invalidas.
- [x] Coherencia booking-pago-factura.
- [x] Emision automatica de factura al confirmar pago.
- [x] Escritura de booking_history al confirmar/cancelar por estado de pago.

### Pruebas pendientes
- [x] Integracion de create payment con escenarios completos.
- [x] Integracion de update status con emision automatica de factura.
- [x] Integracion de listados por rol para payments/invoices.

---

## 2026-03-21 - Fase 3 (Mensajeria y soporte)

### Alcance entregado
1. Endpoints de mensajeria implementados:
   - POST /threads
   - GET /threads/me
   - GET /threads/{thread_id}
   - POST /threads/{thread_id}/messages
   - PATCH /threads/{thread_id}/status
2. Endpoints de soporte implementados:
   - POST /support/tickets
   - GET /support/tickets/me
   - GET /support/tickets/{ticket_id}
   - PATCH /support/tickets/{ticket_id}/status
   - POST /support/tickets/{ticket_id}/attachments

### Trabajo tecnico realizado
1. Modelado ORM de `threads`, `messages`, `support_tickets` y `support_ticket_attachments`.
2. Reglas de acceso por participante para hilos entre cliente y profesional.
3. Soporte de creacion de hilo desde reserva o por contacto directo segun rol activo.
4. Matriz simple de estados para hilos (`open` <-> `closed`).
5. Workflow de estados de soporte con restricciones por rol:
   - admin: flujo completo
   - propietario: apertura/cierre dentro del alcance MVP
6. Validacion server-side de adjuntos por tipo MIME y extension permitida.

### Validacion
1. Pruebas unitarias de reglas de mensajeria y soporte.
2. Estado tras fase (incluyendo integracion): 40 pruebas superadas.

### Decisiones cerradas
1. Los adjuntos de soporte se registran como metadatos; la subida binaria real se difiere.
2. El admin puede ver todos los tickets e hilos; cliente y profesional solo acceden a sus recursos.
3. El hilo puede abrirse desde una reserva existente o por contacto directo segun el rol autenticado.

### Limitaciones conocidas
1. La cobertura de integracion de threads/support esta en progreso; quedan casos de borde adicionales.
2. Falta marcar mensajes como leidos y emitir notificaciones reales.
3. La subida fisica y almacenamiento seguro de archivos adjuntos aun no esta implementada.

### Siguiente fase
Pruebas de integracion y endurecimiento transversal.

---

## Checklist - Fase 3 (Mensajeria y soporte)

### Alcance
- [x] Crear hilo entre cliente y profesional.
- [x] Listar hilos por contexto de rol.
- [x] Consultar detalle de hilo con mensajes.
- [x] Enviar mensajes dentro de un hilo.
- [x] Abrir/cerrar hilo.
- [x] Crear ticket de soporte.
- [x] Listar tickets por contexto de visibilidad.
- [x] Consultar ticket con adjuntos.
- [x] Actualizar estado de ticket.
- [x] Registrar adjuntos por metadatos.

### Criterios
- [x] Acceso solo para participantes del hilo o admin.
- [x] Posibilidad de asociar hilo a una reserva existente.
- [x] Restriccion de mensajes a hilos abiertos.
- [x] Validacion de workflow de estados de soporte.
- [x] Validacion de tipos de adjunto permitidos.

### Pruebas pendientes
- [x] Integracion completa de create/list/detail para hilos.
- [x] Integracion de cambios de estado de soporte por rol.
- [x] Integracion de adjuntos con escenarios de rechazo HTTP.

---

## 2026-03-28 - Endurecimiento transversal 

### Alcance 
1. Contrato de error unificado para toda la API:
   - `code`
   - `message`
   - `field_errors`
   - `trace_id`
2. Endpoint adicional de mensajeria:
   - PATCH /threads/{thread_id}/messages/{message_id}/read
3. Gestion de sesiones y logout dentro del alcance PI:
   - con soporte persistente de sesiones mediante `auth_sessions`
   - sin tabla dedicada `revoked_refresh_tokens` en esta fase
4. Endpoint de refresh con rotacion y bloqueo de reutilizacion:
   - POST /auth/refresh
5. Estandarizacion de codigos de error semanticos por modulo.

### Trabajo tecnico realizado
1. Handlers globales para `HTTPException`, `RequestValidationError` y excepciones no controladas.
2. Middleware HTTP para generar y propagar `trace_id` en cabecera de respuesta.
3. Implementacion de read-state en mensajes por participante autorizado.
4. Gestion de sesiones activas mediante `sid` y persistencia en `auth_sessions`, con soporte de estado activo o revocado.
5. Logout y revocacion de sesion dentro del alcance actual, sin incorporar todavia una tabla especifica de blacklist de refresh tokens.
6. Generacion de codigos de error por modulo a partir del path de request y del mensaje de error.
7. Endpoints para listar sesiones y revocar una o todas las sesiones del usuario.

### Validacion
1. Pruebas de integracion para:
   - envelope de error en 404 y 422
   - marcado de mensaje como leido
   - idempotencia de logout con desactivacion de sesion en `auth_sessions`
   - refresh rotation y rechazo de reutilizacion de refresh
2. Estado actual global: 45 pruebas superadas.
3. Pruebas adicionales de sesiones por dispositivo:
   - revocacion de una sesion sin afectar otra sesion activa
   - revocacion global y bloqueo de refresh en todas las sesiones
   - manejo 404 semantico en sid inexistente
4. Estado actual global tras sesiones por dispositivo: 48 pruebas superadas.
5. Ampliacion con politicas avanzadas de sesion:
   - expiracion por inactividad (TTL configurable)
   - limite de sesiones activas por usuario
   - supervision y revocacion administrativa
6. Ampliacion con casos limite de read-state y auditoria:
   - 404 en hilo inexistente
   - 404 en mensaje inexistente en hilo valido
   - emisor puede marcar su propio mensaje como leido
   - lectura de mensaje en hilo cerrado permitida
   - read_at visible para el emisor en detalle del hilo (pista de auditoria)
7. Estado actual global tras bloque completo: 56 pruebas superadas.

### Limitaciones conocidas
1. La gestion por `sid` es funcional, pero faltan politicas avanzadas (limite de dispositivos, expiracion configurable, panel admin).
2. El envelope de error unificado convive con mensajes heredados por `detail` en capas internas.

### Siguiente fase
Bloque cerrado. Ambas lineas de trabajo completadas:
- Politicas avanzadas de sesiones: implementadas y cubiertas por pruebas de integracion.
- Casos limite de read-state y auditoria de mensajes: implementados y cubiertos por pruebas de integracion.

### Checklist de cierre del bloque
- [x] Contrato de error unificado operativo.
- [x] Endpoint read-state en mensajeria.
- [x] Revocacion de sesion en logout sin tabla dedicada de blacklist de refresh tokens en esta fase.
- [x] Refresh rotation con rechazo de reutilizacion.
- [x] Codigos de error semanticos por modulo.
- [x] Gestion de sesiones persistente mediante `auth_sessions` dentro del alcance PI.
- [x] Expiracion de sesion por inactividad (TTL configurable via `sesvia_session_idle_days`).
- [x] Limite de sesiones activas por usuario (configurable via `sesvia_max_active_sessions`).
- [x] Supervision y revocacion administrativa de sesiones.
- [x] Casos limite de read-state: 404 hilo/mensaje inexistente, autoread, hilo cerrado.
- [x] Pista de auditoria basica: read_at visible para todos los participantes del hilo.

## Decision final sobre autenticacion y sesiones

Para el alcance del PI, se mantiene la tabla `auth_sessions` dentro del diseno de base de datos. Esta decision permite registrar sesiones activas, sostener un cierre de sesion mas coherente y reforzar el control basico de sesiones sin introducir una complejidad excesiva en esta fase.

La tabla `revoked_refresh_tokens` se difiere para evolucion posterior al PI. Aunque puede ser util para escenarios de revocacion avanzada de refresh tokens y para un control mas estricto del ciclo de vida de tokens, se ha considerado fuera del alcance necesario para la entrega actual.

Decision final:
- `auth_sessions`: incluida en el alcance del PI
- `revoked_refresh_tokens`: pospuesta para una fase posterior al PI

### Validacion complementaria del flujo de sesion y logout

Se realizo una comprobacion funcional basica del flujo de autenticacion dentro del alcance actual del PI. La validacion incluyo inicio de sesion, acceso a ruta protegida, refresh de tokens, cierre de sesion y verificacion del estado persistido en `auth_sessions`.

Resultados verificados:
- el login genera una sesion valida
- una ruta protegida responde correctamente con el access token
- el refresh funciona mientras la sesion sigue activa
- el logout desactiva la sesion en `auth_sessions`
- tras el logout, el refresh del mismo contexto de sesion queda bloqueado

Limitacion conocida:
- en la implementacion actual, las rutas protegidas no comprueban todavia el estado de sesion en cada peticion
- por ello, un access token ya emitido podria seguir siendo valido hasta su expiracion natural
- la invalidacion garantizada en el alcance actual se aplica al nivel de sesion persistida y flujo de refresh

---

## 2026-03-31 - Integracion frontend (bloques funcionales PI)

### Alcance entregado
1. Estabilizacion de bloque de soporte en frontend (cliente/pro/admin):
   - creacion de ticket
   - listado de tickets
   - detalle de ticket
   - cambios de estado segun rol
2. Ajuste de cola de revision admin de profesionales:
   - profesionales ya aprobados dejan de aparecer en la lista de revision
3. Bloque de operativa profesional (MVP) completado:
   - agenda (lectura)
   - servicios (crear, editar, activar/desactivar)
   - disponibilidad (crear, editar, activar/desactivar)
4. Bloque de perfil/configuracion iniciado e integrado:
   - PATCH `/auth/me` desde UI
   - avatar por URL (`avatar_url`)
   - idioma preferido (`lang_pref`)
   - PATCH `/pro-profile/me` para perfil publico profesional

---

## 2026-04-12 - Cambios y estado PI

### Alcance entregado
1. Implementado: soporte bidireccional en tickets de soporte (`support_ticket_replies`), endpoints de respuesta y visualización de hilo completo.
2. Refuerzo de validación y visualización en frontend para tickets, agenda profesional, y perfil de usuario/profesional.
3. Mejoras en la gestión de estado SPA, i18n y validación de IVA en servicios.
4. Eliminadas referencias y código de flujo unidireccional de soporte.
5. Documentación y comentarios actualizados en backend y frontend para reflejar el estado real del código.

### Cambios respecto a lo planificado
1. No se implementó la subida binaria real de adjuntos en soporte (solo metadatos).
2. No se añadió tabla de auditoría para revisiones admin.
3. El calendario visual de reservas en frontend quedó pendiente de pulir.
4. Galería de imágenes por servicio y panel admin avanzado quedan como mejoras futuras.
5. Emails reales, integración OAuth, y reembolsos automáticos no implementados (ver tfg-todos).

### Validación
1. Pruebas manuales y unitarias en backend y frontend para los nuevos endpoints y flujos.
2. Verificación de que los cambios cumplen los requisitos funcionales mínimos para la entrega PI.

### Siguiente fase
1. Cierre de pruebas, entrega y defensa del PI.

---

### Alcance entregado
1. Soporte deja de ser un flujo unidireccional:
   - `POST /support/tickets/{ticket_id}/replies`
   - `GET /support/tickets/{ticket_id}` devuelve tambien respuestas
2. Actualizacion de la documentacion resumen para reflejar el estado real del proyecto.

### Trabajo tecnico realizado
1. Se incorpora el modelo `support_ticket_replies` al esquema principal y a la exportacion de base de datos.
2. Se habilita el hilo de respuestas entre usuario y admin dentro del mismo ticket.
3. Se revisan los README principales para eliminar referencias antiguas al flujo "one-way".

### Decision cerrada
1. El ticket de soporte queda modelado como conversacion ligera texto + adjuntos por metadatos; la subida binaria real sigue diferida.

### Trabajo tecnico realizado
1. Refuerzo de estado y renderizado SPA en `frontend/js/app.js` para nuevas secciones profesionales.
2. Ampliacion de `frontend/js/api.js` con wrappers de perfil:
   - `patchMe`
   - `getMyProProfile`
   - `patchMyProProfile`
3. Ampliacion i18n en `frontend/js/i18n.js` para etiquetas y mensajes de settings/profiles.
4. Reglas de IVA aplicadas en UI para servicios:
   - solo 21% o 10% en modo no exento
   - 0% exclusivamente cuando `tax_exempt = true`
   - validacion preventiva antes de enviar payload
5. Mejora visual de estado de tickets de soporte con pills por color.

### Validacion
1. Verificacion de sintaxis frontend con `node --check` en:
   - `frontend/js/api.js`
   - `frontend/js/app.js`
   - `frontend/js/i18n.js`
2. Regresion backend focalizada en soporte/revision admin:
   - lote de pruebas en verde durante el bloque (incluyendo exclusion de profesionales aprobados en review)

### Decisiones cerradas (alcance PI)
1. Se mantiene soporte MVP sin tabla de respuestas de ticket.
2. Se pospone tabla de auditoria adicional para revisiones admin.
3. Avatar de perfil se cubre con `users.avatar_url` (sin migracion extra en este bloque).

### Limitaciones conocidas
1. Calendario visual de reservas aun no implementado completamente en frontend.
2. Imagenes por servicio (galeria/logo de servicio) requieren evolucion de modelo/API y estrategia de almacenamiento.

### Siguiente fase
1. Completar bloque de calendario/disponibilidad visual.
2. Cerrar ronda de pruebas end-to-end frontend + backend para preparacion de entrega PI.

---

## 2026-04-13 - Google OAuth y ajustes compartidos de cuenta

### Alcance entregado
1. Cierre del bloque de acceso con Google para alta/login y vinculacion posterior desde una cuenta local ya existente.
2. Unificacion del area compartida de cuenta en frontend para cliente y profesional:
   - `Preferences`
   - `Connections`
   - `Security & access`
3. Exposicion del estado de conexiones de autenticacion dentro de `GET /auth/me`.

### Trabajo tecnico realizado
1. Ampliacion de `UserView` y esquemas asociados con `auth_connections` para que frontend conozca proveedores activos por cuenta.
2. Implementacion del endpoint `POST /auth/google/connect/start` y reutilizacion de la callback OAuth existente con modo `connect`.
3. Vinculacion explicita de Google a una cuenta autenticada sin depender de igualdad exacta entre el email principal y el email devuelto por Google.
4. Refuerzo de `frontend/js/app.js` para mostrar estado de conexion local/Google, CTA de vinculacion y mensajeria distinta cuando no existe password local.
5. Actualizacion de `frontend/js/api.js` e `frontend/js/i18n.js` para soportar el nuevo flujo de conexiones y el retorno OAuth desde settings.

### Validacion
1. Nuevas pruebas de integracion en `backend/tests/test_phase44_google_auth.py` para:
   - vinculacion de Google desde una cuenta autenticada
   - rechazo de doble vinculacion
2. Regresion focalizada de sesiones y seguridad en `backend/tests/test_phase36_integration_error_readstate_sessions.py`.
3. Verificacion de sintaxis frontend con `node --check` en:
   - `frontend/js/api.js`
   - `frontend/js/app.js`
   - `frontend/js/i18n.js`

### Decisiones cerradas
1. La conexion de Google se gestiona dentro de `Settings > Connections`, no como pantalla separada.
2. Si el email principal de la cuenta local no coincide con el email devuelto por Google, la vinculacion no sobreescribe ni auto-verifica el email principal.
3. `Security & access` mantiene cambios de email/password solo para cuentas con proveedor local activo.

### Limitaciones conocidas
1. No existe aun accion de desconexion de Google desde la UI.
2. No existe aun conversion guiada desde cuenta solo-Google a cuenta local con password propio.

### Siguiente fase
1. Recuperacion de password.
2. Emails reales / SMTP productivo.

---

## 2026-04-14 - SMTP, recuperacion de password y emails transaccionales

### Alcance entregado
1. Cierre del bloque de correo transaccional real con helper SMTP compartido para autenticacion, reservas, facturacion y notificaciones internas.
2. Implementacion completa de recuperacion de password desde login para cuentas locales.
3. Preferencia compartida de cuenta para recibir avisos por email de mensajes internos y respuestas de soporte.

### Trabajo tecnico realizado
1. Nuevo modulo comun de correo en `backend/app/core/email.py` con envio SMTP, adjuntos y modo estricto para flujos de seguridad.
2. Reutilizacion del correo compartido en autenticacion para verificacion de cuenta, reenvio de verificacion, recuperacion de password y confirmacion de cambios sensibles.
3. Nuevos endpoints `POST /auth/password-reset/request` y `POST /auth/password-reset/confirm`.
4. Envio best-effort de confirmacion de reserva e invoice PDF por email desde reservas y pagos.
5. Avisos por email para mensajes de hilos internos y respuestas admin de soporte, controlados por la preferencia `email_message_notifications`.
6. Ajustes frontend en login y settings para reset de password y preferencia de avisos email.

### Validacion
1. Nuevas pruebas en `backend/tests/test_phase45_email_notifications.py` para reseteo de password, confirmacion de reserva + invoice y preferencia de avisos email.
2. Regresion focalizada en:
   - `backend/tests/test_phase34_integration_booking_payments.py`
   - `backend/tests/test_phase35_integration_messaging_support.py`
   - `backend/tests/test_phase42_billing_security.py`
   - `backend/tests/test_phase44_google_auth.py`
3. Verificacion de sintaxis frontend con `node --check` en:
   - `frontend/js/api.js`
   - `frontend/js/app.js`
   - `frontend/js/i18n.js`

### Decisiones cerradas
1. Los emails de seguridad fallan de forma explicita si SMTP esta activado pero mal configurado.
2. Los emails de negocio son best-effort y no bloquean la operativa principal.
3. Para esta fase PI, el aviso de mensajes por email se resuelve como copia/aviso de hilos internos y soporte, sin notificaciones push.

### Limitaciones conocidas
1. Los recordatorios previos a la cita y los correos de cambio, cancelacion o fallo de pago siguen diferidos.
2. La emision de factura ya envia email automaticamente, pero no existe aun un boton explicito de reenvio manual para una factura ya emitida.
3. El envio real fuera de desarrollo depende de configurar las variables SMTP del entorno.

### Siguiente fase
1. Recordatorios y avisos email avanzados.
2. Reembolsos y politicas de cancelacion.

---

## 2026-04-15 - Retorno de Stripe a citas y aviso de pago fallido

### Alcance entregado
1. Refuerzo del retorno del flujo Stripe para que el cliente vuelva a `Appointments` con la reserva recien creada ya seleccionada.
2. Aviso visual mas claro en el detalle de la cita para estados de pago:
   - confirmado
   - pendiente
   - cancelado por salida del checkout
   - fallido
3. Email de pago fallido con enlace de vuelta a la cita para reintentar el cobro.
4. Consolidacion del flujo de exito: un solo email de confirmacion de reserva con la factura PDF adjunta cuando ya existe.

### Trabajo tecnico realizado
1. Ajustes en `frontend/js/app.js` para mantener el contexto del `booking_id` devuelto por Stripe y abrir directamente el panel lateral de detalle de la cita.
2. Nuevo estado de `flow notice` en citas del cliente con copy e indicadores visuales especificos por resultado del cobro.
3. Ajustes de estilos en `frontend/css/app.css` para banners de estado `success` y `warning`.
4. Nuevo helper backend para email de pago fallido con CTA de reintento.
5. Refactor del flujo de correo de pago exitoso para adjuntar la factura PDF al email de confirmacion en vez de disparar dos correos separados en el camino automatico.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/i18n.js`
3. `pytest backend/tests/test_phase45_email_notifications.py`
4. `pytest backend/tests/test_phase34_integration_booking_payments.py`

### Decisiones cerradas
1. Para el flujo automatico normal de pago confirmado se prioriza un solo correo de confirmacion con factura adjunta, en lugar de dos emails consecutivos.
2. El email de pago fallido cubre fallos reales o expiracion de sesion; abandonar manualmente el checkout sigue resolviendose sobre todo en la UI al volver a `Appointments`.

### Limitaciones conocidas
1. Los recordatorios previos a cita y los avisos de cambio o cancelacion siguen pendientes.
2. No se ha anadido un boton manual de reenvio de factura porque la descarga PDF ya esta cubierta y el correo automatico principal incorpora la factura en el flujo normal.

---

## 2026-04-16 - Busqueda en historial de citas y clientes

### Alcance entregado
1. Busqueda en `Appointments` del cliente sobre todo su historial cargado de citas.
2. Busqueda en `Calendar` del profesional por cliente, contacto, servicio, fecha, estado o identificador.
3. Vista de resultados plana cuando hay busqueda activa, manteniendo filtros de estado y acceso al detalle lateral.

### Trabajo tecnico realizado
1. Nuevos estados `clientAppointmentsSearchQuery` y `proAgendaSearchQuery` en `frontend/js/app.js`.
2. Helper comun de normalizacion para busqueda tolerante a mayusculas y acentos.
3. Barra de busqueda compartida con contador de resultados y boton de limpieza.
4. Estilos responsive en `frontend/css/app.css` y microcopy en `frontend/js/i18n.js`.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/i18n.js`
3. `node --check frontend/js/api.js`

### Decisiones cerradas
1. Se evita crear una nueva entidad backend de clientes por ahora: la base se resuelve sobre `/bookings/me`, que ya devuelve el historial accesible por rol.
2. La busqueda se mantiene del lado frontend porque la agenda actual carga todas las reservas relevantes del usuario para esta fase del TFG.

### Siguiente fase
1. Politica de cancelacion y reembolso en modo test.
2. Acceso admin puntual de solo lectura.

---

## 2026-04-17 - Politicas fijas de cancelacion y reembolso

### Alcance entregado
1. Politica de cancelacion configurable por servicio con tres opciones cerradas:
   - `non_refundable`
   - `flexible_7d`
   - `flexible_24h`
2. Copia de la politica en cada reserva para conservar las condiciones aceptadas por el cliente aunque el servicio cambie despues.
3. Visualizacion de la politica en tarjetas de servicio, flujo de reserva, confirmacion y detalle de cita para cliente y profesional.
4. Reembolso automatico local en modo test al cancelar una reserva pagada dentro de la ventana reembolsable.

### Trabajo tecnico realizado
1. Nuevo modulo `app.core.cancellation` con normalizacion, calculo de fecha limite y elegibilidad de reembolso.
2. Ampliacion de modelos, esquemas, SQL base y actualizacion runtime para `services.cancellation_policy` y `bookings.cancellation_policy`.
3. Actualizacion de endpoints de servicios, publicos y reservas para validar, exponer y aplicar la politica.
4. Actualizacion de `frontend/js/app.js`, `frontend/js/i18n.js` y `frontend/css/app.css` para selector profesional y copy visible de politica.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/i18n.js`
3. `.venv\\Scripts\\python.exe -m pytest backend/tests/test_cancellation_policy_rules.py backend/tests/test_phase34_integration_booking_payments.py`

### Decisiones cerradas
1. La opcion intermedia se fija en 7 dias antes de la cita para evitar campos libres y mantener reglas claras.
2. Las ausencias no presentadas se indican siempre como no reembolsables.
3. La fase actual marca el pago local como `refunded` en modo test; la operacion remota real contra Stripe queda como posible endurecimiento posterior si se despliega con cobros reales.

### Siguiente fase
1. Acceso admin puntual de solo lectura.
2. Estandarizacion de ingles y pulido visual de demo.

---

## 2026-04-18 - Vista admin puntual de solo lectura

### Alcance entregado
1. Vista de detalle por usuario para administracion sin impersonacion.
2. Lectura agrupada de:
   - cuenta
   - perfil profesional, si existe
   - datos fiscales resumidos
   - reservas relacionadas
   - pagos relacionados
   - facturas relacionadas
3. Boton `Ver` en el listado de usuarios del panel admin para abrir el detalle.

### Trabajo tecnico realizado
1. Nuevos esquemas `AdminUserReadOnlyDetailView` y snapshots asociados en `backend/app/schemas.py`.
2. Nuevo endpoint `GET /auth/admin/users/{user_public_uuid}/detail` con control de rol admin.
3. Nuevas funciones frontend en `frontend/js/api.js` y `frontend/js/app.js` para cargar y renderizar el detalle.
4. Estilos responsive en `frontend/css/app.css` y textos i18n para la vista admin.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/api.js`
3. `node --check frontend/js/i18n.js`
4. `.venv\\Scripts\\python.exe -m pytest backend/tests/test_phase41_admin_professional_review.py`

### Decisiones cerradas
1. No se implementa impersonacion para el TFG: se usa visor read-only para evitar riesgos de seguridad y trazabilidad.
2. La vista queda limitada a datos necesarios para soporte/diagnostico y no incluye acciones de escritura.

### Siguiente fase
1. Estandarizacion de ingles y pulido visual de demo.
2. Editar cita solo cliente si sobra tiempo.

---

## 2026-04-19 - Decisiones de alcance para defensa

### Alcance documentado
1. Facturacion:
   - La demo trabaja normalmente con factura simplificada.
   - El backend conserva factura completa para importes > 400 EUR.
   - La UX de cliente para introducir, guardar y gestionar datos fiscales quedo cerrada posteriormente en la seccion `Datos fiscales de cliente para factura completa`.
2. Festivos:
   - Para el TFG se resuelven mediante bloqueos manuales de fechas/horas por el profesional.
   - El calendario nacional/autonomico automatico queda como mejora futura.
3. Admin:
   - Se descarta impersonacion en el TFG.
   - Se documenta el visor puntual read-only como alternativa mas segura para soporte.
4. Seguridad:
   - Se documenta la separacion entre validaciones frontend de UX y validaciones backend reales.
   - Se recogen hashing de passwords, sesiones persistentes, refresh rotation, `trace_id`, variables de entorno y HTTPS.
   - La validacion documental de identidad queda como mejora futura.
5. Frontend:
   - Se justifica la SPA ligera con ES modules y template literals.
   - React u otro framework queda como posible evolucion posterior, no como deuda bloqueante del TFG.
6. Ingles de la UI:
   - Se normaliza el locale ingles hacia ingles US (`en`).
   - Se mantienen formas como `canceled`, `center` y `finalized`.
   - Existe ademas un locale `en-gb` (Ingles UK) que hereda de `en` via `LOCALE_PARENTS` y solo sobreescribe las diferencias ortograficas (`cancelled`, `booking` en lugar de `appointment` donde aplica, etc.).

### Archivos actualizados
1. `README.md`
2. `E4_Ana_Vertedor_App_Plataforma_Gestion_Citas.md`
3. `docs/tfg-todos.md`

### Decision importante
1. No se simplifica la logica de facturacion del backend: la ruta de factura completa para importes superiores a 400 EUR se mantiene.
2. El cierre funcional de la UX de datos fiscales queda documentado mas adelante en este mismo log.

---

## 2026-04-20 - Pulido visual ligero de demo

### Alcance entregado
1. Movimiento suave de entrada para el contenido principal de la SPA.
2. Animacion escalonada discreta para tarjetas de dashboard y grids principales.
3. Respeto de `prefers-reduced-motion` para no forzar animaciones a usuarios que las desactivan.

### Trabajo tecnico realizado
1. Nuevos keyframes `sesvia-page-rise` y `sesvia-card-in` en `frontend/css/app.css`.
2. Aplicacion de animaciones solo bajo `prefers-reduced-motion: no-preference`.
3. Actualizacion de `docs/tfg-todos.md`.

---

## 2026-04-21 - Preparacion de despliegue Railway

### Alcance entregado
1. El backend FastAPI puede servir el frontend estatico desde `/frontend/`, permitiendo una demo Railway de un solo servicio.
2. La API del frontend ya no depende solo de `http://127.0.0.1:8000`:
   - en local mantiene el backend de desarrollo
   - en despliegue usa el mismo origen por defecto
   - permite override con `window.SESVIA_API_BASE`, `<meta name="sesvia-api-base">` o `localStorage.sesvia.api_base`
3. CORS queda configurable por entorno con `SESVIA_CORS_ORIGINS`.

### Trabajo tecnico realizado
1. `frontend/js/api.js`: resolucion dinamica de API base.
2. `backend/app/main.py`: servicio de `index.html` y montaje de `frontend/` como static files.
3. `backend/app/core/config.py` y `backend/.env.example`: nueva variable `SESVIA_CORS_ORIGINS`.
4. `README.md` y `backend/README.md`: notas de despliegue Railway.

### Pendiente de prueba real
1. Publicar en Railway.
2. Verificar `/health`.
3. Abrir `/frontend/`.
4. Probar login, Google OAuth, SMTP real y retorno de Stripe con las URLs publicas.

---

## 2026-04-22 - Opciones pequenas de conexiones y servicios

### Alcance entregado
1. Google conectado puede desvincularse desde `Settings > Connections`.
2. La desvinculacion de Google se bloquea si la cuenta no tiene password local, para evitar dejar al usuario sin acceso.
3. Stripe conectado muestra boton de desconexion en la tarjeta de cobros online del profesional.
4. La desconexion de Stripe limpia solo el enlace local de Sesvia; no elimina la cuenta externa de Stripe.
5. Los servicios muestran placeholder visual segun modo de servicio en catalogo y panel profesional.

### Trabajo tecnico realizado
1. Backend:
   - `DELETE /auth/google/connection`
   - `DELETE /payments/stripe/setup`
2. Frontend:
   - Nuevas llamadas API de desconexion.
   - Botones con modal de confirmacion para Google y Stripe.
   - Placeholders de servicios con estilos dedicados.
3. Tests:
   - Cobertura de desconexion Google segura.
   - Cobertura de limpieza local de Stripe.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/api.js`
3. `node --check frontend/js/i18n.js`
4. `python -m py_compile` sobre endpoints y tests tocados.
5. `pytest backend/tests/test_phase44_google_auth.py`
6. `pytest backend/tests/test_phase46_stripe_disconnect.py`

---

## 2026-04-23 - Datos fiscales de cliente para factura completa

### Alcance entregado
1. Las cuentas cliente pueden gestionar `Datos de facturacion` desde ajustes.
2. Las reservas superiores a 400 EUR exigen datos fiscales antes de ir a pago.
3. En el flujo de reserva, el cliente puede:
   - usar datos fiscales ya guardados
   - introducir datos fiscales nuevos
   - elegir si guardarlos en cuenta o usarlos solo para esa reserva
4. La reserva guarda `billing_detail_id` para que la factura completa use los datos correctos aunque no sean el default de cuenta.

### Trabajo tecnico realizado
1. Backend:
   - `bookings.billing_detail_id` en modelo ORM y SQL base.
   - `BookingCreateRequest` acepta `billing_detail_id`, `billing_detail` y `save_billing_detail`.
   - Los endpoints de billing details ya son accesibles a usuarios autenticados cliente/profesional.
   - La emision de factura completa prioriza los datos fiscales vinculados a la reserva.
2. Frontend:
   - Seccion de datos de facturacion en ajustes de cliente.
   - Bloque de factura completa en el paso de pago de reserva.
   - Checkbox para guardar o no guardar datos fiscales nuevos en cuenta.
3. Tests:
   - Reserva > 400 EUR rechazada sin datos fiscales.
   - Reserva > 400 EUR con datos fiscales puntuales no guardados.
   - Cliente puede crear datos fiscales default sin rol profesional.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/i18n.js`
3. `python -m py_compile` sobre modelos, esquemas, endpoints y tests tocados.
4. `pytest backend/tests/test_phase34_integration_booking_payments.py`
5. `pytest backend/tests/test_phase42_billing_security.py`

---

## i18n — Ingles UK, Frances y sincronizacion de locales

### Cambios
1. Nuevos locales en `frontend/js/i18n.js`:
   - `en-gb` (Ingles UK): hereda de `en` via `LOCALE_PARENTS`; solo almacena diferencias ortograficas (~8 claves: `cancelled`, `booking cancelled`, etc.). No requiere cambios en base de datos.
   - `fr` (Frances): traduccion completa de los 17 bloques (ui, auth, authRecovery, guest, consent, verification, professional, home, authEntry, agendaUi, supportHub, settingsUi, clientPublicProfile, appointmentsUi, publicVerifyUi, printUi, legal).
2. Selector de idioma actualizado con emojis de bandera para diferenciar visualmente los dos ingleses: 🇺🇸 English (US) vs 🇬🇧 English (UK).
3. Locales soportados actualizados: `["es", "ca", "gl", "eu", "en", "en-gb", "fr"]`.
4. Fallback chain extendido: `t()` y `tUi()` consultan el locale activo → padre (si existe via `LOCALE_PARENTS`) → español → ingles.
5. Sincronizacion de claves faltantes:
   - `browseStripePaymentFailed` anadida a `ca`, `gl` y `eu` (existia en `es` y `en`).
   - `messageNotificationsLabel` y `messageNotificationsHint` anadidas a `settingsUi` de `es`, `ca`, `gl`, `eu` y `fr` (existian solo en `en.settingsUi`).
   - `fr.settingsUi` reemplazado: se elimina la plantilla generica inicial y se sustituye por una traduccion completa alineada con la estructura real de los otros locales (Google OAuth, contrasena local, facturacion, preferencias, etc.).
   - `clientBillingDetailsTitle/Body/Hint` y `billingSaveSuccess/Error` presentes en `fr.settingsUi`.

### Archivos actualizados
1. `frontend/js/i18n.js`
2. `docs/later-features.md`
3. `docs/implementation-log.md`

### Validacion
1. `node --check frontend/js/i18n.js`

---

## 2026-04-24 - Actualizacion documental de facturacion completa

### Alcance documentado
1. `README.md` refleja que los datos fiscales de cliente para factura completa ya estan implementados.
2. `backend/README.md` documenta el uso de datos fiscales vinculados a la reserva cuando el importe supera 400 EUR.
3. `docs/tfg-todos.md` marca como cerrado el flujo de datos fiscales del cliente para factura completa.
4. `E4_Ana_Vertedor_App_Plataforma_Gestion_Citas.md` elimina la formulacion de pendiente y describe el comportamiento real para defensa.
5. `docs/later-features.md` conserva como post-TFG los recordatorios, emails avanzados de cambio/cancelacion y reembolsos remotos reales con Stripe.

### Validacion
1. Revision textual con `rg` para evitar contradicciones sobre el estado de la factura completa.

---

## 2026-04-25 - Disponibilidad profesional: turnos partidos, festivos y collapsibles

### Alcance entregado
1. Corrección del bug de pérdida de datos al guardar el horario semanal: el segundo paso de guardado tiene rollback que reactiva los tramos desactivados si falla.
2. Soporte de turno partido (múltiples franjas por día) mediante botón "Añadir franja" en el editor de horario semanal.
3. Tooltip informativo sobre turnos nocturnos que cruzar la medianoche.
4. Festivos nacionales con nombre correcto (ñ y tildes), traducidos a todos los locales mediante `name_key`.
5. Botón de ignorar por festivo individual (localStorage por profesional).
6. Eliminación del aviso de festivos en la vista pública del perfil profesional.
7. Secciones del editor de disponibilidad convertidas a collapsibles independientes:
   - Horario semanal: cerrado por defecto, toggle independiente.
   - Bloques manuales y festivos: cerrado por defecto, toggle independiente.
   - Lista de fechas dentro de bloques: tercer toggle interior independiente.
8. Corrección de todos los errores mojibake (UTF-8 leído como Latin-1) en `app.js`, `i18n.js` y `api.js`: 751 reemplazos en total.

### Trabajo tecnico realizado
1. `frontend/js/app.js`:
   - Rollback en `onSaveWeeklyAvailability` con array `deactivatedIds` y re-activación en error del segundo paso.
   - Nueva función `addProAvailabilityRange(weekday)` con hora de inicio inteligente basada en el último tramo.
   - `renderProAvailabilityEditor()` y `renderProAvailabilityBlocksManager()` con `data-toggle`, chevron SVG y flags de estado.
   - Helpers: `getHolidayLabel()`, `_dismissedHolidaysStorageKey()`, `getDismissedHolidays()`, `dismissHoliday()`.
   - Tres nuevos flags de estado: `proAvailabilityEditorExpanded`, `proAvailabilityBlocksExpanded`, `proAvailabilityBlocksListExpanded`.
   - Manejadores globales de click para `data-holiday-dismiss` y `data-toggle`.
   - SVGs `info` y `chevron` en `_ico`.
2. `frontend/js/i18n.js`:
   - Claves añadidas en los 5 locales: `proAvailabilityAddRange`, `proAvailabilityOvernightHint`, `proAvailabilityBlockDismissHoliday`, `proAvailabilityHolidayPrefix`, `proAvailabilityBlockReadOnlyBadge` (completado en `ca`, `gl`, `eu`).
   - 10 claves `holiday_*` por locale (50 en total): `holiday_ano_nuevo` a `holiday_natividad_del_senor`.
3. `frontend/css/app.css`:
   - `.hint-wrap`, `.hint-bubble` para tooltip (color fijo a `var(--sesvia-6)`).
   - `.availability-card-toggle`, `.availability-block-toggle`, `.availability-block-list-toggle`.
   - `.collapsible-chevron` + rotación con `.is-open`.
4. `backend/app/core/data/holidays/es-national-2026.json`:
   - Nombres de festivos con tildes y ñ correctas.
   - Campo `name_key` añadido a cada entrada.
5. `backend/app/core/holidays.py` y `backend/app/schemas.py`:
   - `HolidayRecord.name_key` y propagación hasta `AvailabilityBlockView`.

### Validacion
1. `node --check frontend/js/app.js`
2. `node --check frontend/js/i18n.js`
3. `node --check frontend/js/api.js`
4. Script Python de detección de mojibake sobre los tres ficheros: 0 patrones restantes.

### Decisiones cerradas
1. El turno nocturno (p. ej. 18:00-02:00) no se divide automáticamente: el profesional añade manualmente la franja del día siguiente; el tooltip informa de esta restricción.
2. Los festivos nacionales son solo lectura en la UI; el profesional puede ignorarlos individualmente sin eliminarlos del sistema.
3. Los collapsibles empiezan cerrados para reducir la carga visual inicial del editor de disponibilidad.

---

## 2026-04-26 - SMTP Resend y nota futura sobre proveedor de email

### Alcance documentado
1. Se valida que el modulo actual de email funciona con SMTP estandar y no requiere integrar de inmediato el SDK de Resend.
2. Se deja registrada como mejora futura la posibilidad de evolucionar `backend/app/core/email.py` hacia una abstraccion de proveedor con adaptadores.

### Decisiones cerradas
1. Para la entrega PI se mantiene SMTP como via principal porque reutiliza el helper existente, evita dependencias nuevas y permite cambiar de proveedor solo mediante variables de entorno.
2. Resend queda configurado como proveedor SMTP de desarrollo/prueba tras las restricciones de SMTP AUTH encontradas en cuentas Microsoft/Outlook.
3. Una migracion posterior al SDK/API de Resend solo compensa si se necesitan capacidades especificas del proveedor: identificadores de mensaje, plantillas, tags, analitica, webhooks o diagnostico de entrega mas granular.
4. La aplicacion no deberia acoplar endpoints de negocio al proveedor: el contrato interno debe seguir siendo un helper comun como `send_email_message(...)`, con SMTP como adaptador actual y Resend API como posible adaptador futuro.

### Nota operativa
1. En local, `SESVIA_EMAIL_DEBUG_ENABLED=true` permite mantener visible el token de respaldo aunque el envio real funcione.
2. En demo/produccion, esta variable debe volver a `false` para no exponer tokens de verificacion en la SPA.

---

## 2026-04-27 - Reenvio manual de facturas por email

### Alcance entregado
1. Nuevo endpoint profesional `POST /invoices/{invoice_id}/email` para reenviar por correo una factura ya emitida.
2. El envio manual usa el PDF generado por backend como adjunto y falla de forma visible si no hay destinatario o si SMTP devuelve error.
3. El detalle de factura profesional incluye boton `Enviar por email` con estado de carga y mensajes de exito/error.
4. `docs/tfg-todos.md` marca cerrado el boton de reenvio manual y mantiene pendiente solo la validacion real desde UI/demo.

### Validacion
1. `python -m pytest backend\tests\test_phase45_email_notifications.py` -> 13 passed.
2. `node --check frontend/js/app.js`
3. `node --check frontend/js/api.js`
4. `node --check frontend/js/i18n.js`

### Nota operativa
1. El envio automatico al emitir factura sigue siendo best-effort para no bloquear la emision.
2. El reenvio manual es estricto porque el profesional ha pedido explicitamente enviar el correo y necesita feedback si falla.
