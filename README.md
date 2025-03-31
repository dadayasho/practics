# How to up the service
Репозиторий для производственной практики

1) Сделать git clone git@github.com:dadayasho/practics.git

2) Выгрузив репозиторий, поднять docker-compose командой

```bash
docker-compose up -d
```
*В данном docker-compose файле содержится sonarqube и postgres конфигурация.*
 
3) После поднятия docker-compose перейти по указанному url и порту.

4) Авторизоваться -> Перейти во вкладку "Projects" -> "Create project" -> "Manually" -> Ввести имя проекта и ключ, по которому будет идти обрпащение при сканирвоании.

5) Далее нажать кнопку "Locally", сгенерировать токен, перейти в CLI в хранилище проекта и начать скан.

6) Скачать через терминал sonar-scanner

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


<project_key> - ключ репозитория \
    <sonarqube_url> - url ссылка с портом на вебсервер sonarqube \
    <generated_token> - сгенерированный токен 

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


<project_key> - ключ репозитория \
<project_name> - название проекта \
<sonarqube_url> url - ссылка с портом на вебсервер sonarqube \
<generated_token> - сгенерированный токен

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






