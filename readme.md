# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. `cd` - это встроенная команда `bash`. `cd` переставляет указатель на текущую директорию. 

Если сделать ее внешней командой, то после перехода в нужную директорию, придется все равно запускать в ней новый bash 

2. `grep qwerty test -c`
```
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
```
vagrant@vagrant:~$ pstree -p | grep \(1\)
systemd(1)-+-VBoxService(839)-+-{VBoxService}(840)
```
4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

Терминал 1
```
vagrant@vagrant:~$ root ls -l /home/vagrant/ 2>/dev/pts/1
```
Терминал 2
```
vagrant@vagrant:~$ -bash: root: command not found
```
5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
```
vagrant@vagrant:~$ cat >5_in
123
vagrant@vagrant:~$ cat <5_in >5_out
vagrant@vagrant:~$ cat 5_out 
123
```

6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? 
Сможете ли вы наблюдать выводимые данные?

Вывести получилось на своей рабочей машине.
```
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

```
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
```
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

```
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