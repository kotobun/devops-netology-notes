# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

### 1. Какой системный вызов делает команда cd?
Команда cd делает системный вызов `chdir()`. Например, при исполнении `cd /tmp` будет использован системный вызов `chdir(/tmp)`

### 2. Попробуйте использовать команду file на объекты разных типов на файловой системе и выясните, где находится база данных file на основании которой она делает свои догадки.

```
% strace file pepka.html
execve("/usr/bin/file", ["file", "pepka.html"], 0x7ffd9c01b648 /* 40 vars */) = 0

...

stat("/home/ec2-user/.magic.mgc", 0x7ffe433345a0) = -1 ENOENT (No such file or directory)
stat("/home/ec2-user/.magic", 0x7ffe433345a0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3

...

write(1, "pepka.html: HTML document, UTF-8"..., 100pepka.html: HTML document, UTF-8 Unicode text, with very long lines, with CRLF, LF line terminators
...
```

Исходя из вывода выше, можно видеть, что file успешно прочитал файлы
по путям `/etc/magic` и `/usr/share/misc/magic.mgc`.

В `/etc/magic` можно добавлять свои инструкции по опознаванию файлов
для `file`.

`/usr/share/misc/magic.mgc` -- бинарный файл, по которому file
определяет уже известные файлы.

Eще file ожидает свои базы данных по вот этим путям:
```
/home/<user_name>/.magic.mgc
/home/<user_name>/.magic
/etc/magic.mgc
```

### 3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

Можно пойти по следующему пути:
1. Найти файл, в который писал сервис в выводе `lsof`
2. Найти файловый дескриптор, в который пишет сервис в наш удаленный
файл: `ls -l /proc/<pid сервиса>/fd | grep "<путь файла>"`
3. Очистить его: `> /proc/<pid сервиса>/fd/<номер>`

### 4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
Нет, не занимают. Зомби процесс -- это такой процесс, который завершил свое исполнение, освободил свои ресурсы, и ожидает, чтобы родитель принял его return code.

### 5. На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты?
```
% sudo /usr/share/bcc/tools/opensnoop
PID    COMM               FD ERR PATH
946    irqbalance          6   0 /proc/interrupts
946    irqbalance          6   0 /proc/stat
946    irqbalance          6   0 /proc/irq/26/smp_affinity
946    irqbalance          6   0 /proc/irq/27/smp_affinity
946    irqbalance          6   0 /proc/irq/1/smp_affinity
946    irqbalance          6   0 /proc/irq/4/smp_affinity
946    irqbalance          6   0 /proc/irq/8/smp_affinity
946    irqbalance          6   0 /proc/irq/12/smp_affinity
946    irqbalance          6   0 /proc/irq/14/smp_affinity
946    irqbalance          6   0 /proc/irq/15/smp_affinity
```

`proc(5)`:
```
/proc/interrupts
    This is used to record the number of interrupts per CPU per IO device.
```

`proc(5)`:
```
/proc/stat
    kernel/system statistics. Varies with architecture.
```

Из докумментации RedHat про /proc/irq:

```
This directory is used to set IRQ to CPU affinity, which allows the system to connect a particular IRQ to only one CPU. Alternatively, it can exclude a CPU from handling any IRQs.
```

### 6. Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.
Утилита `uname` делает системный вызов `uname()`.

`uname(2)`:
```
Part of the utsname information is also accessible via /proc/sys/kernel/{ostype,  hostname, osrelease, version, domainname}.
```

Пример:
```
% cat /proc/sys/kernel/{ostype,hostname,osrelease,version,domainname}
Linux
bepis.nyansq.local
4.18.0-365.el8.x86_64
#1 SMP Thu Feb 10 16:11:23 UTC 2022
(none)
```

### 7. Чем отличается последовательность команд через ; и через && в bash?
Тем, что, если написать команды через `;`, то они будут исполняться последовательно вне зависимости от того, успешно ли завершилась предыдущая команда, а, если написать через `&&`, то команды будут исполняться только при успешном завершении (cо статус кодом 0) предыдущей.

`set -e` :
```
Exit immediately if a command exits with a non-zero status
```
Если используется `set -e`, то нет смысла писать команды через `&&`.

### 8. Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?

`set -euxo pipefail`
```
-e  Exit immediately if a command exits with a non-zero status.
-u  Treat unset variables as an error when substituting.
-x  Print commands and their arguments as they are executed.
-o pipefail
    the return value of a pipeline is the status of
    the last command to exit with a non-zero status,
    or zero if no command exited with a non-zero status
```
Все эти опции хорошо бы использовать в сценариях, потому что они
добавляют слои защиты от ошибок в коде, допущенных тем, кто писал
сценарий, не являющихся для bash чем-то невалидным, а также, например,
позволяют завершить скрипт при неудачном завершении одного из
составляющих вместо продолжения исполнения, что может привести к непродуманным создателем сценария последствиям.

`-x` полезен для дебага, но в принципе, в готовом и отдебаженном
скрипте, он не дает особо преимуществ.

### 9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

```
% ps -o stat
STAT
S+
R+
```

```
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

       For BSD formats and when the stat keyword is used, additional characters may be displayed:
        (Дополнительные знаки/буквы)
               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group
```
