# MuleSoft Certification Prep

Repositorio con el proyecto del curso DEX401 de MuleSoft para preparar la certificación **MuleSoft Certified Developer - Level 1 (Mule 4)**.

## Sobre mí

Técnico Superior en DAM, con prácticas en NTT DATA en el área de Arquitectura de Integración. Documento aquí mi proceso de preparación para la certificación.

### [dex401-course-project](./dex401-course-project)

Proyecto del curso oficial **Anypoint Platform Development: Fundamentals (DEX401)**, desarrollado módulo a módulo. El progreso se documenta a través de los commits del proyecto.

## Progreso general

| Módulo | Estado |
|---|---|
| [1 — Acceso y modificación de eventos Mule](#módulo-1--acceso-y-modificación-de-eventos-mule) | 🔄 En progreso |
| [2 — Estructuración de aplicaciones Mule](#módulo-2--estructuración-de-aplicaciones-mule) | ⏳ Pendiente |
| [3 — Consumo de web services](#módulo-3--consumo-de-web-services) | ⏳ Pendiente |
| [4 — Control de flujo de eventos](#módulo-4--control-de-flujo-de-eventos) | ⏳ Pendiente |
| [5 — Manejo de errores](#módulo-5--manejo-de-errores) | ⏳ Pendiente |
| [6 — Transformaciones DataWeave](#módulo-6--transformaciones-dataweave) | ⏳ Pendiente |
| [7 — Disparadores de flows](#módulo-7--disparadores-de-flows) | ⏳ Pendiente |
| [8 — Procesamiento de registros](#módulo-8--procesamiento-de-registros) | ⏳ Pendiente |

---

## Módulo 1 — Acceso y modificación de eventos Mule

- [x] **1-1** Visualizar datos del evento (HTTP Listener, DataSense Explorer, Logger, query parameters)
- [x] **1-2** Depurar una aplicación Mule con el debugger
- [x] **1-3** Rastrear datos del evento al entrar y salir de una aplicación Mule
- [x] **1-4** Configurar datos de request y response
- [ ] **1-5** Obtener y modificar datos del evento con expresiones DataWeave
- [ ] **1-6** Definir y usar variables

<details>
<summary>Notas de aprendizaje</summary>

**Walkthrough 1-1:** creado el proyecto `apdev-examples` con un HTTP Listener (`GET /hello`), un Set Payload y un Logger. Practicada la diferencia entre **query parameters** (`attributes.queryParams`, van tras el `?` en la URL) y **URI parameters** (`attributes.uriParams`, parámetros dinámicos dentro de la propia ruta). Usado el DataSense Explorer para inspeccionar payload/attributes en cada componente del flow.

**Walkthrough 1-2:** practicado el uso del Mule Debugger sobre `apdev-examples` — breakpoints en componentes (Set Payload, Logger), avance paso a paso con "Next processor" e inspección de `payload`/`attributes`/`queryParams` en cada punto del flow. Provocado intencionadamente un timeout de petición HTTP dejando el debugger pausado sin resumir, y configurado el timeout de Postman (Settings > General > Request timeout in ms) para controlar cuánto tiempo espera el cliente antes de fallar. Aprendida la diferencia entre "Resume" (continúa hasta el siguiente breakpoint o el final) y "Next processor" (avanza componente a componente).

**Walkthrough 1-3:** practicado el rastreo de datos de eventos (Event Data Tracking) a medida que entran y salen de una aplicación Mule, simulando la llamada a un recurso externo mediante un segundo flujo en el mismo proyecto.
* Creado un segundo flujo llamado `goodbyeFlow` con su propio HTTP Listener (GET /goodbye) que expone un endpoint local, un Set Payload ("Goodbye") y un Logger. Asegurado el uso de la misma configuración global existente (`HTTP_Listener_config`) compartiendo el puerto 8081.
* Añadido un componente HTTP Request dentro de `helloFlow` (antes de su Logger original) configurado para invocar dinámicamente mediante GET el recurso `/goodbye` expuesto localmente en `localhost:8081`.
* Configurado el "Response timeout" del HTTP Request de manera temporal a 300000 ms (300 segundos) en la pestaña Response, evitando que la petición sufra un timeout en el cliente emisor mientras se inspeccionan detenidamente los datos en el Mule Debugger.
* Analizado el comportamiento del flujo de datos en tránsito con el Mule Debugger:
  * Al llegar la petición original a `helloFlow`, el payload inicial y los queryParams (`fname=max&lname=mule`) se reciben correctamente.
  * Al hacer "Step into" en el procesador HTTP Request, el evento cruza la barrera de transporte físico (transport boundary): los query parameters originales de la petición externa de Postman se pierden para el nuevo flujo receptor. El payload que entra a `goodbyeFlow` pasa a ser temporalmente el texto "Hello" (que era el payload de salida generado por el componente previo en `helloFlow`).
  * Al retornar la respuesta del HTTP Request hacia `helloFlow`, los atributos del evento mutan por completo: la estructura cambia de `HttpRequestAttributes` (los metadatos de la petición entrante original de Postman) a `HttpResponseAttributes`, exponiendo metadatos específicos de la respuesta HTTP recibida (como `statusCode: 200` y `reasonPhrase`). El payload del flujo principal se sobrescribe con el resultado del servicio externo ("GOODBYE" / "Goodbye").
* Confirmada la recepción final en Postman evaluando las cabeceras de respuesta devueltas por el Listener (Content-Type, Content-Length y Date).

**Walkthrough 1-4:** practicada la configuración manual de datos de solicitud y respuesta (Request and Response Data Customization) en los componentes HTTP Listener e HTTP Request para alterar el flujo del evento Mule.
* Inspeccionada la pestaña "Responses" en la configuración del HTTP Listener de `helloFlow` (`GET /hello`), comprendiendo que por defecto el "Body" de la respuesta devuelta al cliente está mapeado mediante la expresión DataWeave `#[payload]` (devolviendo el payload activo al final del flujo).
* Configurada una cabecera de respuesta (Response Header) personalizada en el HTTP Listener: clave `"name"` con el valor literal `"Max"`, y establecido el "Reason phrase" a `"Success"` (manteniendo el "Status code" en blanco para que devuelva el código 200 por defecto). Validado el cambio en las cabeceras recibidas por el cliente en Advanced REST Client.
* Inspeccionada la pestaña "Body" en la sección Request del componente HTTP Request (`GET /goodbye`), confirmando que por defecto Mule inyecta el `payload` activo del flujo como cuerpo de la petición saliente hacia el servicio externo.
* Añadido un Query Parameter dinámico dentro de la pestaña "Query Parameters" del HTTP Request: clave `"fullName"` con el valor literal `"Max Mule"`. 
* Utilizado el Mule Debugger para verificar la propagación del parámetro a través de la barrera de transporte: al hacer "Step into" hacia `goodbyeFlow`, se comprobó que los atributos del nuevo mensaje entrante (`HttpRequestAttributes`) ahora contienen con éxito el query parameter `fullName=Max Mule` inyectado por el Request del flujo padre.
* Detenido el proyecto y regresado a la perspectiva "Mule Design" para limpiar el entorno de pruebas.
  
</details>

## Módulo 2 — Estructuración de aplicaciones Mule

- [ ] **2-1** Crear y referenciar subflows y private flows
- [ ] **2-2** Disparar flows usando el conector VM
- [ ] **2-3** Encapsular elementos globales en un archivo de configuración separado
- [ ] **2-4** Usar property placeholders en conectores
- [ ] **2-5** Crear un proyecto Mule bien organizado
- [ ] **2-6** Gestionar metadata de un proyecto

## Módulo 3 — Consumo de web services

- [ ] **3-1** Consumir un servicio RESTful que tiene API y conector propio
- [ ] **3-2** Consumir un servicio RESTful
- [ ] **3-3** Consumir un servicio SOAP
- [ ] **3-4** Transformar datos de múltiples servicios a un formato canónico

## Módulo 4 — Control de flujo de eventos

- [ ] **4-1** Multicast de un evento
- [ ] **4-2** Enrutar eventos según condiciones
- [ ] **4-3** Validar eventos

## Módulo 5 — Manejo de errores

- [ ] **5-1** Explorar el manejo de errores por defecto
- [ ] **5-2** Manejar errores a nivel de aplicación
- [ ] **5-3** Manejar tipos de error específicos
- [ ] **5-4** Manejar errores a nivel de flow
- [ ] **5-5** Manejar errores a nivel de processor
- [ ] **5-6** Mapear un error a un tipo de error personalizado
- [ ] **5-7** Integrar con los manejadores de error de APIkit
- [ ] **5-8** Configurar una estrategia de reconexión para un conector

## Módulo 6 — Transformaciones DataWeave

- [ ] **6-1** Crear transformaciones con el componente Transform Message
- [ ] **6-2** Transformar estructuras básicas de JSON, Java y XML
- [ ] **6-3** Transformar estructuras de datos complejas con arrays
- [ ] **6-4** Transformar hacia/desde XML con elementos repetidos
- [ ] **6-5** Definir y usar variables y funciones
- [ ] **6-6** Convertir y dar formato a strings, números y fechas
- [ ] **6-7** Definir y usar tipos de datos personalizados
- [ ] **6-8** Usar funciones de DataWeave

## Módulo 7 — Disparadores de flows

- [ ] **7-1** Disparar un flow al añadirse un nuevo archivo a un directorio
- [ ] **7-2** Disparar un flow al añadirse un registro en base de datos (watermarking)
- [ ] **7-3** Programar un flow y usar watermarking manual
- [ ] **7-4** Publicar y escuchar mensajes JMS

## Módulo 8 — Procesamiento de registros

- [ ] **8-1** Procesar elementos de una colección con el scope For Each
- [ ] **8-2** Procesar registros con el scope Batch Job (opcional)
- [ ] **8-3** Usar filtrado y agregación en un batch step (opcional)

---

## Stack

MuleSoft Anypoint Platform · RAML · DataWeave 2.0 · APIkit · Postman

## Certificación objetivo

MuleSoft Certified Developer - Level 1 (Mule 4) — examen previsto para agosto 2026.
