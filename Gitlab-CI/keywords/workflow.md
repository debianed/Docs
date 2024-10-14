# workflow
Используйте workflow для управления поведением конвейера.

Вы можете использовать некоторые предопределенные переменные CI/CD в конфигурации workflow, но не переменные, которые определяются только при запуске заданий.

**Связанные темы:**
- [Примеры workflow: rules](https://docs.gitlab.com/ee/ci/yaml/workflow.html#workflow-rules-examples)
- [Переключение между конвейерами ответвлений и конвейерами запросов на слияние](https://docs.gitlab.com/ee/ci/yaml/workflow.html#switch-between-branch-pipelines-and-merge-request-pipelines)

## workflow:auto_cancel:on_new_commit
Используйте workflow:auto_cancel:on_new_commit, чтобы настроить поведение функции автоматической отмены избыточных конвейеров.

**Возможные значения:** 
- conservative: Отмените конвейер, но только в том случае, если еще не запущены задания с interruptible: false. По умолчанию, если не определено.
- interruptible: Отменяйте только задания с прерываемым значением: true.
- none: Не выполняйте автоматическую отмену каких-либо заданий.

**Пример:**
```YAML
workflow:
  auto_cancel:
    on_new_commit: interruptible

job1:
  interruptible: true
  script: sleep 60

job2:
  interruptible: false  # Default when not defined.
  script: sleep 60
```

В этом примере:
- Когда новый коммит отправляется в ветку, GitLab создает новый конвейер и запускает job1 и job2.
- Если новый коммит отправляется в ветку до завершения заданий, отменяется только задание 1.

## workflow:auto_cancel:on_job_failure
Используйте workflow:auto_cancel:on_job_failure, чтобы настроить, какие задания следует отменять при сбое одного задания.

**Возможные значения:** 
all: Отмените конвейер и все запущенные задания, как только произойдет сбой одного из них.
none: Не выполняйте автоматическую отмену каких-либо заданий.

**Пример:**
```YAML
stages: [stage_a, stage_b]

workflow:
  auto_cancel:
    on_job_failure: all

job1:
  stage: stage_a
  script: sleep 60

job2:
  stage: stage_a
  script:
    - sleep 30
    - exit 1

job3:
  stage: stage_b
  script:
    - sleep 30
```

В этом примере, если задание 2 завершается неудачей, задание 1 отменяется, если оно все еще выполняется, а задание 3 не запускается.

**Связанные темы:**
- [Автоматическое отключение родительского конвейера от последующего конвейера](https://docs.gitlab.com/ee/ci/pipelines/downstream_pipelines.html#auto-cancel-the-parent-pipeline-from-a-downstream-pipeline)

## workflow:name
Вы можете использовать name в workflow:, чтобы задать имя для конвейеров.
Всем конвейерам присваивается определенное имя. Все начальные и конечные пробелы в названии удаляются.

**Возможные значения:** 
- Строка
- [Переменные CI/CD](https://docs.gitlab.com/ee/ci/variables/where_variables_can_be_used.html#gitlab-ciyml-file)
- Их комбинация

**Пример:**
Простое имя конвейера с предопределенной переменной:
```YAML
workflow:
  name: 'Pipeline for branch: $CI_COMMIT_BRANCH'
```
Конфигурация с различными названиями конвееров в зависимости от условий эксплуатации:
```YAML
variables:
  PROJECT1_PIPELINE_NAME: 'Default pipeline name'  # A default is not required.

workflow:
  name: '$PROJECT1_PIPELINE_NAME'
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        PROJECT1_PIPELINE_NAME: 'MR pipeline: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME'
    - if: '$CI_MERGE_REQUEST_LABELS =~ /pipeline:run-in-ruby3/'
      variables:
        PROJECT1_PIPELINE_NAME: 'Ruby 3 pipeline'
    - when: always  # Other pipelines can run, but use the default name
```

**Дополнительные сведения:**
- Если имя является пустой строкой, конвейеру не присваивается имя. Имя, состоящее только из переменных CI/CD, может быть преобразовано в пустую строку, если все переменные также пусты.
- workflow:rules:variables становятся глобальными переменными, доступными во всех заданиях, включая задания-триггеры, которые по умолчанию передают переменные в нижестоящие конвейеры. Если в нижестоящем конвейере используется та же переменная, она заменяется значением вышестоящей переменной. Убедитесь, что либо:
  - Используйте уникальное имя переменной в конфигурации конвейера каждого проекта, например, PROJECT1_PIPELINE_NAME.
  - Используйте inherit:variables в задании запуска и укажите переменные, которые вы хотите передать в нисходящий конвейер.

## workflow:rules
Ключевое слово rules в workflow аналогично правилам, определенным в jobs, но определяет, будет ли создан весь конвейер.
Если ни одно из правил не принимает значение true, конвейер не запускается.

**Возможные значения:**
Вы можете использовать некоторые из тех же ключевых слов, что и в правилах на уровне заданий:
- rules: if
- rules: changes
- rules: exists
- when (может быть задано только always или never при использовании с workflow)
- variables

**Пример:**
```YAML
workflow:
  rules:
    - if: $CI_COMMIT_TITLE =~ /-draft$/
      when: never
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

В этом примере конвейеры запускаются, если заголовок фиксации (первая строка сообщения о фиксации) не заканчивается на -draft и конвейер предназначен либо для:
- Запрос на слияние
- Ветвь по умолчанию.

**Дополнительные сведения:**
- Если ваши правила соответствуют как конвейерам ответвлений (отличным от ветки по умолчанию), так и конвейерам запросов на слияние, могут появляться дублирующиеся конвейеры.

**Связанные темы:**
- [Общие условия if для рабочего workflow:rules](https://docs.gitlab.com/ee/ci/yaml/workflow.html#common-if-clauses-for-workflowrules).
- [Использование rules для запуска конвейеров запросов на слияние](https://docs.gitlab.com/ee/ci/pipelines/merge_request_pipelines.html#add-jobs-to-merge-request-pipelines).

## workflow:rules:variables
Вы можете использовать переменные в разделе рабочий workflow:rules для определения переменных для конкретных условий конвейера.
Если условие соответствует, создается переменная, которая может использоваться во всех заданиях конвейера. Если переменная уже определена на глобальном уровне, переменная рабочего процесса имеет приоритет и переопределяет глобальную переменную.

**Тип слова:** Глобальное

**Возможные значения:**
Пары "Имя переменной" и "значение":
- В имени могут использоваться только цифры, буквы и символы подчеркивания (_).
- Значение должно быть строковым.

**Пример:**
```YAML
variables:
  DEPLOY_VARIABLE: "default-deploy"

workflow:
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
      variables:
        DEPLOY_VARIABLE: "deploy-production"  # Override globally-defined DEPLOY_VARIABLE
    - if: $CI_COMMIT_REF_NAME =~ /feature/
      variables:
        IS_A_FEATURE: "true"                  # Define a new variable.
    - when: always                            # Run the pipeline in other cases

job1:
  variables:
    DEPLOY_VARIABLE: "job1-default-deploy"
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
      variables:                                   # Override DEPLOY_VARIABLE defined
        DEPLOY_VARIABLE: "job1-deploy-production"  # at the job level.
    - when: on_success                             # Run the job in other cases
  script:
    - echo "Run script with $DEPLOY_VARIABLE as an argument"
    - echo "Run another script if $IS_A_FEATURE exists"

job2:
  script:
    - echo "Run script with $DEPLOY_VARIABLE as an argument"
    - echo "Run another script if $IS_A_FEATURE exists"
```

Когда ветвь является ветвью по умолчанию:
- переменная DEPLOY_VARIABLE для job1 - это job1-deploy-production.
- переменная DEPLOY_VARIABLE для job2 - это deploy-production.

Когда ветвь является функциональной:
- переменная DEPLOY_VARIABLE для job1 имеет значение job1-default-deploy, а значение IS_A_FEATURE равно true.
- переменная DEPLOY_VARIABLE для job2 имеет значение default-deploy, а значение IS_A_FEATURE равно true.

Когда ветвь представляет собой что-то другое:
- переменная DEPLOY_VARIABLE для job1 - это job1-default-deploy.
- переменная DEPLOY_VARIABLE для job2 - это default-deploy.

**Дополнительные сведения:**
- workflow:rules: переменные становятся глобальными переменными, доступными во всех заданиях, включая задания-триггеры, которые по умолчанию передают переменные в нижестоящие конвейеры. Если в нижестоящем конвейере используется та же переменная, она заменяется значением вышестоящей переменной. Убедитесь, что либо:
    - используйте уникальные имена переменных в конфигурации конвейера каждого проекта, например, PROJECT1_VARIABLE_NAME.
    - используйте inherit:переменные в задании запуска и укажите точные переменные, которые вы хотите передать в нисходящий конвейер.

## workflow:rules:auto_cancel
Используйте workflow:rules:auto_cancel для настройки поведения workflow:auto_cancel:on_new_commit или функций workflow:auto_cancel:on_job_failure.

**Возможные значения:**
- on_new_commit: workflow:auto_cancel:on_new_commit
- on_job_failure: workflow:auto_cancel:on_job_failure

**Пример:**
```YAML
workflow:
  auto_cancel:
    on_new_commit: interruptible
    on_job_failure: all
  rules:
    - if: $CI_COMMIT_REF_PROTECTED == 'true'
      auto_cancel:
        on_new_commit: none
        on_job_failure: none
    - when: always                  # Run the pipeline in other cases

test-job1:
  script: sleep 10
  interruptible: false

test-job2:
  script: sleep 10
  interruptible: true
```

В этом примере для параметра workflow:auto_cancel:on_new_commit установлено значение "прерываемый", а для параметра workflow:auto_cancel:on_job_failure по умолчанию установлено значение "все" для всех заданий. Но если конвейер выполняется для защищенной ветви, правило переопределяет значение по умолчанию с помощью on_new_commit: none и on_job_failure: none. Например, если конвейер выполняется для:
- запускается незащищенная ветвь и новая фиксация, test-job1 продолжает выполняться, а test-job2 отменяется.
- запускается защищенная ветвь и новая фиксация, и test-job1 и test-job2 продолжают выполняться.
