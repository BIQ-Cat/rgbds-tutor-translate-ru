# Главный экран

Главный экран показывает простенькое изображение, используя фоновые тайлы, и отображает текст, требующий от игрока нажать **A** для начала игры. Как только игрок нажимает **A**, мы перемещаемся на экран с лором.

<img src="../assets/part3/img/title-screen-large.png" class="pixelated">

Главный экран содержит 3 вещи:

* Текст "Press A to play"
* Данные о тайлах
* Tilemap

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:title-screen-start}}
{{#include ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:title-screen-start}}
```

## Инициализация главного экрана

В функции "InitTitleScreen" мы будем делать следующее:
* отображать графику главного экрана,
* писать "Press A to play",
* включать LCD.


Вот так выглядит функция "InitTitleScreenState":

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:title-screen-init}}
{{#include ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:title-screen-init}}
```

Чтобы писать текст в нашей игре, мы создали функцию "DrawTextTilesLoop". В качестве параметров мы передаем адрес участка тайлмапа, в который будет помещён первый символ, в `de` и адрес нашего текста в `hl`.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/text-utils.asm:draw-text-tiles}}
{{#include ../../galactic-armada/src/main/utils/text-utils.asm:draw-text-tiles}}
```

Функция "DrawTitleScreen" помещает тайлы главного экрана в VRAM и загружает тайлмап фона:

> **ВАЖНО:** Из-за наличия тайлов символов нам нужно добавить смещение в 52 байта в наш тайлмап. Мы написали для этой цели отдельную функцию, поскольку нам придётся такое делать ещё не раз.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:draw-title-screen}}
{{#include ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:draw-title-screen}}
```

Функции "CopyDEintoMemoryAtHL" и "CopyDEintoMemoryAtHL_With52Offset" определены в "src/main/utils/memory-utils.asm":

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/memory-utils.asm:memory-utils}}
{{#include ../../galactic-armada/src/main/utils/memory-utils.asm:memory-utils}}
```

## Обновление главного экрана

Логика обновления экрана - самая простая часть этого урока. Все, что мы будем делать - ждать до момента нажатия кнопки А. После мы перейдем на экран с предысторией.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:update-title-screen}}
{{#include ../../galactic-armada/src/main/states/title-screen/title-screen-state.asm:update-title-screen}}
```

Наша "WaitForKeyFunction" определена в "src/main/utils/input-utils.asm". Мы будем находиться в бесконечном цикле, пока пользователь не нажмет на кнопку.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/input-utils.asm:input-utils}}
{{#include ../../galactic-armada/src/main/utils/input-utils.asm:input-utils}}
```

На этом с главным экраном всё. Далее - экран с предысторией.
