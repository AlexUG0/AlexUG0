# Создание и настройка python-проекта

**Tags**: `Backend`, `Python`
**Type**: `Article`

## Установка зависимостей

### Poetry

Следующий полезный инструмент в Python разработке - [`poetry`](https://python-poetry.org/). Это довольно умный, а главное быстрый менеджер зависимостей для Python. Он позволяет:

1. Инициализировать Python-проекты
2. Создавать виртуальные окружения разработки
3. Управлять версиями зависимостей, выстраивать корректное дерево зависимостей без конфликтов и при том быстрее Conda
4. Разделять зависимости на prod и dev  
5. Собирать проект в пакет и управлять публикацией пакета

Инструкция по установке poetry для всех ключевых ОС на [этой](https://python-poetry.org/docs/#installation) странице официальной документации.

## Делаем проект

### Создание проекта с `poetry`

При помощи `poetry` мы можем создать новый python-проект. Фактически формального понятия того, что является python-проектом - нет, так как сам python оперирует понятиями модулей и пакетов, хотя можно опереться на [PEP 518](https://www.python.org/dev/peps/pep-0518/).

Любой `.py` файл является Python Module.

Папка с `__init__.py` - Python Package.

Однако, если посмотреть на различные Python open source проекты и опереться на тот гайдлайн, который предлагает poetry, то мы получим достаточно хорошо организованную структуру папок, от которой можно успешно оттолкнуться.

Для того, чтобы создать пустой проект при помощи `poetry` есть команда [`new`](https://python-poetry.org/docs/cli/#new):

```bash
poetry new <package-name>
```

Данная команда создает следующую структуру папок:

```bash
my-package
├── pyproject.toml
├── README.rst
├── my_package
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_my_package.py
```

Внимание стоит обратить на файл `pyproject.toml`. Этот файл уже является частью спецификации Python, однако пока нельзя сказать, что он поддерживается целиком и полностью всеми проектами.

Содержательную часть, с точки зрения `poetry` можно прочитать в [документации](https://python-poetry.org/docs/pyproject/).

Так же за нас сразу создали `README.rst`, но я предпочитаю использование Markdown и меняю расширение файла на `.md`. Помимо этого, для нас сделали небольшой сетап для тестов, к тому же в процессе инициализации разработчик может ввести нужную информацию о проекте (версию, автора, описание, лицензию, prod/dev зависимости).

В общем к этому моменту проект уже оброс набором мета-информации и выглядит солидно. Начнем настраивать линтеры.

### Создание виртуального окружения

В этом нам тоже поможет `poetry` чтобы создать или активировать существующее виртуальное окружение вызовите команду:

```bash
poetry shell
```

В общем случае для управления виртуальными окружениями существует команда [`poetry env`](https://python-poetry.org/docs/managing-environments/).

### Установка dev зависимостей

Для того чтобы поставить пакет используется команда `poetry add`. В частности, чтобы поставить некоторую зависимость только для разработки нужно так же прописать флаг `—-dev`. Установим следующие пакеты:

```bash
poetry add --dev flake8 mypy black isort
```

Этот список dev зависимостей - минимальный. Помимо этого, я рекомендую так же установить `pydocstyle`, `bandit`, а лучше [`yastyleguide`](https://github.com/levkovalenko/yastyleguide).

[`flake8`](https://github.com/PyCQA/flake8) - один из многих доступных линтеров Python-кода, следящий за качеством кода с множеством плагином. Сам фактически является оберткой над `pyckdestyle`, `pyflakes`, `mccabe`.

[`mypy`](http://www.mypy-lang.org/) - проверяет аннотации типов Python на корректность. Фактически благодаря этому инструменту в Python как бы «есть» статическая типизация. У него есть альтернатива - `pyright`.

[`black`](https://github.com/psf/black) - форматтер кода, IMHO очень крутой форматтер кода. Есть ряд разработчиков, в том числе те, кто делают [`wemake-python-styleguide`](https://github.com/wemake-services/wemake-python-styleguide), которые считают, что разработчик сам на основе линтера должен форматировать код. Я считаю это неверным, потому что это:

1. Долго
2. Требует отдельного времени на подумать и пофиксить проблемы
3. Может допускать разночтения, что противоречит философии python

Форматтер за программиста, сам возьмет и все приведет к нужному виду, при этом будет гарантировать, что этот общий вид будет у всех разработчиков поддерживаться одинаковым.

`black` выбран в силу того, что он не просто сделан таким образом, что кому-то нравится и, кажется, такое форматирование корректным, а потому что в основе его формата лежат научные исследования.

[`isort`](https://github.com/PyCQA/isort) - инструмент, который за вас форматирует порядок `import`’ов зависимостей в файлах.

### Настройка линтеров и форматтеров

Большая часть инструментов в данный момент начинает использовать `.toml` файлы для конфигурации, однако далеко не все. В частности, `flake8` еще [не умеет](https://github.com/PyCQA/flake8/issues/234) это делать. Поэтому лично я храню конфигурации следующим образом:

1. Конфигурация для `flake8` в `.flake8` в корне проекта
2. Конифигурации для `black`, `mypy`, `isort` в `pyproject.toml`

Не буду приводить листинги моих конфигураций, но отмечу, что `flake8` по умолчанию считает корректной длину строки в 79 символов, а `black` в 88. К тому же, `isort` производит некорректную с точки зрения `black` сортировку импортов и их нужно “подружить”. Для этого в `.flake8` нужно добавить:

```
max-line-length=119
```

А в секции `pyproject.toml`:

```
[tool.isort]
# максимальная длина строки
line_length = 119


[tool.mypy]
disallow_untyped_defs = true
no_implicit_optional = true
warn_return_any = true
exclude = 'venv'

[tool.black]
# Максимальная длина строки
line-length = 119
# Файлы, которые не нужно форматировать
exclude = '''
(
  /(
      \.eggs         # Исключить несколько общих каталогов
    | \.git          # в корне проекта
    | \.hg
    | \.mypy_cache
    | \.tox
    | \.venv
    | dist
  )/
  | foo.py           # Также отдельно исключить файл с именем foo.py
                     # в корне проекта
)
'''
```
