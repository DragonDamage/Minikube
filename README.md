# Minikube


## 0.) Включаем и настраиваем вложенную виртуализацию

0.1) Включаем `SVM` в BIOS

0.2) Удаляем компоненты Windows:
- `Win` + `R` → `Выполнить` → `appwiz.cpl` → `Enter`
- Переходим в верхний левый раздел и нажмите `Включить или отключить функции Windows`
- Далее снимаем флажок с `Hyper-V`, `Подсистема Windows для Linux` и `Платформа виртуальной машины`

0.3) Отключаем безопасность на основе виртуализации:
- `Win` + `R` → «Выполнить» → `cmd` (запускаем от имени администратора)
- Вводим `bcdedit /set hypervisorlaunchtype off`

0.4) Включаем вложенную виртуализацию в VMware:
- `Витруальна машина` → `Изменить настройки` → `Процессор` → ☑ Виртуальный Intel VT-x/EPT или AMD-V/RVI

---

## 1.1.) Установка minikube
```bash
$ sudo apt update              # Обновляемся
$ sudo apt upgrade             # Обновляемся
$ sudo apt  install docker.io  # Устанавливаем Docker
$ sudo apt install curl        # Устанавливаем курл
$ sudo apt install virtualbox  # Устанавливаем VirtualBox
```

#### Устанавливаем kubectl:
```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### Устанавливаем Minikube:
```bash
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
$ minikube start --driver=docker --force  # Запускаем Minikube
$ minikube status  # Проверяем состояние Minikube
$ minikube stop    # Остановить кластер
```

#### Очистка локального состояния:
```bash
$ minikube start  # В таком случае команда minikube start вернёт ошибку: machine does not exist
$ minikube delete # Чтобы исправить это, нужно очистить локальное состояние
```

---

## 1.2.) Запускаем несколько подов `nginx`, `mysql` и `wildfly`

#### NGINX
- Создаем конфиг:
```bash
$ nano nginx.yaml
```
- Пишем конфиг пода:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
```
- Запускаем под:
```bash
kubectl apply -f nginx.yaml
```

#### WILDFLY
- Создаем конфиг:
```bash
$ nano wildfly.yaml
```
- Пишем конфиг пода:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: wildfly-pod
spec:
  containers:
  - name: wildfly-container
    image: jboss/wildfly
```
- Запускаем под:
```bash
$ kubectl apply -f wildfly.yaml
```

#### MYSQL
- Создаем конфиг:
```bash
$ nano mysql.yaml
```
- Пишем конфиг пода:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql-container
    image: mysql:8.0
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: {наш пароль}
```
- Запускаем под:
```bash
$ kubectl apply -f mysql.yaml
```

---

## 1.3.) Проверяем состояние подов контейнеров
```bash
$ kubectl get po -A
```
---

## 1.4.) Посмотреть утилизацию контейнеров
```bash
$ minikube addons enable metrics-server  # Включаем метрики сервера в кластере
$ kubectl top po  # Смотрим утилизацию по CPU и RAM
```

---

## 1.5.) Смотрим логи контейнеров
```bash
$ kubectl logs mysql-pod
$ kubectl logs nginx-pod
$ kubectl logs wildfly-pod
```

---

## 1.6.) Подключаемся к контейнеру
```bash
$ kubectl exec -it nginx-pod -- bash
$ ls -ali  # Проверяем проходят ли команды
```

---

## 1.7.) Останавливаем контейнеры
```bash
$ minikube stop  # Можем остановить все
$ kubectl delete po nginx-pod  # Остановить контейнеры выборочно
```

---

## 2.1.) Собираем свой контейнер используя UBI 7.7 в качестве базового образа
UBI 7.7 - Это образ ОС Linux фактически голый

```
2.2.) С установленным apache (httpd)
2.3.) Httpd должен запускаться при запуске контейнера
2.4.) Заменить содержимое домашней страницы HTTPD по умолчанию (/var/www/html/index.html)
2.5.) Запустить созданный контейнер
2.6.) Проверить доступность  curl http://....
2.7.) В результате должно отобразиться содержимое index.html
```

- Создаём `Dockerfile`
```bash
$ nano Dockerfile
```
- Пишем конфиг `Dockerfile`:
```yml
FROM registry.access.redhat.com/ubi7/ubi:7.7

# Устанавливаем apache (httpd)
RUN yum install -y httpd

# Замена содержимого домашней страницы HTTPD по умолчанию
COPY index.html /var/www/html/

# Указываем команду по умолчанию для запуска контейнера
CMD [ "httpd", "-D", "FOREGROUND" ]
```

- Создаём `index.html`
```bash
$ nano index.html
```
- Пишем конфиг `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Добро пожаловать в мой сайт</title>
</head>
<body>
    <h1>Sber DevOps success</h1>
    <p>Это моя домашняя страница</p>
</body>
</html>
```

- Создаём наш образ
```bash
$ docker build -t my-apache-image /home/andrey/
```

- Запускаем контейнер на основе нашего образа
```bash
$ docker run -d -p 80:80 my-apache-image
```

Ну и идём в браузер `localhost:80`
Либо курлим: `curl http://localhost:80`

---

## 3.) Вопросы по Linux
У меня есть инструкцию по Linux, которую писал несколько лет)
По факту прочтения технической литературы или же при различных кейсов пополнял ей и продолжаю пополнять
Ссыль: https://github.com/DragonDamage/Linux

1. Показать список дисков
`$ lsblk          # показать диски с партициями`


2. Показать список ФС
```bash
$ sudo fdisk -l  # показать диски
$ df -h          # показать диски
```

3. Как посмотреть занятость диска, ФС, каталога
```bash
$ du -hs ./folder  # посмотреть обьём директории
$ ncdu /folder     # посмотреть свободное и занятое место в директориях
```

4. Показать LVM тома и определить есть ли свободное место в группе
```bash
$ lvdisplay  # Отобразить информацию о всех LVM томах, включая их размеры, группы томов и местоположение
$ vgdisplay  # Отобразить информацию о группах томов, включая общий размер, использованное и свободное пространство
$ vgs        # Отобразить информацию о группах томов, включая общий размер и свободное пространство
```

5. Создать каталог, показать содержимое каталога
```bash
$ mkdir directory
$ ls /directory
```

6. Создать символическую ссылку
```bash
$ ln -s /home/andrey/name_directory name_link  # создаём символическую ссылку
```

7. Посмотреть список процессов
```bash
$ ps                     # посмотреть запущенные процессы
$ ps -u                  # посмотреть запущенные процессы под пользаком
$ ps -afx или ps -aux    # показать все запущенные процессы всех пользователей
$ ps auf | grep ssh      # показать пользователей подключенных по ssh
$ ps -T -p [№процесса]   # показать потоки процесса
$ ps -efl                # посмотреть список процессов с PID
```

8. Показать загруженность ОС
```bash
$ top      # диспечер задач ([Shift]+[P] фильтр по cpu, [Shift]+[M] фильтр по RAM, [V] отобразить процессы в виде древа, [Z] подсветить чуть покрасивее, [Shift]+[E] показать в КБ/МБ/ГБ)
$ htop     # диспечер задач с красивым отображением в % загрузки
$ atop     # диспечер задач с красивым отображением в % загрузки
$ free -h  # показать RAM общую и нагруженную
```

9. Показать утилизацию ресурсов
```bash
$ top         # CPU проц
$ free -h     # MEM оперативку
$ df -h       # HARD жесткий диск
$ ip -c a     # Сеть
$ ip -s link  # Отобразить статистику сетевых интерфейсов, включая количество отправленных и принятых пакетов, ошибки, отброшенные пакеты и другую информацию о сетевом трафике
```

10. Посмотреть состояние службы
```bash
$ systemctl status nginx  # Посмотреть состояние службы
```

11. Посмотреть логи службы
```bash
$ journalctl -u nginx.service     # Посмотреть логи службы
$ cat /var/log/apache2/error.log  # Либо посмотреть через файлы журналов
```




## 4. Задание со звездочкой. Дополнительное.
В minikube уже есть развернутый под с nginx.
Рядом поднять отдельный инстанс zabbix (можно на виртуалке, можно в контейнере – не важно)
который будет дергать страницу nginx из предыдущего пункта, по изменению наполнения – отправлять письмо на указанный список адресов почты с алертом и содержанием страницы.
Условно, был 0, стал 1 – прислать на почту письмо “Alarm! Page changed value 0 to 1”.


