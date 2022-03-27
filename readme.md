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
9. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

1 вариант - перенаправляем stderr в stdout, а сам stdout перенаправляется в /dev/null. Но stderr все еще перенаправлен в stdout
`command 2>&1 >/dev/null | grep 'something'` 
 
2 вариант - с помощью промежуточного дескриптора
```
bash 6>&1
command 6>&1 1>&2 2>&6
```
12. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

Данная команда покажет переменные окружения.
Похожий вывод дадут 
* `env`
* `printenv`

13. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
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
14. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

sse4_2
15. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

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
16. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
Решить не удалось. Ругается на права 
```
vagrant@vagrant:~$ reptyr 2797
Unable to attach to pid 2797: Operation not permitted
The kernel denied permission while attaching. If your uid matches
the target's, check the value of /proc/sys/kernel/yama/ptrace_scope.
For more information, see /etc/sysctl.d/10-ptrace.conf

```
В файле `/etc/sysctl.d/10-ptrace.conf` заменила значение на `kernel.yama.ptrace_scope = 0`
К сожалению это не помогло. 
17. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

`tee` выводит данные одновременно в файл, указанный в качестве параметра, и в stdout, 
В примере команда `tee` получается на stdin данные полученные через pipe от stdout команды `echo`.
`tee` запущена от рута поэтому имеет права на запись в файл