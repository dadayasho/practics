# Как пользоваться сервисом
Репозиторий для производственной практики

1) Сделать git clone git@github.com:dadayasho/practics.git

2) Выгрузив репозиторий

3) Установить Docker и Docker compose через [ansible-playbook](playbooks/docker-playbook.yml)

4) После установки следует поднять [Docker compose файл](docker-compose.yml) через команду:

```bash
docker compose up -d
```

*В данном docker-compose файле содержится sonarqube и postgres конфигурация.*
 
После поднятия docker-compose перейти по указанному url и порту.
Авторизоваться -> Перейти во вкладку **"Projects"** -> **"Create project"** -> **"Manually"** -> Ввести имя проекта и ключ, по которому будет идти обрпащение при сканирвоании.
Далее нажать кнопку **"Locally"**, сгенерировать токен, перейти в CLI в хранилище проекта и начать скан.
Скачать через терминал sonar-scanner или же с [официального сайта](https://docs.sonarsource.com/sonarqube-server/10.8/analyzing-source-code/scanners/sonarscanner/), распаковав архив в папку /opt.

```bash
cd /opt

sudo unzip /путь/к/скачанному/архиву/sonar-scanner-*.zip

export PATH="$PATH:/opt/sonar-scanner/bin"
```

6) Установить Jenkins через [Jenkins ansible файл](playbooks/jenkins-playbook.yaml)

После установки авторизоваться и установить плагины. Следует перейти в **"Manage Jnekins"** -> **"Plugins"** -> **"Advanced settings"** и вставить [плагины](jenkins-dependecies/plugins) во вкладке **"Deploy plugins"**

Найдите раздел **SonarQube servers** → нажмите **Add SonarQube**.
Заполните поля: 
Name: Произвольное имя (например, SonarQube-Prod).
Server URL: Адрес вашего SonarQube (например, http://localhost:9000).
Server authentication token:
Выберите Secret text → вставьте скопированный токен из SonarQube.
Нажмите Add, затем выберите токен из списка.
Сохраните настройки.

Настройка SonarScanner Tool в Jenkins
Перейдите в Manage Jenkins → Tools → SonarQube Scanner.
Нажмите **Add SonarQube Scanner** → выберите **Install automatically**.
Укажите путь к SonarScanner:
В разделе Global Tool Configuration:
Name: SonarScanner (или другое имя).
SONAR_RUNNER_HOME: Укажите путь к папке SonarScanner (если установлен вручную).
Либо оставьте галочку Install automatically, чтобы Jenkins сам загрузил утилиту.

7) Далее воспольоваться созданными [пайплайнами](jenkins-dependecies/jobs/python) и приступать к работе.

Далее будут рассмотрены особенности скана для Python и Java(Gradle).

## **Python**

### Через CLI

В терминале в разделе проекта запустить команду 
```bash
sonar-scanner \
  -Dsonar.projectKey=<project_key> \
  -Dsonar.sources=. \ # указывает на корневой репозиторий
  -Dsonar.host.url=<sonarqube_url> \
  -Dsonar.login=<generated_token>
```


`<project_key>` - ключ репозитория \
    `<sonarqube_url>` - url ссылка с портом на вебсервер sonarqube \
    `<generated_token>` - сгенерированный токен 

### Через sonar-project.properties

Создать файл sonar-project.properties в каталоге проекта

```bash
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


`<project_key>` - ключ репозитория \
`<project_name>` - название проекта \
`<sonarqube_url>` - ссылка с портом на вебсервер sonarqube \
`<generated_token>` - сгенерированный токен

Далее запустить команду `sonar-scanner`

**Просмотреть результат в самом sonarqube.**

## **Java**

Для анализа проектов, используюших Gradle следует предпринять следующие шаги:

Добавить строки в build.gradle:

```
plugins {
    id 'jacoco'                          
    id "org.sonarqube" version "3.5.0.2730" 
}
```

где `id 'jacoco'` - плагин покрытия тестами \
`id "org.sonarqube" version "3.5.0.2730"` - указание на плагин sonarqube (Версия подходит для текущей сборки).

Так же добавить генерацию отчета Jacoco после тестов.

```
test {
    finalizedBy jacocoTestReport
}

jacocoTestReport {
    reports {
        xml.required = true  
    }
}
```

И самое главное - добавить sonar properties

```
sonar {
    properties {
        property "sonar.projectKey", "project_key" # обязательно
        property "sonar.projectName", "project_name"
        property "sonar.projectVersion", "1.0"
        property "sonar.sources", "src_folder" # обязательно
        property "sonar.tests", "src_test"
        property "sonar.java.binaries", "build/classes/java/main" 
        property "sonar.language", "java"
        property "sonar.host.url", "project_url" # обязательно
        property "sonar.login", "<generated_token>" # обязательно

	property "sonar.coverage.jacoco.xmlReportPaths", "build/reports/jacoco/test/jacocoTestReport.xml" # обязательно, при использовании Jacoco
    }
```
где `project_key` - ключ проекта \
`project_name` - имя проекта \
`src_folfder`  - ссылка на расположение исходного кода\
`src_test` - ссылка на тесты \
`project_url` - ссылка с портом на вебсервер sonarqube \
`generated_token` - сгенерированный токен.

После конфигурации *`build.gradle`* запустить `./gradlew clean build sonar`.






