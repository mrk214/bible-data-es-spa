# La Biblia en formato JSON

📖 Este proyecto es un set de datos de la **biblia** en formato `JSON` (encoding `utf8`), obtenidos mediante web scraping, para divulgar la palabra de Dios.
Es una reestructuración de los datos con la intención de que sea sencillo de implementar en cualquier proyecto.

👨‍💻 En la carpeta **data** se encuentran las traducciones de la biblia; divididas en un archivo `JSON` por cada libro de la biblia.

🙏 Espero poder seguir agregando traducciones mientras sea posible, espero que sea útil para alguien que lo necesite; y sobre todo, espero que sirva para la obra de Dios con la humanidad.

## Estructura de los datos

Cada archivo `JSON` corresponde a un libro de la biblia, y está tipado como `Book` (en **typescript**).
Es por ello que adjunto los tipos en **typescript** con los que fueron estructurados los datos, para que se entienda su funcionamiento.

Los tipos importantes aquí son `Book`, `Chapter` y `ChapterItem`.

```typescript
export type Publisher = {
  name: string
}

export type Copyright = {
  html: string
  text: string
}

export type Language = {
  iso_639_1: string
  iso_639_3: string
  language_tag: string
  local_name: string
  text_direction: string
}

export type Current = {
  usfm: string[]
  human: string
}

export type NextPrev =
  | null
  | (Current & {
      canonical: boolean
      toc: boolean
    })

export type RedLetterWordsSection = {
  text: string
  rl: boolean
}

// Para mayor claridad, he asignado un 'weight' a cada tipo.
// Dependiendo de la versión de la traducción, algunos tipos pueden aparecer más o menos
// frecuentemente. Sin embargo, los tipos esenciales son: 'heading1' y 'verse'.
// Recomiendo utilizar y estilizar todas las opciones.
// El 'weight' puede usarse como referencia para el estilo de la fuente del texto.
export type ChapterItemType =
  | 'section1' // raro      - weight: 900
  | 'section2' // raro      - weight: 800
  | 'heading1' // muy común - weight: 700
  | 'heading2' // común     - weight: 600
  | 'label' //    común     - weight: 500
  | 'verse' //    muy común - weight: 400

export type ChapterItem = {
  type: ChapterItemType
  verse_number: number
  lines: string[]
  rlw_lines: RedLetterWordsSection[][]
}

export type Chapter = {
  chapter_usfm: string
  is_chapter: boolean
  current: Current
  next: NextPrev
  previous: NextPrev
  chapter_text: string
  chapter_html: string
  items: ChapterItem[]
}

export type Book = {
  book_usfm: string
  name: string
  local_title: string
  local_abbreviation: string
  version_id: number
  publisher: Publisher
  copyright: Copyright
  language: Language
  repository: string
  chapters: Chapter[]
}
```

## Explicación

Los datos son en su mayoría autoexplicativos, pero aquí hay algunas aclaraciones:

👉 Cada **libro** (`Book`) contiene **capítulos** (`Chapter[]`), y cada capítulo contiene **items** (`ChapterItem[]`).

👉 En algunas versiones, algunos libros tienen una introducción. Para verificar que un **capítulo** (`Chapter`) es realmente un capítulo y no una introducción, puedes usar la propiedad `is_chapter`.

👉 Si un `Chapter` es una introducción, entonces su propiedad `items` será un array vacío. Si deseas usar el contenido, estará disponible en `chapter_text` y `chapter_html`.

👉 Un `ChapterItem` casi siempre será de tipo **verse** (`verse`) o **heading1** (`heading1`). Sin embargo, hay varios otros tipos que pueden estilizarse adecuadamente: `section1`, `section2`, `heading1`, `heading2`, `label`, `verse`.

👉 Si un `ChapterItem` NO es de tipo **verse** (`verse`), entonces su propiedad `verse_number` siempre será `-1`.

👉 Los versos se dividen en líneas, por lo que `lines` es un array (`string[]`). A veces tiene un solo elemento, y otras veces varios, dependiendo de cómo esté estructurado el verso.

👉 Algunos versos pueden contener un título (u otro elemento) en el medio. En estos casos, el verso continúa después del elemento. Por lo tanto, el `verse_number` puede repetirse.

```json
{
  "type": "verse",
  "verse_number": 3,
  "lines": [
    "Pleasing is the fragrance of your perfumes;",
    "your name is like perfume poured out.",
    "No wonder the young women love you!"
  ],
  "rlw_lines": []
},
{
  "type": "verse",
  "verse_number": 4, // 👈 solo debes mostrar este primer 'verse_number'
  "lines": [
    "Take me away with you—let us hurry!",
    "Let the king bring me into his chambers."
  ],
  "rlw_lines": []
},
{
  "type": "heading1",
  "verse_number": -1,
  "lines": ["Friends"],
  "rlw_lines": []
},
{
  "type": "verse",
  "verse_number": 4, // 👈 y NO mostrar este repetido
  "lines": [
    "We rejoice and delight in you;",
    "we will praise your love more than wine."
  ],
  "rlw_lines": []
}
```

👉 Debido a esto, deberías implementar un método para mostrar el `verse_number` solo la primera vez (si ese es tu objetivo). Por ejemplo:

```typescript
let lastPrintedNumber: number = -1
// Condición para imprimir 'verse_number' solo una vez
if (
  chapterItem.verse_number !== -1 &&
  chapterItem.verse_number !== lastPrintedNumber
) {
  lastPrintedNumber = chapterItem.verse_number
  print(chapterItem.verse_number)
}
```

👉 Cada capítulo (`Chapter`) también contiene el texto completo del capítulo en formato plano en `chapter_text`, con caracteres de salto de línea (`\n`), y el html original en `chapter_html`.

👉 **rlw** significa **red letter words**, es decir, palabras atribuidas a Jesús. Por esto, la propiedad se llama `rlw_lines`.

👉 Para ahorrar espacio, `rlw_lines` generalmente es un array vacío, ya que la mayoría de los versos de la Biblia no contienen **red letter words**.

👉 Solo cuando un verso contiene **red letter words**, `rlw_lines` tendrá contenido.

👉 Para cada elemento en el array `lines`, hay un elemento correspondiente en `rlw_lines`.

👉 Sin embargo, cada elemento de `rlw_lines` también es un array, porque a veces solo una parte de una línea se atribuye a Jesús.

## Datos calculados

Creo que los datos son bastante completos, sin embargo, para evitar redundancia, hay datos que no puse de manera explícita porque se pueden calcular de diferentes maneras (_o porque no los pude obtener_). Por ejemplo:

✅ Para saber cuántos versos hay en un capítulo (`Chapter`), puedes buscar el `verse_number` más grande en el array `items: ChapterItem[]`.

✅ Para verificar si un verso (`ChapterItem`) contiene **red letter words**, simplemente revisa si `rlw_lines` tiene elementos (`rlw_lines.length > 0`). De lo contrario, usa solamente `lines`.

## Links directos por traducción

También puedes acceder a los archivos estáticos con los siguientes links.

### NTV

| Name                    | ID (usfm) | Link                                                                              |
| ----------------------- | --------- | --------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NTV/REV.json) |
