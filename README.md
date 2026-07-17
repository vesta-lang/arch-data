# arch-data

Bases de datos de instrucciones **ya generadas y serializadas** para el
compilador [Vesta](https://github.com/vesta-lang), mas un volcado legible de
todas las instrucciones.

Este repositorio guarda **datos**, no codigo.  Su unico proposito es que nadie
tenga que regenerar la base de datos desde las fuentes microarquitecturales
originales (un XML de ~140 MB) cada vez que quiera compilar: el compilador
consume directamente los ficheros de aqui.

---

## Que contiene

Para cada arquitectura de instrucciones (hoy x86; mas adelante ARM, RISC-V):

| Fichero                        | Contenido |
| :----------------------------- | :-------- |
| `x86/x86.vxisa`                | Sintaxis y semantica por ISA: una fila por forma de instruccion (operandos completos, encoding, efectos, propiedades semanticas). |
| `x86/intel-skylake.vxarch`     | Modelo de coste de una microarquitectura: latencias, throughput, uops y puertos, deduplicados en clases de scheduling. |
| `x86/intel-alderlake-p.vxarch` | idem (Golden Cove). |
| `x86/amd-zen4.vxarch`          | idem. |
| `x86/instr_form_ids.h`         | Cabecera C generada: `enum InstrFormID`, `kInstrFormCount` y `kInstrChecksum[]`. |
| `index.html`                   | Visor **visual** de todas las formas: tabla con panel de ayuda, busqueda, filtro por `iclass`, selector de microarquitectura, paginacion y filas expandibles.  Autocontenido; se abre en el navegador o se sirve por GitHub Pages. |
| `dump/x86-instructions.txt`    | Volcado en texto plano de todas las formas, para buscar con `grep` o comparar con `diff`. |

Cada forma de instruccion tiene un identificador estable (`FormID`) derivado de
su firma estructural, no de su nombre.  Los detalles del modelo estan en
[`doc/GENERATION.md`](doc/GENERATION.md).

---

## Para que sirve

La base de datos describe, para cada instruccion:

- **Semantica**: que operandos lee y escribe, si toca memoria o flags, si es una
  barrera de memoria, si serializa, si es atomica, si salta o llama.
- **Coste por microarquitectura**: latencias (por camino operando-a-operando),
  throughput reciproco, numero de micro-operaciones y reparto de puertos.

Generarla desde cero implica descargar y procesar un XML de ~140 MB
([uops.info](https://uops.info/)), lo que es lento y depende de una fuente
externa.  Al versionar aqui el resultado ya serializado, el compilador dispone
de los datos de forma inmediata y reproducible, sin dependencias de red ni pasos
de generacion en el arranque.

---

## Como lo usa el compilador

El compilador **nunca** lee el XML original; carga estos ficheros y los usa para:

- **Analisis del ensamblador en linea** (`asm { }`): a partir de la semantica de
  cada instruccion deriva los efectos y los contratos del bloque (que registros
  toca, si puede fallar, cuanto stack usa) y decide si puede reoptimizarlo.
- **Modelo de coste / scheduling**: consulta las latencias y el reparto de
  puertos por microarquitectura para ordenar instrucciones y estimar coste.
- **Servidor de lenguaje (LSP)**: muestra en el editor que hace cada
  instruccion, como se usa y su coste en la microarquitectura seleccionada.

El identificador `FormID` (un indice denso) es la clave con la que el compilador
indexa estas tablas en memoria.

---

## Procedencia y versiones

Los ficheros de este repositorio se generaron de forma reproducible:

- **Fuente**: `instructions.xml` de uops.info, fecha `2026-03-29`,
  `sha256 = e5e702caaf04c4fc1f75192a9bbe4d8554b3e174f7933a14d124cc33ca677ab3`
  (el hash queda tambien en la cabecera de cada `.vxisa`/`.vxarch`).
- **Generador**: `tools/import/` del repositorio del compilador Vesta,
  commit `4030e35`.
- **Version del formato**: 1.  **Esquema de identidad**: 1.

Como se generan y como actualizar este repositorio: ver
[`doc/GENERATION.md`](doc/GENERATION.md).

---

## Estructura

```text
arch-data/
├── README.md
├── index.html                 visor visual (entrada de GitHub Pages)
├── .nojekyll                  Pages sirve los ficheros tal cual
├── doc/
│   └── GENERATION.md          como se generan estos ficheros
├── x86/
│   ├── x86.vxisa              sintaxis + semantica ISA
│   ├── intel-skylake.vxarch   coste por microarquitectura
│   ├── intel-alderlake-p.vxarch
│   ├── amd-zen4.vxarch
│   └── instr_form_ids.h       cabecera C (FormID + checksums)
└── dump/
    └── x86-instructions.txt   volcado en texto plano de todas las formas
```

---

## Ver online (GitHub Pages)

El visor `index.html` es una pagina estatica autocontenida.  Con GitHub Pages
activado en este repositorio (rama por defecto, carpeta raiz), queda publicado en:

```text
https://vesta-lang.github.io/arch-data/
```

Tambien se puede abrir el fichero `index.html` directamente en el navegador sin
servidor.  El `.nojekyll` evita que Pages procese el sitio con Jekyll.
