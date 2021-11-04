# HomeWork4

Устанавливаем необходимое окружение для выполнения домашней работы, изменив количество дисков с 6 до 8

1. Определить алгоритм с наилучшим сжатием

a) Создаем 4 mirror pool из дисков 

[vagrant@server ~]$ sudo -i
[root@server ~]# zpool create gzip mirror /dev/sdb /dev/sdc
[root@server ~]# zpool create zle mirror /dev/sdd /dev/sde
[root@server ~]# zpool create lzjb mirror /dev/sdf /dev/sdh
[root@server ~]# zpool create lz4 mirror /dev/sdg /dev/sdi

б) Создаем файловые системы с разными видами сжатия

[root@server ~]# zfs create -o compression=gzip gzip/gzip
[root@server ~]# zfs create -o compression=zle zle/zle
[root@server ~]# zfs create -o compression=lzjb lzjb/lzjb
[root@server ~]# zfs create -o compression=lz4 lz4/lz4


в) скачаем файл “Война и мир” и расположим на своей файловой системе

[root@server gzip]# wget -o War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8
[root@server gzip]# cp War_and_Peace.txt /zle/zle/
[root@server gzip]# cp War_and_Peace.txt /lzjb/lzjb/
[root@server gzip]# cp War_and_Peace.txt /lz4/lz4/

г) Проверяем, какое сжатие отработало лучше

[root@server gzip]# zfs list
NAME        USED  AVAIL     REFER  MOUNTPOINT
gzip        141K   832M     25.5K  /gzip
gzip/gzip  25.5K   832M     25.5K  /gzip/gzip
lz4         124K   832M     25.5K  /lz4
lz4/lz4    25.5K   832M     25.5K  /lz4/lz4
lzjb        124K   832M     25.5K  /lzjb
lzjb/lzjb  25.5K   832M     25.5K  /lzjb/lzjb
zle         125K   832M     25.5K  /zle
zle/zle      26K   832M       26K  /zle/zle

2. Определить настройки pool’a
Загружаем архив с файлами с гугл диска https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg

[root@server tar]# wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg" -O test.tar.gz && rm -rf /tmp/cookies.txt

2021-11-04 08:18:56 (7.86 MB/s) - 'test.tar.gz' saved [7275140/7275140]

[root@server tar]# tar -xf test.tar.gz 
[root@server tar]# ls
test.tar.gz  zpoolexport

Импортируем пул

[root@server tar]# zpool import -d ${PWD}/zpoolexport
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                               ONLINE
	  mirror-0                         ONLINE
	    /bendin/tar/zpoolexport/filea  ONLINE
	    /bendin/tar/zpoolexport/fileb  ONLINE


[root@server /]# zpool import -d /bendin/tar/zpoolexport otus
[root@server /]# zpool list otus
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus     480M  2.21M   478M        -         -     0%     0%  1.00x    ONLINE  -

[root@server /]# zfs get compression,compressratio otus
NAME  PROPERTY       VALUE     SOURCE
otus  compression    zle       local
otus  compressratio  1.00x     -

3. Найти сообщение от преподавателей 

Згружаем файл из гугл диска https://drive.google.com/file/d/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG/view?usp=sharing 

wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG" -O otus_task2.file && rm -rf /tmp/cookies.txt

Файл был получен командой
zfs send otus/storage@task2 > otus_task2.file

Создаем файловую систему otus/storage и восстанавливаемся

[root@server bendin]# zfs create otus/storage
[root@server bendin]# zfs receive otus/storage/task2 < otus_task2.file

[root@server bendin]# cd /otus/storage/
[root@server storage]# ls
task2
[root@server storage]# cd task2/
[root@server task2]# ls
10M.file   Moby_Dick.txt      cinderella.tar    homework4.txt  world.sql
Limbo.txt  War_and_Peace.txt  for_examaple.txt  task1
[root@server task2]# cd task1/
[root@server task1]# ls
README  file_mess
[root@server task1]# cd file_mess/

[root@server file_mess]# cat secret_message 
https://github.com/sindresorhus/awesome









