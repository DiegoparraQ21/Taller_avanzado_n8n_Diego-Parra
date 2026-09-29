Taller Avanzado de Automatización Corporativa con n8n

Este repositorio contiene la documentación, arquitectura conceptual y desglose de requerimientos para el Taller Avanzado de Automatización Corporativa con n8n.

El objetivo principal de esta etapa es resolver problemas empresariales complejos abordando gestión de estado, manejo de binarios y archivos, validaciones estructuradas, deduplicación, colecciones de datos, procesos de aprobación humana y trazabilidad en tiempo real, utilizando nodos nativos de n8n e integraciones de ecosistema Google Workspace.

📁 https://drive.google.com/drive/folders/1Ud6Ci8WDB_GtOKwBqeUlqLHnQ6uPj33V?usp=sharing

Puedes acceder a los videos explicativos con el paso a paso de la solución y pruebas de cada ejercicio en el siguiente enlace:

👉 [ Insertar Enlace de Google Drive Aquí ]

🚀 Resumen de Retos

📩 RETO 11 - Radicación Automática de Facturas

Contexto: El área de cuentas por pagar recibe facturas de proveedores por correo electrónico. La gestión manual de descarga, extracción de datos, almacenamiento y registro genera cuellos de botella e ineficiencias.

Objetivo: Automatizar la recepción de correos con facturas adjuntas en formato PDF (con texto seleccionable), extraer los campos clave mediante IA, evitar registros/archivos duplicados y mantener la trazabilidad.

Requerimientos Clave:

Deduplicación: Evaluar la combinación única NIT + número de factura. Evitar deduplicar por simple nombre de archivo o ID de mensaje.

Estructura de Datos Extraídos: Proveedor, NIT, número de factura, fecha de emisión, fecha de vencimiento, subtotal, IVA, total y moneda.

Preservación de Binario: Conservar el archivo PDF original para subirlo a Google Drive mientras en una rama paralela se procesa la extracción del texto.

Almacenamiento: Guardar el PDF en Google Drive y registrar la información consolidada junto al enlace del archivo en Google Sheets.

Componentes / Nodos Clave: Gmail Trigger, Extract From File, HTTP Request (o nodo de IA/LLM), Remove Duplicates, Google Drive y Google Sheets.

🛡️ RETO 12 - Gobernanza de Entregables en Google Drive

Contexto: Distintos equipos suben archivos a una carpeta compartida sin estandarización de nombres ni control de extensiones, dificultando la clasificación documental corporativa.

Objetivo: Evaluar automáticamente cada archivo nuevo que ingrese a una carpeta de entrada utilizando únicamente sus metadatos (sin descargar el contenido ni usar IA), moverlo a la carpeta correspondiente (ACEPTADOS o RECHAZADOS) y registrar la trazabilidad del proceso.

Convención Corporativa de Nombre:

Patrón: CLIENTE_PROYECTO_TIPODOCUMENTO_FECHA_VERSION.ext (Ej: ACME_NEBULA_INFORME_20260924_V03.pdf)

Tipos permitidos: INFORME, CONTRATO, ACTA, ENTREGABLE.

Extensiones permitidas: .pdf, .docx.

Formato de Fecha y Versión: Fecha en AAAAMMDD y versión V seguida de dos dígitos (ej. V01).

Requerimientos Clave:

Auditoría: Guardar en Google Sheets la fecha de procesamiento, ID, nombre de archivo, tipo MIME, estado (ACEPTADO/RECHAZADO), motivo del rechazo y carpeta de destino.

Manejo eficiente: Trabajar directamente sobre metadatos mediante expresiones regulares (Regex) sin descargar el binario.

Componentes / Nodos Clave: Google Drive Trigger, Expresiones / Regex, If / Switch, Google Drive (Move Operation) y Google Sheets.

⏰ RETO 13 - Monitor Corporativo de Vencimientos

Contexto: Gestión manual de fechas de expiración de pólizas, contratos, licencias y certificaciones, lo cual incrementa el riesgo de no renovación oportuna.

Objetivo: Ejecutar una revisión diaria sobre una base de datos en Google Sheets (Vencimientos), clasificar los registros según su proximidad de vencimiento y enviar un único reporte consolidado por correo, actualizando posteriormente las filas para evitar notificaciones duplicadas en el mismo día.

Criterios de Clasificación:

VENCIDO

PRÓXIMO EN 7 DÍAS

PRÓXIMO EN 15 DÍAS

PRÓXIMO EN 30 DÍAS

SIN ALERTA

Requerimientos Clave:

Consolidación: Agrupar múltiples alertas en un solo cuerpo de correo HTML/Texto (evitar un correo por cada fila).

Control de Notificación: Actualizar la columna de fecha de última alerta en Google Sheets. Si una fila ya fue alertada en la fecha actual, no debe volverse a notificar.

Componentes / Nodos Clave: Schedule Trigger, Google Sheets (Read/Update), Edit Fields / Manejo de Fechas, Aggregate, Split Out, If y Gmail.

🛍️ RETOS 14 Y 15 - Sistema de Solicitudes y Orden de Compra

Los retos 14 y 15 conforman un sistema desacoplado compuesto por dos workflows independientes.

📥 RETO 14 - Recepción y Cálculo de Solicitudes de Compra

Objetivo: Recibir un payload JSON vía POST Webhook desde una intranet/aplicación corporativa con un arreglo dinámico de ítems (items).

Requerimientos Clave:

Validar presencia de datos requeridos.

Generar un ID único de solicitud.

Iterar y procesar cualquier cantidad de elementos en items calculando subtotales por ítem y el valor total general.

Registrar la solicitud en Google Sheets con estado inicial PENDIENTE_APROBACION.

Desencadenar de forma asíncrona el Reto 15 enviando la solicitud procesada.

Responder inmediatamente a la aplicación cliente (Respond to Webhook) con el ID, total y estado, sin bloquearse esperando la aprobación humana.

Componentes / Nodos Clave: Webhook, Split Out, Edit Fields, Aggregate, Respond to Webhook y Execute Sub-workflow.

⚖️ RETO 15 - Aprobación y Emisión de Orden de Compra

Objetivo: Recibir las solicitudes generadas por el Reto 14, evaluar el nivel de aprobación en función del monto total, requerir la decisión humana (vía mail interactivo) y generar el documento de Orden de Compra si es aprobada.

Niveles de Aprobación:

Aprobación Automática.

Aprobación por Jefe Directo.

Aprobación Financiera.

Requerimientos Clave:

Ejecución Asíncrona: Inicia mediante When Executed by Another Workflow.

Aprobación Humana: Utiliza el mecanismo de pausa y espera (Send and Wait for Approval).

Flujo Aprobado: Genera la Orden de Compra en Google Docs con la plantilla institucional (ID, fecha, solicitante, centro de costos, justificación, detalle de ítems, total y aprobador), guarda el documento en Google Drive, notifica al área de compras y al solicitante, y actualiza el estado a EMITIDA en Google Sheets.

Flujo Rechazado: Actualiza el estado a RECHAZADA en Google Sheets e informa al solicitante.

Componentes / Nodos Clave: When Executed by Another Workflow, Switch, Gmail (Send and Wait for Approval), Google Sheets, Google Docs y Google Drive.

🛠️ Tecnologías Utilizadas

Orquestador: n8n (Community / Cloud Edition)

Servicios Integrados:

Google Drive API

Google Sheets API

Google Docs API

Gmail API / Webhook Integration

Formatos de Intercambio: JSON, Data Binaria, Markdown / HTML.