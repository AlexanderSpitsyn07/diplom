## Этапы выполнения:


### Создание облачной инфраструктуры

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
2. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform: 
   б. Альтернативный вариант: S3 bucket в созданном ЯО аккаунте
3. Настройте [workspaces](https://www.terraform.io/docs/language/state/workspaces.html) 
   а. Рекомендуемый вариант: создайте два workspace: *stage* и *prod*. В случае выбора этого варианта все последующие шаги должны учитывать факт существования нескольких workspace. 
![Screenshot](1-1.png)
![Screenshot](1-2.png)
4. Создайте VPC с подсетями в разных зонах доступности.
![Screenshot](1-3.png)
![Screenshot](1-4.png)
5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
6. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера. 
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений. 
![Screenshot](2-1.png)

   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/) 
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
![Screenshot](2-2.png)
![Screenshot](2-3.png)

---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.
https://github.com/AlexanderSpitsyn07/test-for-dockerfile
   б. Подготовьте Dockerfile для создания образа приложения. 
![Screenshot](3.png)

Ожидаемый результат:
1. Git репозиторий с тестовым приложением и Dockerfile.
https://github.com/AlexanderSpitsyn07/test-for-dockerfile
2. Регистр с собранным docker image. В качестве регистра может быть DockerHub.
![Screenshot](3-1.png)
---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера. 
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
Альтернативный вариант:
Для организации конфигурации можно использовать [helm charts](https://helm.sh/)

![Screenshot](4-1.png)
![Screenshot](4-6.png)
![Screenshot](4-8.png)
![Screenshot](4-9.png)
![Screenshot](4-10.png)
![Screenshot](4-11.png)
![Screenshot](4-12.png)
![Screenshot](4-13.png)


2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.
![Screenshot](4-2.png)
![Screenshot](4-3.png)
![Screenshot](4-4.png)
![Screenshot](4-5.png)

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
https://github.com/AlexanderSpitsyn07/repoan
2. Http доступ к web интерфейсу grafana.
http://130.193.51.22:32433
UserName: admin
Password: prom-operator
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
В качестве основного Dashboard экспортировал https://grafana.com/grafana/dashboards/315, скриншоты выше.
4. Http доступ к тестовому приложению.
http://130.193.51.22:32764/


---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Использовал jenkins.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
![Screenshot](5.png)
http://130.193.51.22:8080/
admin/admin
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистр, а также деплой соответствующего Docker образа в кластер Kubernetes.
![Screenshot](5-1.png)
![Screenshot](5-2.png)
![Screenshot](5-3.png)
![Screenshot](5-4.png)
Файл с pipiline в этом же репозитории.
---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
https://github.com/AlexanderSpitsyn07/dipter
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
https://github.com/AlexanderSpitsyn07/repoan
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
https://github.com/AlexanderSpitsyn07/test-for-dockerfile
docker pull chehvostik/webserver:latest
5. Репозиторий с конфигурацией Kubernetes кластера.
https://github.com/AlexanderSpitsyn07/repoan
6. Ссылка на тестовое приложение - http://130.193.51.22:32764 - и веб интерфейс Grafana с данными доступа.
http://130.193.51.22:32433
UserName: admin
Password: prom-operator
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)

