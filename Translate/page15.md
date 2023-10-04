# Тайлы

::: tip:💭

"Тайлы" назывались по-разному в документации прошлого.
Обычно их называли "паттернами" или "characters", последнее привело к появлению аббревиатуры "CHR", которая иногда используется для обозначения тайлов.

Например, в NES данные тайлов обычно предоставляются картриджем либо в [CHR ROM, либо в CHR RAM](http://wiki.nesdev.com/w/index.php/CHR_ROM_vs._CHR_RAM).
Термин "CHR" обычно не используется в Game Boy, поскольку обмен информацией между сообществами приводит к "утечке" терминов, поэтому некоторые называют область видеопамяти, где хранятся тайлы, например, "CHR RAM" или "CHR VRAM".

Как и в случае со всем подобным жаргоном, значение которого может зависеть от того, с кем вы разговариваете, я буду придерживаться "тайлов" во всем этом руководстве для согласованности, поскольку сейчас это стандартный термин в сообществе разработчиков GB.

:::

Что ж, копирование этих данных вслепую - это прекрасно, но почему именно эти являются "графическими"?

<figure>
  <img src="../assets/img/ah_yes_pixels.png" alt="Screenshot of some tile definitions in the code">
  <figcaption><q>А, точно, пиксели.</q></figcaption>
</figure>

Давайте разбираться!

## Рука помощи

Теперь выяснение формата с помощью одного только объяснения будет очень запутанным; но, к счастью, BGB помог нам разобраться благодаря своему *VRAM viewer*у.
Вы можете открыть его либо выбрав "Window", затем "VRAM viewer", либо нажав клавишу F5, когда открыт отладчик.
Он также находится в разделе "Other" в контекстном меню экрана.

![Screenshot of the VRAM viewer's "Tiles" tab](../assets/img/vram_viewer.png)

По умолчанию он должен открываться на вкладке "Tiles", как на изображении, но если нет, пожалуйста, выберите её.
Мы перейдем к другим вкладкам в свое время.
На этом изображении показаны тайлы, присутствующие в видеопамяти Game Boy (или "<abbr title="Video RAM">VRAM</abbr>").

::: tip:🤔

Я призываю вас поэкспериментировать с VRAM viewerом, наведите курсор на объекты, установите и снимите флажки, посмотрите сами, что к чему. На любые вопросы, которые у вас могут возникнуть, мы ответим в свое время, не волнуйтесь! И если то, что вы видите позже, не соответствует моим скриншотам, убедитесь, что флажки соответствуют моим.

:::

Не обращайте внимания на надпись "Nintendo" в левом верхнем углу; мы сами ее туда не ставили, и позже мы увидим, почему она там есть.
Мы также будем игнорировать правую половину тайловой сетки, поскольку это эксклюзивно для Game Boy Color: мы доберемся туда, когда доберемся туда.

## Короткая грунтовка

Возможно, вы слышали о тайлах раньше, тем более что они были действительно популярны в 8-битных и 16-битных системах.
Это не случайно: тайлы очень полезны.
Вместо того, чтобы сохранять каждый пиксель на экране (144 × 160 пикселей × 2 бита / пиксель = 46080 бит = 5760 байт, по сравнению с 8192 байтами VRAMа консоли), пиксели группируются в тайлы, а затем тайлы собираются различными способами для получения конечного изображения.

В частности, тайлы можно использовать повторно очень легко и практически бесплатно, экономя много памяти!
Кроме того, одновременное управление целыми фрагментами намного дешевле, чем управление отдельными пикселями, так что это также экономит время обработки.

Понятие "тайл" очень обобщенное, но в Game Boy тайлы *всегда* 8 на 8 пикселей.
Часто аппаратные тайлы группируются, чтобы манипулировать ими как более крупными тайлами (часто 16 × 16); чтобы избежать путаницы, они называются **мета-тайлами**.

### "bpp"?

Возможно, вам интересно, откуда взялся этот "2 бита на пиксель" ранее...
Это то, что называется "разрядностью".

Видите ли, цвета *не* хранятся в самих тайлах!
Вместо этого они работают как книжка—раскраска: сам тайл содержит 8 на 8 *индексов*, а не цветов; вы даете аппаратному обеспечению тайл и набор цветов — **палитру** - и оно раскрашивает их!
(Это также объясняет, почему в то время была очень распространена замена цветов: вы могли создавать различные варианты врагов, сохраняя крошечные палитры вместо большой различной графики.)

В любом случае, как бы то ни было, палитры Game Boy имеют размер в 4 цвета.[^pal_size]
Это означает, что индексы в этих палитрах, хранящиеся в тайлах, могут быть представлены только *двумя битами*!
Это называется "2 бита на пиксель", отмечено "2bpp".

Имея это в виду, мы готовы объяснить, как эти байты превращаются в пиксели!

## Кодирование

Как я уже объяснял, каждый пиксель занимает 2 бита.
Поскольку в байте содержится 8 бит, вы могли бы ожидать, что каждый байт будет содержать 4 пикселя... и вы не были бы ни полностью правы, ни полностью неправы.
Смотрите, каждая строка из 8 пикселей хранится в 2 байтах, но ни один из этих байтов не содержит информации для 4 пикселей.
(Представьте себе, что это банкнота в 100 рублей, разорванная пополам: ни одна из половинок ничего не стоит, но вся купюра стоит, ну, 100 рублей.)

Для каждого пикселя наименее значимый бит его индекса хранится в первом байте, а наиболее значимый бит хранится во втором байте.
Поскольку каждый байт представляет собой набор одного из битов для каждого пикселя, он называется **битовой плоскостью**.

Крайний левый пиксель сохраняется в крайнем левом бите обоих байтов, пиксель справа от него - во втором крайнем левом бите и так далее.
Первая пара байтов хранит самую верхнюю строку, второй байт - строку под ней и так далее.

Вот более наглядная демонстрация; обратите внимание, что она предназначена для SNES, но ее формат 2BPP такой же, как у Game Boy.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/txkHN6izK2Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Поначалу такое кодирование может показаться немного странным, и так оно и есть; оно сделано для того, чтобы аппаратному обеспечению было удобнее декодировать, сохраняя схему простой и маломощной.
Это даже делает возможным несколько крутых трюков, как мы увидим (намного) позже!

Вы можете прочитать больше о кодировке [здесь](https://gbdev.io/pandocs/Tile_Data.html) и [на сайте ShantyTown](https://www.huderlem.com/demos/gameboy2bpp.html).

В следующем уроке мы увидим, как наносятся цвета!

---

[^pal_size]:
Другие консоли могут иметь различную разрядность; например, SNES имеет 2bpp, 4bpp и 8bpp в зависимости от графического режима и нескольких других параметров.