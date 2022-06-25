# devops-DZ3.3
# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

>1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.
### Ответ
```bash
vagrant@vagrant:~$ strace /bin/bash -c 'cd /tmp' 2>&1 | grep tmp
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffe972d6c70 /* 23 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```
Интересующий нас вызов - последняя строка. Вернулся 0, значит `cd` выполнился успешно

>2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
### Ответ
```bash
vagrant@vagrant:~$ strace file /dev/tty1 2>&1 | grep open
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
```
Очевидно, база лежит в `/usr/share/misc/magic.mgc`

>3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
### Ответ
В документации к RedHat предлагается выполнить
```bash
$ echo > /proc/pid/fd/fd_number
```
где `pid`,`fd` можно взять из 
```bash
$ lsof | egrep "deleted|COMMAND"
COMMAND       PID    TID TASKCMD     USER   FD  TYPE  DEVICE    SIZE/OFF      NODE NAME
ora         25575   8194 oracle    oracle   33   REG   65,65  4294983680  31014933 /oradata/DATAPRE/file.dbf (deleted)
```

>4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
### Ответ
Нет, не занимают. Только `pid` не освобождают.

>5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
### Ответ
```bash
vagrant@vagrant:~$ sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
814    vminfo              5   0 /var/run/utmp
627    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
627    dbus-daemon        19   0 /usr/share/dbus-1/system-services
627    dbus-daemon        -1   2 /lib/dbus-1/system-services
627    dbus-daemon        19   0 /var/lib/snapd/dbus-1/system-services/
814    vminfo              5   0 /var/run/utmp
627    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
627    dbus-daemon        19   0 /usr/share/dbus-1/system-services
627    dbus-daemon        -1   2 /lib/dbus-1/system-services
627    dbus-daemon        19   0 /var/lib/snapd/dbus-1/system-services/
```


>6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
### Ответ
```bash
vagrant@vagrant:~$ strace uname -a
execve("/usr/bin/uname", ["uname", "-a"], 0x7ffd89d77368 /* 23 vars */) = 0
.................
uname({sysname="Linux", nodename="vagrant", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
write(1, "Linux vagrant 5.4.0-91-generic #"..., 106Linux vagrant 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
) = 106
.................
exit_group(0)                           = ?
+++ exited with 0 +++


```
```
UNAME(2)                Linux Programmer's Manual               UNAME(2)
NAME         top
       uname - get name and information about current kernel
................
Part of the utsname information is also accessible via
       /proc/sys/kernel/{ostype, hostname, osrelease, version,
       domainname}.
...............
```
>7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
### Ответ
```bash
```
Оператор `;` дает возможность задать в одной строке несколько команд, которые будут выполнены последовательно, одна за другой.
Оператор `&&` являются управляющим оператором. Если в командной строке стоит command1 && command2, то command2 выполняется в том, и только в том случае, если статус выхода из команды command1 равен нулю, что говорит об успешном ее завершении. В общем - логическое "И" для выполнения команд
>-e      Exit  immediately  if a pipeline (which may consist of a
 single simple command),  a subshell command enclosed in parentheses,
 or one of the commands executed as part of a command list enclosed by
 braces (see SHELL GRAMMAR above) exits with a non-zero  status.
Таким образом, после `set -e` использование `&&` не имеет смысла

>8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
### Ответ
```bash
```


>9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
### Ответ

```bash
```

 
