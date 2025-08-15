# Начало работы

На этом уроке мы начнем писать новый проект.
Мы напишем клон [Breakout](https://ru.wikipedia.org/wiki/Breakout_(%D0%B8%D0%B3%D1%80%D0%B0)) / [Arkanoid](https://ru.wikipedia.org/wiki/Arkanoid), который назовем "Unbricked"!
(Хотя можете дать ему любое имя, это все-таки _ваш_ проект.)

Откройте терминал и создайте новую папку (`mkdir unbricked`), перейдите в нее (`cd unbricked`), как вы это делали для ["Hello, world!"](../part1/hello_world.md).

Создайте файл `main.asm` и подключите в него `hardware.inc`.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:includes}}
{{#include ../../unbricked/getting-started/main.asm:includes}}
```
Зачем нужен `hardware.inc`?
Тот код, который мы пишем, взаимодействует только с процессором, но не со всей консолю (по крайней мере, напрямую).
Чтобы взаимодействовать с другими компонентами (например, с графикой), ипользуется [Memory-Mapped <abbr title="Input/Output">I/O</abbr>](https://en.wikipedia.org/wiki/Memory-mapped_I/O) (MMIO), проще говоря, [память](../part1/memory.md) в определенном промежутке (адреса $FF00–FF7F), при обращении выполняющая особые функции.

Эти байты памяти являются _аппаратными регистрами_ (не путать с [регистрами процессора](../part1/registers.md)) - особым внутренним интерфейсом консоли.
Например, аппаратный регистр "состояния PPU" находятся на $FF41.
В этих адресах хранятся параметры графической системы, их можно менять по нужде.
Но помнить все ([возможные регистры](https://gbdev.io/pandocs/Power_Up_Sequence.html#hardware-registers)) очень сложно, здесь и вступает в игру `hardware.inc`!
`hardware.inc` определяет каждому регистру свою константу (например, `rSTAT` для ранее упомянутого регистра "PPU статуса"), плюс еще несколько дополнительных для чтения и записи регистров.

::: tip

Если после прочтения данной информации у вас побаливает голова, то рекомендуем посмотреть на примеры с `rLCDC` и `LCDCF_ON`.

Кстати, `r` означает "register", а `F` в `LCDCF` означает "flag".

:::

Следом, мы выделяем место под заголовок.
[Помните из Ⅰ части](../part1/header.md), что заголовок - это место, где хранится информация, на которую опирается gameboy, не забудьте про него.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:header}}
{{#include ../../unbricked/getting-started/main.asm:header}}
```

Из заголовка мы переходим в `EntryPoint`, поэтому давайте напишем следующее:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:entry}}
{{#include ../../unbricked/getting-started/main.asm:entry}}
```

Следующий участок кода ждет исполнения "VBlank". Только во время "VBlank" можно выключить экран (если сделать это в любое другое время, то можно поломать реальный gameboy).
Мы объясним, что такое VBlank, позже.

Выключение экрана очень важно, потому что загрузка новых тайлов при включенном экране - нелёгкая задача. Мы затронем эту тему в 3 части.

Мы загружаем тайлы в VRAM при помощи этого кода:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_tiles}}
{{#include ../../unbricked/getting-started/main.asm:copy_tiles}}
```

Этот цикл может вам напомнить `CopyTiles`  [из первой части](../part1/jumps.md#conditional-jumps).
Мы начинаем копировать из `Tiles` в `$9000`, который является частью VRAM, где и будут храниться наши [тайлы](../part1/tiles.md).
Повторим то, что `$9000`, это место, где находится нулевой тайл, соответственно, первый тайл попадает в `$9001` и т.д.
Чтобы получить кол-во байтов, которые нужно скопировать, мы делаем прямо как в Ⅰ части: используем другую метку в конце, названную `TilesEnd`, разница между `TilesEnd` (= адрес последнего байта тайлов) и `Tiles` (= адрес первого байта) будет равнятся кол-ву байтов.

Однако, мы пока ничего не нарисовали для отображения.
Мы к этому вернемся!

Почти закончили, теперь напишем другой цикл, на этот раз для копирования [tilemap'а](../part1/tilemap.md).

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_map}}
{{#include ../../unbricked/getting-started/main.asm:copy_map}}
```

Отметим, что тело цикла идентично `CopyTiles`, 3 разных значения, загружены в `de`, `hl`, и `bc`.
Они определяет места: откуда копировать, куда копировать и размер данных.

::: tip "[<abbr title="Don't Repeat Yourself">DRY</abbr>](https://en.wikipedia.org/wiki/Don't_Repeat_Yourself)"

Наверное, вы заметили дублирование кода, но пока не беспокойтесь об этом: мы скоро увидим как писать полноценные _функции_, которые можно будет переиспользовать.
Но сейчас есть более важные темы.

:::

Наконец, давайте включим экран, и выберем [фоновую палитру](../part1/palettes.md).
Вместо написания таких чисел: `%10000001` ($81 или 129, на ваш вкус), мы используем 2 константы, предоставленные  `hardware.inc`: `LCDCF_ON` и `LCDCF_BGON`.
Когда записываем данные в [`rLCDC`](https://gbdev.io/pandocs/LCDC), LCDCF_ON включает PPU и экран, а LCDCF_BGON позволяет нам рисовать на заднем фоне.
(Есть также другие элементы, которые могут быть нарисованы, но мы их пока не включаем.)
Комбинировать константы можно при помощи `|`: (оператора _побитового "или"_); мы рассмотрим его позже.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:end}}
{{#include ../../unbricked/getting-started/main.asm:end}}
```

Прежде чем скомпилировать ROM, надо настроить графику.
Мы нарисуем на экране следующее:

![Layout of unbricked](../assets/part2/img/tilemap.png)

В `hello-world.asm`, мы копировали тайлы вручную; делали это для того чтобы было понятно, как программа работает под капотом, но выполнять все это руками очень непрактично!
Здесь же, мы предлагаем путь короче, который позволит проще записывать строчки пикселей.
Для каждой строки, вместо копирования [битовых плоскостей](../part1/tiles.md#encoding) напрямую, мы будем использовать обратную кавычку (`` ` ``) и 8 символов.
Каждый символ определяет один пиксель слева направо; это должен быть 0, 1, 2, или 3, показывающие цветовой индекс в [палитре](../part1/palettes.md).

::: tip

Если вам не нравится наши цветовые символы, вы можете использовать [флаг `-g` RGBASM'а](https://rgbds.gbdev.io/docs/v0.5.2/rgbasm.1#g) или [`OPT g`](https://rgbds.gbdev.io/docs/v0.5.2/rgbasm.5/#Changing_options_while_assembling) чтобы выбрать другие.
Например, `rgbasm -g '.xXO' (...)` или `OPT g.xXO` заставит использовать символы `.`, `x`, `X`, `O` вместо `0`, `1`, `2` и `3` соответственно.

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
$ rgbasm -o main.o main.asm
$ rgblink -o unbricked.gb main.o
$ rgbfix -v -p 0xFF unbricked.gb
```

Если вы запускаете ROM в эмуляторе, то должны увидить следующее:

![Screenshot of our game](../assets/part2/img/screenshot.png)

Кажется, не хватает белого квадрата!
Вы могли заметить такой комментарий:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:custom_logo}}
{{#include ../../unbricked/getting-started/main.asm:custom_logo}}
```

Тайлы, которые нужны для отображение логотипа, остались пусты.
Вы можете использовать наши или сделать свой!

- **Логотип RGBDS**

  ![The RGBDS Logo](../assets/part2/img/rgbds.png)

  [Исходный код](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/rgbds.asm)

- **Уточка**

  ![A pixel-art duck](../assets/part2/img/duck.png)

  [Исходный код](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/duck.asm)

- **Хвост**

  ![A silhouette of a tail](../assets/part2/img/tail.png)

  [Исходный код](https://github.com/gbdev/gb-asm-tutorial/raw/master/unbricked/getting-started/tail.asm)

Добавьте код выбранного логотипа после комментария, пересоберите игру и вы должны увидеть его в правом нижнем углу!
