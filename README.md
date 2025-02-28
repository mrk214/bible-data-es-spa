# La Biblia en formato JSON

📖 Este proyecto es un conjunto de datos de la **Biblia** en formato `JSON`
(encoding `utf8`), obtenidos mediante web scraping. Los datos están
estructurados de manera estándar y consistente, con la intención de que sea
sencillo implementar cualquiera de las versiones (_traducciones_) disponibles
aquí.

👨‍💻 En la carpeta **data** se encuentran las diferentes versiones, divididas en
archivos `JSON` por cada libro de la Biblia.

🙏 Espero poder seguir agregando traducciones mientras sea posible, espero que
sea útil para alguien que lo necesite; y, sobre todo, espero que sirva para la
obra de Dios.

## Estructura de los datos

Cada archivo `JSON` corresponde a un libro de la Biblia, y fue tipado como
`Book` (con **TypeScript**).
Es por ello que adjunto los tipos en **TypeScript** con los que fueron
estructurados los datos, para que se entienda su funcionamiento.

Los tipos importantes aquí son `Book`, `Chapter`, `ChapterItem` y
`ChapterItemType`.

```typescript
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

export type Chapter = {
  chapter_usfm: string
  is_chapter: boolean
  current: Current
  next: NextPrev | null
  previous: NextPrev | null
  chapter_text: string
  chapter_html: string
  items: ChapterItem[]
}

export type ChapterItem = {
  type: ChapterItemType
  verse_numbers: number[]
  lines: string[]
  rlw_lines: RedLetterWordsSection[][]
}

// Dependiendo de la versión, algunos ChapterItemType pueden aparecer más o menos.
// Los ChapterItemType esenciales son: 'heading1' y 'verse'.
// Puse comentarios que pueden usarse como referencia para los estilos. 👇👇👇
// (Solo es una referencia; puedes poner los estilos que quieras.)
export type ChapterItemType =
  | 'section1' // raro      - weight: 900 - h1
  | 'section2' // raro      - weight: 800 - h2
  | 'heading1' // muy común - weight: 700 - h3
  | 'heading2' // común     - weight: 600 - h4
  | 'label' //    común     - weight: 500 - italic
  | 'verse' //    muy común - weight: 400 - regular text
```

Demás tipos.

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

export type NextPrev = Current & {
  canonical: boolean
  toc: boolean
}

export type RedLetterWordsSection = {
  text: string
  rl: boolean
}
```

## Explicación

Los datos son en su mayoría autoexplicativos, pero dejo algunas aclaraciones ya
que, según la versión, pueden variar algunos datos (_pero no la estructura, la
estructura es la misma para todas las versiones_):

👉 Cada **libro** (`Book`) contiene **capítulos** (`Chapter[]`), y cada capítulo
contiene **items** (`ChapterItem[]`).

👉 En algunas versiones, algunos libros tienen una introducción.
Para verificar que un **capítulo** (`Chapter`) es realmente un capítulo y no una
introducción, puedes usar la propiedad `is_chapter`.

👉 Si un `Chapter` es una introducción, entonces su propiedad `items` será un
array vacío. Si deseas usar el contenido, estará disponible en las propiedades
`chapter_text` y `chapter_html`.

👉 Un `ChapterItem` casi siempre será de tipo **verse** (`verse`) o **heading1**
(`heading1`). Sin embargo, hay varios otros tipos: `section1`, `section2`,
`heading1`, `heading2`, `label`, `verse`.

👉 Si un `ChapterItem` **no es de tipo verse** (`verse`), entonces su propiedad
`verse_numbers` siempre será un array vacío `[]`.

👉 La razón por la que `verse_numbers` es un array es porque algunas versiones
agrupan varios versículos en un solo párrafo de texto, sin que se pueda saber
dónde comienza o termina cada versículo: `"verse_numbers":[5,6,7]`.

👉 Pero lo más común siempre será que `verse_numbers` contenga un solo item:
`"verse_numbers":[7]`.

👉 Los versículos (`ChapterItem` _de tipo_ `verse`) se dividen en líneas, por lo
que `lines` es un array (`string[]`).
A veces tiene un solo elemento, y otras veces varios, dependiendo de cómo esté
estructurado el versículo (_esto no lo decidí yo, sino que así viene desde la
fuente de datos, y puede cambiar de un capítulo a otro_).

👉 Por otro lado, en los `ChapterItem` de tipo **diferente a** `verse`, como
`heading1`, su propiedad `lines` siempre será un array de un solo elemento:
`"lines":["Hello world"]`.

👉 Algunas veces se puede encontrar un título (_u otro elemento_) en medio de un
versículo. En estos casos, el versículo continúa después del elemento. Por lo
tanto, un mismo `verse_numbers` puede repetirse en más de un `ChapterItem`.

- Esto se debe tener en cuenta a la hora de buscar un versículo, por ejemplo,
  ya que **aunque no es común**, se pueden encontrar varios `ChapterItem` con el
  mismo array en `verse_numbers`.

- También porque, generalmente, solo se quiere mostrar el número del versículo
  la primera vez, al principio del versículo.

👉 Cada capítulo (`Chapter`) también contiene el texto completo del capítulo en
formato plano en `chapter_text`, con caracteres de salto de línea (`\n`), y el
HTML original en `chapter_html`.

👉 **rlw** significa **red letter words**, es decir, las palabras atribuidas a
Jesús. Por esto, `ChapterItem` tiene una propiedad llamada `rlw_lines`.

👉 Para ahorrar espacio, `rlw_lines` casi siempre será un array vacío, ya que la
mayoría de los versículos de la Biblia no contienen **red letter words** (_y
tampoco en todas las versiones vienen marcadas_).

👉 El array `rlw_lines` solo tendrá items cuando el versículo contenga
**red letter words**. Puedes usar este dato como una validación.

👉 También debes saber que, si el versículo contiene **red letter words**,
entonces por cada elemento en el array `lines` existe un elemento en el
array `rlw_lines`.

- **if** `rlw_lines.length > 0` **then** `rlw_lines.length === lines.length`

👉 Sin embargo, cada elemento de `rlw_lines` también es un array, porque a veces
solo una parte de una línea se atribuye a Jesús.

## Ejemplo de código

🤯 Para entender todo lo anterior, lo mejor es ir al código y revisar los datos
y su estructura.

🧠 Por eso, para ilustrar mejor los **diferentes casos que se pueden encontrar**
en los datos, hice un ejemplo de código en el cual genero un archivo `README.md`
a partir de leer algunos capítulos especiales que tienen **diferentes niveles
de complejidad**.

🤔 Estoy seguro de que te puede servir para guiarte y darte ideas.

🚀 Revisa el código en el
[repositorio del ejemplo aquí](https://github.com/mrk214/reading-json-files).

## Datos calculados

Creo que los datos son bastante completos; sin embargo, **para evitar algo de
redundancia y mantener cierta flexibilidad** y libertad, hay datos que no
se encuentran de manera explícita, porque **se pueden calcular** de
diferentes maneras.

Así que, si no encuentras un dato en particular de manera explícita, es muy
probable que se pueda calcular con código (_solo hay que programarlo_ 👨‍💻).

## Archivos alojados en GitHub

Puedes clonar el repositorio y trabajar con los archivos de manera convencional
o, si prefieres, acceder directamente a los `archivos JSON servidos en GitHub`.

En cada carpeta de versión hay un archivo `README.md` que incluye:

- La **cantidad de libros** que contiene esa versión.
- Y los **links directos a los archivos JSON alojados en GitHub**.

| usfm      | name                                           | link                                                                                                  |
| --------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `DHH94I`  | Biblia Dios Habla Hoy                          | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/DHH94I/README.md)  |
| `DHHS94`  | Dios habla Hoy Estándar                        | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/DHHS94/README.md)  |
| `LBLA`    | La Biblia de las Américas                      | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/LBLA/README.md)    |
| `NBLA`    | Nueva Biblia de las Américas                   | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/NBLA/README.md)    |
| `NTV`     | Nueva Traducción Viviente                      | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/NTV/README.md)     |
| `NVI`     | Nueva Versión Internacional                    | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/NVI/README.md)     |
| `RVA2015` | Reina Valera Actualizada                       | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/RVA2015/README.md) |
| `RVC`     | Reina Valera Contemporánea                     | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/RVC/README.md)     |
| `RVR1960` | Biblia Reina Valera 1960                       | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/RVR1960/README.md) |
| `TLA`     | Traducción en Lenguaje Actual                  | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/TLA/README.md)     |
| `TLAI`    | Traducción en Lenguaje Actual Interconfesional | [readme](https://github.com/mrk214/bible-data-es-spa/blob/main/data/es___spa___spa/TLAI/README.md)    |

## Capítulos especiales

En algunas versiones, hay casos especiales de algunos capítulos
(`Chapter`) que tienen un `chapter_usfm` en un formato diferente a los demás.

Por ejemplo, `PSA.42_1`, cuando el formato común es `PSA.42`.

Para entender un poco mejor su estructura y cómo manejarlos (_por ejemplo,
en una búsqueda_), recomiendo revisar los siguientes campos en cada uno
de esos capítulos:
`chapter_usfm`, `next?.usfm?.[0]`, `chapter_text`, `chapter_html`, `items`.

🔎 **DHH94I**

- `PSA.42`
- `PSA.42_1`
- `PSA.43_1`
- `SIR.1`
- `SIR.1_1`

🔎 **DHHS94**

- `PSA.42`
- `PSA.42_1`
- `PSA.43_1`
- `SIR.1`
- `SIR.1_1`

🔎 **TLAI**

- `BEL.1_1`
- `LJE.1_1`
- `S3Y.1_1`
- `SUS.1_1`
