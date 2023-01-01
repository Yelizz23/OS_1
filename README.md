# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.
   ```
    Системный вызов chdir("/tmp")`
   ```

    ![snimok]( /Users/antonbannikov/Desktop/chdir.png)

---

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

   ```
    База данных находится в: /usr/share/misc/magic.mgc`
   ```
    ![snimok]( /Users/antonbannikov/Desktop/strace_file.png)

---

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
   ```
    Чтобы освободить место на файловой системе нужно:
    lsof | grep deleted
    ls -la /proc/PID/fd
    cat /dev/null > /proc/PID/fd/descriptor_num
    ls -la /proc/PID/fd
   ```

---

4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
   ```
    В отличии от сирот зомби-процессы не занимают ресурсы памяти. Зомби лишь блокируют записи в таблице процессов (за ними сохраняются номера PID). У таблицы процессов ограниченная память для каждого пользователя и это может в дальнейшем привести к проблеме - не способность создавать новые дочерние процессы
   ```

---

5. В iovisor BCC есть утилита `opensnoop`:
   ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
    
   ```
    Не получается установить пакет bpfcc-tools для Ubuntu 20.04. 
    Устанавливаю все по документации, все устанавливается но при вызове команды strace dpkg -L bpfcc-tools | grep sbin/opensnoop
    пишет что пакет bpfcc-tools не установлен. Не могу понять что делаю не так, подскажите пожалуйста. 
   ```
    ![snimok]( /Users/antonbannikov/Desktop/problem.png)

---

6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
   ```
    uname(2) - Part of the utsname information is also accessible via /proc/sys/kernel/{os‐type, hostname, osrelease, version, domainname}.
   ```

---

7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?

   ```
    Последовательность команд через && в bash: 
    - bash интерпретирует && как логический оператор И
    - выполнит команды последовательно, то есть вторая команда выполнится только в случаи УСПЕШНОГО(0) выполнения первой команды

    Последовательность команд через ; в bash: 
    - позволяет запускать несколько команд одновременно

    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
    - set -e прекращает выполнение команд если результат отличен от нуля, у всех команд кроме последней в последовательности.
    - функционал очень похож на &&, и я думаю что не имеет смысла использовать set -e, так как обе команды прекрящают выполнение при ошибке, хотя могу ошибаться
   ```

---

8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
   ```
    -e: прерывает выполнение команды при ошибке, кроме последней в последовательности
    -u: прекращает выполнение команды, если встретились незаданные параметры и переменные
    -x: вывод команд и их аргументов при выполнении
    -o: выводит статус команды, которая завершилась с ошибкой.

    set -euxo pipefail: хорошо использовать при отладке, показывает выполнение команд и ошибки 
   ```

---

9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

    ![snimok]( /Users/antonbannikov/Desktop/stat.png)
   ```
    Самый наиболее часто встречающийся статус у процессов в системе: 
       S -> Спящее состояние: прерываемое (interruptible sleep (waiting for an event to complete)
       I -> фоновые процессы ядра (бездействующие) - Idle kernel thread
       R -> исполняемый файл (running or runnable (on run queue))
    Дополнительные символы -> дополнительные характеристики, такие как например приоритет:
       <  -> high-priority (not nice to other users)
       N  -> low-priority (nice to other users)
       L  -> has pages locked into memory (for real-time and custom IO)
       s  -> is a session leader
       l  -> is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
       +  -> is in the foreground process group
```

