# homework12

Задание 1 Запустить nginx на порту 4881 тремя способами. Файл - task1.txt

Способ 1:
Строка 4 - проверяем статус сервера, видим статус failed
Строка 8 - видим сообщение об ошибке доступа.
Строка 14 - находим в логе сообщение об ошибке.
Строка 16 - передаем на вход утилиты audit2why сообщение из лога.
Строка 25 - видим предложенный утилитой вариант решения проблемы.
Строка 26 - применяем его: setsebool -P nis_enabled on (ключ -P сохраняет настройку параметра)
Строка 27 - перезапускаем сервис nginx.
Строка 31 - видим, что сервис запустился.
Строка 46 - возвращаем настройку nis_enabled в исходное состояние для проверки следующего способа

Способ 2:
Строка 51 - проверим какие порты входят в имеющийся selinux тип для http траффика (semanage port -l | grep http)
Строки 52-56 - видим, что требуемого порта 4881 нет в списке разрешенных
Строка 57 добавляем порт 4881 в тип http_port_t (semanage port -a -t http_port_t -p tcp 4881)
Строка 58 - проверяем список разрешенных портов
Строка 59 - видим, что требуемый порт появился в списке разрешенных
Строка 61 - рестартуем сервис nginx
Строка 65 - видим сервис запущен.
Строка 78 - удаляем порт 4881 из списка разрешенных для проверки следещего способа (semanage port -d -t http_port_t -p tcp 4881)

Способ 3:
Строка 84 - ищем в логе сообщения об ошибках nginx и передаем строки с сообщениями на вход утилиты audit2allow (grep nginx /var/log/audit/audit.log  | audit2allow -M nginx) (audit2allow -M генерирует модуль selinux)
Строка 88 - видим предложенный утилитой способ решения проблемы - загрузка дополнительного модуля selinux
Строка 90 - Загружаем модуль semodule -i nginx.pp
Строка 91 - запускаем сервис nginx
Строка 95 - видим сервис запущен.
Строки 109, 110 - убеждаемся, что дополнительный модуль загружен (semodule -l | grep nginx)

Задание 2 Выяснить причину невозможности обновить зону DNS при включенном selinux, устранить ее. Файл task2_client.txt вывод с консоли ВМ клиента, файл task2_ns01.txt вывод с консоли ВМ сервера DNS

task2_client.txt Строка 24 - пытаемся обновить зону ddns.lab
task2_client.txt Строка 29 - видим, что обновить не удается
task2_client.txt Строка 34 - пытаемся в логе найти сообщение об ошибке selinux и с помощью утилиты audit2why понять, что можно предпринять. (cat /var/log/audit/audit.log | audit2why)
task2_client.txt Строка 35 - вывод пустой, в логе не нашлось ничего интересного по ошибкам selinux
Идем проверять сервер
task2_ns01.txt Строка 3 - ищем в логе сообщения по ошибкам selinux и с помощью утилиты audit2why понять, что можно предпринять. (cat /var/log/audit/audit.log | audit2why)
task2_ns01.txt Строка 4 - видим, что ошибка найдена. Смотрим внимательно и видим, что ошибка в контексте безопасности вместо типа named_t используется etc_t
task2_ns01.txt Строка 9 - видим предложенный утилитой вариант решения проблемы, генерация модуля утилитой audit2allow
task2_ns01.txt Строка 11 - проверим какой контекст безопасности используется в каталоге конфигурации сервера (ls -laZ /etc/named), ключ Z отображает атрибуты контекста безопасности
task2_ns01.txt Строки 12-18 - в выводе видим, что действительно тип etc_t
task2_ns01.txt Строка 19 - посмотрим какой контекст должен соотвествовать каталогу конфигурации сервера semanage fcontext -l | grep named
task2_ns01.txt Строка 21 - видим, что правильный тип контекста безопасности named_zone_t
Принимаем решение править тип контекста безопасности для /etc/named с etc_t на named_zone_t. Вариант с генерацией модуля тоже рабочий, но в данном случае это будет скорее workaround (вариант обойти проблему). Вариант с отключением selinux не рассматриваем.
task2_ns01.txt Строка 95 - изменяем тип контекста безопасности (chcon -R -t named_zone_t /etc/named)
task2_ns01.txt Строка 96 - проверим изменение типа ls -laZ /etc/named в выводе команды видим, что тип стал named_zone_t
Идем на клиент проверять возможность внести изменения в зону.
task2_client.txt Строки 38-43 - видим, что изменения успешно внесены.
task2_client.txt Строка 44 - проверяем командой dig, в выводе видим, что www.ddns.lab.		60	IN	A	192.168.50.15
перезагружаем стенд, убеждаемся в работоспособности.
