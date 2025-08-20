# Структура проекта

Эта глава даст вам примерное представление того, как проект Galactic Armada будет структурирован. В перечень будут включены папки, ресурсы, инструменты, точка входа и процесс сборки.

Код может быть найден [здесь](https://github.com/gbdev/gb-asm-tutorial/tree/master/galactic-armada).

## Архитектура

В целях организации кода большая часть программы была разбита на функции. Это уменьшает дублирование и упрощает логики.

Вот так будет выглядить архитектура нашего проекта:

:::tip

Сгенерированные файлы не должны включаться в систему контроля версий, так как они просто загромождают репозиторий. Все папки, отмеченные знаком \*, содержат ресурсы, созданные Makefile'ом. Они должны быть исключены из репозитория.

О системе контроля версий и репозиториях [тут](https://git-scm.com/book/ru/v2/%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%9E-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B5-%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%BE%D0%BB%D1%8F-%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D0%B9).

:::

- `libs` - Дополнительный код для работы с вводом и спрайтами.
- `src`
  - `generated` - результат работы RGBGFX. \*
  - `resources` - Тут находится несколько PNG и Aseprite файлов для использование в купе с RGBGFX
  - `main` - Весь основной код на RGBASM
    - `states`
      - `gameplay` - для файлов, относящихся к игровому процессу
        - `objects` - для игровых объектов, таких как игрок, враг, пуля
          - collision - для обработки коллизий между объектами
      - `story` - для файлов, связанных с экраном истории
      - `title-screen` - для файлов, связанных с главным экраном
    - `utils` - Дополнительные функции для помощи в разработке
      - `macros`
- `dist` - Сюда попадает готовый ROM. \*
- `obj` - Файлы, образованные в процессе сборки. \*
- `Makefile` - Главный сборочный файл; именно он описывает, как собрать ROM

## Ресурсы фона и спрайтов

Далее идут фоны и спрайты, которые будут использованы в Galactic Armada:

- Фоны
  - Star Field
  - Title Screen
  - Text Font (только тайлы)
- Спрайты
  - Enemy Ship
  - Player Ship
  - Bullet

<img class="pixelated" src="../assets/part3/img/star-field.png">

<img class="pixelated" src="../assets/part3/img/title-screen.png">

<br>

<img class="pixelated" src="../assets/part3/img/text-font.png" height="48px">

<br>

<img class="pixelated sprites" src="../assets/part3/img/player-ship.png" height="48px">

<img class="pixelated sprites" src="../assets/part3/img/enemy-ship.png" height="48px">

<img class="pixelated sprites" src="../assets/part3/img/bullet.png" height="48x">


Эти картинки изначально были созданы в Aseprite. Сами файлы .aseprite вы также можете найти в репозитории. Они были экспортированы в формат PNG с **особой цветовой палитрой**. При запуске команды `make` они конвертируются в `.2bpp` и `.tilemap` файлы с помощью утилиты RGBGFX.

> Программа **`rgbgfx`** конвертирует PNG изображения в формат, понятный консолям Game Boy и Game Boy Color и наоборот.
>
> Главный функционал **`rgbgfx`** заключается в разбивании PNG изображений на *[квадраты](https://rgbds.gbdev.io/docs/v0.6.1/rgbgfx.1#squares)* 8 на 8 пикселей, конвертации каждого из них в тайлы 1bpp или 2bpp и сохранениии их в файл. У нее также возможность сгенерировать tile map, attribute map и/или палитру; больше информации об этой программе можете найти ниже.

Ссылка на RGBGFX: [https://rgbds.gbdev.io/docs/v0.6.1/rgbgfx.1](https://rgbds.gbdev.io/docs/v0.6.1/rgbgfx.1)

Мы используем его для конвертации наших картинок в форматы .2bpp и .tilemap (бинарники).

```bash,linenos,start={{#line_no_of "" ../../galactic-armada/Makefile:generate-graphics}}
{{#include ../../galactic-armada/Makefile:generate-graphics}}
```

Для получения доступа к бинарным данным используется команда INCBIN.

```rgbasm
; in src/main/states/gameplay/objects/player.asm
{{#include ../../galactic-armada/src/main/states/gameplay/objects/player.asm:player-tile-data}}

; in src/main/states/gameplay/objects/enemies.asm
{{#include ../../galactic-armada/src/main/states/gameplay/objects/enemies.asm:enemies-tile-data}}

; in src/main/states/gameplay/objects/bullets.asm
{{#include ../../galactic-armada/src/main/states/gameplay/objects/bullets.asm:bullets-tile-data}}
```

:::tip Подключение бинарников

При разработке такого рода проектов у вас обязательно будут графические ресурсы, информация об уровне и т.д, которые вы захотите брать извне. Используйте **`INCBIN`** для подключения бинарного файла "как есть". Если файл не был найден в текущей папке, путь до него будет передан [rgbasm(1)](https://rgbds.gbdev.io/docs/v0.6.1/rgbasm.1) (см. флаг **`-i`**), и поиск будет произведён относительно текущей директории командной строки.

```
INCBIN "titlepic.bin"
INCBIN "sprites/hero.bin"
```

Вы можете подключать любую часть файла при помощи **`INCBIN`**. Пример ниже подключает 256 байтов из data.bin, начиная с 78 байта.

```
INCBIN "data.bin",78,256
```

Аргумент длины опционален. Если указана только стартовая позиция, байты будут копироваться до конца файла.

См. также: [Including binary files - RGBASM documentation](https://rgbds.gbdev.io/docs/v0.6.1/rgbasm.5#Including_binary_files)

:::

## Сборка

Сборка производится с помощью Makefile. Он может быть запущен с помощью команды `make`. Make является предустановленной программой в Linux, MacOS и других UNIX-подобных системах. При работе на Windows рекомендуем воспользоваться [cygwin](https://www.cygwin.com/).

Если кратко, то наш Makefile умеет:

- Чистить сгенерированные папки
- Воссоздавать сгенерированные папки
- Конвертировать PNG, находящиеся в src/resources, в форматы `.2bpp` и `.tilemap`
- Конвертировать `.asm` в `.o`
- Использовать `.o` файлы для сборки ROM фалйа
- Использовать RGBFIX.
