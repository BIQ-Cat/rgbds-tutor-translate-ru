# Игра

На экране игрового состояния мы будем контролировать шатл, который будет уметь перемещаться по всем 4 направлениям и стрелять по инопланетянам. При уничтожении вражеского корабля игрок получает одно очко. 

![rgbds-shmup-gameplay.gif](../assets/part3/img/rgbds-shmup-gameplay.gif)

Игровое состояние - это ключевая часть проекта. Оно потребовало больше всего времени на создание, поэтому эту часть проекта мы разделили на несколько подстраниц. На каждой из них мы будем разбирать важные игровые концепции.

В игровом состоянии мы объявляем следующие переменные:

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:gameplay-data-variables}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:gameplay-data-variables}}
```

Для упрощения логики наш счёт будет занимать шесть байтов. Каждый байт представляет одну цифру в счёте.

## Инициализация игрового состояния:

Когда игра запускается, нам нужно сделать следующее:
- сбросить счёт игрока до нуля.
- сбосить количество жизней до трёх.
- инициализировать все игровые элементы (фон, игрока, пули и врагов)
- Разрешить STAT-прерывания для нашего интерфейса HUD. (о прерываниях в следующих уроках)
- Вывести счёт и количество жизней на интерфейс HUD.
- Установить позицию окна на 7,0 (о слое окна тоже в следующих уроках)
- Зажечь LCD со включенным слоем окна на $9C00

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:init-gameplay-state}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:init-gameplay-state}}
```

Логика инициализации игрока, врагов, пуль и фона будет объяснена на следующих уроках. Каждое состояние отвечает за включение LCD. Состоянию игрового процесса нужно использовать слой окна, поэтому мы должны его включить перед возвратом из функции.

## Обновление игрового состояния

Функция "UpdateGameplayState" не очень сложная. Большая часть логики была разделена между разными файлами, отвечающими за фон, игрока, врагов и пули.

Во время игрового процесса нам нужно делать следующее:
* Принимать пользовательский ввод
* Сбрасывать *Теневой* OAM (о ней чуть позже)
* Сбрасывать спрайт теневого OAM
* Обновлять наши игровые элементы (игрок, фон, враги, пули)
* Удалять ненужные спрайты 
* Завершать игру,если мы потеряли последнюю жизнь
* Внутри VBlank:
    * Переместить объекты из теневого в обычный OAM
    * Обновлять тайлмап.

Мы будем обрабатывать ввод пользователя как в прошлом уроке. Мы всегда сохраняем состояние кнопок Game boy в переменной "wLastKeys".

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-state-start}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-state-start}}
```

Далее нам надо сбросить наш теневой OAM и текущий адрес спрайта теневого OAM.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-oam}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-oam}}
```

Так как мы будем манипулировать большим количеством спрайтов на экране, мы не собираемся работать напрямую с OAM. Мы определим несколько "теневых" ("копий") спрайтов OAM, которые станут нашими новыми объектами. В конце игрового цикла мы копируем объекты из теневого OAM в обычный OAM.

Каждый объект использует случайный спрайт теневого OAM. Нам нужно следить за тем, какой теневой спрайт сейчас используется. Для этого мы создали 16-битный указатель "wLastOAMAddress", определенный в "src/main/utils/sprites.asm". Он указывает на следующий неактивный теневой спрайт.

Когда мы сбрасывает адрес текущего спрайта теневого OAM, мы просто устанавливаем адрес первого теневого спрайта в переменную "mLastOAMAddress".

> **ВАЖНО:** Мы также следим за тем, сколько было использовано теневых спрайтов. Внутри функции "ResetOAMSpriteAddress" мы будем сбрасывать этот счетчик.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/utils/sprites-utils.asm:reset-oam-sprite-address}}
{{#include ../../galactic-armada/src/main/utils/sprites-utils.asm:reset-oam-sprite-address}}
```

Затем мы обновим наши игровые элементы:

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-elements}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-elements}}
```

На данный момент большая часть работы на данной итерации игрового цикла сделана. Мы очистим все ненужные объекты. Это очень важно, ведь число активных спрайтов меняется от кадра к кадру. Если часть ненужных спрайтов осталась на экране, они будут выглядеть странно и/или вводить в заблуждение игрока.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-clear-sprites}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-clear-sprites}}
```

Функция очистки ненужных объектов перемещает объект за экран на всех спрайтах теневого OAM, начиная с `wLastOAMAddress`.

### Конец игрового цикла

Здесь мы должны будем проверять, должна ли игра продолжиться. Когда фаза VBlank начинается, мы проверяем, потерял ли игрок все жизни. Если да - завершаем игровое состояние, нет - продолжаем. Мы заканчиваем игру также, как и начали: мы обновим значение 'wGameState' и выполни переход к "NextGameState".

Если игрок не потерял все свои жизни, мы скопируем теневые спрайты в настоящий OAM и прокрутим фон.

```rgbasm,linenos,start={{#line_no_of "" ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-end-update}}
{{#include ../../galactic-armada/src/main/states/gameplay/gameplay-state.asm:update-gameplay-end-update}}
```
