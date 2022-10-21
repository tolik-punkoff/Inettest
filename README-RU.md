# Скрипт для тестирования сетевых соединений.

## Что умеет

+ Последовательно проводить два вида тестов: ping и получение страницы (или файла) с web-сервера

+ Оповещать пользователя об ошибках или пройденных тестах, с цветным текстом.

+ Выдавать звуковые сигналы об успешном завершении теста, или об ошибке на PC-speaker.

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-main.png?raw=true)

Для звуковых оповещений должен быть [настроен PC-speaker и установлена утилита beep](https://tolik-punkoff.com/2018/01/14/pc-speaker-v-linux-ili-kak-sdelat-beep-iz-konsoli-vstroennym-dinamikom-pk/) ([копия](https://lj.rossia.org/users/hex_laden/380240.html)). 

Последовательность операций задается в конфигурационном файле.

## Конфигурационный файл

Конфигурационный файл состоит из строк следующего формата:

`операция|адрес|отображаемое описание|прервать/игнорировать`

Пока поддерживаются только две операции:

`ping` - для ping'а адреса

`getp` - для получения страницы или файла с WEB-сервера (с помощью wget)

адрес - адрес, WWW для `getp`, IP или WWW для `ping`.

`отображаемое описание` - описание операции, текст, который будет выводиться на консоль.

`прервать/игнорировать` - при ошибке операции, если указано ключевое слово `break`, скрипт останавливает работу и выводит сообщение о том, что в ходе тестирования произошли ошибки. Если указать иное значение, например, `skip`, то скрипт продолжит производить тесты, пока не закончится конфигурационный файл, или пока не будет следующий сбой в тесте, где указана опция `break`.

Названия операций и `break` нечувстивительны к регистру (т.е. можно написать `Ping`, `ping` или `Getp`, `GeTP` или `BreaK`, `breaK`).

Первые два поля (операция и адрес) являются обязательными. Если они не будут указаны, скрипт будет прерван на строке с ошибкой:

`...`

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[ДАННЫЕ УДАЛЕНЫ]|Provider IP|break`

`getp`

`...`

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-config-error.png?raw=true)

Если не будет указано поле описания, то оно будет по умолчанию установлено в значение No desription:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-no-desription.png?raw=true)

Если последнее поле не будет заполнено, то оно принимает значение break.

Если в первом поле будет неверно указанный код операции, то скрипт его проигнорирует, выдав соответствующее сообщение, продолжит выполнять другие операции, но завершится с ошибкой:

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[ДАННЫЕ УДАЛЕНЫ]|Provider IP|break`

`getp|[ДАННЫЕ УДАЛЕНЫ]|Provider page`

`1234|[ДАННЫЕ УДАЛЕНЫ]|VPN Server IP|break`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-unknow-operation.png?raw=true)

Для разделения полей используется символ `|`

В конфигурационном файле можно использовать комментарии, начинающиеся с символа `#`, все, что находится после этого символа - игнорируется. Пустые строки также игнорируются.

## Пример конфигурационного файла (без данных)

`#inettest.cfg example`

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|x.x.x.x|Provider IP|break #Change x.x.x.x to your provider ip`

`getp|myprovider.net|Provider site page|skip #change myprovider.net to real site your provider`

`ping|x.x.x.x|VPN Server IP|break #Change x.x.x.x to real VPN provider IP`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

## Переменные скрипта

Скрипт не имеет параметров командной строки, основные настройки осуществляются через переменные самого скрипта:

`CONFIG` - путь к конфигурационному файлу, например `CONFIG="./inettest.cfg"`. Если конфигурационный файл не будет найден, скрипт выдаст ошибку:

`CRITICAL ERROR: Config file ./inettest.cfg not exist!` 

`NOCOLOR` - если значение равно `0`, включить вывод цветного текста на консоль, если `1` - отключить. По умолчанию `0`

`NOCOLOR=1`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-no-color.png?raw=true)

`NOSOUNDP` - включение ( по умолчанию `0`) или отключение (`1`) звука в процессе тестов. Звук выдается после каждого отдельного теста.

`NOSOUNDF` - аналогично предыдущей переменной, только звук звучит после окончания всех тестов или их прерывания.

`NOADDR` - `0`, включить тестируемый адрес в вывод скрипта, `1` - не включать.

`NOADDR=0`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-noaddr-0.png?raw=true)

`NOADDR=1`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-main.png?raw=true)

`PACKETS` - количество пакетов для команды `ping` (по умолчанию `PACKETS=3`)

`TIMEOUT` - тайм-аут для получения страницы или файла (в скрипте делается с помощью `wget`, по умолчанию `TIMEOUT=5`)

## Тест при ошибке сети

Конфиг:

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[ДАННЫЕ УДАЛЕНЫ]|Provider IP|skip`

`getp|[ДАННЫЕ УДАЛЕНЫ]Provider page|skip`

`ping|[ДАННЫЕ УДАЛЕНЫ]|VPN Server IP|break`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

Результат:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-network-errors.png?raw=true)

## Коды ошибок

`ping`:

1 — No reply (не один из пакетов до пингуемого адреса не дошел)

2 — Other error (другая ошибка, в большинстве случаев — «сеть недоступна»).

`getp` (`wget`):

1 — Иная / общая ошибка (generic error code)

2 — Ошибка в параметрах командной строки или файлах конфигурации (.wgetrc или .netrc)

3 — Ошибка файлового ввода/вывода (I/O error)

4 — Ошибка сети (например, при обрыве связи)

5 — Ошибка SSL

6 — Ошибка идентификации (неправильное имя пользователя или пароль)

7 — Ошибка протокола

8 — Ошибка сервера (например, нужный файл на сервере не найден, ошибка 404)

## Коды возврата скрипта

0 - Ошибок в ходе тестов не произошло.

1 - Произошла хотя бы одна ошибка.