# How to up the service
Репозиторий для производственной практики

1)Сделать git clone git@github.com:dadayasho/practics.git

2)Выгрузив репозиторий, поднять docker-compose командой

```
docker-compose up -d
```
В данном docker-compose файле содержится sonarqube и postgres конфигурация.

3)После поднятия docker-compose перейти по указанному url и порту.

4)Авторизоваться -> Перейти во вкладку "Projects" -> "Create project" -> "Manually" -> Ввести имя проекта и ключ, по которому будет идти обрпащение при сканирвоании.

5)Далее нажать кнопку "Locally", сгенерировать токен, перейти в CLI в хранилище проекта и начать скан.

6) Скачать через терминал sonar-scanner

Далее будут рассмотрены особенности скана для Python и Java(Gradle).

## Python

### Через CLI

В терминале в разделе проекта запустить команду 
```
sonar-scanner \
  -Dsonar.projectKey=<project_key> \
  -Dsonar.sources=. \ # указывает на корневой репозиторий
  -Dsonar.host.url=<sonarqube_url> \
  -Dsonar.login=<generated_token>

```
<project_key> - ключ репозитория \
    <sonarqube_url> - url ссылка с портом на вебсервер sonarqube \
    <generated_token> - сгенерированный токен 

### Через sonar-project.properties

Создать файл sonar-project.properties в каталоге проекта

```
# Ключ проекта (уникальный)
sonar.projectKey=<project_key>
sonar.projectName=<project_name>
sonar.projectVersion=1.0

# Пути к исходному коду
sonar.sources=src # Папка исходного кода  
sonar.tests=tests  

# Язык анализа
sonar.language=py

# Покрытие кода (если есть, опционально)
sonar.python.coverage.reportPaths=coverage.xml

# Адрес сервера SonarQube
sonar.host.url=<sonarqube_url>
sonar.login=<sonarqube_token>
```

<project_key> - ключ репозитория \
	<project_name> - название проекта \
    <sonarqube_url> url - ссылка с портом на вебсервер sonarqube \
    <generated_token> - сгенерированный токен

Далее запустить команду `sonar-scanner`

**Просмотреть результат в самом sonarqube.**


