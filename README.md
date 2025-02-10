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

### Nueva Traducción Viviente (NTV)

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

### Reina Valera 1960 (RVR1960)

| Name                    | ID (usfm) | Link                                                                                  |
| ----------------------- | --------- | ------------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVR1960/REV.json) |

### Reina Valera Actualizada (RVA2015)

| Name                    | ID (usfm) | Link                                                                                  |
| ----------------------- | --------- | ------------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVA2015/REV.json) |

### Reina Valera Contemporánea (RVC)

| Name                    | ID (usfm) | Link                                                                              |
| ----------------------- | --------- | --------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/RVC/REV.json) |

### Nueva Versión Internacional (NVI)

| Name                    | ID (usfm) | Link                                                                              |
| ----------------------- | --------- | --------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NVI/REV.json) |

### La Biblia de las Américas (LBLA)

| Name                    | ID (usfm) | Link                                                                               |
| ----------------------- | --------- | ---------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/LBLA/REV.json) |

### Nueva Biblia de las Américas (NBLA)

| Name                    | ID (usfm) | Link                                                                               |
| ----------------------- | --------- | ---------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/NBLA/REV.json) |

### Dios habla Hoy Estándar (DHHS94) - unchecked

| Name                    | ID (usfm) | Link                                                                                 |
| ----------------------- | --------- | ------------------------------------------------------------------------------------ |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHHS94/REV.json) |

### Biblia Dios Habla Hoy (DHH94I) - unchecked

| Name                    | ID (usfm) | Link                                                                                 |
| ----------------------- | --------- | ------------------------------------------------------------------------------------ |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/DHH94I/REV.json) |

### Traducción en Lenguaje Actual (TLA) - unchecked

| Name                    | ID (usfm) | Link                                                                              |
| ----------------------- | --------- | --------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLA/REV.json) |

### Traducción en Lenguaje Actual Interconfesional (TLAI) - unchecked

| Name                    | ID (usfm) | Link                                                                               |
| ----------------------- | --------- | ---------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___spa/TLAI/REV.json) |
