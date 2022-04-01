# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. `cd` - это встроенная команда `bash`. `cd` переставляет указатель на текущую директорию. 

Если сделать ее внешней командой, то после перехода в нужную директорию, придется все равно запускать в ней новый bash 

2. `grep qwerty test -c`
```bash
vagrant@vagrant:~$ cat >test
kajdasi njdshu
dsfjdskf qwerty
qwerty hfhfhf45 jgugj
qwer dzfnsdfj hghg
qwerty12
vagrant@vagrant:~$ grep qwerty test | wc -l
3
vagrant@vagrant:~$ grep qwerty test -c
3
```
3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
```bash
vagrant@vagrant:~$ pstree -p | grep \(1\)
systemd(1)-+-VBoxService(839)-+-{VBoxService}(840)
```
4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

Терминал 1
```bash
vagrant@vagrant:~$ root ls -l /home/vagrant/ 2>/dev/pts/1
```
Терминал 2
```bash
vagrant@vagrant:~$ -bash: root: command not found
```
5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
```bash
vagrant@vagrant:~$ cat >5_in
123
vagrant@vagrant:~$ cat <5_in >5_out
vagrant@vagrant:~$ cat 5_out 
123
```

6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? 
Сможете ли вы наблюдать выводимые данные?

Вывести получилось на своей рабочей машине.
```bash
$ tty 
/dev/pts/2
$ echo Follow white rabbit >/dev/tty3
```
Далее переключилась к терминалу 3: Ctrl+Alt+F3 и увидела там вывод команды

7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?

`bash 5>&1` создает дескриптор 5 и перенаправляет вывод в него на stout. 
`echo netology > /proc/$$/fd/5` выведет на устройство вывода слово netology. поскольку вывод с дескриптора 5 был перенаправлен в stout
Если запустить только 2 команду, без предварительного запуска 1ой, то получим ошибку. поскольку такого дескриптора нет

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

```bash
vagrant@vagrant:~$ bash 6>&1
vagrant@vagrant:~$ ls -lR 6>&1 1>&2 2>&6
.:
total 16
-rw-rw-r-- 1 vagrant vagrant  0 Mar 19 19:32 0.txt
-rw-rw-r-- 1 vagrant vagrant  0 Mar 19 19:32 1.txt
-rw-rw-r-- 1 vagrant vagrant  0 Mar 19 19:32 2.txt
-rw-rw-r-- 1 vagrant vagrant  4 Mar 25 19:25 5_in
-rw-rw-r-- 1 vagrant vagrant  4 Mar 25 19:25 5_out
-rw-rw-r-- 1 vagrant vagrant 78 Mar 27 19:32 hardcopy.0
-rw-rw-r-- 1 vagrant vagrant 22 Mar 27 18:39 test
```


9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

Данная команда покажет переменные окружения.
Похожий вывод дадут 
* `env`
* `printenv`

10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
```bash
line 290
/proc/[pid]/cmdline
This read-only file holds  the  complete  command  line  for  the
process,  unless  the  process  is a zombie.  In the latter case,
there is nothing in this file: that is, a read on this file  will
return  0  characters.  The command-line arguments appear in this
file as a set of strings separated by null bytes ('\0'),  with  a
further null byte after the last string.

line 338
/proc/[pid]/exe
Under Linux 2.2 and later, this file is a symbolic link  contain‐
ing  the  actual pathname of the executed command.  This symbolic
link can be dereferenced normally; attempting  to  open  it  will
open  the  executable.   You can even type /proc/[pid]/exe to run
another copy of the same executable that is being run by  process
[pid].  If the pathname has been unlinked, the symbolic link will
contain the string '(deleted)' appended to the original pathname.
In  a  multithreaded  process, the contents of this symbolic link
are not available if the main thread has already terminated (typ‐
ically by calling pthread_exit(3)).
```

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

sse4_2

12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

     ```bash
     vagrant@netology1:~$ ssh localhost 'tty'
     not a tty
     ```

     Почитайте, почему так происходит, и как изменить поведение.

    * При открытии нового терминала и `vagrant ssh` дефолтная shell команда `bash -l` (https://www.vagrantup.com/docs/vagrantfile/ssh_settings)
    * Поэтому в данному случае команды shell вызываются сначала из `/etc/profile`. После этого обращение идет к `/root/.profile` т.к. большинство команд vagrant запускается от рута. (https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Bash-Startup-Files)
    * В Ubuntu в `/root/.profile` находится команда `mesg n` которая запрещает другим пользователям писать в его терминалы
    * В общем случае это полезно. Но в случае vagrant польза не ясна.
    * Чтобы обойти эту проблему я нашла несколько способов. Например, заменить `msg n`  на `tty -s && mesg n` в файле `/root/.profile`. Для проверки наличия устройства tty  

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.

В файле `/etc/sysctl.d/10-ptrace.conf` заменила значение на `kernel.yama.ptrace_scope = 0`
Сразу не сработало. После перезагрузки все получилось.
Далее пример использования

```bash
vagrant@vagrant:~$ top

top - 18:47:09 up 13 min,  1 user,  load average: 0.00, 0.01, 0.00
Tasks: 109 total,   1 running, 108 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    981.0 total,    383.0 free,    136.5 used,    461.5 buff/cache
MiB Swap:   1962.0 total,   1962.0 free,      0.0 used.    700.4 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND  
      1 root      20   0  103128  12744   8372 S   0.0   1.3   0:01.38 systemd  
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd 
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp   
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par+ 
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker+ 
      8 root      20   0       0      0      0 I   0.0   0.0   0:00.10 kworker+ 
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_perc+ 
     10 root      20   0       0      0      0 S   0.0   0.0   0:00.07 ksoftir+ 
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.15 rcu_sch+ 
     12 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 migrati+ 
     13 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 idle_in+ 
     14 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0  
     15 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/1  
     16 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 idle_in+ 
     17 root      rt   0       0      0      0 S   0.0   0.0   0:00.21 migrati+ 
     18 root      20   0       0      0      0 S   0.0   0.0   0:00.05 ksoftir+ 
     20 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker+ 
[1]+  Stopped                 top
vagrant@vagrant:~$ bg
[1]+ top &
vagrant@vagrant:~$ 


[1]+  Stopped                 top
vagrant@vagrant:~$ disown top
-bash: warning: deleting stopped job 1 with process group 1596

reptyr 1596
vagrant@vagrant:~$ top

top - 18:47:09 up 13 min,  1 user,  load average: 0.00, 0.01, 0.00
Tasks: 109 total,   1 running, 108 sleeping,   0 stopped,   0 zombie

``` 
Запущенный процесс успешно переместился

14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

`tee` выводит данные одновременно в файл, указанный в качестве параметра, и в stdout, 
В примере команда `tee` получается на stdin данные полученные через pipe от stdout команды `echo`.
`tee` запущена от рута поэтому имеет права на запись в файл

# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.

Использовала команду:  
```bash
strace /bin/bash -c 'cd /tmp'
```
Полностью результат не привожу. К `cd /tmp` относится вот этот: `chdir("/tmp")` 

2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
   
Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.

Попробовала strace для команды `file`: 
```bash
vagrant@vagrant:~$ strace file /dev/tty
```

Из результата команды:
```bash
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
```
Предполагаю, что БД тут `/usr/share/misc/magic.mgc`

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

```bash
vagrant@vagrant:~$ ping google.com > google.log
^Z
[2]+  Stopped                 ping google.com > google.log
vagrant@vagrant:~$ bg
[2]+ ping google.com > google.log &
vagrant@vagrant:~$ rm -rf google.log 
vagrant@vagrant:~$ ps aux | grep ping
vagrant     2362  0.0  0.2   8960  2724 pts/0    S    20:01   0:00 ping google.com
vagrant     2366  0.0  0.0   8160   732 pts/0    S+   20:02   0:00 grep --color=auto ping
vagrant@vagrant:~$ sudo lsof -p 2362
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
ping    2362 vagrant  cwd    DIR  253,0  3284992 1051845 /home/vagrant
ping    2362 vagrant  rtd    DIR  253,0     4096       2 /
ping    2362 vagrant  txt    REG  253,0    72776 1835881 /usr/bin/ping
ping    2362 vagrant  mem    REG  253,0    31176 1841606 /usr/lib/x86_64-linux-gnu/libnss_dns-2.31.so
ping    2362 vagrant  mem    REG  253,0    51832 1841607 /usr/lib/x86_64-linux-gnu/libnss_files-2.31.so
ping    2362 vagrant  mem    REG  253,0   201272 1836505 /usr/lib/locale/C.UTF-8/LC_CTYPE
ping    2362 vagrant  mem    REG  253,0       50 1836510 /usr/lib/locale/C.UTF-8/LC_NUMERIC
ping    2362 vagrant  mem    REG  253,0     3360 1836513 /usr/lib/locale/C.UTF-8/LC_TIME
ping    2362 vagrant  mem    REG  253,0  1518110 1836504 /usr/lib/locale/C.UTF-8/LC_COLLATE
ping    2362 vagrant  mem    REG  253,0      270 1836508 /usr/lib/locale/C.UTF-8/LC_MONETARY
ping    2362 vagrant  mem    REG  253,0       48 1836514 /usr/lib/locale/C.UTF-8/LC_MESSAGES/SYS_LC_MESSAGES
ping    2362 vagrant  mem    REG  253,0       34 1836511 /usr/lib/locale/C.UTF-8/LC_PAPER
ping    2362 vagrant  mem    REG  253,0       62 1836509 /usr/lib/locale/C.UTF-8/LC_NAME
ping    2362 vagrant  mem    REG  253,0      131 1836503 /usr/lib/locale/C.UTF-8/LC_ADDRESS
ping    2362 vagrant  mem    REG  253,0       47 1836512 /usr/lib/locale/C.UTF-8/LC_TELEPHONE
ping    2362 vagrant  mem    REG  253,0       23 1836507 /usr/lib/locale/C.UTF-8/LC_MEASUREMENT
ping    2362 vagrant  mem    REG  253,0  3035952 1835290 /usr/lib/locale/locale-archive
ping    2362 vagrant  mem    REG  253,0   137584 1841525 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.28.0
ping    2362 vagrant  mem    REG  253,0  2029224 1841468 /usr/lib/x86_64-linux-gnu/libc-2.31.so
ping    2362 vagrant  mem    REG  253,0   101320 1841650 /usr/lib/x86_64-linux-gnu/libresolv-2.31.so
ping    2362 vagrant  mem    REG  253,0  1168056 1835853 /usr/lib/x86_64-linux-gnu/libgcrypt.so.20.2.5
ping    2362 vagrant  mem    REG  253,0    31120 1841471 /usr/lib/x86_64-linux-gnu/libcap.so.2.32
ping    2362 vagrant  mem    REG  253,0    27002     682 /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
ping    2362 vagrant  mem    REG  253,0   191472 1841428 /usr/lib/x86_64-linux-gnu/ld-2.31.so
ping    2362 vagrant  mem    REG  253,0      252 1836506 /usr/lib/locale/C.UTF-8/LC_IDENTIFICATION
ping    2362 vagrant    0u   CHR  136,0      0t0       3 /dev/pts/0
ping    2362 vagrant    1w   REG  253,0     6067 1052318 /home/vagrant/google.log (deleted)
ping    2362 vagrant    2u   CHR  136,0      0t0       3 /dev/pts/0
ping    2362 vagrant    3u  icmp             0t0   34671 00000000:0003->00000000:0000
ping    2362 vagrant    4u  sock    0,9      0t0   34672 protocol: PINGv6
```

Открыла другую консоль. Ввела в ней команду  
```bash
vagrant@vagrant:~$ sudo sh -c '>/proc/2362/fd/1'
```

После этого удаленный файл перестал расти. Проверяла также по `lsof`

4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Нет, зомби не занимают памяти (как процессы-сироты), но блокируют записи в таблице процессов, размер которой ограничен для каждого пользователя и системы в целом.

5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).

```bash
vagrant@vagrant:~$ sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
1      systemd            12   0 /proc/639/cgroup
1      systemd            12   0 /proc/385/cgroup
832    vminfo              6   0 /var/run/utmp
628    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
628    dbus-daemon        20   0 /usr/share/dbus-1/system-services
628    dbus-daemon        -1   2 /lib/dbus-1/system-services
628    dbus-daemon        20   0 /var/lib/snapd/dbus-1/system-services/
632    irqbalance          6   0 /proc/interrupts
632    irqbalance          6   0 /proc/stat
632    irqbalance          6   0 /proc/irq/20/smp_affinity
632    irqbalance          6   0 /proc/irq/0/smp_affinity
632    irqbalance          6   0 /proc/irq/1/smp_affinity
632    irqbalance          6   0 /proc/irq/8/smp_affinity
632    irqbalance          6   0 /proc/irq/12/smp_affinity
632    irqbalance          6   0 /proc/irq/14/smp_affinity
632    irqbalance          6   0 /proc/irq/15/smp_affinity
832    vminfo              6   0 /var/run/utmp
628    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
628    dbus-daemon        20   0 /usr/share/dbus-1/system-services
```
 
9. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.

При вожу часть `strace uname`
```bash
uname({sysname="Linux", nodename="vagrant", ...}) = 0
```
Цитата из `man 2 uname`:
```
Part of the utsname information is also accessible  via  /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
```
10. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
     ```bash
     root@netology1:~# test -d /tmp/some_dir; echo Hi
     Hi
     root@netology1:~# test -d /tmp/some_dir && echo Hi
     root@netology1:~#
     ```
     Есть ли смысл использовать в bash `&&`, если применить `set -e`?

Оператор точка с запятой позволяет запускать несколько команд за один раз, и выполнение команды происходит последовательно.
Оператор AND (&&) будет выполнять вторую команду только в том случае, если при выполнении первой команды SUCCEEDS, т.е. состояние выхода первой команды равно «0» — программа выполнена успешно.

set -e  Exit immediately if a command exits with a non-zero status.
Тогда нет смысла использовать `&&` совместно с `set -e`

11. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?

```
-e  Exit immediately if a command exits with a non-zero status.
-u  Treat unset variables as an error when substituting.
-x  Print commands and their arguments as they are executed.
-o option-name
pipefail     the return value of a pipeline is the status of the last command to exit with a non-zero status, or zero if no command exited with a non-zero status
```

В совокупности эти опции позволяют писать более безопасные сценарии, поскольку увеличивается детализация логгирования, завершается сценарий при наличии ошибок.
Эти опции можно сказать делают то, что уже встроено в высокоуровневых языках программирования.

12. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

Выдержка из man:
```
PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display
       to describe the state of a process:

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

       For BSD formats and when the stat keyword is used, additional characters may be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group
```

Самые частые:
Ss - ожидающие пробуждения
T - остановлен