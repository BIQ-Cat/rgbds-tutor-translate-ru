# Начало работы

В этом уроке мы начнем писать новый проект.
Мы напишем клон [Breakout](https://en.wikipedia.org/wiki/Breakout_%28video_game%29) / [Arkanoid](https://en.wikipedia.org/wiki/Arkanoid), который назовем "Unbricked"!
(Хотя, дайте ему любое имя, это все-таки _ваш_ проект.)

Откройте терминал и создайте новую папку (`mkdir unbricked`), перейдите в нее (`cd unbricked`), как вы это делали для ["Hello, world!"](../part1/hello_world.md).

Создайте файл `main.asm` и подключите в него `hardware.inc`.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:includes}}
{{#include ../../unbricked/getting-started/main.asm:includes}}
```
Зачем нужен `hardware.inc`?
Тот код, который мы пишем, взаимодействует только с процессором, но не со всей консолью(по крайней мере, напрямую).
Чтобы взаимодействовать с другими компонентами (например, с графикой), ипользуется [Memory-Mapped <abbr title="Input/Output">I/O</abbr>](https://en.wikipedia.org/wiki/Memory-mapped_I/O) (MMIO), проще говоря, [память](../part1/memory.md) в определенном промежутке (адреса $FF00–FF7F) при обращении выполняют особые функции.

Эти байты памяти являются интерфейсом для консоли, которые называются _регистрами <abbr title="аппаратное обеспечение">АО</abbr>_ (не путать с [регистрами процессора](../part1/registers.md)).
Например, регистры "PPU статуса" находятся на $FF41.
В этих адресах хранятся параметры графической системы, их можно менять по нужде.
Но помнить весь ([бесконечный список регистров](https://gbdev.io/pandocs/Power_Up_Sequence.html#hardware-registers)) очень сложно, здесь и вступает в игру `hardware.inc`!
`hardware.inc` определяет каждому регистру свою константу (например, `rSTAT` для ранее упомянутого регистра "PPU статуса"), плюс еще несколько дополнительных для чтения и записи регистров.

::: tip

Если после прочтения данной информации у вас побаливает голова, то рекомендуем посмотреть на примеры с `rLCDC` и `LCDCF_ON`.

Кстати, `r` означает "register", `F` в `LCDCF` означает "flag".

:::

Следом, мы выделяем место под заголовок.
[Помните из Ⅰ части](../part1/header.md), что заголовок - это место, где хранится информация, на которую опирается gameboy, поэтому вы бы не хотели оставить его пустым.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:header}}
{{#include ../../unbricked/getting-started/main.asm:header}}
```

Из заголовка мы переходим в `EntryPoint`, поэтому давайте напишем следующее:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:entry}}
{{#include ../../unbricked/getting-started/main.asm:entry}}
```

Следующий участок кода ждет "VBlank", только во время которого можно выключить экран (если выключить экран в любое другое время, то можно поломать реальный gameboy).
Мы объясним что такое VBlank позже.

Выключение экрана очень важно, потому что загрузка новых тайлов при включенном экране очень неприятна—мы затронем эту тему в 3 части.

Говоря о тайлах, мы их загрузим в VRAM, используя этот код:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_tiles}}
{{#include ../../unbricked/getting-started/main.asm:copy_tiles}}
```

Этот цикл может вам напомнить тот, который [из первой части Ⅰ](../part1/jumps.md#conditional-jumps).
Мы начинаем копировать из `Tiles` в `$9000`, который является частью VRAM, где и будут храниться наши [тайлы](../part1/tiles.md).
Повторим то, что `$9000`, это место, где находится нулевой тайл, тайлы далее идут следом.
Чтобы получить кол-во байтов, которые нужно скопировать, мы делаем прямо как в Ⅰ части: используем другую метку в конце, названную `TilesEnd`, разница между `TilesEnd` (= адрес последнего байта тайлов) и `Tiles` (= адрес первого байта) будет равнятся кол-ву байтов.

Однако, мы пока ничего не нарисовали для отображения.
Мы к этому вернемся!

Почти закончили, теперь напишем другой цикл, на этот раз для копирования [tilemap'а](../part1/tilemap.md).

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_map}}
{{#include ../../unbricked/getting-started/main.asm:copy_map}}
```

Отметим, что тело цикла идентично `CopyTiles`, 3 разных значения, загружены в `de`, `hl`, и `bc`.
Они определяет места: откуда копировать, куда копировать и размер.

::: tip "[<abbr title="Don't Repeat Yourself">DRY</abbr>](https://en.wikipedia.org/wiki/Don't_Repeat_Yourself)"

Наверное, вы заметили, что код повторяется, и вы правы, мы скоро увидим как писать полноценные _функции_, которые можно будет переиспользовать.
Но сейчас есть более важные темы.

:::

Наконец-то, давайте включим экран, и выберем [фоновую палитру](../part1/palettes.md).
Вместо написания таких чисел: `%10000001` ($81 или 129, на ваш вкус), мы используем 2 константы, предоставленные  `hardware.inc`: `LCDCF_ON` и `LCDCF_BGON`.
Когда записываем данные в [`rLCDC`](https://gbdev.io/pandocs/LCDC), LCDCF_ON включает PPU и экран, а LCDCF_BGON позволяет нам рисовать на заднем фоне.
(Есть также другие элементы, которые могут быть нарисованы, но мы их пока не включаем.)
Комбинировать константы можно при помощи `|`: (оператора _побитового "или"_); мы рассмотрим это позже.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:end}}
{{#include ../../unbricked/getting-started/main.asm:end}}
```

Прежде чем скомпилировать ROM, надо настроить графику.
Мы нарисуем на экане следующее:

![Layout of unbricked](../assets/part2/img/tilemap.png)

В `hello-world.asm`, мы копировали тайлы вручную; делали это для того чтобы было понятно, как программа работает под капотом, но выполнять все это руками очень непрактично!
Здесь же, мы предлагаем более дружественный к глазам путь, который позволит проще записывать строчки пикселей.
Для каждой строки, вместо копирования [битовых плоскостей](../part1/tiles.md#encoding) напрямую, мы будем использовать тильду (`` ` ``) и 8 символов.
Каждый символ определяет один пиксель слева направо; это должен быть 0, 1, 2, или 3, показывающие цветовой индекс в [палитре](../part1/palettes.md).

::: tip

Если вам не нравится наши цветовые символы, вы можете использовать [`-g` option RGBASM'а](https://rgbds.gbdev.io/docs/v0.5.2/rgbasm.1#g) или [`OPT g`](https://rgbds.gbdev.io/docs/v0.5.2/rgbasm.5/#Changing_options_while_assembling) чтобы выбрать другие.
Например, `rgbasm -g '.xXO' (...)` или `OPT g.xXO` заставит использовать символы `.`, `x`, `X`, `O` вместо тех, что выше.

:::

Например:

```rgbasm
	dw `01230123 ; эквивалентно `db $55,$33`
```

Вы, наверное, заметили, что здесь используется `dw` вместо `db`; разницу между ними мы рассмотрим позже...
У нас уже есть нарисованные тайлы для проекта, поэтому вы можете скопировать этот [файл](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/tileset.asm), и вставить его в конец вашего кода.

Затем скопируйте Tilemap также из этого [файла](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/tilemap.asm) и вставьте его после `TilesEnd`.

Теперь же вы можете скомпилировать ROM с помощью следующих команд:

```console
$ rgbasm -L -o main.o main.asm
$ rgblink -o unbricked.gb main.o
$ rgbfix -v -p 0xFF unbricked.gb
```

Если вы запускаете ROM в эмуляторе, то должны увидить следующее:

![Screenshot of our game](../assets/part2/img/screenshot.png)

That white square seems to be missing!
You may have noticed this comment earlier, somewhere in the tile data:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:custom_logo}}
{{#include ../../unbricked/getting-started/main.asm:custom_logo}}
```

The logo tiles were left intentionally blank so that you can choose your own.
You can use one of the following pre-made logos, or try coming up with your own!

- **RGBDS Logo**

  ![The RGBDS Logo](../assets/part2/img/rgbds.png)

  [Source](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/rgbds.asm)

- **Duck**

  ![A pixel-art duck](../assets/part2/img/duck.png)

  [Source](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/duck.asm)

- **Tail**

  ![A silhouette of a tail](../assets/part2/img/tail.png)

  [Source](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/tail.asm)

Add your chosen logo's data (click one of the "Source" links above) after the comment, build the game again, and you should see your logo of choice in the bottom-right!
