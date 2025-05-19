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
• Уменьшил файловую систему до 4Gb. Результат см. на скриншоте ["resize_4Gb"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/resize_4G.png)   



## 🧭 Оглавление

- [🛠️ Основные принципы LVM](#pvl)
- [⚙️ Форматирование и монтирование файловой системы](#ext4)
- [✂️ Расширение файловой системы](#resize_max)
- [✂️ Уменьшение файловой системы](#resize_min)

---

<a id="pvl"></a>
## 🛠️ Создание Physical Volume, Volume Group и Logical Volume

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

<a id="ext4"></a>
## ⚙️ Форматирование и монтирование файловой системы

```bash
# Форматирование логического тома
mkfs.ext4 /dev/otus/test
# Монтирование логического тома
mkdir /lvm
mount /dev/otus/test /lvm
```

---

<a id="resize_max"></a>
## ✂️ Расширение файловой системы

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

<a id="resize_min"></a>
## ✂️ Уменьшение файловой системы

```bash
# Отключение логического тома   
umount /lvm   
# Проверка файловой системы   
e2fsck -f /dev/otus/test   
# Уменьшение файловой системы   
resize2fs /dev/otus/test 4G   
# Уменьшение логического тома   
lvreduce -L 4G /dev/otus/test   
# Монтирование логического тома
mount /dev/otus/test /lvm
```

---
