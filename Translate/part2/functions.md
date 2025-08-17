# Функции

Несмотря на малый объем написанного нами кода, уже можно выделить повторяющиеся блоки.
Чтобы исправить это, воспользуемся функциями.

Наприме, мы используем один и тот же алгоритм для копирования в памяти в трех местах.
Давайте напишем функцию `Memcpy` (по аналогии с [функцией `memcpy` в языке С](https://man7.org/linux/man-pages/man3/memcpy.3.html)). Разместим ее сразу после `jp Main`:

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/functions/main.asm:memcpy}}
{{#include ../../unbricked/functions/main.asm:memcpy}}
```

Первое, что вы должны заметить - новая инструкция `ret`.
Она _возвращает_ интерпретатор к тому месту, где была _вызвана_ функция.
Во многих языках мы имеем определенный конец у функции: в C или Rust у нас есть `}`, в Pascal или Lua - `end` и так далее.
Однако, наша функция завершается тогда и только тогда, когда мы используем инструкцию `ret`.
Иначе результат будет непредсказуем.

Взгляните на комментарии над функцией, объясняющие, какие регистры отвечают за аргументы.
В assembly нет привычных нам _параметров_, поэтому эти комментарии очень важны для описания работы функции.
Мы увидим больше подобных комментариев далее.

Как мы уже говорили, есть три места, где можно использовать функцию `Memcpy`.
Найдите все эти места и замените их на вызов `Memcpy`; для этого воспользуйтесь инструкцией `call`.
Регистры выполняют роль параметров, поэтому мы оставим все как есть.

<div class="table-wrapper"><table><thead><tr><th>До</th><th>После</th></tr></thead><tbody><tr><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_tiles}}
{{#include ../../unbricked/getting-started/main.asm:copy_tiles}}
```

</td><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/functions/main.asm:copy_tiles}}
{{#include ../../unbricked/functions/main.asm:copy_tiles}}
```

</td></tr><tr><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/getting-started/main.asm:copy_map}}
{{#include ../../unbricked/getting-started/main.asm:copy_map}}
```

</td><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/functions/main.asm:copy_map}}
{{#include ../../unbricked/functions/main.asm:copy_map}}
```

</td></tr><tr><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/objects/main.asm:copy-paddle}}
{{#include ../../unbricked/objects/main.asm:copy-paddle}}
```

</td><td>

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/functions/main.asm:copy_paddle}}
{{#include ../../unbricked/functions/main.asm:copy_paddle}}
```

</td></tr></tbody></table></div>

В следующей главе мы напишем еще одну функцию, на этот раз для получения ввода от игрока.
