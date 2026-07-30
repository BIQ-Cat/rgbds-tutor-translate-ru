# Hello World!

В этом уроке мы начнем с "ассемблирования" нашей первой программы.
Остальная часть этой главы будет посвящена объяснению того, как и почему все работает.

Обратите внимание на то, что нам нужно будет вводить много команд, поэтому откройте терминал.
Лучше всего будет создать новую папку (например, `mkdir gb_hello_world`).

Сохраните следующие файлы и поместите их в только что созданную папку:
- [`hello-world.asm`](../assets/hello-world.asm)
- [`hardware.inc`](https://raw.githubusercontent.com/gbdev/hardware.inc/v4.0/hardware.inc)

Затем перейдите в эту папку в терминале (`cd gb_hello_world`) и выполните следующие команды:

::tip CONVENTION

Чтобы уточнить, где начинается новая команда, я добавил `$` перед каждой, не вводите их!

:::

```console
$ rgbasm -o hello-world.o hello-world.asm
$ rgblink -o hello-world.gb hello-world.o
$ rgbfix -v -p 0xFF hello-world.gb
```

<style>
	.box.danger ol {
		list-style-type: symbols(fixed "👎" "👍" "👍");
	}
</style>

:::danger:‼️

Будьте осторожны с аргументами! Некоторые флаги, такие как `-o`, используют аргумент после них в качестве параметра:

1. `rgbasm -o hello-world.asm hello-world.o` не будет работать (и может исказить `hello-world.asm`!)
2. `rgbasm hello-world.asm -o hello-world.o` заработает

Если вам нужны пробелы в аргументе, вы должны заключить его в кавычки:

1. `rgbasm -o hello world.o hello world.asm` не будет работать
2. `rgbasm -o "hello world.o" "hello world.asm"` заработает

:::

Все должно выглядить так:
<script id="asciicast-weljUlcp1KC5GqS9jqV62dy5m" src="https://asciinema.celforyon.fr/a/weljUlcp1KC5GqS9jqV62dy5m.js" async></script>

(Если вы столкнулись с ошибкой, которую не можете устранить самостоятельно, не бойтесь [обращаться к нам](../index.md#feedback)!)

Поздравляю!
Вы только что собрали свой первый ROM файл для Game Boy!
Теперь нам просто нужно запустить его; откройте Emulicious, нажмите File → Open File и откройте `hello-world.gb`.

<video controls poster="../assets/vid/hello_world.poster.png">
	<source src="../assets/vid/hello_world.webm" type="video/webm">
	<source src="../assets/vid/hello_world.mp4" type="video/mp4">

	<img src="../assets/vid/hello_world.gif" alt="Video demonstration in Emulicious">
</video>

Вы также можете воспользоваться флэш-картриджем (я использую [EverDrive GB X5](https://krikzz.com/store/home/47-everdrive-gb.html), но есть много альтернатив), загрузите на него свой ROM файл и запустите его на реальной консоли!

![Picture of the Hello World running on a physical DMG](../assets/img/hello_dmg.jpg)

Что ж, теперь, когда у нас все работает, пришло время посмотреть за кулисы...
