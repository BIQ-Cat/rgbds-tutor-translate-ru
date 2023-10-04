# Первоначальная настройка

Во-первых, мы должны настроить нашу среду разработки. 
Нам понадобится:

1. Cреда разработки POSIX
2. [RGBDS](https://rgbds.gbdev.io/install) v0.5.1 (хотя v0.5.0 должна быть совместима)
3. GNU Make (предпочтительно последнюю версию)
4. Редактор кода
5. Эмулятор

::: tip:❓😕

Следующие инструкции по установке предоставляются на основе принципа, что "все сработает с первого раза", но они могут быть устаревшими или по какой-то причине не работать для вас. 
Не волнуйтесь, мы здесь, чтобы помочь: [спросите в GBDev](../help-feedback.md), и мы поможем вам с установкой всего!

:::

## Инструменты

### Linux & macOS

Хорошие новости: вы уже выполнили шаг 1!
Вам просто нужно установить [RGBDS](https://rgbds.gbdev.io/install), и, возможно, обновить GNU Make.

#### macOS

На момент написания этой статьи, macOS (до 11.0, текущей последней версии) поставляется с очень устаревшей версией GNU Make.
Вы можете проверить это, открыв терминал и введя `make --version`, в котором, помимо прочего, должно быть указано "GNU Make" и дата.

Если ваш Make слишком старый, вы можете обновить его с помощью [Homebrew](https://brew.sh) формулы [`make`](https://formulae.brew.sh/formula/make#default).
На момент написания этой статьи вам должны вывести предупреждение о том, что обновленный Make был установлен как `gmake`; вы можете либо следовать предложению и использовать его в качестве `make` по умолчанию, либо использовать `gmake` вместо `make` в этом руководстве.

#### Linux

После установки RGBDS откройте терминал и введите `make --version`, чтобы проверить вашу версию Make.

Если `make` не может быть найден, возможно вам потребуется установить `build-essentials` вашего дистрибутива.

### Windows

Печальная правда заключается в том, что Windows - ужасная ОС для разработки; однако вы можете установить среды, которые решают большинство проблем.

В Windows 10 лучше всего использовать [WSL](https://docs.microsoft.com/en-us/windows/wsl), что позволяет запускать дистрибутив Linux в Windows.
Установите WSL 1 или WSL 2, потом дистрибутив по вашему выбору, а затем снова выполните эти действия, но для установленного вами дистрибутива Linux.

Если WSL не подходит, вы можете использовать [MSYS2](https://www.msys2.org) или [Cygwin](https://www.cygwin.com) вместо него; затем ознакомьтесь с [инструкциями по установке Windows RGBDS](https://rgbds.gbdev.io/install).
Насколько мне известно, оба они предоставляют достаточно актуальную версию GNU Make.

::: tip

Если вы программировали для других консолей, таких как GBA, проверьте, не установлен ли MSYS2 на вашем компьютере.
Это связано с тем, что devkitPro, популярный пакет домашней разработки, включает в себя MSYS2.

::: 

## Редактор кода

Подойдет любой редактор кода; лично я использую [Sublime Text](https://www.sublimetext.com) с его [синтаксическим пакетом RGBDS](https://packagecontrol.io/packages/RGBDS); однако вы можете использовать любой текстовый редактор, включая Блокнот, если у вас хватит ума.
В GBDev есть [раздел о пакетах подсветки синтаксиса](https://gbdev.io/resources#syntax-highlighting-packages), посмотрите там, поддерживает ли ваш любимый редактор RGBDS.

## Эмулятор

Использование эмулятора для игры в игры - это одно, а использование его для программирования игр - совсем другое.
Два аспекта, которые должен выполнять эмулятор, чтобы обеспечить приятный опыт программирования, заключаются в следующем:
- **Средства отладки**:
  Когда ваш код выходит из строя на реальной консоли, очень трудно понять почему и как.
  Нет никакого консольного вывода, никакого способа провести `отладку` программы, ничего.
  Однако эмулятор может предоставить средства отладки, позволяющие контролировать выполнение, проверять память и т.д.
  Это жизненно важно, если вы хотите, чтобы разработка под Game Boy была *веселой*, поверьте мне!
- **Точность**:
  Точность означает "насколько что-то соответствует оригинальной консоли".
  Использование плохого эмулятора для игры в игры может сработать, но использование его для *разработки* игры может привести к случайной несовместимости вашей игры с реальной консолью.
  Для получения дополнительной информации прочтите [эту статью об Ars Technica](https://arstechnica.com/?post_type=post&p=44524) (особенно раздел <q>Эмулятор для каждой игры</q> вверху 2 страницы).
  Вы можете сравнить точность эмуляторов [тут](https://daid.github.io/GBEmulatorShootout/).

Эмулятор, который я буду использовать для этого руководства, - это [Emulicious](https://emulicious.net/).
Пользователи всех ОС могут запустить этот эмулятор, установив Java runtime.
Также, доступны другие эмуляторы с поддержкой отладки, например [Mesen2](https://www.mesen.ca/), [BGB](https://bgb.bircd.org) (только на Windows/с помощью Wine), [SameBoy](https://sameboy.github.io) (GUI только под macOS); у них похожие возможности, однако имеют разные интерфейсы