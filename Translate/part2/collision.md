# Коллизия

Возможность перемещать ракетку, конечно, прекрасная, но нам нужен еще один объект для игры: мячик!
Как и в случае с ракеткой, первым делом мы должны создать тайл для мяча и загрузить его в VRAM.

## Графика

Добавьте этот участок кода сразу после тайла ракетки:
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:ball-sprite}}
{{#include ../../unbricked/collision/main.asm:ball-sprite}}
```

Теперь скопируйте эти тайлы в VRAM после копирования тайлов ракетки.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:ball-copy}}
{{#include ../../unbricked/collision/main.asm:ball-copy}}
```

Также нужно инициализировать объект в OAM после инициализации ракетки. Это можно сделать следующим образом:
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:oam}}
{{#include ../../unbricked/collision/main.asm:oam}}
```

Когда мяч отскакивает от стенки, он меняет направление движения по x и y.
Давайте создадим две переменные, которые будут следить за направлением мяча: `wBallMomentumX` и `wBallMomentumY`.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:ram}}
{{#include ../../unbricked/collision/main.asm:ram}}
```

Нам нужно инициализировать переменные перед входом в игровой цикл, поэтому давайте установим им начальное значение сразу после операции с OAM.
Когда wBallMomentumX равняется 1, а wBallMomentumY равняется -1, мяч будет лететь в правый верхний угол.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:init}}
{{#include ../../unbricked/collision/main.asm:init}}
```

## Подготовка

Теперь самое веселое!
Добавьте этот участок кода в начало игрового цикла, он управляет мячом, в зависимости от значения переменных.
Обратите внимание, что мы используем `+ 4` для Y и `+ 5` для X (координат мяча).
Этот метод обращения к объекту может показаться достаточно кустарным, однако на данный момент у нас совсем мало объеков, поэтому можем пока оставить все как есть.
В будущем мы воспользуемся гораздо более простым способом работы с OAM.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:momentum}}
{{#include ../../unbricked/collision/main.asm:momentum}}
```

Вы, наверное, захотите перекомпилировать игру, чтобы увидеть изменения.
Если вы это сделаете, то мяч не будет скакать туда-сюда, а просто вылетет за экран. 

Чтобы исправить эту проблему, нам нужно добавить обработку коллизий мяча.
Так как нам придется проверять коллизию несколько раз, мы напишем функцию для этого.

::: tip

Пожалуйста, не зацикливайтесь на деталях этой функции, она использует техники и инстркуции, которые мы еще не проходили.
Ее суть в том, что она конвертирует позицию спрайта в адрес на тайлмапе.
Таким образом, мы можем проверить тайл мяча на участие в коллизии.

:::

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:get-tile}}
{{#include ../../unbricked/collision/main.asm:get-tile}}
```

Следующая функция называется `IsWallTile`, она будет содержать список тайлов, от которых мяч может отскочить.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:is-wall-tile}}
{{#include ../../unbricked/collision/main.asm:is-wall-tile}}
```

Эта функция может выглядить странно на первый взгляд.
Вместо записи результата в *регистр*, например `a`, мы возвращаем его в качестве [*флага*](../part1/operations.md#flags): `Z`!
Если тайл совпадает, функция находит стену, завершается, устанавливает флаг `Z`.

## Собираем воедино

Теперь пора использовать данные функции для реализации коллизий.
Добавьте следующий код после обновления позиции мяча:
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:first-tile-collision}}
{{#include ../../unbricked/collision/main.asm:first-tile-collision}}
```

Вы, наверное, заметили инструкции `sub` перед вызовом `GetTileByPixel`.
Помните, что координаты (0, 0) находятся за пределами экрана, а координаты объекта сдвинуты на 8 по X и на 16 по Y? Вычитание как раз позволяет нам избавиться от этого сдвига.

Однако вы, наверное, заметили, что мы вычитаем дополнительный пиксель из координаты Y.
Это потому, что наш код проверяет тайл _над_ мячом. Если бы мы не прибавили 1, мы бы получили координату тайла, на котором мяч _находится в данный момент_.
Теперь осталось добавить коллизии снизу, слева и справа, чтобы реагировать на все виды стен (а не только на потолок). Что ж, за работу!

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:tile-collision}}
{{#include ../../unbricked/collision/main.asm:tile-collision}}
```

Теперь мячик отскакивает от стен!
Перед тем, как закончить эту главу, давайте реализуем коллизию ракетки и мяча.

## Коллизия ракетки

Коллизия ракетки гораздо проще коллизии с тайлмапом. Всё, что нам нужно - это сравнить координаты объектов.
Для этих операций нам понадобится [флаг *carry* (переноса)](../part1/operations.md#flags).
Флаг переноса обозначается прописной буквой `C`, аналогично нулевому флагу (`Z`). Главное - не перепутайте флаг `C` с регистром `c`!

:::tip Напоминание о сравнениях

Как в случае с флагом `Z`, вы можете использовать `carry`, чтобы совершать условные переходы.
Однако, когда мы используем `Z`, мы проверяем числа на равенство, в случае с `C` же мы проверяем, какое из чисел наибольшее.
Например, `cp a, b` устанавливает флаг `C`, если `a < b`, и снимает его, если `a >= b`.
(Если вам нужно проверить, что `a <= b` или `a > b`, вы можете использовать `Z` и `C` одновременно с двумя инструкциями `jp`.)

:::

Armed with this knowledge, let's work through the paddle bounce code:
Вооружившись этими знаниями, давайте попробуем написать коллизию для мяча и ракетки.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:paddle-bounce}}
{{#include ../../unbricked/collision/main.asm:paddle-bounce}}
```

Проверка позиции по Y очень простая, ибо наша ракетка плоская.
Однако проверка по X немного сложнее: она содержит 2 условия, которые увеличивают область коллизии мяча и ракетки
Во-первых, нам надо прибавить 16 к позиции мяча; если мяч находится в 16 пикселях от ракетки, он отскакивать не должен. Тем самым, мы увеличили область коллизи на 16 пикселей.
Затем мы отнимаем эти 16 пикселей чтобы получить снова координаты мяча. Для проверки левого края мы отнимаем 8 пикселей от позиции мяча; если мяч находится в 8 пикселях от ракетки, он отскакивать не должен. Тем самым, мы увеличили область коллизии еще на 8 пикселей.

<svg viewBox="-10 -10 860 520">
	<style>
		text { text-anchor: middle; fill: var(--fg); font-size: 20px; }
		.left { text-anchor: start; }
		.right { text-anchor: end; }
		.grid { stroke: var(--fg); opacity: 0.7; }
		.ball { stroke: teal; }
		.paddle { stroke: orange; }
		.excl { stroke: red; } text.excl { stroke: initial; fill: red; font-family: "Source Code Pro", Consolas, "Ubuntu Mono", Menlo, "DejaVu Sans Mono", monospace, monospace !important; }
		/* Overlays */
		rect, polyline { opacity: 0.5; stroke-width: 3; }
		/* Arrow */
		polygon { stroke: inherit; fill: var(--bg); }
		use + line { stroke-dasharray: 0 32 999; stroke-width: 2; }
	</style>
	<defs>
		<polygon id="arrow-head" points="0,0 -40,-16 -32,0 -40,16" stroke="context-stroke"/>
		<pattern id="ball-hatched" viewBox="0 0 4 4" width="8" height="8" patternUnits="userSpaceOnUse">
			<line x1="5" y1="-1" x2="-1" y2="5" class="ball"/>
			<line x1="5" y1="3" x2="3" y2="5" class="ball"/>
			<line x1="1" y1="-1" x2="-1" y2="1" class="ball"/>
		</pattern>
		<pattern id="paddle-hatched" viewBox="0 0 4 4" width="8" height="8" patternUnits="userSpaceOnUse">
			<line x1="5" y1="-1" x2="-1" y2="5" class="paddle"/>
			<line x1="5" y1="3" x2="3" y2="5" class="paddle"/>
			<line x1="1" y1="-1" x2="-1" y2="1" class="paddle"/>
		</pattern>
		<pattern id="excl-hatched" viewBox="0 0 4 4" width="8" height="8" patternUnits="userSpaceOnUse">
			<line x1="5" y1="-1" x2="-1" y2="5" class="excl"/>
			<line x1="5" y1="3" x2="3" y2="5" class="excl"/>
			<line x1="1" y1="-1" x2="-1" y2="1" class="excl"/>
		</pattern>
	</defs>
	<image x="128" y="0" width="256" height="256" href="../assets/part2/img/ball.png"/>
	<rect x="128" y="0" width="32" height="32" fill="url(#ball-hatched)"/>
	<image x="288" y="256" width="256" height="256" href="../assets/part2/img/paddle.png"/>
	<rect x="288" y="256" width="32" height="32" fill="url(#paddle-hatched)"/>
	<line class="grid" x1="-10" y1="0" x2="850" y2="0"/>
	<line class="grid" x1="-10" y1="32" x2="850" y2="32"/>
	<line class="grid" x1="-10" y1="64" x2="850" y2="64"/>
	<line class="grid" x1="-10" y1="96" x2="850" y2="96"/>
	<line class="grid" x1="-10" y1="128" x2="850" y2="128"/>
	<line class="grid" x1="-10" y1="160" x2="850" y2="160"/>
	<line class="grid" x1="-10" y1="192" x2="850" y2="192"/>
	<line class="grid" x1="-10" y1="224" x2="850" y2="224"/>
	<line class="grid" x1="-10" y1="256" x2="850" y2="256"/>
	<line class="grid" x1="-10" y1="288" x2="850" y2="288"/>
	<line class="grid" x1="-10" y1="320" x2="850" y2="320"/>
	<line class="grid" x1="-10" y1="352" x2="850" y2="352"/>
	<line class="grid" x1="0" y1="-20" x2="0" y2="351"/>
	<line class="grid" x1="32" y1="-20" x2="32" y2="351"/>
	<line class="grid" x1="64" y1="-20" x2="64" y2="351"/>
	<line class="grid" x1="96" y1="-20" x2="96" y2="351"/>
	<line class="grid" x1="128" y1="-20" x2="128" y2="351"/>
	<line class="grid" x1="160" y1="-20" x2="160" y2="351"/>
	<line class="grid" x1="192" y1="-20" x2="192" y2="351"/>
	<line class="grid" x1="224" y1="-20" x2="224" y2="351"/>
	<line class="grid" x1="256" y1="-20" x2="256" y2="351"/>
	<line class="grid" x1="288" y1="-20" x2="288" y2="351"/>
	<line class="grid" x1="320" y1="-20" x2="320" y2="351"/>
	<line class="grid" x1="352" y1="-20" x2="352" y2="351"/>
	<line class="grid" x1="384" y1="-20" x2="384" y2="351"/>
	<line class="grid" x1="416" y1="-20" x2="416" y2="351"/>
	<line class="grid" x1="448" y1="-20" x2="448" y2="351"/>
	<line class="grid" x1="480" y1="-20" x2="480" y2="351"/>
	<line class="grid" x1="512" y1="-20" x2="512" y2="351"/>
	<line class="grid" x1="544" y1="-20" x2="544" y2="351"/>
	<line class="grid" x1="576" y1="-20" x2="576" y2="351"/>
	<line class="grid" x1="608" y1="-20" x2="608" y2="351"/>
	<line class="grid" x1="640" y1="-20" x2="640" y2="351"/>
	<line class="grid" x1="672" y1="-20" x2="672" y2="351"/>
	<line class="grid" x1="704" y1="-20" x2="704" y2="351"/>
	<line class="grid" x1="736" y1="-20" x2="736" y2="351"/>
	<line class="grid" x1="768" y1="-20" x2="768" y2="351"/>
	<line class="grid" x1="800" y1="-20" x2="800" y2="351"/>
	<line class="grid" x1="832" y1="-20" x2="832" y2="351"/>
	<rect x="128" y="0" width="256" height="256" class="ball" style="fill: none;"/>
	<polyline points="288,352 288,256 544,256 544,352" class="paddle" style="fill: none;"/>
	<rect x="-15" y="-15" width="47" height="440" class="excl" fill="url(#excl-hatched)"/>
	<text x="40" y="430" class="excl left">jp c, DoNotBounce</text>
	<rect x="800" y="-15" width="52" height="510" class="excl" fill="url(#excl-hatched)"/>
	<text x="790" y="500" class="excl right">jp nc, DoNotBounce</text>
	<use href="#arrow-head" x="48" y="380" transform="rotate(-180,48,380)" class="paddle"/><line x1="48" y1="380" x2="304" y2="380" class="paddle"/>
	<text x="176" y="400">- 8</text>
	<use href="#arrow-head" x="304" y="450" class="paddle"/><line x1="304" y1="450" x2="48" y2="450" class="paddle"/>
	<use href="#arrow-head" x="816" y="450" class="paddle"/><line x1="816" y1="450" x2="304" y2="450" class="paddle"/>
	<text x="432" y="470">+ 8 + 16</text>
</svg>

:::tip Толщина ракетки

Вы наверное думаете, почему мы проверяем 16 пикселей справа, но только 8 пикселей слева.
Все дело в том, что в OAM сохраняются координаты левого верхнего угла объекта, поэтому центр нашей ракетки находится на 4 пикселя правее позиции в OAM.
Фактически, мы проверяем 12 пикселей справа и слева от центра ракетки.

Может показаться, что 12 пикселей - это много, но мы хотим немного облегчить задачу игроку. Вы, если хотите, можете сами поиграться со значениями.

:::

## BONUS: tweaking the bounce height

Вы, наверное, заметили, что мячик немного заходит за ракетку перед отскоком. Все потому что программа проверяет коллизию верхней части мяча с верхней частью ракетки. Попробуйте самостоятельно изменить алгоритм, чтобы мяч отскакивал, когда его низ соприкасается с верхом ракетки.

Подсказка: вы можете решить данную пробелму одной иструкцией!

<details><summary>Ответ:</summary>

```diff linenos,start={{#line_no_of "" ../../unbricked/collision/main.asm:paddle-bounce}}
	ld a, [_OAMRAM]
	ld b, a
	ld a, [_OAMRAM + 4]
+	add a, 6
	cp a, b
```

Либо вы можете добавить `sub a, 6` сразу после `ld a, [_OAMRAM]`.

В любом случае, попробуйте поиграться с числом `6`; посмотрите, что лучше подходит!

</details>
