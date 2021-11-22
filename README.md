# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

 ### 1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.

Это тот самый в последней строке?
```
vagrant@vagrant:~$ strace -o cd_log /bin/bash -c 'cd /tmp' && grep tmp cd_log
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffc698b2a10 /* 24 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```
 
 ---
 
 ### 2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:  
 ```
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.
```
Посмотрим что покажет file:  
```
vagrant@vagrant:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@vagrant:~$ file /dev/sda{1,2,3}
/dev/sda1: block special (8/1)
/dev/sda2: block special (8/2)
/dev/sda3: cannot open `/dev/sda3' (No such file or directory)
vagrant@vagrant:~$ file mail
mail: ASCII text
vagrant@vagrant:~$ file /dev/tty{1..9}
/dev/tty1: character special (4/1)
/dev/tty2: character special (4/2)
/dev/tty3: character special (4/3)
/dev/tty4: character special (4/4)
/dev/tty5: character special (4/5)
/dev/tty6: character special (4/6)
/dev/tty7: character special (4/7)
/dev/tty8: character special (4/8)
/dev/tty9: character special (4/9)
vagrant@vagrant:~$ file /home/vagrant/
/home/vagrant/: directory

```
 Затем выполним `vagrant@vagrant:~$ strace -o file_log file /home/vagrant/`.  Сразу сложно сказать куда обращается file за поиском данных, но на странице `man file` есть упоминание:  
 FILES  
     /usr/share/misc/magic.mgc  Default compiled list of magic.  
     /usr/share/misc/magic      Directory containing default magic files.  
 И если грепнуть файл вывода strace  то видно что туда file и обращается(помимо библиотек, конечно):  
 ```
 vagrant@vagrant:~$ grep -i magic file_log
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
stat("/home/vagrant/.magic.mgc", 0x7ffef41c9e30) = -1 ENOENT (No such file or directory)
stat("/home/vagrant/.magic", 0x7ffef41c9e30) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
read(3, "# Magic local data for file(1) c"..., 4096) = 111
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
```

---

 ### 3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).  

Сначала находим этот файл как-то так к примеру: `lsof -p PIDпроцеса приложения | grep deleted`, затем можно его обнулить скажем так: `cat /dev/null > deleted_log_file`. Конечно это не решит проблему навсегда, ведь приложение продолжает писать инфу в этот лог-файл. 

---

 ### 4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби-процессы не занимают память, но могут привести к тому что пользователь под которым они запущены не сможет запустить новые процессы. Зомби-процессы блокируют записи в таблице процессора и при достижении лимита это может стать проблемой и даже привести к невозможности пользователя подключиться к системе. Дисковая подсистема будет также нагружена в зависимости от размера файлов зомби-процесса, как я понял(поправьте если это не так). 

---
 ### 5. В iovisor BCC есть утилита `opensnoop`:  
```
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```
На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).  

``` 
 vagrant@vagrant:/usr/sbin$ sudo ./opensnoop-bpfcc -d 1
PID    COMM               FD ERR PATH
583    irqbalance          6   0 /proc/interrupts
583    irqbalance          6   0 /proc/stat
583    irqbalance          6   0 /proc/irq/20/smp_affinity
583    irqbalance          6   0 /proc/irq/0/smp_affinity
583    irqbalance          6   0 /proc/irq/1/smp_affinity
583    irqbalance          6   0 /proc/irq/8/smp_affinity
583    irqbalance          6   0 /proc/irq/12/smp_affinity
583    irqbalance          6   0 /proc/irq/14/smp_affinity
583    irqbalance          6   0 /proc/irq/15/smp_affinity
```
 
 ---
 
 ### 6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.

Судя по выводу strace команда uname использует системные вызовы uname и arch_prtcl.  Прошу ваш комментарий по этому вопросу. Сиистемных вызово там перечислено немало, но в основном стандартные - досту к памяти, открыть файл и т.д.  

Из man 2 uname:
Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.  

---

 ### 7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
 ```
 root@netology1:~# test -d /tmp/some_dir; echo Hi
 Hi
 root@netology1:~# test -d /tmp/some_dir && echo Hi
 root@netology1:~#  
```
 Есть ли смысл использовать в bash `&&`, если применить `set -e`?  
 
Последовательность команд через `;` это просто последовательное выполнение команд, вне зависимости от результатов предыдущей команды. Если запустить их через `&&` то команда 2 будет выполнена только в случае успешного завершения команды 1 и т.д.  
По поводу `set -e`: очень смешно, я думал консоль глючит. Только потом почитал man и догнал о чём речь ) На полноценную замену `&&` пожалуй не тянет.  Но в случаях когда после какой-либо ошибки дальнейшее выполнение команд нежелательно может пригодиться.

---

 ### 8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?  


 ### 9. 
 
PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display to describe the
       state of a process:

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

