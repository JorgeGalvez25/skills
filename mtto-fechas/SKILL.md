---
name: mtto-fechas
description: Revisa el canal de Slack #mtto y compara los standups diarios del equipo de mantenimiento con una captura del tablero de tickets (columnas PROCESO / ENTREGADO con TT#####, responsable y fecha) para decir a qué tickets hay que moverles la fecha, cuáles revisar/cerrar y cuáles van bien. Usar cuando digan "/mtto-fechas", adjunten la imagen del tablero, o pregunten "a cuál le muevo la fecha".
---

# Revisión de fechas del tablero vs. #mtto (/mtto-fechas)

El usuario (Jorge Gálvez, jefe de mantenimiento) invoca esta skill y adjunta una **captura del tablero**.
No hace falta que explique nada más: haz todo el flujo y entrega el resultado.

## 1. Leer la imagen

Extrae de la captura, por cada tarjeta: **columna** (PROCESO / ENTREGADO), **TT#####**, cliente/descripción corta,
**responsable** y **fecha** (formato dd/mm/aaaa). Ignora el número del círculo junto al reloj (no interpretarlo).
Si no hay imagen adjunta, pídela y detente.

## 2. Leer el canal #mtto

- Canal: **#mtto**, privado, ID `C02CGD1D9NX` (el usuario a veces escribe "mmto"; es el mismo).
- Herramienta: `slack_read_channel` con `channel_id=C02CGD1D9NX`, `limit=100` y `oldest` = timestamp Unix
  de **hace ~12 días** (calcúlalo con cuidado a partir de la fecha de hoy; un error de año trae meses de
  mensajes y la salida se desborda). Si aun así se desborda, acorta la ventana en lugar de procesar el archivo.
- Si el conector de Slack no está autorizado, díselo al usuario (autorizar en conectores de claude.ai o `/mcp`) y detente.
- Si algún mensaje relevante tiene hilo, léelo con `slack_read_thread`.

### Cómo son los mensajes

Cada desarrollador publica su standup diario (~17:00) mencionando a Jorge, sin títulos de día:
directamente las secciones **standby** y **seguimiento**, con líneas `TT#####: estatus` (cómo queda cada caso al cierre del día).
Los standups anteriores al 08/10/2026 traen bloques **Ayer** (o "Viernes") y **Hoy**; de esos usa solo el bloque "Hoy".
Un caso que deja de aparecer en el standup sin estatus de cierre (RSP, LDP, CEC, cancelado) se trata como "dejó de aparecer".
Equipo: Martín Reyes, Miguel Armenta, Susana Gámez, Saulo López.

Vocabulario de estatus:
- **RSP** (Revisión SCRUM Pendiente): el cliente dio por bueno el cambio o la corrección del fallo; falta integrarlo a MAIN y las pruebas previas a liberar la versión. Vuelve a seguimiento cuando entra a integración y pruebas con el tester.
- **LDP**: el tester terminó sus pruebas y mandó el correo de liberación de pruebas (cambios, mejoras y fallos). "En espera de LDP" sigue abierto.
- **CEC**: se manda a cerrar.
- **Cancelado** (`TT#####: Cancelado` o la línea `CANCELADO TT##### | ...`): ya no se le da seguimiento. "Probablemente se cancele" no cuenta.
- **Entregado / en espera de respuesta**: esperando retroalimentación de soporte/cliente.
- **Pausa / pausado / standby / en espera de logs**: detenido.
- "x2", "x3", "x4"… = veces que se ha repetido la misma acción (indica estancamiento).

Usa el **último standup de cada responsable** como estatus vigente (los casos del más reciente),
y los anteriores para ver la tendencia.

## 3. Cruzar y clasificar

Hoy = fecha actual. Para cada ticket del tablero:

1. **Mover fecha**: la fecha ya pasó o es hoy, y el canal muestra que sigue abierto
   (standby, pausa, revisando, en espera de logs/respuesta, entregado sin respuesta, o sin evidencia de entrega).
2. **Revisar estatus**: la fecha ya pasó y el ticket **dejó de aparecer** en los standups, o la última
   mención fue "mandar a RSP"/"LDP"/"entrega"/"cancelado". Probablemente necesita cambiar de columna o cerrarse, no una fecha nueva
   (si se canceló, la acción sugerida es cerrarlo como cancelado en el tablero).
3. **Sin cambio**: va en tiempo, o el canal indica cierre (CEC) / siguiente paso claro. Si la fecha es futura pero
   el ticket lleva mucho tiempo pausado ("x3", "pausado"), menciónalo como riesgo.
4. **Sin información**: no aparece en el canal; dilo, no inventes.

Nunca supongas un estatus que el canal no diga; cita brevemente lo que dice el standup y de qué día.

## 4. Respuesta

En español, breve, con este formato:

- Una línea con el rango de mensajes leídos (ej. "Leí #mtto del 21/09 al 01/10").
- **Mover fecha**: tabla `Ticket (responsable) | Fecha actual | Qué dicen en el canal`, del más atrasado al menos.
- **Revisar estatus**: viñetas con la acción sugerida (pasar a Entregado, confirmar RSP o LDP, cerrar).
- **Sin cambio**: viñetas cortas.
- **Sin información** (si aplica).

No envíes mensajes a Slack ni modifiques nada: solo es lectura y recomendación. Si el usuario después pide
mandar recordatorios a los responsables, redacta el mensaje en mrkdwn y pide confirmación antes de enviarlo.
