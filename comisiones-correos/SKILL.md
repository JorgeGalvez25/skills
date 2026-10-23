---
name: comisiones-correos
description: Prepara los correos de autorización de comisiones de los cambios internos de Mantenimiento. Se ejecuta desde la carpeta del mes (C:\Users\jorge\Dropbox\MTTO\Reportes Mensuales\Comisiones\<año>\YYYYMM-Mes), toma los HorasCambioTT#####.xlsx de CambiosInternos\<colaborador> tal como estén guardados (incluso si el usuario los editó) y deja un borrador en Outlook con ese Excel adjunto. Usar cuando digan "/comisiones-correos", "genera los correos de autorización", "prepara los borradores de comisiones", "ya edité el archivo, haz el correo" o "vuelve a adjuntar el Excel".
argument-hint: "[TT##### ...]"
---

# Correos de autorización de comisiones (/comisiones-correos)

Preparas el correo con el que Jorge pide la autorización del pago de comisiones de cada cambio interno.
Los `HorasCambioTT#####.xlsx` los genera `/comisiones-internos`. El usuario puede editarlos después; esta skill **siempre adjunta el archivo como esté guardado en ese momento**, nunca lo regenera ni lo modifica.

- **Carpeta de la skill** (`$SK`): `C:\Users\jorge\.claude\skills\comisiones-correos`.
- **Carpeta del mes** (`$MES`): la carpeta de trabajo actual, con nombre `YYYYMM-Mes`.
- **Títulos de los cambios:** `C:\Users\jorge\Dropbox\MTTO\ReportesMTTO\CambiosProceso.xlsx` (columna A = ID, columna B = Descripción).
- El script corre en PowerShell 5.1, usa Excel y Outlook por COM y está guardado en UTF-8 con BOM. Si lo editas, consérvalo con BOM; sin él, los acentos salen corruptos.

## Paso 1 — Generar

Para todos los cambios del mes:

```powershell
& 'C:\Users\jorge\.claude\skills\comisiones-correos\scripts\correos.ps1'
```

Si el usuario nombra uno o varios TT (por ejemplo, los que editó), pásalos con `-TT`:

```powershell
& 'C:\Users\jorge\.claude\skills\comisiones-correos\scripts\correos.ps1' -TT TT105878,TT102841
```

Usa la herramienta PowerShell con un timeout amplio (unos 120000 ms).

Por cada `HorasCambioTT#####.xlsx`, el script:

1. **Revisa que el Excel no esté abierto.** Si lo está, se detiene. Pídele al usuario que lo guarde y lo cierre, y vuelve a correrlo.
2. **Prepara el borrador en Outlook** (Borradores) con el Excel actual adjunto. **No lo envía.** Si ya había un borrador con el mismo asunto, lo borra (queda en Elementos eliminados) y crea uno nuevo. Así el adjunto siempre es la versión editada y no se duplican borradores.

Imprime cada TT con la fecha y hora en que se guardó el Excel adjunto.

Si falla con `no está en CambiosProceso.xlsx`, pregunta al usuario el título del caso. No lo inventes.

## Redacción fija del correo

Tomada del que Jorge envía cada mes:

- **Para:** `jorgeorduno@imagensoft.com.mx`
- **Asunto:** `Autorización comisiones cambio interno TT#####`
- **Cuerpo:**

```
Ingeniero buen día, solicito de su autorización para el pago de comisiones del cambio interno <TÍTULO>, realizado por <Colaborador>.


Jorge Armando Gálvez Martínez
Analista - Desarrollador
```

En el cuerpo:

- `<TÍTULO>` es el título de CambiosProceso sin espacios alrededor de los guiones, por ejemplo `TT105878-IMAGENSOFT-CAMBIO-CARTAPORTE UBICACION Y MERCANCIA`.
- `<Colaborador>` es el nombre de pila, es decir, el nombre de la carpeta en `CambiosInternos`.

## Paso 2 — Entregar (en el chat, breve)

1. Los TT procesados, con la fecha de guardado del Excel adjunto.
2. Cuáles borradores se reemplazaron.
3. Que los borradores están en Outlook, listos para revisar y enviar.
