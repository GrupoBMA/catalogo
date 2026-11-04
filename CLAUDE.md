# CLAUDE.md

Guía para Claude Code en la app **Catálogo de proveedores** de Grupo BMA. Todo en español (UI, datos, comentarios y nombres de funciones), igual que la app de administración.

**Regla de mantenimiento:** cada cambio significativo se documenta aquí en el mismo commit. Lo que todavía no se sabe se marca como *pendiente* en vez de suponerlo.

## Qué es

Catálogo de los productos que ofrecen los proveedores (sobre todo fábricas en China), para tenerlos ordenados y **comparar modelos** entre proveedores. Hay dos líneas, que siguen cómo catalogan sus páginas:

- **SUNL** (sunldemexico.com): motocarros, mototaxis, ATV, areneros, UTV, minimotos, motocicletas, movilidad eléctrica, refacciones. Primero el tipo y luego los atributos (cilindrada, cabina, tipo de caja, rodado, ejes, carga…). Es el modelo a seguir.
- **ALZATOR** (alzator.com.mx): maquinaria pesada. Industrial y logística (patín, apiladora, montacargas, montacargas todo terreno y telescópico, plataformas de elevación, grúas y polipastos) y Construcción (mini y excavadoras, minicargador, trascavo, retroexcavadora).

Las categorías y los atributos que se comparan en cada una están en `CATEGORIAS` de `build_catalogo.py` (archivo local, fuera del repo).

## Estado actual (4-nov-2026)

- `index.html`: página de prueba con datos **inventados** (refacciones y proveedores ficticios) para ver el funcionamiento.
- La captura de catálogos se hace en local: `build_catalogo.py` genera `datos/productos.json` y `Catalogo comparativo.xlsx`. El avance (qué catálogos faltan) está en `AVANCE.md`, también local.

## Regla estricta: nada con precios en el repo

El repo es **público** (GitHub Pages en el plan gratis). Los catálogos de proveedores, los precios y los datos capturados **nunca** se suben: `Catalogos/`, `datos/`, `build_catalogo.py`, los Excel y `AVANCE.md` están en `.gitignore`. Revisar `git status` antes de cada commit.

## Repositorio y despliegue

- Repo **`GrupoBMA/catalogo`** en la organización `GrupoBMA`, hermano de `GrupoBMA/administracion`. Se publica con GitHub Pages desde `main` en <https://grupobma.github.io/catalogo/>. **Cada push a `main` sale en vivo.**
- Clon local: `OneDrive\Claude Code\Catalogo Proveedores`.
- Flujo: editar, commit descriptivo, `git push origin main` y decirle al usuario dónde mirarlo en la página en vivo.

## Datos y seguridad (pendiente)

1. Mismo proyecto de Supabase que administración y GVF (`jlfendrdwptwmttrahre`), con tablas **separadas** en un esquema propio (nombre tentativo `catalogo`), expuesto en la API de Supabase.
2. Inicio de sesión antes de mostrar cualquier precio.
3. RLS en todas las tablas y candado de escrituras en `localhost`, como en GVF.
4. Supabase es la fuente de verdad.

Mientras no exista el inicio de sesión, la página publicada solo lleva datos inventados.
