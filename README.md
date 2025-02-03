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

export type ChapterItemType = 'heading' | 'verse'

export type ChapterItem = {
  type: ChapterItemType
  verse_number: number
  lines: string[]
  rlw_lines: RedLetterWordsSection[][]
}

export type Chapter = {
  chapter_usfm: string
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

Creo que los datos son bastante autoexplicativos, sin embargo voy a aclarar algunos detalles:

👉 Cada **libro** (`Book`) tiene **capítulos** (`Chapter[]`), y cada capítulo tiene **items** (`ChapterItem[]`).

👉 Cada `ChapterItem` puede ser del tipo **título** (`heading`) o **versículo** (`verse`). Si es de tipo **título** (`heading`) entonces `verse_number` siempre será `-1`.

👉 Los versos vienen separados por líneas, por eso `lines` es un arreglo (`string[]`). A veces es un arreglo de un solo item, y otras veces un arreglo de varios items, dependiendo de como esté dividido el versículo.

👉 Pero dentro del capítulo (`Chapter`) también se puede obtener el capítulo completo en texto plano, en la propiedad `chapter_text`, que tiene saltos de línea (`\n`).

👉 **rlw** significa **red letter words**, es decir, las palabras atribuidas a Jesús. Por eso el nombre de la propiedad `rlw_lines`.

👉 Por ahorrar un poco de datos `rlw_lines` casi siempre es un arreglo vacío; ya que en la mayoría de los versículos de la biblia no hay **red letter words**.

👉 Sólo cuando un versículo tiene **red letter words**, `rlw_lines` tendrá líneas.

👉 Por cada item del arreglo `lines` habrá un item en el arreglo `rlw_lines`.

👉 Solo que cada item de `rlw_lines` es otro arreglo, ya que no siempre una línea completa es atribuida a Jesús, a veces es sólo una parte. Pero la estructura siempre será la misma para conservar la consistencia de los datos.

## Datos calculados

Creo que los datos son bastante completos, sin embargo, para evitar redundancia, hay datos que no puse de manera explícita porque se pueden calcular de diferentes maneras (*o porque no los pude obtener*).
Solo es cuestión de usar los datos que sí hay. Por ejemplo:

✅ Para saber cuantos versículos tiene un capítulo (`Chapter`), se pueden contar cuantos `ChapterItem` hay de tipo `verse`.

✅ Para saber si un versículo (`ChapterItem`) tiene **red letter words** basta con verificar si el arreglo `rlw_lines` tiene items (`rlw_lines.length > 0`). Sino, solamente se usa el arreglo `lines`.

Y así mismo, se pueden sacar más datos con programación según los requerimientos.

## Links directos por traducción

También puedes acceder a los archivos estáticos con los siguientes links.

### NTV

| Name                    | ID (usfm) | Link                                                                                   |
| ----------------------- | --------- | -------------------------------------------------------------------------------------- |
| Génesis                 | `GEN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/GEN.json) |
| Éxodo                   | `EXO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/EXO.json) |
| Levítico                | `LEV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/LEV.json) |
| Números                 | `NUM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/NUM.json) |
| Deuteronomio            | `DEU`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/DEU.json) |
| Josué                   | `JOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JOS.json) |
| Jueces                  | `JDG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JDG.json) |
| Rut                     | `RUT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/RUT.json) |
| 1 Samuel                | `1SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1SA.json) |
| 2 Samuel                | `2SA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2SA.json) |
| 1 Reyes                 | `1KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1KI.json) |
| 2 Reyes                 | `2KI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2KI.json) |
| 1 Crónicas              | `1CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1CH.json) |
| 2 Crónicas              | `2CH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2CH.json) |
| Esdras                  | `EZR`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/EZR.json) |
| Nehemías                | `NEH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/NEH.json) |
| Ester                   | `EST`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/EST.json) |
| Job                     | `JOB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JOB.json) |
| Salmos                  | `PSA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/PSA.json) |
| Proverbios              | `PRO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/PRO.json) |
| Eclesiastés             | `ECC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ECC.json) |
| Cantar de los Cantares  | `SNG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/SNG.json) |
| Isaías                  | `ISA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ISA.json) |
| Jeremías                | `JER`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JER.json) |
| Lamentaciones           | `LAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/LAM.json) |
| Ezequiel                | `EZK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/EZK.json) |
| Daniel                  | `DAN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/DAN.json) |
| Oseas                   | `HOS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/HOS.json) |
| Joel                    | `JOL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JOL.json) |
| Amós                    | `AMO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/AMO.json) |
| Abdías                  | `OBA`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/OBA.json) |
| Jonás                   | `JON`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JON.json) |
| Miqueas                 | `MIC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/MIC.json) |
| Nahúm                   | `NAM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/NAM.json) |
| Habacuc                 | `HAB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/HAB.json) |
| Sofonías                | `ZEP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ZEP.json) |
| Hageo                   | `HAG`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/HAG.json) |
| Zacarías                | `ZEC`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ZEC.json) |
| Malaquías               | `MAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/MAL.json) |
| Mateo                   | `MAT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/MAT.json) |
| Marcos                  | `MRK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/MRK.json) |
| Lucas                   | `LUK`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/LUK.json) |
| Juan                    | `JHN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JHN.json) |
| Hechos de los Apóstoles | `ACT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ACT.json) |
| Romanos                 | `ROM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/ROM.json) |
| 1 Corintios             | `1CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1CO.json) |
| 2 Corintios             | `2CO`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2CO.json) |
| Gálatas                 | `GAL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/GAL.json) |
| Efesios                 | `EPH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/EPH.json) |
| Filipenses              | `PHP`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/PHP.json) |
| Colosenses              | `COL`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/COL.json) |
| 1 Tesalonicenses        | `1TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1TH.json) |
| 2 Tesalonicenses        | `2TH`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2TH.json) |
| 1 Timoteo               | `1TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1TI.json) |
| 2 Timoteo               | `2TI`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2TI.json) |
| Tito                    | `TIT`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/TIT.json) |
| Filemón                 | `PHM`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/PHM.json) |
| Hebreos                 | `HEB`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/HEB.json) |
| Santiago                | `JAS`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JAS.json) |
| 1 Pedro                 | `1PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1PE.json) |
| 2 Pedro                 | `2PE`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2PE.json) |
| 1 Juan                  | `1JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/1JN.json) |
| 2 Juan                  | `2JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/2JN.json) |
| 3 Juan                  | `3JN`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/3JN.json) |
| Judas                   | `JUD`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/JUD.json) |
| Apocalipsis             | `REV`     | [raw file](https://jsckdm.github.io/bible-data-es-spa/data/es___es___spa/NTV/REV.json) |
