# Como se generan estos ficheros

Los datos de este repositorio son **generados**, no escritos a mano.  El
generador es la unica fuente de verdad; aqui solo se versiona su salida.  Este
documento explica de donde salen y como regenerarlos.

---

## Fuente de datos

- **uops.info** &mdash; `instructions.xml` (~140 MB): sintaxis, operandos,
  encoding y mediciones de latencia/throughput/puertos por microarquitectura.
  <https://uops.info/>
- **Overlay semantico manual** &mdash; lo que uops.info no codifica
  (serializante, barrera, atomica, salto, llamada): un fichero pequeno y estable
  dentro del generador.

El `.vxisa` y cada `.vxarch` llevan en su cabecera el `sha256` y la fecha del XML
con el que se generaron, de modo que la procedencia es verificable a partir de
los propios ficheros.

---

## Generador

El generador vive en el repositorio del compilador Vesta, en `tools/import/`
(pipeline importar &rarr; optimizar &rarr; serializar).  **No** se duplica aqui
para no tener dos copias que puedan divergir.

La documentacion completa del generador (arquitectura, formato, modelo de
identidad `FormID`) esta en el `README.md` de ese directorio.

---

## Regenerar la base de datos

Desde la raiz del repositorio del compilador, con el `instructions.xml`
descargado de uops.info:

```bash
# 1. bases de datos serializadas (.vxisa, .vxarch, instr_form_ids.h)
python tools/import/build_database.py <instructions.xml> <arch-data>/x86 \
    tools/import/overlay_x86_semantics.def

# 2. sitio visual (index.html + analyzer.html + assets/, entrada de GitHub Pages)
python tools/import/dump_html.py <arch-data>/x86 <arch-data>

# 3. volcado en texto plano
python tools/import/dump_db.py <arch-data>/x86 "" --limit 0 \
    > <arch-data>/dump/x86-instructions.txt
```

Sustituye `<arch-data>` por la ruta de este repositorio.

---

## Procedencia de la version actual

| Dato                     | Valor |
| :----------------------- | :---- |
| Fuente                   | uops.info `instructions.xml` |
| Fecha del XML            | `2026-03-29` |
| `sha256` del XML         | `e5e702caaf04c4fc1f75192a9bbe4d8554b3e174f7933a14d124cc33ca677ab3` |
| Commit del generador     | `4030e35` (repositorio del compilador Vesta) |
| Version del formato      | `1` |
| Esquema de identidad     | `1` |
| Formas de instruccion    | `22252` |

---

## Cuando y como actualizar este repositorio

Regenera y sube un commit cuando cambie cualquiera de estos:

- una **nueva version del `instructions.xml`** de uops.info (cambia el `sha256`);
- una **microarquitectura nueva** anadida al generador;
- una correccion o ampliacion del **overlay semantico**;
- un cambio de **formato** o de **esquema de identidad** en el generador (sube la
  version correspondiente en la cabecera).

Al hacer commit, actualiza la tabla de procedencia de arriba y la seccion
"Procedencia y versiones" del `README.md` con los nuevos valores.
