# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Задание 1

Создание Deployment приложения, использующего локальный PV, созданный вручную.

1. Пишу манифест Deployment приложения, состоящего из контейнеров busybox и multitool.

Применяю Deployment и проверяю его статус:

![image](https://github.com/user-attachments/assets/6bb0cf2a-7572-4bdb-a0c3-f6711008e56a)

Статус пода в состоянии Pending. Ввиду того, что не готов PVC

2. Пишу манифест PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

Применяю манифесты PV и PVC и проверяю их статусы:

![image](https://github.com/user-attachments/assets/ca4a370c-6924-4a62-82fa-e45385c63f18)

PV и PVC запущены.

Теперь под, который ожидал создания PVC находится в статусе RUNNING:

![image](https://github.com/user-attachments/assets/485c3891-faf8-4b7d-93b7-6d80cf2694da)

3. Проверю, сможет ли multitool прочитать файл, в который busybox пишет данные каждые пять секунд в общей директории.

Проверить доступность файла можно из контейнера `multitool`:

![image](https://github.com/user-attachments/assets/a9e5b803-379b-4ab4-ad0f-510a6ca2bdf7)

Multitool имеет доступ к файлу и может прочитать его.

4. Удаляю Deployment и PVC:

![image](https://github.com/user-attachments/assets/b142feeb-8f3c-42bc-8e34-1cf9a238d11c)

 Проверю, что после этого произошло с PV:

![image](https://github.com/user-attachments/assets/4b68c58f-b5a6-4940-9548-8dc9f30976dd)

При установленной политике Retain, когда PVC удаляется, PV остается в кластере вместе с данными, которые на нем хранятся.
После удаления PVC, статус PV изменился на Released, и не сможет быть повторно привязан, пока не будет очищен (освобожден) администратором.

5. Проверю, сохранился ли файл на локальном диске ноды:

![image](https://github.com/user-attachments/assets/52ea2bb9-1468-4451-add4-b6852362bf02)

Файл присутствует в директории `/tmp/pv-data`.

Удаляю PV:

![image](https://github.com/user-attachments/assets/69964237-1793-452f-adc3-2abf8d38818f)

Файл все еще будет доступен в локальной директории ноды (`/tmp/pv-data/data.log`), потому что мы использовали hostPath, что означает, что данные хранятся на диске ноды, и удаление PV не влияет на сам файл. В случае если в манифесте PV политика persistentVolumeReclaimPolicy будет установлена в Recycle, то файл будет удален.

6. Ссылка на манифест - [Deployment](https://github.com/PatKolzin/kuber-2.2/blob/main/src/deployment.yaml)

   Ссылка на манифест - [PV](https://github.com/PatKolzin/kuber-2.2/blob/main/src/pv.yaml)

   Ссылка на манифест - [PVC](https://github.com/PatKolzin/kuber-2.2/blob/main/src/pvc.yaml)

------

### Задания 2

Создание Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Устанавливаю и настраиваю NFS-сервер на ноде с MicroK8S:

![image](https://github.com/user-attachments/assets/6f77d295-f81e-4e73-9704-45023fd9a73e)
![image](https://github.com/user-attachments/assets/b82e43a6-addf-46c5-9aca-f1fff8287c12)

2. Пишу манифест Deployment-PVC и манифест SC и применяю их:

![image](https://github.com/user-attachments/assets/c7b492f9-81a8-4707-a2f6-4661a4a4627c)

После чего под и PVC переходят из статуса - pending в статус - bound.

![image](https://github.com/user-attachments/assets/fe3ea2d3-01c0-449a-8dc5-9b083832d56e)

Проверяю PV, убеждаюсь, что он создан автоматически:

![image](https://github.com/user-attachments/assets/eea6cd26-d04c-4650-ad6f-85cf0287ead4)

3. Проверю возможность чтения и записи файла изнутри пода. Для этого войду в оболочку контейнера пода и создам файл:

![image](https://github.com/user-attachments/assets/314735bf-5e87-492c-b7f2-e7882b5443bc)

Через `describe pv` проверяем, по какому пути смонтировалась NFS директория:

![image](https://github.com/user-attachments/assets/2d3208e7-eb32-433d-8d32-f63cc8ca12a1)

Переходим в эту директорию, и видим созданный из контейнера пода файл:

![image](https://github.com/user-attachments/assets/714305bf-2724-451a-90e1-1ab83a7fb368)

Это говорит о том, что NFS работает и из пода файл доступен для чтения и записи.

4. Ссылка на манифест [Deployment-PVC](https://github.com/PatKolzin/kuber-2.2/blob/main/src/deployment-multitool.yaml)

   Ссылка на манифест [SC](https://github.com/PatKolzin/kuber-2.2/blob/main/src/sc.yaml)
