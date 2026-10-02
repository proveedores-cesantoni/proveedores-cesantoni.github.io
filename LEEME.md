# Plataforma de proveedores CESANTONI | Somos Logística (versión web)

Página independiente, sin servidor propio y sin Google Apps Script:

| Parte | Servicio (plan gratuito, sin tarjeta) |
|---|---|
| Página (portal `index.html` y panel `admin.html`) | GitHub Pages |
| Accesos (proveedores con folio + clave, personal con correo + contraseña) | Firebase Authentication |
| Registros, documentos (guardados por partes), bitácora y usuarios | Cloud Firestore (plan Spark) |
| Correos de avisos | Gmail por Apps Script (~100 destinatarios al día) y EmailJS de respaldo (200 al mes) |
| Recuperación de clave / contraseña | Correo de Firebase Authentication |

Portal: `https://proveedores-cesantoni.github.io/`
Panel: `https://proveedores-cesantoni.github.io/admin.html`

Las colecciones usan el prefijo `pv_`, así que no tocan los datos de la versión de Apps Script.

## Puesta en marcha (una sola vez)

1. **Authentication**: consola de Firebase > Authentication > Comenzar > Correo electrónico/contraseña > Habilitar.
   En *Configuración > Dominios autorizados* agrega `proveedores-cesantoni.github.io`.
   En *Plantillas*, cambia el idioma a Español.
2. **Firestore**: Firestore Database > Reglas > pega `firestore.rules` > Publicar.
3. **App web**: Configuración del proyecto > Tus apps > `</>` (Web) > registra la app y copia
   `apiKey`, `authDomain`, `projectId` y `appId` en `js/config.js`.
4. **EmailJS** (https://www.emailjs.com, plan Free):
   - Email Services > Add > Gmail > conecta el correo que enviará los avisos → anota el *Service ID*.
   - Email Templates > Create: *To Email* `{{to_email}}`, *Subject* `{{subject}}`, *From name*
     `CESANTONI Somos Logística`, y en el contenido (botón «Edit Content» > «Code») escribe `{{{html}}}` → anota el *Template ID*.
   - Account > General: *Public Key*. Account > Security: agrega el dominio `proveedores-cesantoni.github.io` en *Allowed origins*.
   - Copia los tres valores en `js/config.js`.
5. **Correos por Gmail** (opcional, recomendado): en https://script.google.com crea un proyecto nuevo
   «Correo proveedores CESANTONI», pega `apps-script-correo/Correo.gs` y `appsscript.json` (repositorio privado),
   Implementar > Nueva implementación > Aplicación web (Ejecutar como: yo; Acceso: cualquier usuario) y pega la URL `/exec`
   en `js/config.js` > `correo.url`. Si falla o se acaba el límite diario, la página usa EmailJS.
6. **Primer administrador**: abre el panel; la primera vez pide crear la cuenta del administrador principal.
7. **Destinatarios**: en *Panel > Destinatarios* agrega los correos del equipo que recibirán las alertas internas.

## Correos automáticos (dos diseños de tarjeta)

| Momento | Al proveedor (tarjeta clara con folio) | Al equipo (tarjeta de alerta oscura) |
|---|---|---|
| Genera su folio | Folio, clave y botón para continuar | «Nuevo registro» (si el destinatario lo activó) |
| Envía su registro completo | «¡Registro recibido!» | «Registro completo» con datos y lista de documentos |
| Se le pide corrección | Motivo de cada documento | — |
| Reenvía correcciones | «Recibimos tus correcciones» | «Correcciones recibidas» |
| Resolución final | Aprobado / no aprobado | — |

## Panel

Tablero con indicadores y tarjetas de alerta (registros por revisar, correcciones sin respuesta,
documentos por vencer, registros sin avance y correos no entregados), lista con filtros y CSV,
expediente con visor de documentos (zoom, giro, PDF, versiones y descarga), revisión por documento,
resolución, historial, correos con vista previa, destinatarios, usuarios, sistema y respaldo.

## Límites gratuitos

- Firestore: 1 GiB en total, 50 000 lecturas y 20 000 escrituras al día. Los documentos (máx. 5 MB c/u)
  se guardan dentro de Firestore; 1 GiB alcanza para unos 200–300 expedientes completos.
  El uso se ve en *Panel > Sistema*.
- Gmail (Apps Script): unos 100 destinatarios al día con una cuenta de Gmail. EmailJS: 200 correos al mes de respaldo.
  Los que fallen quedan en *Panel > Correos* para reintentar o borrar.
- Sin lectura automática (OCR): las fechas y vigencias se revisan manualmente.

## Resumen diario, fotos y aviso de privacidad

- **Resumen diario** (Apps Script `apps-script-correo`, función `instalarResumen`): a las 8:00 envía a los destinatarios con
  «Resumen diario» un correo con los pendientes del tablero. Si no hay pendientes no envía nada.
- **Fotos optimizadas**: JPG, PNG y WEBP se reducen en el navegador (lado mayor 2200 px) antes de subirlas. Los PDF no cambian.
- **Aviso de privacidad**: `aviso-privacidad.html`; razón social, domicilio y correo ARCO en `js/config.js` > `aviso`.

## Respaldo

*Panel > Sistema > Descargar respaldo (JSON)* descarga registros, documentos (datos), usuarios,
destinatarios y correos. Hazlo periódicamente.

## Pruebas

`tests/web/e2e.mjs` (en el repositorio privado) recorre portal y panel con el emulador oficial de Firebase
y las mismas reglas de seguridad: 33 de 33 verificaciones correctas.

---

# Solicitudes de movimientos (carpeta `solicitudes/`)

Plataforma aparte para que las áreas (Customer Service, Mercadotecnia, Ventas, Calidad…) pidan a Logística devoluciones,
recolecciones, envíos, traslados, movimientos de Mercadotecnia, renta de unidad y maniobras, y les den seguimiento.

| Liga | Para quién |
|---|---|
| `https://proveedores-cesantoni.github.io/solicitudes/` | Áreas que piden movimientos (formulario y «Mis solicitudes») |
| `https://proveedores-cesantoni.github.io/solicitudes/admin.html` | Logística (base y seguimiento) |

- **Base separada:** colecciones `sm_` en el mismo proyecto de Firebase. No lee ni escribe nada de `pv_` (proveedores) y tiene su propio personal.
- **Quien solicita:** al mandar su primera solicitud recibe por correo su folio (SOL-0001…) y una clave; con su correo y esa clave entra a «Mis solicitudes».
- **Devoluciones:** fotos obligatorias del material y checklist de 7 puntos; si no cumple todo, llega marcada «No cumple».
- **Paquetería:** un renglón por tipo de paquete (cantidad, largo, ancho, alto y peso c/u) con totales y peso volumétrico.
- **Logística:** folio CSTEXT (el mismo de «Fletes 2026»), categorización, responsable, transportista, unidad, guía, fecha programada,
  mensajes a quien solicita y notas internas. Solo seguimiento: no se capturan importes.
- **Personal:** el primer acceso al panel crea al administrador principal. A quien ya tiene cuenta (por ejemplo, del panel de proveedores)
  se le invita y entra con su misma contraseña.
- **Correos:** usan el mismo Gmail (Apps Script) y EmailJS de respaldo. Mientras `Correo.gs` solo reconozca al personal de proveedores,
  los avisos de solicitudes salen por EmailJS (200 al mes en total).

## Puesta en marcha (una sola vez)

1. **Reglas:** Firebase > Firestore Database > Reglas > pega **todo** `firestore.rules` (incluye el bloque «Solicitudes de movimientos») > Publicar.
2. Abre el panel de solicitudes y crea el administrador principal; en *Configuración* captura los correos de aviso y da de alta al personal.
3. Comparte la liga del formulario con las áreas.

Pruebas: `tests/solicitudes-web/e2e.mjs` (repositorio privado) con el emulador de Firebase y estas mismas reglas: 45 de 45 correctas.
La prueba de proveedores (`tests/web/e2e.mjs`) sigue en 33 de 33 con las reglas combinadas.
