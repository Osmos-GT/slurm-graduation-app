# Репозитарий приложения YELB.

Это репозитарий с кодом учебного приложения Yelb.  

Код, необходимый для создания и настройки инфраструктуры, необходимой для запуска этого приложения, находится в [другом репозитарии](https://yelb-repo.gitlab.yandexcloud.net/yelb/yelb-iac). Используется инфраструктура Yandex cloud.

Приложение состоит из следующих сервисов:

* yelb-appserver
* yelb-ui
* база данных Postgresql
* сервер Redis

Построенный в данном репозитарии пайплайн собирает, проверяет и деплоит yelb-appserver и yelb-ui. База данных и Redis деплоятся в пайплайне репозитория IaC в рамках подготовки инфраструктуры.

После установки приложение будет доступно по адресу https://s046013.fun/

## Пайплайн

Пайплайн предусматривает, что загрузку в хранилище образов и деплой проходят только протестированные образы.

Пайплайн имеет следующие этапы:

#### Lint
Параллельно выполняющиеся yamllint и helm lint.

#### Build
Параллельная сборка appserver и ui.

#### Test-appserver
Тестирование appserver-сервиса.

#### Test-ui
Тестирование ui-сервиса.

#### Push
Загрузка собранных образов после их успешного тестирования.

#### Cleanup
Удаляются все собранные образы appserver и ui, кроме последних. Это позволяет ускорить следующую сборку с помощью кеша, и в то же время не захламлять ноду, где работает раннер.

#### Obtain-deploy-token
Если в хранилище нет переменных с деплой-токеном, то в пайпплайн добавляется этот этап.  
Полученные через API имя пользователя и пароль добавляются как новые переменные.

#### Deploy
Деплой приложения в k8s.

## Переменные

IaC-репозитарий и настоящий репозитарий находятся в одной группе Gitlab, поэтому используют одно хранилище переменных.

Перед началом работы необходимо добавить в Gitlab следующие переменные:

- GITLAB_PRIVATE_TOKEN  
Private token аккаунта в гитлабе, использующийся для работы с API gitlab. 

- SA_KEY  
Ключ сервисного аккаунта, от имени которого Terraform будет создавать и удалять ресурсы.
Можно получить так:  
```yc iam key create --service-account-name <имя сервисного аккаунта> --output key.json --folder-id <id каталога в облаке>```

- TF_VAR_YC_CLOUD_ID  
ID облака

- TF_VAR_YC_FOLDER_ID  
ID каталога в облаке

- YELB_DB_USER  
Имя пользователя для подключения к базе данных

- YELB_DB_PASS  
Пароль для подключения к базе данных

- YELB_DB_PORT  
Номер порта для подключения к базе данных

В ходе выполнения IaC-пайплайна через API гитлаба будет создана переменная YELB_DB_ADDR с адресом мастер-хоста базы данных.

При выполнении пайплайна в данном репозитарии через API будут запрошены в Gitlab и добавлены переменные DEPLOY_TOKEN_USER и DEPLOY_TOKEN_PASS для доступа к хранилищу образов в случае, если они отсутствуют.