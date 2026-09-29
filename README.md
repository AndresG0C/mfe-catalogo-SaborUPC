# mfe-catalogo

Micro frontend del catálogo de platos. Publica el evento `carrito:agregar`
cuando el usuario agrega un plato.

## Puerto
`http://localhost:8082`

Ejemplo:
    python -m http.server 8082

## Tecnología
Vue 3 desde CDN (unpkg, `vue.esm-browser.prod.js`). Sin build.

## Contrato de montaje / desmontaje
- `window.renderCatalogo(idContenedor)`
- `window.unmountCatalogo(idContenedor)`

## Eventos
- **Publica:** `carrito:agregar`
  - `detail`: `{ id, nombre, precio }`
  - Versión: 1
- **Escucha:** ninguno.

## Funcionalidad
- Lista de platos con categoría y precio.
- Búsqueda por nombre.
- Filtro por categoría.
- Botón "Agregar al carrito" → dispara `carrito:agregar`.

## Modo independiente
Abrir `http://localhost:8082/` para ver el catálogo sin el contenedor.
Los eventos que publica se muestran en una lista debajo.

## Prueba de contrato
Abrir `http://localhost:8082/contrato.html`.
Verifica automáticamente funciones expuestas, montaje y estructura del evento.

## Versión visible
`VERSION = '2.0.0'` (mostrado en pantalla bajo el título).

## Despliegue en Netlify

Este micro frontend se despliega en Netlify como sitio estático. Además de
los archivos del propio MFE (`index.html`, `catalogo.js`, `contrato.html`),
el repositorio incluye un archivo **`netlify.toml`** en la raíz.

### ¿Qué contiene `netlify.toml`?

```toml
[[headers]]
  for = "/*"
  [headers.values]
    Access-Control-Allow-Origin = "*"
```

### ¿Por qué es necesario?

Los micro frontends de SaborUPC viven en **orígenes distintos**. El
`mfe-catalogo` está en Netlify (`https://<algo>.netlify.app`), mientras que
el contenedor single-spa corre en otro dominio (por ejemplo,
`http://localhost:9000` en desarrollo o su propia URL en producción).
Cuando el contenedor carga `catalogo.js` desde el origen de Netlify, el
navegador aplica la **política de CORS**: solo permite la carga si el servidor
de Netlify responde con la cabecera `Access-Control-Allow-Origin`.

Por defecto, Netlify **no añade esa cabecera** a los archivos estáticos. Sin
ella, el navegador bloquea la carga del script, y el contenedor muestra el
error:

```
Access to script at 'https://<algo>.netlify.app/catalogo.js' from origin
'http://localhost:9000' has been blocked by CORS policy: No
'Access-Control-Allow-Origin' header is present on the requested resource.
```

El archivo `netlify.toml` **resuelve ese bloqueo** indicándole a Netlify que
incluya la cabecera `Access-Control-Allow-Origin: *` en todas las respuestas.
Con eso, cualquier origen (el contenedor en local, en Render, en Vercel,
etc.) puede importar `catalogo.js` sin que el navegador lo rechace.

### ¿Por qué `*` y no un dominio específico?

Durante el desarrollo y la demostración del taller, el contenedor puede
estar corriendo en distintos orígenes (`localhost:9000`, una URL de Render,
una URL de Vercel, etc.). Usar `*` evita tener que actualizar el archivo
cada vez que cambia el origen del contenedor.

En un entorno de producción real conviene restringirlo al dominio del
contenedor. Por ejemplo:

```toml
[[headers]]
  for = "/*"
  [headers.values]
    Access-Control-Allow-Origin = "https://mi-contenedor.example.com"
```

### ¿Cómo se aplica?

1. El archivo `netlify.toml` vive en la raíz del repositorio, junto a
   `index.html`, `catalogo.js` y `contrato.html`.
2. Al hacer `git push`, Netlify detecta el cambio y redespliega el sitio.
3. A partir de ese momento, todas las respuestas de Netlify incluyen la
   cabecera CORS.
4. El contenedor puede cargar `catalogo.js` desde Netlify sin bloqueo.

### Verificación

Abrir en el navegador:

```
https://<algo>.netlify.app/catalogo.js
```

En DevTools → pestaña **Network**, inspeccionar la respuesta y comprobar
que incluye:

```
Access-Control-Allow-Origin: *
```

Si la cabecera aparece, el contenedor podrá cargar el MFE desde cualquier
origen.

### Alternativa: archivo `_headers`

Netlify también soporta un archivo llamado `_headers` con la misma
funcionalidad:

```
/*
  Access-Control-Allow-Origin: *
```

Ambos enfoques son equivalentes. En este proyecto se eligió `netlify.toml`
porque es un formato más declarativo y queda agrupado con el resto de la
configuración de despliegue.