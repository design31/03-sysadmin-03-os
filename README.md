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

---

### 3. 
