# Двоичная и шестнадцатеричная системы счисления

Прежде чем мы поговорим о коде, необходимо немного ознакомиться с базовым материалом.
При программировании на низком уровне понимание _[двоичной системы счисления](https://en.wikipedia.org/wiki/Binary_number)_ и _[шестнадцатеричной системы счисления](https://en.wikipedia.org/wiki/Hexadecimal)_ является обязательным.
Поскольку вы, возможно, уже знаете об обеих из них, краткое изложение информации, относящейся к RGBDS, доступно в конце этого урока.

Итак, что такое двоичная система счисления?
Это другой способ представления чисел, в том, что называется _система с основанием 2_.
Мы привыкли считать в [системе с основанием 10](https://en.wikipedia.org/wiki/Decimal), таким образом, у нас есть 10 цифр: 0, 1, 2, 3, 4, 5, 6, 7, 8, и 9.
Вот как здесь работают цифры:

```
  42 =                       4 × 10   + 2
     =                       4 × 10^1 + 2 × 10^0
                                  ↑          ↑
     Мы умножаем каждую цифру числа на 10 возведенную в степень номера разряда (справа налево начиная с 0).

1024 = 1 × 1000 + 0 × 100  + 2 × 10   + 4
     = 1 × 10^3 + 0 × 10^2 + 2 × 10^1 + 4 × 10^0
       ↑          ↑          ↑          ↑
Здесь мы видим, как с помощью цифр делают числа!
```

::: tip:ℹ️

`^` это "возведение в степень". Напоминаю, что `X^N` - это умножение `X` на себя `N` раз, и если число возвести в 0 степень, получится 1.

:::

Десятичные цифры образуют уникальное _разложение_ чисел по степеням 10.
Но зачем останавливаться на степенях 10?
Вместо этого мы могли бы использовать другие системы, например с основанием 2.
(Почему именно двоичная система счисления, будет объяснено позже.)

Двоичная система счисления имеет основание 2, имеет только 2 цифры, названые _битами_: 0 и 1.
Таким образом, мы можем обобщить принцип, изложенный выше, и записать эти два числа аналогичным образом:

```
  42 =                                                    1 × 32  + 0 × 16  + 1 × 8   + 0 × 4   + 1 × 2   + 0
     =                                                    1 × 2^5 + 0 × 2^4 + 1 × 2^3 + 0 × 2^2 + 1 × 2^1 + 0 × 2^0
                                                              ↑         ↑         ↑         ↑         ↑         ↑
                                          Из-за того что мы считаем в base 2, мы видим в формуле 2 вместо 10!

1024 = 1 × 1024 + 0 × 512 + 0 × 256 + 0 × 128 + 0 × 64  + 0 × 32  + 0 × 16  + 0 × 8   + 0 × 4   + 0 × 2   + 0
     = 1 × 2^10 + 0 × 2^9 + 0 × 2^8 + 0 × 2^7 + 0 × 2^6 + 0 × 2^5 + 0 × 2^4 + 0 × 2^3 + 0 × 2^2 + 0 × 2^1 + 0 × 2^0
       ↑          ↑         ↑         ↑         ↑         ↑         ↑         ↑         ↑         ↑         ↑
```

Итак, применяя тот же принцип, мы можем сказать, что в двоичной системе 42 записывается как `101010`, а 1024 - как `10000000000`.
Поскольку вы не можете отличить 10 (десятичное число 10) от 2 (двоичное число 10), RGBDS распознает двоичные числа с префиксом в виде процента: 10 - это 10, а %10 - 2.

Хорошо, но почему именно система счисления с основанием 2?
Довольно удобно, что бит может быть только 0 или 1, которые легко представить как "есть электричество" или "нет электричества", пусто или заполнено и т.д.!
Если вы хотите дома создать однобитную память, просто возьмите коробку.
Если коробка пуста, значит она хранит 0; если же коробка содержит _что-то_, она хранит 1.
Таким образом, компьютеры в основном манипулируют двоичными числами, и это имеет _множество_ последствий, как мы увидим на протяжении всего этого урока.

## Шестнадцатеричная система счисления

Напомним, что десятичная система счисления непрактична для работы с компьютером, вместо этого полагаются на двоичные числа.
Хорошо, но с двоичным кодом действительно непрактично работать.
Возьмите %10000000000, оно же 2048; когда в десятичной системе требуется только 4 цифры, в двоичной вместо этого требуется 12!
И, вы заметили сколько тут нулей?
К счастью, шестнадцатеричная система здесь, чтобы спасти твой мозг от перегрузки!

Шестнадцатеричная система работает точно так же, как и любая другая система, но с 16 цифрами, называемыми _нибблами_: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E и F.

```
  42 =            2 × 16   + 10
     =            2 × 16^1 + A × 16^0

1024 = 4 × 256  + 0 × 16   + 0
     = 4 × 16^2 + 0 × 16^1 + 0 × 16^0
```

Как и в двоичном формате, мы будем использовать префикс для обозначения шестнадцатеричного числа, а именно `$`.
Так, 42 = $2A, и 1024 = $400.
Это _намного_ компактнее, чем двоичная система, и немного больше, чем десятичная; но кое-что делает шестнадцатеричную систему очень интересной, то, что один ниббл соответствует 4 битам!

| Нибблы | Биты  |
| :----: | :---: |
|   $0   | %0000 |
|   $1   | %0001 |
|   $2   | %0010 |
|   $3   | %0011 |
|   $4   | %0100 |
|   $5   | %0101 |
|   $6   | %0110 |
|   $7   | %0111 |
|   $8   | %1000 |
|   $9   | %1001 |
|   $A   | %1010 |
|   $B   | %1011 |
|   $C   | %1100 |
|   $D   | %1101 |
|   $E   | %1110 |
|   $F   | %1111 |

Это позволяет очень легко перемещаться между двоичной и шестнадцатеричной системами счисления, сохраняя при этом компактность.
Таким образом, шестнадцатеричная система используется намного чаще, чем двоичная.
И, не волнуйтесь, десятичную систему все еще можно использовать 😜

(Примечание: можно было бы указать, что восьмеричная система (система с основанием 8), также подойдет для этого; однако в первую очередь мы будем иметь дело с 8-битными юнитами, для которых шестнадцатеричная система работает намного лучше, чем восьмеричная. RGBDS поддерживает восьмеричную систему с префиксом `&`, но я еще не видел, чтобы оно использовалось.)

::: tip:💡

Если у вас возникли проблемы с преобразованием десятичной системы счисления в двоичную/шестнадцатеричную, проверьте, нет ли в вашем любимом калькуляторе режима "программист" или способа преобразования систем счисления.

:::

## Заключение

- В RGBDS шестнадцатеричный префикс является `$`, а двоичный префикс - `%`.
- Шестнадцатеричная система счисления может использоваться в качестве "компактной двоичной" системы счисления.
- Использование двоичной или шестнадцатеричной системы счисления полезно, когда важны отдельные биты; в другом случае десятичная система работает так же хорошо.
- Для тех случаев, когда цифры становятся слишком длинными, RGBASM допускает подчеркивание между цифрами (`123_465`, `%10_1010`, `$ DE_AD_BE_EF` и т.д.)
