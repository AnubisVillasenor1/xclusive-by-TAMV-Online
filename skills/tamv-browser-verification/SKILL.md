---
name: tamv-browser-verification
description: Verificación web y QA de Xclusive / TAMV ONLINE con agent-browser. Úsala para smoke tests de rutas, revisión visual y responsive, accesibilidad, reproducción de bugs con evidencia observable y comprobación de que un build DEMO/LIVE no filtra credenciales ni datos. Triggers include "abre la web", "prueba esta ruta", "haz un smoke test", "revisa la app", "QA de navegador", "captura evidencia", "dogfooding", "revisa el checkout en modo test".
allowed-tools: Bash(agent-browser:*)
---

# TAMV ONLINE — Verificación Web Soberana

**Skill propietaria de TAMV ONLINE NETWORK para Xclusive — Sophisticated Elegance.**

Esta skill convierte `agent-browser` en un protocolo de verificación con marca TAMV: observar,
medir, evidenciar y detenerse. No es una herramienta para “hacer clic hasta que algo funcione”.

> **Principio rector:** la IA propone y verifica; el humano decide.
> Una captura demuestra lo observado, no aprueba producción.

Identidad de método de esta skill:

```text
[TINA].[YUN].[QA].[VERIFICACION_WEB].[V1].[MEDIO].[EVIDENCIA_OBSERVADA]
```

---

## 1. Preflight obligatorio

Antes de tocar una página:

1. Confirma que la CLI está instalada y qué skills sirve la versión presente:

   ```sh
   agent-browser skills list
   ```

2. Carga el manual real de la versión instalada. **No improvises banderas ni comandos de
   memoria**: el contenido lo sirve la propia CLI.

   ```sh
   agent-browser skills get core
   ```

3. Carga una referencia especializada solo si aplica:

   | Necesidad                             | Referencia                                              |
   | ------------------------------------- | ------------------------------------------------------- |
   | QA exploratorio, bug hunt, dogfooding | `agent-browser skills get dogfood`                      |
   | App de escritorio Electron            | `agent-browser skills get electron`                     |
   | Slack                                 | `agent-browser skills get slack`                        |
   | HAR → cliente de API                  | `agent-browser skills get derive-client`                |
   | Herramientas WebMCP de una página     | `agent-browser skills get webmcp-gen`                   |
   | Navegador en Vercel Sandbox           | `agent-browser skills get vercel-sandbox`               |
   | Deploy protegido de Vercel            | `agent-browser skills get protected-vercel-deployments` |
   | Navegador cloud de AWS Bedrock        | `agent-browser skills get agentcore`                    |

4. Declara el entorno y el alcance antes de abrir nada:
   - Entorno: `local` / `preview` / `producción (solo lectura)`.
   - URL base.
   - Commit o build que se está verificando.
   - Rutas y flujos incluidos.
   - Qué **no** se va a tocar.

Si `agent-browser` no está instalado, no lo simules. Informa y usa el fallback de la sección 8.

---

## 2. Contrato de seguridad TAMV

### 2.1 Permitido

| Entorno                         | Alcance                                                                                     |
| ------------------------------- | ------------------------------------------------------------------------------------------- |
| Local LIVE (`npm run dev`)      | Rutas públicas, autenticación con cuentas de prueba, estados de error.                      |
| Local DEMO (`npm run dev:demo`) | Flujos completos con datos ficticios, incluido comercio simulado.                           |
| Preview / staging               | Solo con autorización; datos de prueba; checkout en modo test.                              |
| Producción                      | Solo lectura: rutas públicas, `GET /health.json`, cabeceras, ausencia de credenciales demo. |

### 2.2 Prohibido

- Credenciales reales, tokens de producción, datos personales o contenido privado de terceros.
- Pagos, retiros, reembolsos, suscripciones o cualquier movimiento de dinero real.
- Eludir CAPTCHA, MFA, rate limits, allowlists o controles de acceso.
- Registro masivo, spam, scraping de personas, extracción de directorios o recolección de datos.
- Pruebas de carga o estrés contra producción.
- Publicar capturas con correos, tokens, cookies, URLs firmadas, IDs de sesión o contenido privado.
- Modificar datos de producción para “ver si funciona”.

### 2.3 Parada inmediata

Detente y pide instrucciones humanas si observas:

- CAPTCHA o MFA no previsto.
- Un aviso legal o consentimiento nuevo que no estabas autorizado a aceptar.
- Error 5xx reproducible o pérdida de datos.
- Exposición de secretos, datos personales, contenido privado o una URL firmada.
- Cualquier acción que pueda mover dinero, borrar una cuenta o alterar derechos de acceso.
- Un flujo que se comporta distinto entre DEMO y LIVE sin explicación documentada.

---

## 3. Flujo canónico de verificación

La sintaxis exacta la confirma `skills get core`. El orden lógico es siempre el mismo:

1. **Abrir** la URL objetivo en una sesión identificable.
2. **Observar** con snapshot de accesibilidad antes de actuar; no adivines selectores.
3. **Actuar** sobre referencias de elemento (`@eN`) o entradas explícitas.
4. **Esperar** la condición real (texto, URL, estado, respuesta de red), no un tiempo arbitrario.
5. **Afirmar** el resultado con una comprobación observable.
6. **Evidenciar** con captura redactada y nota de lo observado.
7. **Cerrar** la sesión y liberar el estado.

Reglas de interacción:

- Un snapshot por acción relevante; los refs caducan.
- No reutilices `@eN` de una página anterior.
- Prefiere nombres accesibles, roles y etiquetas visibles.
- Si un clic no produce cambio observable, no repitas a ciegas: inspecciona consola, red y estado.
- No “arregles” la página desde el navegador para que la prueba pase.

---

## 4. Matriz de verificación Xclusive

### 4.1 Smoke de rutas

Comprueba al menos, según el alcance autorizado:

| Ruta                                                   | Qué observar                                                           |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
| `/`                                                    | Render, hero, navegación, video con póster y `prefers-reduced-motion`. |
| `/explorar`                                            | Catálogo, imágenes optimizadas, estados vacío/error.                   |
| `/creadoras`, `/creadora/$slug`                        | Perfil, portada, acceso según sesión.                                  |
| `/clips`, `/salones`, `/salon/$id`                     | Listado, acceso, reproducción y permisos.                              |
| `/membresias`                                          | Precios y estados; nada de cobros reales.                              |
| `/boveda`                                              | Solo con entitlement; URL firmada efímera.                             |
| `/entrar`, `/registro`, `/recuperar`, `/restablecer`   | Formularios, errores y ausencia de credenciales demo en LIVE.          |
| `/cuenta`, `/estudio`, `/admin`                        | Redirección controlada o acceso denegado sin rol.                      |
| `/legal`, `/legal/$doc`                                | Documento, versión y aceptación.                                       |
| `/health.json`                                         | `200` y `Cache-Control: no-store`.                                     |
| `/robots.txt`, `/sitemap.xml`, `/manifest.webmanifest` | Respuesta válida y contenido coherente.                                |

### 4.2 Frontera DEMO / LIVE

En un build LIVE verifica:

- `/entrar` no muestra cuentas demo.
- El bundle no contiene `demo1234` ni credenciales de demostración.
- `VITE_DEMO_MODE` no se puede activar desde el navegador.
- Un correo fuera de `ADMIN_EMAILS` no obtiene rol de administrador.
- Sin sesión, las rutas protegidas redirigen o deniegan; no muestran datos.

En un build DEMO verifica:

- Las cuentas demo existen y el comercio está etiquetado como simulado.
- Ningún dato demo se presenta como real.
- Reiniciar datos desde **Mi cuenta → Datos** funciona y no rompe la sesión.

### 4.3 Comercio en modo test

Solo con PSP en sandbox o con el modo DEMO:

- El precio mostrado coincide con el resuelto por el servidor.
- Sin configuración de pago, el checkout falla cerrado y **no** crea pedido.
- El acceso no se concede al volver del checkout; solo tras webhook verificado.
- Un pago fallido no crea entitlement.
- Reintentar no duplica orden ni cobro (idempotencia).
- Los estados `pending`, `paid`, `failed`, `refunded` se muestran sin ambigüedad.

Nunca introduzcas números de tarjeta reales. Nunca confirmes un pago real.

### 4.4 Responsive y accesibilidad

- Anchos objetivo: 360, 390, 768, 1024, 1440.
- Sin scroll horizontal, solapamientos ni texto cortado.
- Foco visible y orden de tabulación lógico.
- Formularios con etiqueta, ayuda y error asociados.
- Contraste suficiente en texto, botones y estados deshabilitados.
- Imágenes con texto alternativo útil; decorativas vacías.
- Diálogos con foco atrapado y cierre por teclado.
- Video respeta `prefers-reduced-motion` y no arranca con audio.

### 4.5 Cabeceras y superficie expuesta

Solo lectura, sin explotar nada:

- `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`,
  `Referrer-Policy`, `Permissions-Policy`, HSTS y `Cross-Origin-Opener-Policy`.
- Ausencia de `Access-Control-Allow-Origin: *` en endpoints que no lo requieran.
- Cookies de sesión con `HttpOnly`, `Secure` y `SameSite` cuando aplique.
- Errores que no revelan rutas internas, stack traces, claves ni SQL.
- Respuestas de auth que no confirman si un correo existe.

---

## 5. Protocolo de evidencia

### 5.1 Artefactos

Guarda capturas, videos y notas en una ruta ignorada por Git:

```text
.output/qa/<YYYY-MM-DD>-<slug>/
```

Nunca escribas evidencia en `public/`, `src/assets/` ni en una ruta servida por la aplicación.

### 5.2 Redacción

Antes de adjuntar cualquier artefacto:

- Recorta o difumina correos, nombres, tokens, cookies, URLs firmadas e IDs de sesión.
- No copies el valor de una cookie ni de un header de autorización en el reporte.
- Describe el dato sensible por su tipo (“correo de prueba”), no por su valor.
- Si no puedes redactar sin destruir la evidencia, no la adjuntes: describe el hallazgo.

### 5.3 Sello de evidencia

Cada evidencia debe declarar:

```text
[TAMV][QA][<ENTORNO>][<RUTA>][<FECHA>]
observado: <resultado concreto>
esperado: <criterio de aceptación>
fuente: <captura | snapshot | header | respuesta>
limitación: <lo que no se pudo comprobar>
estado: draft | auto_generated | human_reviewed
```

---

## 6. Clasificación de hallazgos

| Nivel  | Criterio                                                                | Acción                                        |
| ------ | ----------------------------------------------------------------------- | --------------------------------------------- |
| **P0** | Dinero, acceso, secretos, datos personales o pérdida de datos.          | Detener, no continuar, escalar de inmediato.  |
| **P1** | Flujo crítico roto, error 5xx reproducible, regresión de autenticación. | Reportar con evidencia y bloquear la entrega. |
| **P2** | Fallo funcional no crítico, accesibilidad, responsive o copy.           | Reportar y priorizar.                         |
| **P3** | Mejora visual, consistencia o rendimiento menor.                        | Registrar como propuesta.                     |

No mezcles niveles para “suavizar” un P0. No cierres un hallazgo sin explicar qué lo resolvió.

---

## 7. Formato de entrega

```md
## Alcance

- Entorno:
- URL base:
- Build / commit:
- Rutas y flujos:

## Resultado observado

- <ruta>: <resultado> — evidencia: <archivo o nota>

## Hallazgos

### P0/P1/P2/P3 — <título>

- Esperado:
- Observado:
- Pasos mínimos:
- Evidencia redactada:
- Impacto:
- Recomendación:

## No verificado

- <flujo> — motivo: <sin credenciales | sin PSP sandbox | fuera de alcance | bloqueado por CAPTCHA/MFA>

## Estado

- skill: tamv-browser-verification
- state: draft | auto_generated | human_reviewed

## Siguiente decisión humana

- <aprobación, corrección o escalado requerido>
```

---

## 8. Fallback sin `agent-browser`

Si la CLI no está disponible o no puede instalarse:

1. Declara: “`agent-browser` no está instalado; no se ejecutó verificación automatizada”.
2. Ofrece el plan manual equivalente: rutas, anchos, estados y cabeceras a comprobar.
3. Pide capturas humanas si el usuario puede aportarlas.
4. No inventes resultados, no cites capturas que no viste y no marques nada como `verified`.

---

## 9. Regla final

**Observar no es aprobar.** Esta skill produce evidencia; la autoridad para declarar producción
sigue siendo humana. Si hay conflicto entre “terminar la prueba” y “proteger a una persona, un
dato o un pago”, se detiene la prueba.
