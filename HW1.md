# Урок 1. Механизмы пространства имен

## Задание: необходимо продемонстрировать изоляцию одного и того же приложения (как решено на семинаре - командного интерпретатора) в различных пространствах имен

### Формат сдачи ДЗ: предоставить доказательства выполнения задания посредством ссылки на google-документ с правами на комментирование/редактирование

### Результатом работы будет: текст объяснения, логи выполнения, история команд и скриншоты (важно придерживаться такой последовательности)

# Создание директории

1.  *создадим директорию*

    mkdir lorafolder

2.  *создадим среду*

    lorafolder/bin

3.  *скопируем оболочку*

    cp /bin/bash lorafolder/bin

4.  *узнаем какие библиотеки использует оболочка*

    ldd /bin/bash

5.  *создадим папку lib*

    mkdir lorafolder/lib

5.  **создадим папку lib64*

    mkdir lorafolder/lib64

6.  **скопировать нужные библиотеки*

cp /lib/x86_64-linux-gnu/libtinfo.so.6 lorafolder/lib
cp /lib/x86_64-linux-gnu/libc.so.6 lorafolder/lib
cp /lib64/ld-linux-x86-64.so.2 lorafolder/lib64/

8.  **переходим в папку*

    cd lorafolder

#### увидим структуру  _____bin   lib   lib64

9.  *командой chroot  переходим в bash -5.2*

    sudo chroot lorafolder

##### Таким образом мы получили изоляцию на уровне файловой системы

# Теперь  подготовим папку для работы с ls

1. *копируем ls*

   cp /bin/ls lorafolder/bin

2. *проверяем библиотеки*

    ldd /bin/ls

3. *скоприровать нужные бибилиотеки*

   cp /lib/x86_64-linux-gnu/libselinux.so.1 lorafolder/lib/
   cp /lib/x86_64-linux-gnu/libpcre2-8.so.0 lorafolder/lib/

4. *командой chroot  переходим в bash -5.2*

   sudo chroot lorafolder

5. *можем применять команду ls*

# Создаем изоляцию

1. **просмотр списка всех ns*

   lsns

2. *зайти в папку proc*

   cd /proc

3. *проверить список всех запущенных приложений*

   ls

4. *создание  текстовый ns*

   ip netns list

5. *создаем имя для ns*  

   ip netns add testns

6. **запускаем*

   ip netns list

7. *запускаем процесс (оболочку)*

   ip netns exec testns bash

8. * включить сетевой интерфейс с localhost*

   ip link set dev lo up

# Cоздадим виртуальную сеть

1. **создаем виртуальный интерфейс*

    ip link add veth0 type veth peer name veth1

2. *привязать veth1 к ранее созданному ns*

    ip link set veth1 netns testns

3. **проверяем*

    ip a

4. *присваиваем IP нашему интерфейсу veth0*

    ip addr add 10.0.0.1/24 dev veth0

5. *включаем наш интерфейс*

    ip link set dev veth0 up

6. *проверяем*

    ip a

# теперь подключаемся к новому ns

1. *подключаемся*

   ip netns exec testns bash

2. *смотрим сетевые интерфейсы*

   ip a

3. *Присвоим IP нашему интерфейсу veth1*

    ip addr add 10.0.0.4/24 dev veth1 up

4. *поднимаем*

    ip link set dev veth1 up

5. **смотрим*

    ip a

6. *Пропилинговать интерфейс из другого ns*

    ping 10.0.0.1

# Утилита unshare

1. #### создадим ns типа net

   sudo unshare --net bash

2. #### посмотреть что мы в другом ns

   ip a

# что бы посмотреть только те процессы кот. крутятся  в конкретном ns есть команда

* sudo unshare --pid --net --fork --mount-proc /bin/bash
* ps
