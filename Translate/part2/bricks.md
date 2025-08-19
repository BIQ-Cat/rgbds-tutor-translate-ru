# Блоки

Наш мячик пока только прыгает из стороны в сторону. Сейчас мы научим его ломать блоки.

Прежде, чем начнём, давайте познакимся с созданием *констант*.
Мы уже пользовались до этого некоторыми константами, например `rLCDC` из файла `hardware.inc`, однако мы также можем создавать свои собственные для любых задач.
Давайте определим три константы в начале файла, в которые запишем ID тайлов, содержащих левую и правую части блока, а также ID пустого тайла.
```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/bricks/main.asm:constants}}
{{#include ../../unbricked/bricks/main.asm:constants}}
```

Константы, как мы говорили ранее, представляют из себя информацию, к которой мы обращаемся по имени.
Написание имени константы будет равносильно написанию числа, которое ей присвоено: `ld a, BRICK_LEFT` равноценно `ld a, $05`.
Первый вариант, как мы видим, для нас гораздо понятней.

## Разрушение блоков

Теперь мы напишем функцию, которая определяет и разрушает блоки.
Наши блоки состоят из двух тайлов, и при попадании в один из них нам потребуется удалить оба, чтобы убрать весь блок.
Если мы задеваем тайл с левой частью блока (которому соответствует константа `BRICK_LEFT`), нам будет нужно удалить этот тайл и тайл справа от него.
Если же мы задеваем тайл с правой частью, нам, соответственно, понадобится дополнительно удалить тот, что слева.

```rgbasm,linenos,start={{#line_no_of "" ../../unbricked/bricks/main.asm:check-for-brick}}
{{#include ../../unbricked/bricks/main.asm:check-for-brick}}
```

Просто вставьте `call CheckAndHandleBrick` во все проверки отскоков.
Убедитесь в том, что ничего не пропустили.
Вызов должен идти прямо перед модификацией направления мяча.

```diff,linenos,start={{#line_no_of "" ../../unbricked/bricks/main.asm:updated-bounce}}
BounceOnTop:
	; Remember to offset the OAM position!
	; (8, 16) in OAM coordinates is (0, 0) on the screen.
	ld a, [_OAMRAM + 4]
	sub a, 16 + 1
	ld c, a
	ld a, [_OAMRAM + 5]
	sub a, 8
	ld b, a
	call GetTileByPixel ; Returns tile address in hl
	ld a, [hl]
	call IsWallTile
	jp nz, BounceOnRight
+	call CheckAndHandleBrick
	ld a, 1
	ld [wBallMomentumY], a

BounceOnRight:
	ld a, [_OAMRAM + 4]
	sub a, 16
	ld c, a
	ld a, [_OAMRAM + 5]
	sub a, 8 - 1
	ld b, a
	call GetTileByPixel
	ld a, [hl]
	call IsWallTile
	jp nz, BounceOnLeft
+	call CheckAndHandleBrick
	ld a, -1
	ld [wBallMomentumX], a

BounceOnLeft:
	ld a, [_OAMRAM + 4]
	sub a, 16
	ld c, a
	ld a, [_OAMRAM + 5]
	sub a, 8 + 1
	ld b, a
	call GetTileByPixel
	ld a, [hl]
	call IsWallTile
	jp nz, BounceOnBottom
+	call CheckAndHandleBrick
	ld a, 1
	ld [wBallMomentumX], a

BounceOnBottom:
	ld a, [_OAMRAM + 4]
	sub a, 16 - 1
	ld c, a
	ld a, [_OAMRAM + 5]
	sub a, 8
	ld b, a
	call GetTileByPixel
	ld a, [hl]
	call IsWallTile
	jp nz, BounceDone
+	call CheckAndHandleBrick
	ld a, -1
	ld [wBallMomentumY], a
BounceDone:
```

На этом всё!
Довольно просто, не правда ли?
