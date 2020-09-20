# Домашнее задание  по теме "Установка и настройка GitLab и агентов сборки"

## Тема: Регистрация Gitlab Runner и запуск простейшего пайплайна

## Цель: Научиться запускать runner, создать простой шаблонный пайплайн

1) сделать репозиторий
2) запустить и привязать к проекту gitlab-runner
3) написать простейший .gitlab-ci.yml пайплайн по примеру с занятия
4) добиться успешного выполнения пайплайна на своём раннере
Критерии оценки: Успехом выполнения задания считается успешное выполнение пайплайна, можно приложить скриншот, или дать доступ к проекту (достаточно роли reporter) на логин 

---


## Решение

1. Сделали репозиторий
https://gitlab.com/vasiliev-alexey/otus-ci-cd

2. запустить и привязать к проекту gitlab-runner

* ознакомились с документацией по [GitLab Runner](https://docs.gitlab.com/runner/install/)
* Создали локально виртуальную машину на Ubuntu 20.04
* Установили runner
~~~ sh
 # Скачиваем устновочный пакет
  curl -LJO  https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb

# Установка 
sudo dpkg -i  gitlab-runner_amd64.deb

# Конфигурация
    #Создаем токен для Runner  в GitLab Сервер
    # Токен Gitlab-CI доступен на панели настроек CI / CD из пользовательского интерфейса:
    # https://gitlab.com/<account>/<repo>/settings/ci_cd

# Регестрируем
  gitlab-runner register

    https://gitlab.com/

Please enter the gitlab-ci token for this runner:
__masked__

Please enter the gitlab-ci description for this runner:
[my-runner]: my-runner

Please enter the gitlab-ci tags for this runner (comma separated):
my-runner,foobar
Registering runner... succeeded                     runner=66m_339h

Please enter the executor: docker-ssh+machine, docker, docker-ssh, parallels, shell, ssh, virtualbox, docker+machine, kubernetes:
docker

Please enter the default Docker image (e.g. ruby:2.1):
alpine:latest

Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 

~~~

3. написать простейший .gitlab-ci.yml пайплайн по примеру с занятия

Взял за шаблон - предоставленный GitLab  шаблон - и добавил в него свои теги.

~~~ yaml
# This file is a template, and might need editing before it works on your project.
# see https://docs.gitlab.com/ce/ci/yaml/README.html for all available options

# you can delete this line if you're not using Docker
image: busybox:latest

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"


build1:
  stage: build
  script:
    - echo "Do your build here"
  tags:
  - avasiliev

test1:
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
  tags:
  - avasiliev

test2:
  stage: test
  script:
    - echo "Do another parallel test here"
    - echo "For example run a lint test"
  tags:
  - avasiliev


deploy1:
  stage: deploy
  script:
    - echo "Do your deploy here"

~~~

4. добиться успешного выполнения пайплайна на своём раннере

Проверил на созданной виртуальной машине  - в момент исполнения pipeline запускаются контейнеры от имени runner

~~~ sh
➜  ~ sudo docker  ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
c9bf59f46f90        6858809bf669        "sh -c 'if [ -x /usr…"   1 second ago        Up Less than a second                       runner-2-bjxdnw-project-21056958-concurrent-0-4f871171ce1a5744-build-2
~~~

В логах GitLab - сообщения из Pipeline [присутствуют](https://gitlab.com/vasiliev-alexey/otus-ci-cd/-/jobs/746840638)



## Статьи
[Установка GitLab-Runner-а в Unix/Linux](https://linux-notes.org/ustanovka-gitlab-runner-a-v-unix-linux/)

[Настройка Gitlab-CI раннера на своем собственном сервер](http://itisgood.ru/2020/04/13/nastrojka-gitlab-ci-rannera-na-svoem-sobstvennom-server/)

