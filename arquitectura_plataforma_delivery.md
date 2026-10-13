# Propuesta técnica: Plataforma de gestión y administración de deliveries

**Fecha:** 12 de octubre de 2026  
**Estado:** Propuesta inicial / MVP

## 1. Visión y alcance

Crear una plataforma móvil para administrar una operación de motorizados que realizan entregas puerta a puerta. **No es inicialmente una aplicación para que consumidores hagan pedidos.** El dueño/administrador recibe solicitudes por WhatsApp u otros canales, crea y asigna carreras, controla su ejecución, supervisa la ubicación de los motorizados y administra ingresos, comisiones, liquidaciones, gastos y utilidad.

Se desea iniciar aprovechando capas gratuitas para desarrollo y pilotos, sin bloquear el crecimiento posterior a múltiples zonas, sucursales, ciudades o empresas. En el futuro podrían añadirse una aplicación para clientes y un modelo SaaS multiempresa.

### Objetivos funcionales
- Registrar motorizados y permitir que descarguen e inicien sesión en la app.
- Crear carreras, establecer su precio y asignarlas manualmente.
- Mostrar ubicación en tiempo real durante la jornada autorizada.
- Registrar estados, entregas, novedades y evidencias.
- Calcular cuánto se cobra, cuánto corresponde al motorizado y la utilidad de la empresa.
- Distinguir quién recibió el dinero y gestionar cuentas por cobrar, pagos y cierres de caja.
- Enviar notificaciones de nuevas asignaciones y actualizaciones automáticas de la app.
- Evolucionar hacia automatización por WhatsApp e IA.

## 2. Stack tecnológico recomendado

| Capa | Selección | Motivo |
|---|---|---|
| Lenguaje principal | TypeScript | Compartir conocimientos y tipos entre frontend y backend |
| Móvil Android/iOS | React Native + Expo | Una base de código para ambas plataformas |
| Backend | NestJS | API modular y lógica de negocio centralizada |
| ORM | Prisma | Modelado y acceso a datos |
| Base de datos | PostgreSQL | Transacciones, reportes y buena portabilidad |
| Extensión geoespacial | PostGIS | Búsquedas por cercanía, zonas y coordenadas |
| Proveedor inicial de BD | Supabase | PostgreSQL administrado y herramientas adicionales; plan gratuito limitado |
| Tiempo real | WebSockets / Socket.IO | Actualizaciones de carreras y posiciones |
| Notificaciones | Expo Push / Firebase Cloud Messaging | Avisos de asignaciones y novedades |
| Compilación y OTA | Expo EAS Build / EAS Update | Distribución y actualización de código compatible |
| Mapas | Datos OpenStreetMap + proveedor adecuado de mapas/tiles | Visualización geográfica; revisar condiciones de uso |
| Panel web futuro | Next.js | Administración desde escritorio |
| Caché local móvil | SQLite | Guardar temporalmente eventos sin conexión |
| WhatsApp | WhatsApp Business Platform | Integración futura de solicitudes y notificaciones |
| IA | Una API entre OpenAI, Anthropic o Google | Interpretación de mensajes y reportes cuando exista necesidad real |

### Alternativas de lenguaje móvil
- **React Native:** recomendado por velocidad de desarrollo y soporte Android/iOS con TypeScript.
- **Kotlin:** excelente opción Android nativa, con control fino de capacidades del sistema, pero no sustituye una app iOS.
- **Swift:** excelente opción iOS nativa, pero no sustituye una app Android.

### Alternativas de base de datos
- **PostgreSQL + PostGIS:** recomendación principal por costo de licencia, transacciones y capacidades geográficas.
- **MySQL:** alternativa válida, pero se prefiere el ecosistema geoespacial de PostGIS.
- **SQL Server:** técnicamente sólido, aunque menos conveniente para el objetivo inicial de costos y portabilidad.

## 3. Productos e interfaces

### Aplicación del administrador
- Alta y gestión de motorizados y comercios.
- Creación, cotización y asignación de carreras.
- Vista de mapa con motorizados autorizados y estados de disponibilidad.
- Tarifas, comisiones, ingresos, pagos pendientes y gastos.
- Cierres de caja, liquidaciones y reportes.

### Aplicación del motorizado
- Registro/inicio de sesión y perfil.
- Inicio y fin de jornada.
- Recepción y aceptación/rechazo de carreras según reglas de negocio.
- Visualización de direcciones y apertura de navegación GPS.
- Cambio de estados, prueba de entrega y reporte de novedades.
- Consulta de ganancias y liquidaciones.
- Sincronización de eventos al recuperar conexión.

### Panel web (cuando crezca la operación)
- Mapa amplio y supervisión de carreras simultáneas.
- Gestión masiva, auditoría, contabilidad y exportación de reportes.
- Permisos para otros operadores y administradores.

## 4. Flujo operativo de una carrera

1. Un comercio solicita un delivery por WhatsApp u otro medio.
2. El administrador registra origen, destino, comercio, precio y observaciones.
3. Se asigna un motorizado disponible.
4. El motorizado recibe el evento y una notificación push; confirma la asignación.
5. Actualiza el estado durante recogida, traslado y entrega.
6. Se registra quién recibió el dinero, método de pago, evidencia y posibles novedades.
7. El sistema calcula pago al motorizado, margen y saldos pendientes.
8. La carrera entra al cierre y a la liquidación correspondiente.

**Ejemplo ilustrativo:** carrera cobrada a $3, pago al motorizado $2, utilidad **bruta** $1. La utilidad neta requiere descontar los demás costos, descuentos e impuestos aplicables.

**Conceptos separados:** valor cobrado, pago al motorizado, margen bruto, gastos operativos, utilidad neta, dinero efectivamente recibido, cuentas por cobrar y saldo de caja. Registrar si cobró el motorizado, la empresa o si el comercio pagará posteriormente. Evitar confundir facturación con efectivo disponible.

## 5. Arquitectura propuesta

```text
React Native + Expo (administrador / motorizado)
             |
       REST + WebSockets
             |
      NestJS (monolito modular)
             |
  +----------+----------------------------+
  | Usuarios, organizaciones y permisos   |
  | Motorizados y jornadas                |
  | Comercios, tarifas y carreras         |
  | Asignaciones y estados                |
  | Ubicación y zonas                     |
  | Cobros, comisiones y liquidaciones    |
  | Gastos, reportes y auditoría          |
  +---------------------------------------+
             |
     PostgreSQL + PostGIS
             |
  Integraciones: push, mapas, WhatsApp, IA
```

Empezar con **monolito modular**, no microservicios. Separar módulos únicamente cuando haya razones operativas y de escala. El backend debe validar permisos, tarifas, cambios de estado y movimientos financieros; no confiar estos cálculos a la app móvil ni a un modelo de IA.

## 6. GPS y ubicación en tiempo real

- Utilizar `expo-location` y `expo-task-manager` con configuración de permisos y capacidades nativas apropiadas.
- Seguir ubicación durante la jornada y con consentimiento informado; detenerla al terminar.
- Ajustar frecuencia: mayor durante una carrera activa, menor en espera y ninguna fuera de jornada.
- Mantener una posición reciente por motorizado y conservar historial solo durante el plazo necesario.
- PostGIS permite cercanía, pertenencia a zonas y consultas geográficas; no reemplaza por sí solo a un motor de rutas o estimaciones viales.
- Android/iOS restringen el trabajo en segundo plano. No prometer seguimiento continuo si se fuerza el cierre, se revocan permisos o el sistema suspende la aplicación.
- Probar con builds de desarrollo, no depender únicamente de Expo Go.
- Proteger acceso a posiciones, definir retención y respetar normativa de privacidad aplicable.

### Dimensionamiento ilustrativo

Con 100 motorizados, 8 horas diarias, 26 días al mes y una actualización cada 10 segundos:

`100 × 8 × 3600 / 10 × 26 = 7.488.000 actualizaciones/mes`.

Es un cálculo de eventos emitidos, **no** necesariamente mensajes facturables o escrituras efectivas. Evaluar agregación, cambios por distancia y políticas de almacenamiento para evitar costos innecesarios.

## 7. Servicios externos y costos iniciales

| Servicio | Uso | Consideración |
|---|---|---|
| Supabase Free | PostgreSQL administrado y utilidades | Cuotas de BD, almacenamiento, Realtime y posibles pausas; verificar plan vigente |
| Expo EAS | Builds y OTA | Cuotas de compilaciones y usuarios activos para actualizaciones; verificar precios vigentes |
| FCM / Expo Push | Push | Revisar límites, configuración y comportamiento de cada plataforma |
| OpenStreetMap | Datos cartográficos | Los datos abiertos no implican hosting ilimitado de tiles; elegir proveedor apto para producción |
| Vercel | Panel Next.js | Revisar restricciones del plan gratuito para uso comercial; alternativa de hosting de pago |
| Backend NestJS | API | Desarrollo local gratuito; producción requiere hosting disponible y monitoreo |
| WhatsApp Business Platform | Mensajería futura | Precios y reglas según país, categoría y proveedor |
| API de IA | Automatización futura | Cobro por consumo; las suscripciones de chat no incluyen automáticamente uso de API |

Se puede prototipar con un gasto mensual de infraestructura cercano a **$0**, pero no asumir operación comercial gratuita ilimitada. Considerar dominio, publicación en tiendas, dispositivos, proveedor de mapas, backend de producción, backups y observabilidad. Las cifras y cuotas de los proveedores cambian: verificarlas antes de contratar.

## 8. Actualizaciones de la aplicación

Usar Expo EAS Update para publicar cambios compatibles de JavaScript y recursos sin obligar a reinstalar la app. Los cambios de código nativo, permisos o dependencias que lo requieran necesitarán una nueva compilación y publicación por las tiendas. Gestionar versiones y compatibilidad de runtime, y tener un mecanismo de reversión.

## 9. WhatsApp e inteligencia artificial

**No integrar las tres API de IA desde el comienzo.** Primera etapa: flujo manual sólido. Segunda etapa: WhatsApp Business Platform. Tercera etapa: evaluar un solo proveedor (OpenAI, Anthropic o Google) según costo y calidad del caso de uso.

Ejemplo futuro: recibir «Necesito un motorizado para recoger en la farmacia y entregar en el barrio central; valor $3». Un modelo extrae campos y propone una carrera **pendiente de confirmación**. El administrador verifica origen, destino, tarifa y asignación. La IA no debe autorizar por sí sola pagos, liquidaciones ni obligaciones económicas.

## 10. Modelo de datos inicial sugerido

Entidades a considerar:

- `organizations`, `users`, `roles`, `organization_members`
- `drivers`, `driver_documents`, `shifts`, `driver_last_locations`, `driver_location_history`
- `merchants`, `addresses`, `service_zones`, `pricing_rules`
- `deliveries`, `delivery_assignments`, `delivery_status_events`, `delivery_proofs`
- `charges`, `payments`, `driver_earnings`, `driver_settlements`, `cash_closings`, `expenses`
- `notifications`, `audit_logs`

Agregar `organization_id` donde corresponda y aplicar controles estrictos de aislamiento de datos, incluso si al inicio solo existe una empresa. Usar transacciones e idempotencia para operaciones monetarias; guardar historial auditable en vez de sobrescribir movimientos financieros sin trazabilidad.

## 11. Plan por fases

1. **MVP operativo:** autenticación, roles, motorizados, comercios, tarifas, carreras, asignación manual, estados y comisiones.
2. **Operación móvil:** jornadas, push, GPS, confirmación de entrega y sincronización offline.
3. **Control financiero:** cuentas por cobrar, pagos, gastos, cierres de caja, liquidaciones y reportes.
4. **Automatización:** WhatsApp Business, extracción asistida por IA y sugerencias de asignación.
5. **Escalabilidad:** panel web completo, sucursales, varias organizaciones, optimización de rutas y eventual app para clientes.

## 12. Decisiones finales

**Elección recomendada:** TypeScript + React Native/Expo + NestJS + Prisma + PostgreSQL/PostGIS + Supabase inicial + Socket.IO + Expo Push/FCM + EAS Update. Next.js para el panel web. WhatsApp Business e IA después de validar la operación básica.

Priorizar tres resultados: **asignar carreras, visualizar motorizados y cuadrar correctamente cada dólar de la operación**. Diseñar desde el inicio para crecimiento, pero evitar la complejidad de implementar todas las funciones futuras en el MVP.
