# Домашнее задание к занятию «Защита сети», Лебедев А.И., FOPS-10


------

### Подготовка к выполнению заданий

1. Подготовка защищаемой системы:

- установите **Suricata**,
- установите **Fail2Ban**.

2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

------

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*  

### Решение:  

- Имеем kali-linux и машину для тестов, где установлены и настроены **Suricata** и **Fil2Ban**;

- Над настройками Suricata пришлось немного задержаться, т.к. он упорно не хотел записывать правила в мой файл по дефолту, а писал его в /var/lib... Что ж. Скопируем его в наш путь по дефолту и запустим. Работает:

![sur](img/1.JPG)  

 - Предпримем SYN-сканирование:

![kali](img/sS.JPG)   

  - В фаст-логах видим, что суриката отметила потенциально подозрительный трафик, предупреждает о том, что предпринимается попытка "вытянуть" информацию, показывает ip-адрес злоумышленника, порт с которого ведется сканирование, а также порты на машине, на которые сканирование направлено

![sur](img/sSk.JPG)  

  - Предпримем другие виды атак и посмотрим логи. Итак, в логи не попадает TCP ACK (sA) сканирование. Полагаю, что т.к. этот вид сканирования отличнается от других тем, что не получает информацию о том открыт или закрыт порт и, верятно, требует более углубленно настройки от Суриката. Остальные виды сканирования попадают в логи и выдается та же информация.

![sur](img/kali.JPG)  

![sur](img/bad.JPG)  

 - По fail2ban же, вот, что я могу сказать. Я попытался настроить его так (создать правило), чтобы он смотрел в лог Суриката (eve.json), выхватывал оттуда строчку SYN, если она там есть и сразу банил этот ip-адрес. К несчастью, у меня не вышло. Не видел он это правило и все. Собственно, отказался от идею ввиду того, что реализация данного действа съела бы у меня несколько дней. В основных же логах fail2ban, типа sshd никакой активности нет. Но, оно и понятно, т.к. у нас идет не подбор паролей пока.

------

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.



*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*  

### Решение:    

 - Настроим File2Ban на превентивную защиту ssh-порта и попробуем пробиться из kali:

![sur](img/pass.JPG)  

  - Что ж, пароль и логин он подбирает, но пролезть на машину не может. На другую машину - запросто пролезает. При том, пароль и логин он подбирает будучи уже в бане:

![sur](img/block.JPG)  

   - Нормальная ли это ситуация? 



---
