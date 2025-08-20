## Точка входа

Начнем эту часть с того же, с чего начали предыдущую: с секции "заголовка" по адресу $100. Мы также объявим глобальные переменные, которые будут использованы во время игры.

- `wLastKeys` и `wCurKeys` будут использоваться для считывания с крестовины.
- `wGameState` будет отслеживать текущее состояние игры.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/GalacticArmada.asm:entry-point}}
{{#include ../../galactic-armada/src/main/GalacticArmada.asm:entry-point}}
```

после метки `EntryPoint` мы сделаем следующее:

- зададим начальное состояние игры
- инициализируем библиотеку для работы со спрайтами [gb-sprobj-lib](https://github.com/eievui5/gb-sprobj-lib)
- настроим дисплейные регистры
- загрузим данные о тайлах для нашего шрифта в VRAM.

Данные о тайлах, которые мы загрузим, будут использованы во всех игровых состояниях, поэтому мы это делаем сейчас.

<img class="pixelated" src="../assets/part3/img/text-font-large.png">

Этот набор букв называется “Area51”. Этот и другие пиксельные шрифты 8x8 можно найти здесь: [https://damieng.com/typography/zx-origins/](https://damieng.com/typography/zx-origins/). Наши 52 тайла мы разместим в самом начале области фона VRAM.

![TextFontDiagram.png](../assets/part3/img/TextFontDiagram.png)

Важная вещь, на которую стоит обратить внимание. Сопоставление тайла букве надо определить в коде. Это позволит RGBDS понять, какой байт надо сопоставить с конкретной буквой.

Для выполнения сопоставления в Galactic Armada посмотрим на “text-font.png”. Пробел - это первый символ на картинке, а значит, первый символ в VRAM. Алфавит начинается с 26 символа. Особые символы можно добавить отдельно. Пока что это всё, что нам нужно. Мы определим сопоставления в "src/main/utils/macros/text-macros.inc".

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/macros/text-macros.inc:charmap}}
{{#include ../../galactic-armada/src/main/utils/macros/text-macros.inc:charmap}}
```

Вернемся к точке входа. Мы будем ждать VBlank для того, чтобы начать работу. Мы также отключим LCD перед загрузкой тайлов в VRAM.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/GalacticArmada.asm:entry-point-end}}
{{#include ../../galactic-armada/src/main/GalacticArmada.asm:entry-point-end}}
```

:::tip

Даже если мы не определили цветовую палитру, [emulicious](https://emulicious.net/) может за нас выбрать цвета по умолчанию, если мы находимся в "автоматическом" ("Automatic") режиме или в режиме "Gameboy Color". 

:::

:::tip

Вместо инструкции `ld a, 0` мы можем использовать `xor a`, чтобы обнулить регистр. Она занимает на 1 байт меньше, что многое значит для Game Boy.

:::

В коде выше вы видели  вызов функции `WaitForOneVBLank`. Мы определим необходимые для работы с VBlank функции (в т.ч. и ее) в файле "src/main/utils/vblank-utils.asm":

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/vblank-utils.asm:vblank-utils}}
{{#include ../../galactic-armada/src/main/utils/vblank-utils.asm:vblank-utils}}
```

В следующей главе мы займемся меткой `NextGameState`, которая будет переключать игрока между игровыми состояниями.
