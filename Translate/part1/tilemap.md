# Tilemap

::: tip:🧐

Некоторые пишут "tile map", некоторые "tilemap".
Я буду использовать последний вариант, я придерживаюсь его в коде (`Tilemap`, а не `TileMap`), а также позже, когда мы поговорим о картах атрибутов ("attrmap" и `Attrmap` вместо `AttrMap`).

:::

Мы *почти* закончили.
Мы видели, что графика в Game Boy состоит из "тайлов" размером 8 × 8, и мы видели, как отрисовываются цвета.

Но мы еще не видели, как эти тайлы складываются в цельную картинку!

Тайлы - это, по сути, сетка из пикселей; ну, Tilemap - это, по сути, сетка из тайлов!
Чтобы обеспечить дешевое повторное использование, тайлы не хранятся непосредственно в tilemap; вместо этого на тайлы ссылается *ID*, который вы можете увидеть в VRAM viewer'е Emulicious.

<figure>
  <img src="../assets/img/tile_id.png" alt="Screenshot highlighting where a tile's ID can be seen">
  <figcaption>
    ID отображается в шестнадцатеричном формате без префикса, так что это номер тайла $10, он же 16.
    Как вы, возможно, заметили, тайлы отображаются строками по 16, поэтому их легче найти по шестнадцатеричному ID.
    Изящно!
  </figcaption>
</figure>

ID тайлов - это числа, как и все, с чем имеют дело компьютеры.
ID хранятся в байтах, поэтому существует 256 возможных ID тайлов.
Однако проницательный читатель наверняка заметил, что всего существует 384 тайла[^tile_blocks]!
В силу [принципа Дирихле](https://en.wikipedia.org/wiki/Pigeonhole_principle) это означает, что некоторые ID ссылаются на несколько тайлов одновременно.

Действительно, Emulicious сообщает, что первые 128 тайлов имеют те же ID, что и последние 128.
Существует механизм для выбора того, ссылаются ли ID 0-127 на первые или последние 128 тайлов, но для простоты мы пока не будем обращать на это внимания, поэтому, пожалуйста, пока игнорируйте первые (самые верхние) 128 тайлов.

Теперь, пожалуйста, обратите свое внимание на вкладку "Tilemap viewer", изображенную ниже.

![Screenshot of the tilemap viewer](../assets/img/tilemap_viewer.png)

::: tip

Вы можете заметить, что показанное изображение больше, чем то, что отображается на экране.
В данный момент на экране отображается только часть tilemap, очерченная более толстой рамкой в Tilemap viewer'е.
Мы объясним это более подробно в 2 части.

:::

Здесь мы сможем увидеть возможности повторного использования тайла в полную силу.
Если что, вот тайлы, которые наш Hello World загружает в VRAM:

![Enlarged view of the tiles loaded in VRAM](../assets/img/hello_world_tiles.png)

Вы можете видеть, что мы загрузили только один "пустой" тайл ($00, первый он же. верхний левый), но его можно переиспользовать, чтобы покрыть весь фон без каких-либо дополнительных затрат!

Переиспользование может быть более "хирым", тайл $01 используется для верхнего левого угла H, E, L, L и W (красные линии на рисунке ниже)!
R, L и D также имеют общий верхний левый тайл ($2D, синие линии на рисунке); и так далее.
Вы можете подтвердить это, наведя курсор на тайлы во вкладке BG map, которая показывает ID тайла в этой позиции.

<figure>
  <img src="../assets/img/hello_world_mappings.svg" alt="Diagram of some tile mappings">
  <figcaption>
    Вот несколько примеров повторного использования тайла. Не все линии, однако, проиллюстрированы для сохранения порядка.
  </figcaption>
</figure>

В целом, мы можем предположить, что отображение графики в Game Boy состоит из загрузки "шаблонов" (тайлов), а затем указания консоли, какой тайл отображать для каждого заданного местоположения.

---

[^tile_blocks]:
Еще более проницательный читатель, должно быть, заметил, что 384 = 3 × 128.
Таким образом, тайлы часто концептуально группируются в три "блока" по 128 тайлов в каждом, которые Emulicious показывает разделенными более толстыми горизонтальными линиями.