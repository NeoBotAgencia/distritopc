# Recreación de Diseño Web

## Flujo de Trabajo

Al recibir una tarea de recreacion, lo primero es identificar el tipo de entrada:
- **imagen adjunta** -> usarla directamente como referencia
- **URL proporcionada** -> capturarla con Puppeteer para obtener la imagen de referencia visual antes de empezar

### Si la entrada es una URL

1. Capturar la URL con Puppeteer a **1440px de ancho**(desktop) y tambien a **390px**(mobile) si el diseño es reponsive. Capturar y analizar la referencia son la unica accion - pasar directamente a generar el html sin pasos intermedios:
   ```bash
   node -e"
   const puppeteer = require('puppeteer');
   
   (async () => {
     const browser = await puppeteer.launch();
     const page = await browser.newPage();
     await page.setViewport({ width: 1440, height: 900 });
     await page.goto('https://distritopc.com/', { waitUntil: 'networkidle0' });
     await page.screenshot({ path: 'reference.png', fullPage: true });
     await browser.close();
   })();
   ```
2. Usar `reference.png` como imagen de referencia para el resto del flujo.
3. Si la URL requiere scroll para ver secciones distintas, capturar cada seccion en el mismo script antes de continuar

### Si la entrada en una imagen

Usarla directamente como referencia. No hay paso previo de captura

---

## Flujo de Trabajo Principal

### 1. Analizar la referencia

Inspeccionar la imagen de referencia e identificar todo lo necesario antes de escribir una sola linea de codigo. El analisis y la generacion ocurren en el mismo paso mental - no hay vuelta atras a esta fase:
- Estructura general (header, hero, card, footer, etc.)
- Paleta de colores (valores hex exactos si son visibles)
- Tipografia(tamaños, pesos, familias)
- Espaciados y grid
- Componentes intecractivos (botones, inputs, navs)

### 2. Generar `index.html`

crear un unico archivo `index.html` con:
- Tailwind CSS (vía CDN) - `<script src="https://cdn.tailwindcss.com"></script>`
- Todo el contenido en linea - sin archivos externos salvo que se pida
- Imagenes de placeholders de `https://placehold.co/` cuando no haya fuente real
- Estructura responsive mobile-first


### 3. Capturar el resultado

```bash
node -e"
const puppeteer = require('puppeteer');
(async () => {
  const browser = await puppeteer.launch({args:['--no-sandbox']});
  const page = await browser.newPage();
  await page.setViewport({ width: 1440, height: 900 });
  await page.goto('file:///ruta/al/index.html', { waitUntil: 'networkidle0' });
  await page.screenshot({ path: 'result.png', fullPage: true });
  await browser.close();
})();
```


### 4. Comparar con la referencia

Comparar `result.png` contra la referencia. Anotar **todas** las discrepancias en una sola pasada antes de tocar el codigo -no corregir una a una:

| Aspecto | que medir|
|---|---|
| Espaciado y padding | diferencias en px|
| Tipografia | Tamaño, peso, line-height en px |
| Colores | valores hex exactos |
| Alineación y posicionamiento | diferencias en px |
| Radios de borde, sombras y efectos | diferencias en px |
| Comportamiento responsive | comportamiento del diseño en diferentes tamaños de pantalla |
| Tamaño y ubicación de imágenes/iconos | diferencias en px |

Ser **especifico** en cada discrepancia encontrada. Ejemplo:
- "El heading es de 32px pero la referencia muestra 24px"
- "El gap entre cards es 16px pero deberia ser 24px"
- "El color del boton es #3882f6 pero la referencia muestra #2563EB"

### 5. Corregir 

Aplicar **todos** los ajustes identificados en el paso anterior en una unica edicion del `index.html`. No realizar ediciones parciales ni recapturar entre correcciones individuales.

### 6. Re-capturar y repetir

Volver al paso 3 y repetir la comparacion.

**Realizar siempre un minimo de 2 rondas completas de comparacion.**
Solo detener cuando:
- El usuario lo indique explícitamente, o
- no queden diferencias visibles (dentro de ~2-3px en todos los elementos)


---

## Reglas

- No añadir caracteristicas, secciones ni contenido que no esten en la referencia
- Igualar la referencia exactamente - no "mejorar" el diseño
- Si el usuario proporciona clases CSS o tokens de estilo, usarlos literalmente
- Mantener el codigo limpio pero sin abstraer en exceso - clases Tailwind inline estan bien
- Si la Url no es accesible (error de red, auth requerida, etc.), notificarlo y pedir una captura manual
- Si Puppeteer no esta instalado: `npm install puppeteer` antes de usarlo
