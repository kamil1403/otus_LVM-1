<p align="center">
  <img src="https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/lvm.jpg" alt="RAID Banner" width="800">
</p>

<h1 align="center">otus_LVM-1</h1>
<p align="center">Дата: 15-05-2025<br>Автор: Kamil Ibragimov</p>

## Домашнее задание: работа с LVM
Задание:   
• Настроить LVM в Ubuntu 24.04 Server   
• Создать Physical Volume, Volume Group и Logical Volume   
• Отформатировать и смонтировать файловую систему   
• Расширить файловую систему за счёт нового диска   
• Выполнить resize   
• Проверить корректность работы   

Критерии оценки:   
• Задание считается выполненым, если проведены все манипуляции с разделами, прописано монтирование в fstab, проведена работа со снепшотами    

Результат:   
• Создал Physical Volume, Volume Group и Logical Volume. Результат см. на скриншоте ["lvm_create"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/lvm_create.png)  
• Отформатировал том и смонтировал файловую систему в каталог /lvm. Результат см. на скриншоте ["mkfs.ext4"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/mkfs.ext4.png)  
• Расширил файловую систему за счет нового диска на 5Gb. Результат см. на скриншоте ["resize"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/resize.png) 



## 🧭 Оглавление

- [✍🏻 Скрипт для сборки RAID 1](#script)
- [✍🏻 Скрипт симуляции отказа диска](#fixraid)
- [📀 Диски и устройства](#hard)
- [🧱 Разметка и подготовка диска](#partition)
- [🗃️ Сборка RAID 1](#setup)
- [💥 Симуляция отказа диска](#failure)
- [⚙️ Создание разделов на RAID](#GPT)
- [🛠️ Форматирование и монтирование файловой системы](#ext4)
- [✂️ Основные принципы LVM](#pvl)

---

<a id="script"></a>
## ✍🏻 Скрипт для сборки RAID 1
Запуск через sudo bash script.sh   
Дать права chmod +x script.sh

```bash
#!/bin/bash

mkdir -p /mnt/raid01
apt-get install -y mdadm
mdadm --create --verbose /dev/md127 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdb2
mkfs.ext4 /dev/md127
mount /dev/md127 /mnt/raid01
echo "/dev/md127 /mnt/raid01 ext4 defaults 0 0" >> /etc/fstab
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u
```

---

<a id="fixraid"></a>
## ✍🏻 Скрипт симуляции отказа диска
Запуск через sudo bash script.sh   
Дать права chmod +x script.sh

```bash
#!/bin/bash

mdadm /dev/md127 --fail /dev/sdb1
mdadm /dev/md127 --remove /dev/sdb1
mdadm --zero-superblock /dev/sdb3 >/dev/null 2>&1
mdadm /dev/md127 --add /dev/sdb3
watch -n 1 cat /proc/mdstat
```

---

<a id="hard"></a>
## 📀 Команды для работы с LVM

```bash
# Показывает структуру дисков и разделов в виде дерева
lsblk
lsblk | grep -E 'sd|NAME'
# Выводит список физических дисков и их разделов в системе
ls -la /dev/sd*
```

---

<a id="partition"></a>
## 🧱 Разметка и подготовка диска

```bash
# Показывает текущую таблицу разделов диска /dev/sdb (размеры, типы, начальные сектора)
fdisk -l /dev/sdb
# Запуск интерактивного режима для управления разделами диска
fdisk /dev/sdb
# Действия внутри fdisk:
# m — меню команд
# p — текущая таблица разделов
# n — создать раздел (primary/extended)
# w — записать изменения и выйти
# Форматирует раздел /dev/sdb1 в файловую систему ext4
mkfs.ext4 /dev/sdb1
# Отображает древовидную структуру дисков и разделов (имена, размеры, точки монтирования)
lsblk
# Показывает UUID и тип файловой системы для всех разделов (нужен для настройки /etc/fstab)
blkid
# Создает директории /mnt/disk1, /disk2 и т.д. для монтирования разделов
# Флаг -p игнорирует ошибки, если директории уже существуют
mkdir -p /mnt/disk1 /mnt/disk2 /mnt/disk3 /mnt/disk4
# Монтирует раздел /dev/sdb1 в директорию /mnt/disk1 (временное подключение)
mount /dev/sdb1 /mnt/disk1
# Редактирует файл fstab для автоматического монтирования разделов при загрузке
nano /etc/fstab
# Пример строки
UUID=1234-ABCD /mnt/disk1 ext4 defaults 0 2
```

---

<a id="setup"></a>
## 🗃️ Сборка RAID 1 с использованием mdadm

```bash
# Создает директорию /mnt/raid01 для монтирования RAID-массива
mkdir /mnt/raid01
# Показывает статус RAID-массивов в системе (активные массивы, процесс синхронизации, устройства)
cat /proc/mdstat
# Устанавливает утилиту mdadm для управления программными RAID-массивами в Linux
apt install mdadm
# Создает RAID 1 (зеркало) из двух разделов, где /dev/md127 — имя массива, -l 1 — уровень RAID (зеркалирование), -n 2 — количество устройств
mdadm --create /dev/md127 -l 1 -n 2 /dev/sdb1 /dev/sdb2
# Проверка прогресса синхронизации RAID (например, resync = 50%).
cat /proc/mdstat
# Выводит детальную информацию о массиве:
# Состояние (active, degraded)
# Список устройств (/dev/sdb1, /dev/sdb2)
# UUID массива
mdadm -D /dev/md127
# Форматирует RAID-массив в файловую систему ext4 (готов к хранению данных)
mkfs.ext4 /dev/md127
# Монтирует массив в директорию /mnt/raid01 для доступа к данным
mount /dev/md127 /mnt/raid01
# Показывает свободное место на всех смонтированных файловых системах, включая RAID
df -h
# Копирует содержимое /var/log/ в RAID-массив для резервирования логов
cp -r /var/log/* /mnt/raid01
# Редактирует файл fstab для автоматического монтирования разделов при загрузке
nano /etc/fstab
# Пример строки для автоматического монтирования RAID
/dev/md127 /mnt/raid01 ext4 defaults 0 0
```

---

<a id="failure"></a>
## 💥 Симуляция отказа диска в RAID 1 и его замена

```bash
# Выводит детальную информацию о массиве:
# Состояние (active, degraded)
# Список устройств (/dev/sdb1, /dev/sdb2)
# UUID массива
mdadm -D /dev/md127
# Помечает диск /dev/sdb1 как неисправный (инициирует процесс замены)
mdadm /dev/md127 --fail /dev/sdb1
# Удаляет неисправный диск /dev/sdb1 из массива (после --fail)
mdadm /dev/md127 --remove /dev/sdb1
# Проверяет метаданные RAID на диске (например, был ли диск частью массива)
mdadm --examine /dev/sdX
# Очищает суперблок RAID на диске перед повторным использованием (удаляет следы участия в массиве)
mdadm --zero-superblock /dev/sdX
# Добавляет новый диск /dev/sdb3 в массив для восстановления (начнется ресинхронизация данных)
mdadm /dev/md127 --add /dev/sdb3
# Отображает текущий статус RAID, включая прогресс ресинхронизации (например, resync = 75%)
cat /proc/mdstat
```

---

<a id="GPT"></a>
## ⚙️ Расширение файловой системы

```bash
# Инициализация нового PV   
pvcreate /dev/sdc   
vgextend otus /dev/sdc   
# Расширение логического тома   
lvextend -l+80%FREE /dev/otus/test
# Расширение файловой системы
resize2fs /dev/otus/test
```

---

<a id="ext4"></a>
## 🛠️ Форматирование и монтирование файловой системы

```bash
# Форматирование логического тома
mkfs.ext4 /dev/otus/test
# Монтирование логического тома
mkdir /lvm
mount /dev/otus/test /lvm
```

---

<a id="pvl"></a>
## ✂️ Создание Physical Volume, Volume Group и Logical Volume

```bash
# LVM (Logical Volume Manager) — это система управления дисками в Linux, позволяющая гибко объединять и перераспределять пространство на физических носителях   
# LVM использует три уровня:
# 1. PV (Physical Volume) — это "сырой" диск или раздел, подготовленный под использование в LVM   
pvcreate /dev/sdb
# 2. VG (Volume Group) — это контейнер, в который объединяются один или несколько PV   
vgcreate otus /dev/sdb
# 3. LV — это "кусок" из группы VG, который работает как обычный раздел, только гибче   
lvcreate -L 10G -n test otus
# Создание логического тома на 80% от доступного места   
lvcreate -l+80%FREE -n test otus   
# Удаление логического тома   
lvremove /dev/otus/test   
# Просмотр и управление LVM   
# Подробно:  
pvdispaly 
vgdisplay   
lvdisplay   
# Кратко:   
pvs   
vgs   
lvs   
```

---
