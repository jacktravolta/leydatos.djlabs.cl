# leydatos.djlabs.cl — front independiente (Ley de Datos 21.719)

Clon del front público de **Ley de Datos 21.719** (`public/ley-datos/` de la
plataforma `asistente-judicial`), levantado como **dominio propio** que
habla con la plataforma por **REST**.

```
Navegador ──REST JSON + cookies──▶ plataforma (corazón ley_datos_21719)
   │
   └── CORS: el backend debe permitir este origen exacto
```

Nada de esto se sirve desde la plataforma: el front tiene su propio
document root, sus propias rutas (todas relativas) y su propia perilla de
despliegue.

## Qué hay aquí

| Archivo | Qué hace |
|---|---|
| `index.html` | La landing: hero, escaneo de sitio, cómo funciona, qué ofrece, acceso (login/registro), recargas, contacto y los modales de términos/privacidad. |
| `app.html` | El panel: escáner de sitios, historial de análisis, asistente, billetera, recargas, fuentes del RAG, conversaciones de visitantes, perfil y uso. |
| `contrasena.html`, `verificar.html` | Cerrar sesión con contraseña y confirmar el correo. |
| `debug-sources.html` | Diagnóstico de las fuentes del RAG (para soporte). |
| `js/` | Los módulos compartidos con la plataforma (`chat-widget`, `checkout`, `topup`, `notifications`, `uso`, …), hablando por `apiUrl()`. |
| `css/tokens.css`, `css/quill.snow.css` | El sistema de diseño (la piel la define el heart, por REST). |
| `api-config.js` | **La única perilla del despliegue**: el origen del backend para el navegador. |

## Configuración

`api-config.js` (un solo archivo, se reescribe en el deploy):

```js
window.__API_BASE__ = 'https://api.djlabs.cl';
```

El valor por defecto es **producción**: el front siempre habla con la API,
nunca consigo mismo. Dos cosas para que funcione:

1. El origen del front debe estar en la allowlist `cors.origins` del backend
   (coincidencia exacta de esquema+host+puerto).
2. La cookie de sesión debe viajar: `api.djlabs.cl` y `*.djlabs.cl` son el
   **mismo sitio** (mismo dominio registrable), así que `SameSite=Lax` +
   HTTPS alcanza — no hace falta `None`.

**En local** el origen se cambia sin editar el archivo: `?api=http://127.0.0.1:8099`
(solo se respeta en `localhost`), o el arnés E2E lo inyecta antes de cargar
la página.

El **heart** no se configura aquí: el clon trabaja con `ley_datos_21719`, el
corazón del producto. La piel y el tema los aplica el backend (REST
`/ui/theme?heart=ley_datos_21719`), igual que en la plataforma.

## Cómo probarlo

```bash
# En local, contra la plataforma real
php -S 127.0.0.1:8814 -t .

# Batería propia: no toca la plataforma real
php tests/test_leydatos_front.php
```

La batería verifica el contrato que el clon no puede romper: las páginas se
sirven, **cero llamadas a `/api/*.php`**, rutas relativas (el front funciona
en la raíz de su dominio), verbos REST correctos (GET para consultar, DELETE
para borrar, PUT para actualizar) y que el clon no arrastre el heart ni el
copy de otro producto.

## Las reglas que este front no rompe

- **Cero `/api/*.php`**: el panel, el chat, las fuentes, el TTS y la
  descarga de documentos van por REST JSON contra la plataforma.
- **El asistente se presenta con el nombre de SU heart**: el backend resuelve
  el nombre y la persona por el heart de la petición.
- **La pausa por verificación de correo se explica**: si la cuenta está en
  pausa, el chat muestra el aviso y el botón para reenviar el enlace (nada
  de errores genéricos).
- **Los errores se explican sin culpar a nadie** y sin dejar pantallas en
  blanco.
