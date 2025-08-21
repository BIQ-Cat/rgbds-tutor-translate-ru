# Экран с предысторией

Этот экран показывает 2 страницы текста с сюжетом, а затем меняет состояние на игровое.

<img src="../assets/part3/img/GalacticArmada-1.png" class="pixelated" height="288px">

<img src="../assets/part3/img/GalacticArmada-2.png" class="pixelated" height="288px">

## Инициализация экрана Предыстории

Внутри `InitStoryState` мы просто включаем LCD. Большая часть логики экрана находится в функции обновления состояния.

:::tip

Поскольку мы подключили файл с сопоставлениями символов буквам, наш текст будет иметь правильный шрифт.

:::

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/story/story-state.asm:init-story-state}}
{{#include ../../galactic-armada/src/main/states/story/story-state.asm:init-story-state}}
```

## Обновление экрана сюжета

Вот данные для нашего экрана сюжета. Мы их определяем прямо над `UpdateStoryState`.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-data}}
{{#include ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-data}}
```

Лор игры показывается с использованием эффекта "пишущей машинки". Этот эффект достигается таким же образом, как и появление строки ***press A to play***, но теперь мы ждём 3 VBlank'а между буквами, что создает дополнительную задержку.

> Вы также можете поместить это число в переменную и сделать задержку настраиваемой в параметрах игры.

Для этого эффекта мы определили функцию в "src/main/utils/text-utils.asm".

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/text-utils.asm:typewriter-effect}}
{{#include ../../galactic-armada/src/main/utils/text-utils.asm:typewriter-effect}}
```

Мы вызываем `DrawText_WithTypewriterEffect` так же, как и `DrawTextTilesLoop`. В качестве аргументов передаем адрес участка тайлмапа, с которого начинается печать, в `de` и адрес нашего текста в `hl`.

Мы запустим ее 4 раза, а потом будем ждать нажатия A.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-page1}}
{{#include ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-page1}}
```

Как только пользователь нажал кнопку, нам необходимо показать 2 страницу. Чтобы напечатать текст со второй страницы, нам надо сначала стереть текст с первой, а для этого надо очистить фон. Все, что функция `ClearBackground` делает - выключает экран, заполняет фон первым тайлом, затем включает lcd. Мы определили эту функцию в "src/main/utils/background.utils.asm".

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/background-utils.asm:background-utils}}
{{#include ../../galactic-armada/src/main/utils/background-utils.asm:background-utils}}
```

Возвращаясь к экрану с предысторией: после показа первой страницы текста и очистки фона мы делаем всё то же самое со второй страницей:

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-page2}}
{{#include ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-page2}}
```

После того, как мы показали игроку сюжет, нужно переместить его в следующее состояние: экран игрового процесса. Мы закончим исполнение `UpdateStoryState` изменяя значение переменной и переходя на метку `NextGameState`, как было сказано ранее.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-end}}
{{#include ../../galactic-armada/src/main/states/story/story-state.asm:story-screen-end}}
```
