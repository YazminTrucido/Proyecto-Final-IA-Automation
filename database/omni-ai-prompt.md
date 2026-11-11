# Instrucción para Airtable Omni AI

Este es el texto que se le dio a Omni AI para generar la base. Los nombres de tablas y campos tienen que quedar exactamente así, porque el workflow de n8n los usa tal cual.

```
Creá una base llamada "Estudio Lumen · Gestor de Leads" con 4 tablas vinculadas.
Usá EXACTAMENTE estos nombres de tabla y de campo (con mayúsculas, tildes y espacios iguales):

1) Tabla "Servicios": Nombre (texto, campo principal), Descripción (texto largo),
   Precio desde USD (número entero), Plazo dias (número entero), Activo (casilla).
   Cargá 5 servicios de ejemplo de una agencia de diseño web, todos activos.

2) Tabla "Leads": Nombre (texto, principal), Email (email), Teléfono (teléfono),
   Empresa (texto), Mensaje (texto largo), Presupuesto USD (número entero),
   Fuente (selección única: Google, Instagram, Recomendación, Otro, Sin dato),
   Estado (selección única: Pendiente, Datos incompletos, Procesado por IA,
   Esperando aprobación, Aprobado por Humano, Enviado, Rechazado por humano,
   Sin respuesta (vencido), Descartado (spam), Error (revisión manual)),
   Categoria IA (selección única: VIP, Calificado, Frío, Spam), Puntaje IA (número),
   Resumen IA (texto largo), Servicio sugerido (vínculo a Servicios),
   Asunto propuesto (texto), Borrador respuesta (texto largo), Tokens usados (número),
   Gmail Thread ID (texto), Gmail Message ID (texto), Thread ID aprobacion (texto),
   Fecha ingreso, Fecha decision y Fecha respuesta (fecha con hora),
   Ultimo error (texto largo).

3) Tabla "Interacciones": Interaccion (texto, principal), Lead (vínculo a Leads),
   Tipo (selección única: Borrador IA, Email enviado), Canal (selección única:
   Interno, Gmail), Contenido (texto largo), Gmail Thread ID (texto),
   Tokens usados (número), Fecha (fecha con hora).

4) Tabla "Errores": Error (texto, principal), Lead (vínculo a Leads),
   Tipo (selección única: Datos incompletos, Lead duplicado, Fallo API IA,
   Respuesta IA inválida, Fallo Gmail), Nodo (texto), Mensaje (texto largo),
   Fecha (fecha con hora), ID ejecucion (texto), Resuelto (casilla).

No crees campos extra ni registros de ejemplo en Leads, Interacciones ni Errores.
```
