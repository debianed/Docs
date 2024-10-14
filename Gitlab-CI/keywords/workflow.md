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

## workflow:rules

## workflow:rules:variables

## workflow:rules:auto_cancel

