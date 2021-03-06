## Домашнее задание к занятию "13.2 разделы и монтирование"  </br>
Приложение запущено и работает, но время от времени появляется необходимость передавать между бекендами данные. А сам бекенд генерирует статику для фронта. Нужно оптимизировать это. </br>

### Задание 1: подключить для тестового конфига общую папку </br>
* в поде подключена общая папка между контейнерами (например, /static); </br>
* после записи чего-либо в контейнере с беком файлы можно получить из контейнера с фронтом. </br>
Установил nfs сервис через helm, и создал 2 pvc. YAML для pvc: [make_2_volumes.yaml](https://github.com/murzinvit/13.02_kubernetes_config_mounts/blob/02e0d8a13c6d3df1eb7df4ae46d261d26e364173/make_2_volumes.yaml), YAML для pod: [make_pod3.yaml](https://github.com/murzinvit/13.02_kubernetes_config_mounts/blob/02e0d8a13c6d3df1eb7df4ae46d261d26e364173/make_pod3.yaml) </br>
Важный момент установка на каждой ноде драйвера nfs-common </br>
Первый для БД, второй как общая папка для контейнеров: </br>
![2_claims_db_nfs](https://github.com/murzinvit/screen/blob/507b8da79b31bd402904c0a9c28fca6cca8c735f/Kuber_2_claims_db_nfs.jpg) </br>
Зашёл в контейнер backend в поде netology-devops6-app и в папке /mnt создал файл - hello: </br>
![Kuber_exec_backend](https://github.com/murzinvit/screen/blob/cc93d778950d0c902cbf0c14d0a54dc641207ba8/Kuber_exec_backend.jpg) </br>
Зашёл в контейнер frontend в папке /mnt видно файл hello: </br>
![Kuber_exec_frontend](https://github.com/murzinvit/screen/blob/cc93d778950d0c902cbf0c14d0a54dc641207ba8/Kuber_exec_frontend.jpg) </br>

### Задание 2: подключить общую папку для прода </br>
Поработав на stage, доработки нужно отправить на прод. В продуктиве у нас контейнеры крутятся в разных подах, поэтому потребуется PV и связь через PVC. </br>
Сам PV должен быть связан с NFS сервером. Требования: </br>
* все бекенды подключаются к одному PV в режиме ReadWriteMany; </br>
* фронтенды тоже подключаются к этому же PV с таким же режимом; </br>
* файлы, созданные бекендом, должны быть доступны фронту. </br>
Создал 2 pvc в неймспейс prod: YAML для создания pvc: [2_pvc_in_prod.yaml](https://github.com/murzinvit/13.02_kubernetes_config_mounts/blob/e8ee65769ed4eec97f601a1edb20c0127de4985d/2_pvc_in_prod.yaml) </br>
Поднял 3 деплоймента из. YAML для создания deployments: [make_deployment_app.yaml](https://github.com/murzinvit/13.02_kubernetes_config_mounts/blob/e8ee65769ed4eec97f601a1edb20c0127de4985d/make_deployment_app.yaml) </br>
Зашёл на под в деплойменте backend и создал файл hello2 в папке /mnt (где примонтирован pvc): </br>
![Kuber_hello2_in_deployment](https://github.com/murzinvit/screen/blob/dc6224596a2298da2236da21169d8ce5bc52bb61/Kuber_hello2_in_deployment.jpg) </br>
Зашёл в под в деплойменте frontend в папку /mnt: </br>
![Kuber_hello2_in_front](https://github.com/murzinvit/screen/blob/584f3bc470582e5543e0d23b4a01dfa47c6de594/Kuber_hello2_in_front.jpg) </br>


### Рабочие заметки: </br>
Настройка отдельно nfs сервера: http://debian-help.ru/articles/nastroika-nfs-servera-debian/ </br>
Установка helm и установка nfs-server через helm: (производить под пользователем на сервере, у которого есть доступ до kubectl) </br>
* установить helm: curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash </br>
* добавить репозиторий чартов: helm repo add stable https://charts.helm.sh/stable && helm repo update </br>
* установить nfs-server через helm: helm install nfs-server stable/nfs-server-provisioner </br>
`kubectl exec netology-devops6-app -c frontend -it -- bash` </br>
