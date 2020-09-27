#  Автоматическая сборка и тестирование 

## Цель: написать первый пайплайн для автоматической сборки проекта и модульного тестирования через несколько утилит, линтеры и анализаторы.

### Домашнее задание
Написание пайплайна для модульного тестирования и сборки
1. определить стадии движения продукта для выбранного проекта
2. описать в файле gitlab-ci пайплайн для поочередного выполнения
3. добавить модульные тесты, линтеры
4. добавить сборку
5. вынести сборку и тестирование в отдельные ci-файлы (в другом репозитории)
6. включить вынесенные файлы в исходный пайплайн как зависимости
5. (необязательно) добавить шаг деплоя 

---

## Решение
* Создаем инфраструктурный проект по созданию кластера Kubernetes на языке HCL, используя инструмент Terraform
* Исходные коды размещаем в [проекте](https://gitlab.com/vasiliev-alexey/gke-tf-cluster/)
* Общая часть по валидации и созданию плана инфраструктуры - вынесена в отдельный [вспомогательный проект](https://gitlab.com/vasiliev-alexey/otus-ci-cd)  
* Деплой  (Создание и разрушение инфраструктуры - вынесены в в ручные операции)

[общий конфиг сборки](https://gitlab.com/vasiliev-alexey/gke-tf-cluster/-/blob/master/.gitlab-ci.yml) выглядит так

~~~ yaml
include:
  - project: 'vasiliev-alexey/ci-cd-dummy'
    file: '/tf_jobs/plan.yml'
  - project: 'vasiliev-alexey/ci-cd-dummy'
    file: '/tf_jobs/tf_lint.yml'
stages:
 - validate
 - lint
 - plan
 - apply
 - destroy

image:
  name: hashicorp/terraform:0.12.26
  entrypoint: [""]

.common_tags:
  tags:
    - docker



before_script:
  - GOOGLE_BACKEND_CREDENTIALS=$service_account
  - TF_LOG=1 terraform init  || true


tf_lint:
  stage: lint
  extends: 
   - .common_tags
   - .terralint


validate:
  extends: .common_tags
  stage: validate
  script:
    - terraform validate
  artifacts:
    paths:
      - .terraform

plan:
  extends: 
   - .common_tags
   - .template_tf_plan
  stage: plan

apply:
  extends: .common_tags
  stage: apply
  script: 
    - GOOGLE_CLOUD_KEYFILE_JSON=$service_account
    - echo $GOOGLE_CLOUD_KEYFILE_JSON
    - terraform apply -auto-approve -var project_name=${PROJECT_ID} -var cred_file=${service_account}
  when: manual
  only:
    - master

destroy:
  extends: .common_tags
  stage: apply
  script: terraform destroy -auto-approve -var project_name=${PROJECT_ID} -var cred_file=${service_account}
  when: manual
  only:
    - master
~~~

ps: Есть проблемы с сохранением backend на cloud bucket -  решим позднее