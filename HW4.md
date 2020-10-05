#  Gitlab CI и Docker: Services, Artifacts, Rules 

## Цель:  Добавить условия для сборки докер-образа - если менялся код приложения или докерфайл

### Домашнее задание
Добавить условия для сборки докер-образа - если менялся код приложения или докерфайл

---

## Решение
В рамках другого курса, реализовал сборку Docker-образов для  типового приложения на основе  архитектурного шаблона микросервисов.

* [репозиторий с исходными кодами](https://gitlab.com/vasiliev-alexey/microservices-demo)
* [Пайплайн для сборки и шаблоны для Job ](https://gitlab.com/vasiliev-alexey/microservices-demo/-/blob/master/.gitlab-ci.yml)
* [Таски для сборки образов ](https://gitlab.com/vasiliev-alexey/microservices-demo/-/blob/master/ci_cd/build.yaml)
* [Таски для тегирования образов по шаблону семвер от тегов гита ](https://gitlab.com/vasiliev-alexey/microservices-demo/-/blob/master/ci_cd/build.yaml)


Шаблон сборки Docker-образов микросервиса

~~~ yaml
.build_common:
  extends: .docker_template
  stage: build
  only:
    changes:
      - src/$SERVICE_NAME/**/* # только для изменения исходного кода
  script:
    - cd src/$SERVICE_NAME  
    - docker build . -f Dockerfile -t $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_SHORT_SHA
    - docker push $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_SHORT_SHA
~~~

Шаблон для тегирования Docker-образов по тегу от гита

~~~ yaml
.push_common:
  extends: .docker_template
  stage: push
  variables:
    GIT_STRATEGY: none
  only: # только для тегов по семверу
    refs:
      - /^v\d+\.\d+\.\d+/
    changes:
      - src/$SERVICE_NAME/**/*
  script:
    - docker pull $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_SHORT_SHA
    - docker tag  $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_SHORT_SHA $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_TAG
    - docker push $DOCKER_REGISTRY_USER/$SERVICE_NAME:$CI_COMMIT_TAG

~~~