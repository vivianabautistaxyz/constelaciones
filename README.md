# Constelaciones

Blog estático con [Astro](https://astro.build), editable desde un panel visual
([Decap CMS](https://decapcms.org)) o directamente en Markdown, publicado en
GitHub Pages.

- **Sitio**: https://vivianabautistaxyz.github.io/constelaciones/
- **Editor**: https://vivianabautistaxyz.github.io/constelaciones/admin/

Ver [ARCHITECTURE.md](./ARCHITECTURE.md) para el por qué de las decisiones de diseño.

## Estructura

```text
/
├── public/
│   └── admin/             # Decap CMS (panel de edición en /admin)
├── src/
│   ├── content/posts/     # Posts en Markdown (fuente de verdad)
│   ├── layouts/
│   └── pages/
├── content.config.ts      # Schema de la colección "posts"
└── .github/workflows/     # Deploy automático a GitHub Pages
```

## Escribir un post

- **Desde el panel**: entra a `/admin`, inicia sesión con GitHub, "New Post".
- **A mano**: crea un `.md` en `src/content/posts/` con este frontmatter:

  ```yaml
  ---
  title: Mi post
  date: 2026-01-01
  description: Opcional
  draft: false
  ---
  ```

Cualquiera de los dos métodos termina en un commit a `main`, que dispara el
deploy automático (ver abajo).

## Comandos locales

| Comando           | Acción                                      |
| :----------------- | :------------------------------------------ |
| `npm install`       | Instala dependencias                        |
| `npm run dev`       | Server local en `localhost:4321`            |
| `npm run build`     | Build de producción a `./dist/`             |
| `npm run preview`   | Previsualiza el build localmente            |

## Deploy

Cada push a `main` dispara `.github/workflows/deploy.yml`, que hace build y
publica a GitHub Pages. No requiere pasos manuales.
