# CLAUDE.md

Guía para Claude Code en la app **Catálogo de proveedores** de Grupo BMA. Todo en español (UI, datos, comentarios y nombres de funciones), igual que la app de administración.

**Regla de mantenimiento:** cada cambio significativo se documenta aquí en el mismo commit. Lo que todavía no se sabe se marca como *pendiente* en vez de suponerlo.

## Qué es

Catálogo de los productos que ofrecen los proveedores (sobre todo fábricas en China), para tenerlos ordenados y **comparar modelos** entre proveedores. Hay dos líneas, que siguen cómo catalogan sus páginas:

- **SUNL** (sunldemexico.com): motocarros, mototaxis, ATV, areneros, UTV, minimotos, motocicletas, vehículos de carga, movilidad eléctrica (patines, motonetas y triciclos personales), refacciones. Primero el tipo y luego los atributos (cilindrada, cabina, tipo de caja, rodado, ejes, carga…). Es el modelo a seguir.
- **ALZATOR** (alzator.com.mx): maquinaria pesada. Industrial y logística (patín, apiladora, montacargas, montacargas todo terreno y telescópico, plataformas de elevación, mesas elevadoras, grúas y polipastos, que incluye el equipo para tambos) y Construcción (mini y excavadoras, minicargador, trascavo, retroexcavadora, minidumper) y Agrícola (tractores).

Las categorías y los atributos que se comparan en cada una están en `CATEGORIAS` de `build_catalogo.py` (archivo local, fuera del repo).

## Estado actual (4-nov-2026)

`index.html` es la app (antes fue el piloto de refacciones; se conservó su diseño). Un solo archivo, sin librerías.

- **Datos:** viven en Supabase (ver "Datos y seguridad"). Al abrir pide usuario y contraseña; sin sesión no se ve nada. La página publicada y la local son iguales: las dos leen de Supabase.
- **Subir datos:** solo en la computadora (localhost). Si existe `datos/productos.json`, junto a "Salir" aparece "Subir datos de esta computadora", que **reemplaza todo** el catálogo de Supabase por ese archivo (función `catalogo_subir`). Flujo: `python build_catalogo.py`, abrir la app local, entrar y subir.
- **Formato del JSON:** `{categorias: {LINEA: {Familia: {Categoría: [campos a comparar]}}}, etiquetas: {campo: "Nombre"}, productos: [...]}`. Cada producto: `id, linea, familia, categoria, proveedor, modelo, nombre, precio, moneda, incoterm, fecha, atributos{}`, y opcionales `medidas_mm, empaque, piezas_40hq, notas, archivo`.
- **Navegación:** línea (SUNL / ALZATOR) → tipo (botones agrupados por familia, solo los que tienen productos) → filtros por característica (selects, solo los de `FILTRABLES` con 2 o más valores, más Proveedor). La lista va de menor a mayor precio. El **buscador es general**: con texto busca en todo el catálogo (las dos líneas y todos los tipos, sin filtros) en nombre, modelo, proveedor, tipo, familia, línea y características; cada palabra debe aparecer y acepta plurales ("atv's" → ATV). Los resultados se ordenan por línea, tipo y precio, y muestran su tipo arriba. "Borrar búsqueda" o escoger un tipo regresa a la navegación normal.
- **Sin precio:** un producto puede venir con `precio: null` (folletos); se muestra "Sin precio", va al final de la lista y no cuenta para el mejor precio al comparar.
- **Ficha:** precio, incoterm, piezas por 40HQ, todas las características (las que faltan dicen "No indicado"), notas de captura y archivo fuente.
- **Comparar:** casilla en cada producto (también se pueden mezclar tipos); la barra de abajo abre la tabla a todo lo ancho. Mejor precio en verde (solo si todos están en la misma moneda); lo que es igual en todos sale en gris.
- **Celular** (≤820 px): lista o ficha, una a la vez, con botón para regresar.
- **Probar:** `.claude/launch.json` levanta `python -m http.server 8766`. Ya no hay datos de ejemplo: para probar hay que entrar con la cuenta.

La captura de catálogos se hace en local: `build_catalogo.py` genera `datos/productos.json` y `Catalogo comparativo.xlsx`. El avance (qué catálogos faltan) está en `AVANCE.md`, también local.

## Regla estricta: nada con precios en el repo

El repo es **público** (GitHub Pages en el plan gratis). Los catálogos de proveedores, los precios y los datos capturados **nunca** se suben: `Catalogos/`, `datos/`, `build_catalogo.py`, los Excel y `AVANCE.md` están en `.gitignore`. Revisar `git status` antes de cada commit.

## Repositorio y despliegue

- Repo **`GrupoBMA/catalogo`** en la organización `GrupoBMA`, hermano de `GrupoBMA/administracion`. Se publica con GitHub Pages desde `main` en <https://grupobma.github.io/catalogo/>. **Cada push a `main` sale en vivo.**
- Clon local: `OneDrive\Claude Code\Catalogo Proveedores`.
- Flujo: editar, commit descriptivo, `git push origin main` y decirle al usuario dónde mirarlo en la página en vivo.

## Datos y seguridad (desde el 5-nov-2026)

- **Supabase:** mismo proyecto que GVF y administración (`jlfendrdwptwmttrahre`). Esquema propio `catalogo` con `productos`, `config` (categorías y etiquetas) y `usuarios` (lista de cuentas con acceso). El esquema **no** está expuesto en la API y sus tablas tienen RLS sin reglas: nadie las lee directo.
- **Única puerta:** `public.catalogo_datos()` (leer todo) y `public.catalogo_subir(datos)` (reemplazar todo). Las dos son `security definer` y primero revisan que la sesión esté en `catalogo.usuarios`; `anon` no puede llamarlas.
- **Cuentas:** solo una, la del usuario. Se crea en el panel de Supabase (Authentication → Users → Add user, con "Auto Confirm"), con correo `usuario@catalogo.bma`; en la app se escribe solo `usuario`. Después se agrega su id a `catalogo.usuarios`. Si olvida la contraseña, se cambia desde el panel.
- **GVF:** al hacer esto se encontró que las 26 tablas de GVF dejaban entrar a cualquier cuenta con sesión. Ahora sus reglas (`solo_gvf_*`) solo dejan pasar a las cuentas de `acceso.gvf_usuarios` (hoy, la de GVF). Una cuenta nueva de GVF hay que agregarla ahí.
- La llave del código es la pública (publishable); sin sesión no da acceso a nada.
