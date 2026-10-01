# Arquitectura

Este documento explica las decisiones detrás del proyecto, no solo qué hace
el código. Está pensado para que, al volver en unos meses, se entienda el
"por qué" sin tener que reconstruir el razonamiento desde cero.

## Objetivo

Un blog con experiencia de edición visual (tipo Notion), hosteado 100% en
GitHub Pages, sin backend propio que mantener.

## Piezas y por qué

### Astro como generador estático

Genera HTML estático en build time — no hay servidor que correr, calza
directo con GitHub Pages (que solo sirve archivos estáticos). Los posts
viven como Markdown en `src/content/posts/` usando Astro Content
Collections, con un schema (`content.config.ts`) que valida `title`, `date`,
`description` y `draft` en build time.

### Decap CMS para edición visual

Decap CMS es un cliente que corre en el navegador (`public/admin/`), lee/
escribe archivos del repo vía la API de GitHub, y no requiere servidor
propio — todo el estado vive en los commits de git. Eso es lo que permite
"editar como en Notion" sin salirse del modelo 100%-estático-en-Pages.

### El problema: OAuth necesita un secreto, GitHub Pages no puede guardar secretos

Para que Decap pueda commitear en tu nombre, necesita hacer el flujo OAuth
de GitHub, que requiere intercambiar un `client_secret` por un token — y
ese intercambio **tiene que pasar en un servidor**, nunca en el navegador
(un secreto en código estático servido por Pages sería público).

GitHub Pages no ejecuta servidor, así que esa única pieza (el intercambio
OAuth) vive aparte, en **`decap-oauth-provider`**, un repo distinto
desplegado en Vercel (dos funciones serverless: `/api/auth` y
`/api/callback`, con rewrites a `/auth` y `/callback` porque así es como
Decap las llama). El blog en sí sigue siendo 100% estático en GitHub Pages;
Vercel solo resuelve el login.

Esto es la única parte "no estática" de todo el sistema, y existe
exclusivamente por esta restricción de seguridad, no por necesidad del blog.

### Por qué dos repos separados y no uno solo

El proveedor OAuth no tiene nada que ver con el contenido del blog ni con
su ciclo de deploy — se despliega distinto (Vercel vs. GitHub Pages), se
actualiza distinto (casi nunca), y mezclarlo en el mismo repo acoplaría dos
cosas sin relación. Si en el futuro se cambia de CMS o de hosting del
blog, el proveedor OAuth no se ve afectado.

### Token de GitHub sin expiración

La GitHub OAuth App se configuró con "Expire user access tokens"
**desactivado**. El proveedor OAuth no implementa el flujo de
`refresh_token` (hubiera sido una pieza más de código y estado que
mantener para un beneficio marginal en un proyecto personal de un solo
usuario). La contrapartida: si GitHub revoca el token (ej. cambio de
password, revocación manual), hay que volver a autorizar la app — un caso
raro y de bajo costo.

### Deploy

`.github/workflows/deploy.yml` hace build en cada push a `main` y publica a
GitHub Pages vía `actions/deploy-pages`. No hay paso manual: tanto los
posts escritos a mano como los publicados desde `/admin` llegan a `main`
como commits normales y disparan el mismo pipeline.

## Diagrama

```
┌─────────────┐   commit vía API   ┌──────────────────┐
│  /admin      │ ─────────────────▶│  GitHub repo      │
│ (Decap CMS)  │                    │  constelaciones   │
└─────┬────────┘                    └────────┬──────────┘
      │ login OAuth                          │ push a main
      ▼                                      ▼
┌─────────────────────┐            ┌──────────────────────┐
│ decap-oauth-provider │            │ GitHub Actions        │
│ (Vercel, serverless) │            │ build Astro + deploy  │
└──────────────────────┘            └──────────┬────────────┘
                                                ▼
                                     ┌──────────────────────┐
                                     │   GitHub Pages        │
                                     │ (sitio público)        │
                                     └──────────────────────┘
```
