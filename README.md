# Xclusive — Sophisticated Elegance

Club editorial privado para creadoras independientes: membresías, clips, salones en vivo,
bóveda protegida, mensajería, estudio de creadora, administración y centro legal. Estética de
club privado; arquitectura de bóveda.

- **Sitio en producción:** https://sophisticated-elegance.vercel.app
- **Repositorio:** https://github.com/AnubisVillasenor1/xclusive-by-TAMV-Online
- **Ecosistema:** TAMV ONLINE NETWORK · RDM Digital Hub · Nodo Cero, Real del Monte, Hidalgo.

> **Estado del documento:** `draft`. Este README describe lo que existe en el repositorio y
> separa explícitamente lo implementado de lo especificado. No certifica el producto, no
> sustituye una auditoría independiente y no declara producción lista. Las reglas operativas
> del repositorio viven en [`AGENTS.md`](AGENTS.md).

---

## 1. Qué es

Xclusive es una aplicación web de una sola página con SSR (TanStack Start) para publicar
contenido editorial de creadoras, vender acceso a piezas y membresías, y operar una bóveda
privada de medios. Está construida para que la autoridad —identidad, rol, precio y acceso— se
resuelva en el servidor, nunca en el navegador.

La plataforma está **navegable y funcional en lo editorial y lo operativo**, y permanece
**bloqueada en lo económico y lo regulatorio**: no hay cobro real, ni KYC, ni retiro efectivo
mientras no existan proveedores aprobados y configuración de producción.

---

## 2. Estado real

**Preparación estimada: 70/100.** Es una estimación documentada con matriz ponderada, no una
métrica automática ni una auditoría externa. El detalle, la matriz antes/después y el historial
de iteraciones están en [`docs/AUDITORIA-REMEDIACION.md`](docs/AUDITORIA-REMEDIACION.md).

Etiquetas: `IMPLEMENTED` · `PARTIAL` · `BLOCKED` · `SPEC` · `ABSENT` · `MANAGED` · `REFERENCE`
(definidas en `AGENTS.md`, §0.2).

| Dominio                            | Estado             | Dónde vive                                                                   | Límite verificable                                                  |
| ---------------------------------- | ------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Autenticación y roles              | `IMPLEMENTED`      | `src/lib/auth/*`, `src/integrations/supabase/*`                              | Rol resuelto en servidor con allowlists; sin allowlist no hay admin |
| Contenido de creadoras             | `IMPLEMENTED`      | `db/0002_contenido_creadoras.sql`, `src/lib/content/*`, `/clips`, `/estudio` | Sujeto a moderación y propiedad; no todo el dominio está persistido |
| Precio y reparto                   | `IMPLEMENTED`      | `src/lib/payments/orders.server.ts`, `splits.ts`                             | El precio lo fija el servidor; sin PSP no hay cobro                 |
| Comercio y webhook                 | `BLOCKED`          | `src/routes/api/payments/webhook.ts`                                         | Responde 503 sin `PAYMENT_PROVIDER` y secretos: fallo cerrado       |
| ARCO, denuncias y retiros          | `IMPLEMENTED`      | `xclusive_data_requests`, `xclusive_reports`, `xclusive_payout_requests`     | Los retiros se registran **retenidos**, no pagados                  |
| Bóveda privada                     | `PARTIAL`          | bucket `vault`, `getVaultDownloadUrl`                                        | URL firmada de 60 s; faltan revocación y bitácora de descargas      |
| MFA y recuperación                 | `PARTIAL`          | `/recuperar`, `/restablecer`, `SecurityTab`                                  | No se exige AAL2 en cada consulta ni en RLS                         |
| Membresías                         | `PARTIAL`          | `src/lib/auth/auth.functions.ts`, `db/0002_*.sql`                            | RLS de `memberships` pendiente según la auditoría                   |
| KYC, AML y edad certificada        | `SPEC` / `BLOCKED` | Solo referencias en `docs/PRODUCCION.md`                                     | Sin proveedor ni tablas; la aceptación es declarativa               |
| Doble divisa (consumo/liquidación) | `SPEC`             | Sin tablas ni UI                                                             | No debe presentarse como función disponible                         |
| Agencias y subagencias             | `SPEC`             | Sin tablas, rutas ni cálculo                                                 | No implementar sin esquema aprobado                                 |
| Academia y laboratorio             | `ABSENT`           | Sin rutas ni tablas                                                          | No inventar rutas ni progreso de estudiantes                        |
| `tamv-pack` / manifiesto           | `ABSENT`           | Pertenece al repo maestro TAMV Creator Intelligence                          | Aquí solo existe `skills/` local                                    |
| Nodos CROWN/ISA/SOPHIA/ORION/ARGUS | `REFERENCE`        | Arquitectura de gobierno del ecosistema                                      | Sin runtime; ningún agente puede invocar su autoridad               |
| DSP, stems y audio                 | `SPEC`             | Solo documentación de intención                                              | No simular procesamiento ni resultados                              |
| Observabilidad                     | `PARTIAL`          | `src/lib/error-capture.ts`, `lovable-error-reporting.ts`                     | Sin OpenTelemetry, métricas de negocio ni alertas                   |
| Esquema gestionado                 | `MANAGED`          | `supabase/migrations/`, `drizzle/`                                           | Se cambia desde Lovable, no a mano                                  |

### 2.1 Lo que sigue bloqueado (P0)

1. **Cobro real:** requiere `PAYMENT_PROVIDER=stripe`, `STRIPE_SECRET_KEY` y
   `STRIPE_WEBHOOK_SECRET`. Sin ellos el checkout no crea pedido.
2. **KYC/AML y edad certificada:** requieren proveedor externo aprobado; hoy la edad es
   declarativa.
3. **Retiro efectivo:** requiere PSP, KYC, cuenta bancaria y maduración; hoy solo se registra la
   intención como retenida.
4. **Mensajería persistente:** el hilo vive en el navegador; falta migrarlo al servidor.
5. **Correo transaccional:** no hay notificaciones fuera del navegador.

Nada de lo anterior se declara terminado: son dependencias externas y de cumplimiento, no
tareas de interfaz.

---

## 3. Arquitectura

| Capa       | Tecnología                                                         |
| ---------- | ------------------------------------------------------------------ |
| Runtime    | Node.js 22 (CI), React 19, TypeScript 5.8 estricto                 |
| Framework  | TanStack Start + TanStack Router (SSR, server functions)           |
| Build      | Vite 8 + `@lovable.dev/vite-tanstack-config` + Nitro               |
| UI         | Tailwind CSS v4, primitivas shadcn/ui, sistema visual `lux/`       |
| Datos      | PostgreSQL + Supabase (Auth, Storage, RLS), Drizzle                |
| Pagos      | Stripe (server-side, opcional y fail-closed)                       |
| Pruebas    | Vitest 4 + Testing Library (jsdom)                                 |
| Despliegue | Vercel (canónico); Cloudflare Workers documentado como alternativa |

`tsconfig.json` activa `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes` y
`noUncheckedSideEffectImports`.

```text
src/
├─ components/
│  ├─ lux/        primitivas, tarjetas, checkout y desglose de comisiones
│  ├─ platform/   shell del estudio, compositor de contenido, bóveda, cuenta y admin
│  ├─ site/       cabecera, pie, layouts, age gate, cookies, tour, video editorial
│  └─ ui/         base shadcn/ui
├─ lib/
│  ├─ auth/       sesión, roles de servidor, allowlists, rate limit, auditoría, legal
│  ├─ content/    server functions de contenido, ARCO, denuncias y retiros
│  ├─ payments/   precio de servidor, órdenes, partida doble, Stripe, entitlements
│  ├─ store.tsx   estado de cliente (nunca autoridad de identidad en LIVE)
│  └─ pricing.ts  tarifas centralizadas (base + 15 %)
├─ routes/        páginas TanStack y `api/payments/webhook.ts`
├─ test/          pruebas de integración y de UI
└─ styles.css     tokens y utilidades (aura, btn-lux, lift, mystic-rule)
```

### 3.1 Flujo de una compra

```text
1. El cliente pide `startCheckout` con el id de la pieza.
2. El servidor resuelve precio, comisión y reparto; nunca confía en el importe del cliente.
3. Se crea la orden con clave de idempotencia y estado `pending`.
4. Si hay PSP configurado, se abre la sesión del proveedor; si no, el checkout falla cerrado.
5. El webhook firmado marca la orden como pagada y escribe el libro mayor.
6. El entitlement se concede solo después del webhook verificado, no al volver del checkout.
```

La UI muestra el desglose (precio, comisión de plataforma, coste estimado del PSP y neto de la
creadora) antes de confirmar.

---

## 4. Rutas

| Ruta                            | Acceso               | Qué hace                                         |
| ------------------------------- | -------------------- | ------------------------------------------------ |
| `/`                             | Público              | Portada editorial, hero, lookbook y video        |
| `/explorar`, `/clips`           | Público              | Catálogo y piezas publicadas                     |
| `/creadoras`, `/creadora/$slug` | Público              | Directorio y perfil de creadora                  |
| `/salones`, `/salon/$id`        | Público / sesión     | Salones en vivo, aforo y nivel requerido         |
| `/membresias`                   | Público              | Tarifas y niveles; sin cobro real                |
| `/boveda`                       | Sesión + entitlement | Reproducción y descarga firmada                  |
| `/mensajes`                     | Sesión               | Hilo de mensajería (estado en navegador)         |
| `/cuenta`                       | Sesión               | Perfil, membresías, compras, ARCO, seguridad     |
| `/estudio`                      | Rol creadora         | Publicación, métricas y solicitud de retiro      |
| `/admin`                        | Rol admin            | Operación y métricas reales                      |
| `/entrar`, `/registro`          | Público              | Supabase Auth; DEMO solo en builds DEMO          |
| `/recuperar`, `/restablecer`    | Público              | Recuperación de acceso                           |
| `/legal`, `/legal/$doc`         | Público              | Nueve documentos legales versionados             |
| `/ayuda`, `/ecosistema`         | Público              | Ayuda y mapa de capacidades                      |
| `/api/payments/webhook`         | Servidor (firmado)   | Verificación del PSP y cumplimiento de la orden  |
| `/health.json`                  | Público              | Salud del despliegue (`Cache-Control: no-store`) |

---

## 5. Funcionalidad implementada

### Experiencia pública

- Portada editorial con hero cinematográfico, retrato, lookbook de cinco piezas y video con
  póster y respeto a `prefers-reduced-motion`.
- Directorio de diez creadoras con feed, niveles, salones y verificación declarada.
- Perfil de creadora con portada, biografía, membresías y contenido protegido.
- Clips y bundles con precio, comisión y desglose antes de confirmar.
- Salones con aforo, horario, nivel requerido y estado real de la función.
- Centro legal con nueve documentos versionados (`2026.11-demo`), aceptación con hash y derechos
  ARCO.
- Age gate, consentimiento de cookies y consentimiento legal versionado.

### Guía interactiva

`src/components/site/PlatformTour.tsx`: recorrido de doce pasos que navega entre rutas, con
tarjeta contextual, progreso `NN / 12`, teclado (`←`, `→`, `Esc`), foco atrapado y persistencia en
`localStorage` (`xclusive.tour.v1`).

### Cuenta, estudio y administración

- ARCO: `submitDataRequest` / `listMyDataRequests` sobre `xclusive_data_requests`; el
  seguimiento sobrevive al cambio de dispositivo.
- Denuncias: `submitReport` sobre `xclusive_reports`, con sesión obligatoria.
- Retiros: `requestPayout` / `listMyPayouts` sobre `xclusive_payout_requests`, mostrados como
  **retenidos** con su motivo real.
- Compras: `getVaultDownloadUrl` verifica el entitlement y firma una URL de 60 s.
- Estudio: `ServerContentComposer` publica portada y archivo privado; saldo calculado con
  `getCreatorEarnings` a partir de pedidos pagados.
- Administración: `getOperationsOverview` con métricas reales; sin allowlist no hay admin.
- Eliminación de cuenta (ARCO) con confirmación explícita y clave de servicio en servidor; si
  falta, se declara no disponible.

### Activos locales

- `public/editorial/` — cinco piezas WebP optimizadas.
- `public/covers/` — portadas procedurales generadas con `scripts/generate-covers.mjs`.
- `public/brand/` — emblema, wordmark y lockup con `srcset`.
- `scripts/fetch-editorial.mjs` documenta la descarga y optimización de las editoriales.

---

## 6. Dinero: reglas y bloqueos

- **Precio de servidor.** `resolveServerPrice` decide el importe; el cliente solo envía
  identificadores. Un importe del navegador no se usa para cobrar.
- **Partida doble.** `xclusive_ledger_entries` registra débito y crédito; toda orden debe cuadrar
  al peso. Se calcula en pesos enteros para no perder centavos por redondeo.
- **Comisión del PSP.** Se estima 3,5 % + IVA 16 % y la asume la plataforma; el margen puede ser
  negativo y se muestra como tal.
- **Idempotencia.** `xclusive_orders.idempotency_key` y `xclusive_webhook_events` evitan doble
  cobro y doble abono. La cobertura de todos los flujos no está demostrada.
- **Fallo cerrado.** Sin PSP aprobado no se crea pedido ni se concede acceso. No hay cobros,
  propinas ni saldos reales en el estado actual.
- **Retiros.** Se registran como intención retenida; no se ejecuta dispersión.
- **Prohibición de simulación.** Nunca se presentan pagos, conversiones, métricas de agencias ni
  resultados de audio como reales. Los datos de práctica se etiquetan `demo`, `mock` o `sample`.

Lo que **no** existe todavía: doble divisa (monedas/diamantes), billetera, agencias y
subagencias, KYC/AML operativo, payout efectivo y conciliación contable externa.

---

## 7. Seguridad y privacidad

- **Identidad en servidor.** La sesión de Supabase se verifica con bearer token en cada server
  function; `requireSupabaseAuth` rechaza lo no autenticado.
- **Roles por allowlist.** `ADMIN_EMAILS` / `CREATOR_EMAILS` y `app_metadata`; `user_metadata`
  nunca concede rol. Sin allowlist no hay administradores.
- **DEMO aislado en compilación.** El modo se fija con `--mode demo` o `VITE_DEMO_MODE=true`; un
  build LIVE elimina cuentas y credenciales de demostración del bundle.
- **RLS.** Activado en las tablas `xclusive_*`; el navegador solo lee sus propias filas y no
  existen políticas de escritura para `authenticated`. Las escrituras usan la clave de servicio
  desde server functions y el webhook firmado.
- **Cabeceras.** CSP, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`,
  `Permissions-Policy`, HSTS y `Cross-Origin-Opener-Policy`, definidas en `public/_headers` y en
  `vite.config.ts` (`routeRules`).
- **Abuso.** Limitador por clave en server functions, más los límites de Supabase Auth.
- **Auditoría.** Eventos `xclusive.audit` con actor, acción, resultado y detalle mínimo; recibos
  legales firmados por el servidor en `xclusive_legal_acceptances`.
- **Secretos.** Nunca en el repositorio. `.env*` está ignorado; solo se versionan plantillas con
  valores ficticios (`.env.example`, `.env.demo`). `SUPABASE_SERVICE_ROLE_KEY` y
  `LOVABLE_API_KEY` son exclusivas de servidor.

---

## 8. Puesta en marcha

Requisitos: **Node.js 22+** y **npm** (el CI usa `npm ci` con `package-lock.json`; el repo
conserva `bun.lock` por compatibilidad con el perfil maestro, pero no es el gestor de este CI).

```sh
git clone https://github.com/AnubisVillasenor1/xclusive-by-TAMV-Online.git
cd xclusive-by-TAMV-Online
npm ci
npm run dev
```

Vite imprime la URL local al arrancar.

### Variables de entorno

Públicas (se incorporan al bundle, no son secretas):

| Variable                        | Para qué                       |
| ------------------------------- | ------------------------------ |
| `VITE_SUPABASE_URL`             | Endpoint del proyecto Supabase |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Clave publicable del cliente   |
| `VITE_SUPABASE_PROJECT_ID`      | Identificador del proyecto     |
| `VITE_SITE_URL`                 | URL canónica del sitio         |
| `VITE_DEMO_MODE`                | `false` en producción          |

Servidor (runtime, nunca con prefijo `VITE_`):

| Variable                    | Para qué                                                |
| --------------------------- | ------------------------------------------------------- |
| `SUPABASE_URL`              | Endpoint usado por las server functions                 |
| `SUPABASE_PUBLISHABLE_KEY`  | Clave publicable para validar sesiones                  |
| `SUPABASE_PROJECT_ID`       | Identificador del proyecto                              |
| `ADMIN_EMAILS`              | Allowlist de administración (separada por comas)        |
| `CREATOR_EMAILS`            | Allowlist de creadoras                                  |
| `CREATOR_SLUGS`             | Vínculo `correo=slug` para el perfil editorial          |
| `SUPABASE_SERVICE_ROLE_KEY` | Solo para borrado de cuenta y operación administrativa  |
| `PAYMENT_PROVIDER`          | `stripe` para activar el cobro (hoy bloqueado)          |
| `STRIPE_SECRET_KEY`         | Crear sesiones de pago reales                           |
| `STRIPE_WEBHOOK_SECRET`     | Firmar y verificar el webhook                           |
| `POSTGRES_URL`              | Solo scripts de mantenimiento (`apply-db`, `preflight`) |

### Cuentas de demostración

Solo existen en builds DEMO (`npm run dev:demo`, `npm run build:demo`):

| Rol      | Correo                    | Contraseña |
| -------- | ------------------------- | ---------- |
| Fan      | `fan@xclusive.demo`       | `demo1234` |
| Creadora | `valentina@xclusive.demo` | `demo1234` |
| Admin    | `admin@xclusive.demo`     | `demo1234` |

En LIVE no se renderizan y el texto de la contraseña no está en el bundle.

---

## 9. Scripts

| Comando                  | Qué hace                                         |
| ------------------------ | ------------------------------------------------ |
| `npm ci`                 | Instalación reproducible (la que usa el CI)      |
| `npm run dev`            | Desarrollo LIVE con SSR y HMR                    |
| `npm run dev:demo`       | Desarrollo DEMO con cuentas locales              |
| `npm run build`          | Build de producción (Vercel/Nitro)               |
| `npm run build:demo`     | Build DEMO con cuentas locales                   |
| `npm run build:dev`      | Build en modo development                        |
| `npm run preview`        | Sirve el build localmente                        |
| `npm run typecheck`      | `tsc --noEmit`                                   |
| `npm test`               | Vitest (una pasada)                              |
| `npm run test:watch`     | Vitest en modo watch                             |
| `npm run lint`           | ESLint                                           |
| `npm run format`         | Prettier sobre el proyecto                       |
| `npx prettier --check .` | Verificación de formato (no hay script dedicado) |

Mantenimiento de base de datos (solo con `POSTGRES_URL` y revisión previa):

```sh
node scripts/preflight.mjs
node scripts/apply-db.mjs db/0001_comercio_auditoria.sql --dry-run
node scripts/apply-db.mjs db/0001_comercio_auditoria.sql
```

`apply-db.mjs` es idempotente, pero opera sobre una base compartida: exige revisión humana antes
de ejecutarse sin `--dry-run`.

---

## 10. Pruebas y calidad

- **9 archivos, 56 casos** en `src/test/` y junto al dominio (`src/lib/**/*.test.ts`):
  autenticación, recibos legales, reparto y partida doble, creadoras, precios, comercio, DEMO,
  tour y rutas.
- Se prueban contratos e invariantes, no implementación: nada de leer el código fuente con
  `fs.readFileSync` ni aserciones sobre conteos que cambian solos.
- El CI de GitHub Actions (`.github/workflows/ci.yml`) ejecuta, en Node 22:
  `npm ci` → `npx tsc --noEmit` → `npm test` → `npm run lint` → `npm run build`.
- La última verificación local documentada (11 de noviembre de 2026) reportó 0 errores de tipos,
  56 pruebas, 0 errores de lint, 16 avisos preexistentes y build correcto. **Esta actualización es
  documental y no re-ejecutó esos comandos en local**; el push dispara el CI que sí los ejecuta.

Estándares obligatorios para pruebas nuevas están en `AGENTS.md`, §11.

---

## 11. Base de datos

Tablas propias con prefijo `xclusive_`, creadas en `db/` para no colisionar con el proyecto de
Supabase compartido con otro producto:

| Tabla                        | Para qué                                                 |
| ---------------------------- | -------------------------------------------------------- |
| `xclusive_orders`            | Pedido con importe, comisión e idempotencia              |
| `xclusive_ledger_entries`    | Libro mayor de partida doble; cuadra al peso por pedido  |
| `xclusive_entitlements`      | Acceso concedido solo tras pago verificado               |
| `xclusive_legal_acceptances` | Recibo legal firmado por el servidor                     |
| `xclusive_audit_log`         | Auditoría estructurada con retención propia              |
| `xclusive_webhook_events`    | Idempotencia de webhooks por identificador del proveedor |
| `xclusive_content`           | Piezas de creadoras, portada y archivo privado           |
| `xclusive_data_requests`     | Solicitudes ARCO con estado y plazo                      |
| `xclusive_reports`           | Denuncias y moderación                                   |
| `xclusive_payout_requests`   | Intenciones de retiro, retenidas con motivo              |

Buckets: `xclusive-public` (portadas y material público) y `vault` (privado, solo URL firmada).
El esquema gestionado por Lovable (`supabase/migrations/`, `drizzle/`) no se edita a mano.

---

## 12. Despliegue

Vercel es el destino canónico. Guía detallada: [`DEPLOY.md`](DEPLOY.md).

```sh
npx vercel pull --yes --environment=production
npx vercel build --prod
npx vercel deploy --prebuilt --prod
```

- Proyecto: `prj_xtn4fMajuBdI5NKfnTDbNmuCFaNn` (team `team_QbYxBq0wdGuWTaCKV6U5TL2N`).
- Alias de producción: https://sophisticated-elegance.vercel.app
- `vercel.json` fija `NITRO_PRESET=vercel`; las cabeceras se escriben en
  `.vercel/output/config.json` desde `vite.config.ts` y se replican en `public/_headers`.
- Cloudflare Workers (`wrangler.jsonc`) queda como alternativa documentada, no validada en este
  repositorio.

### Verificación posterior

1. `GET /health.json` responde 200 con `Cache-Control: no-store`.
2. `/entrar` no muestra cuentas DEMO y el bundle no contiene `demo1234`.
3. Un correo fuera de `ADMIN_EMAILS` no obtiene rol de administrador.
4. `/cuenta`, `/estudio`, `/admin` y `/boveda` responden 200 o redirigen sin errores 500.
5. Los logs muestran eventos `xclusive.audit` en acciones sensibles.

---

## 13. Gobernanza del repositorio

- [`AGENTS.md`](AGENTS.md) es la constitución operativa: jerarquía de prioridades, estados,
  reglas de Lovable, secretos, economía, pruebas y prohibición de afirmaciones no verificables.
- [`skills/`](skills/README.md) contiene las skills locales. `tamv-browser-verification` define
  el protocolo de QA con `agent-browser`, incluido su fallback cuando la CLI no está instalada.
- **Lovable gestiona** base de datos, autenticación, pagos, integraciones, hosting y auditoría de
  plataforma: son zonas de cambio controlado. No se editan `supabase/migrations/` ni `drizzle/`,
  ni archivos generados, ni se reescribe el historial publicado (`main` alimenta el editor).
- **Origen de cada commit:** `engine-generated`, `human-approved` o `human-authored`.
- **Estados de entregable:** `draft`, `auto_generated`, `human_reviewed`, `verified`, `wired`.
  Ningún agente puede autopromoverse a `verified` o `wired`.
- **Conflicto de fuentes:** si el repo maestro tiene `docs/spec/`, prevalece; en este checkout no
  existe, y las fuentes canónicas son `docs/AUDITORIA-REMEDIACION.md`, `docs/PRODUCCION.md`,
  `DEPLOY.md`, `db/*.sql`, `public/_headers`, `vite.config.ts` y `src/routes/README.md`.

Las reglas de oro: no se fuerza el push, no se exponen secretos y no se simulan pagos,
certificaciones ni resultados.

---

## 14. Límites explícitos

- **Contenido adulto deshabilitado** hasta completar verificación de edad certificada,
  consentimiento documentado por creadora y aprobación del procesador de pagos.
- **Sin dinero real:** saldos, comisiones y propinas son demostración visible; no se solicita ni
  se procesa ningún medio de pago.
- **Textos legales en borrador** (`2026.11-demo`): requieren revisión legal, fiscal y de
  protección de datos antes de operar.
- **Consentimiento e imagen:** no se publican fotografías de personas reales sin licencia y
  consentimiento verificables; las editoriales locales son material de referencia estética.
- **Sin certificaciones:** este repositorio no declara ISO 27001, SOC 2, PCI DSS ni cumplimiento
  normativo auditado. No hay auditoría independiente ni prueba de penetración.
- **Sin módulos fantasma:** Academia, agencias, doble divisa, DSP/stems, memoria de cinco scopes
  y runtime de CROWN/ISA/SOPHIA/ORION/ARGUS no existen en este checkout.

---

## 15. Documentación

- [`AGENTS.md`](AGENTS.md) — constitución operativa y matriz de estado.
- [`docs/AUDITORIA-REMEDIACION.md`](docs/AUDITORIA-REMEDIACION.md) — remediación P0, matriz y
  bloqueos.
- [`docs/PRODUCCION.md`](docs/PRODUCCION.md) — blueprint de producción: datos, RBAC/RLS, pagos,
  KYC y puertas de lanzamiento.
- [`DEPLOY.md`](DEPLOY.md) — despliegue, variables y verificación posterior.
- [`db/README.md`](db/README.md) — tablas propias, RLS y aplicación idempotente del esquema.
- [`skills/README.md`](skills/README.md) — registro y reglas de skills.

---

## 16. Autoría

Proyecto impulsado con colaboración de tecnología latinoamericana.

- **TAMV ONLINE NETWORK** · RDM Digital Hub · Nodo Cero, Real del Monte, Hidalgo, México.
- **Autoría técnica y arquitectura de sistemas:** Edwin Oswaldo Castillo Trejo (Anubis
  Villaseñor).
- **ORCID:** `0009-0008-5050-1539` · **ISNI:** `0000000531699229`. Son identificadores de
  registro declarados por la autoría: atribución, no certificación ni aval.

Orgullosamente Realmontenses.
