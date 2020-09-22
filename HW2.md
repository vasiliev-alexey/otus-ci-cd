# Доработка пайплайна

## Цель: Доработать ваш имеющийся пайплайн с предыдущего урока с применением include и extends

---

## Решение

### Extends
По заявлению в документации Extends - позволяет  создавать наследоваение в конфигурации серез механизм слияний конфигураций.
Наследование задается конструкцией

~~~ yaml
# подключим job из другого файла - если мы шарим ее между конфигурациями
include: deploy.yml

...

deploy1:
  stage: deploy
  ## наследуемся включенной job
  extends: .template_deploy
~~~

Результатом - будем смерженный job  вида

~~~ yaml
deploy1:
  stage: deploy
  script:
    - echo "Супер деплой"
~~~


### Include
Команда Include позволяет подключать содержимое иных yaml файлов качестве  источника конфигурации элементов для pipeline.
В качестве источников можно использовать:

* *local* - файлы в локальной директории проекта
* *file* - файлы из директорий других проектов
* *remote* - файлы доступные из инфх источников данных, например по сети
* *template* - файлы из  шаблонов GitLab

Файлы с Include, в момент планирования исполнения, будут смержены с основным  по следующим правилам:
* Иерархическая вложенность
* Include всегда расматриваются в первую очередь, и могут быть переопределены значениями из .gitlab-ci.yml



Использованы в нашем [пайплайне](https://gitlab.com/vasiliev-alexey/otus-ci-cd/-/blob/master/.gitlab-ci.yml) local, file, remote
~~~ yaml
include:
  - project: 'vasiliev-alexey/ci-cd-dummy'
    file: '/scripts/deploy.yml'
  - local: '/deploy.yml'
  - remote: 'https://gist.githubusercontent.com/vasiliev-alexey/896d1e12a72de15a7e75f1d29e078560/raw/20a21d9e486e8c612b31ea28b76ca149a64ce399/deploy.yml'
~~~

Создали 3 деплоя, каждый из котрорых расширяет джобы из того или иного источника.
~~~ yaml
deploy_local:
  variables:
    GITLAB: "вжух и полетело"
  stage: deploy
  extends: .template_deploy


deploy_file:
  variables:
    GITLAB: "вжух и полетело"
  stage: deploy
  extends: .template_deploy_file

deploy_file_remote:
  variables:
    GITLAB: "вжух и полетело"
  stage: deploy
  extends: .template_deploy_remote

~~~

[Успешно отработал сценарий](https://gitlab.com/vasiliev-alexey/otus-ci-cd/-/pipelines/193096336)

~~~ sh
Or perhaps you might print out some debugging details
$ echo "Супер деплой из конфигурации передаваемой через GITHUB Gist" $CI_JOB_STAGE
Супер деплой из конфигурации передаваемой через GITHUB Gist deploy
~~~

---
## Материалы
[GitLab configuration reference - include ](https://docs.gitlab.com/ee/ci/yaml/README.html#include)

[GitLab configuration reference - extends ](https://docs.gitlab.com/ee/ci/yaml/README.html#extends)


[GitLab- примеры include](https://docs.gitlab.com/ee/ci/yaml/includes.html)

[Сборка проектов с GitLab CI: один .gitlab-ci.yml для сотни приложений](https://habr.com/ru/company/flant/blog/340996/)

[Переиспользование кода в pipeline](https://medium.com/@imalik8088/tired-of-repeated-gitlab-ci-files-includes-to-the-rescue-17225532812a)