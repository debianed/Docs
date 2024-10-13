# default
Для некоторых ключевых слов можно задать глобальные значения по умолчанию. Каждое ключевое слово по умолчанию копируется в каждое задание, для которого оно еще не определено. Если для задания уже определено ключевое слово, это значение по умолчанию не используется.

**Тип слова:** Глобальное
**Возможные значения:**
Данные ключевые слова могут иметь пользовательские значения по умолчанию:
- after_script
- artifacts
- before_script
- cache
- hooks
- id_tokens
- image
- interruptible
- retry
- services
- tags
- timeout (однако из-за [проблемы 213634](https://gitlab.com/gitlab-org/gitlab/-/issues/213634) это ключевое слово не имеет никакого эффекта)

**Пример:**
```YAML
default:
  image: ruby:3.0
  retry: 2

rspec:
  script: bundle exec rspec

rspec 2.7:
  image: ruby:2.7
  script: bundle exec rspec
```