# Проделанная работа в процессе выполнения задания к лекции «Подготовка отчёта о тестировании и завершение проекта»

<table align="right">
<td>
<h4 align="center">Окружение:</h4>

* **Операционная система:** macOS Sequoia 15.1 &nbsp; &nbsp; <br>
* **Среда разработки::** Visual Studio Code  <br>
* **Контейнеризация:** Docker <br>
* **Java:** JDK 17 <br>
* **Инструмент тестирования:** Apache JMeter
<br>
</td>
</table>

[Задание.](https://github.com/netology-code/loadqa-homeworks/blob/main/5.Metrics/homework_lecture5.md)

#### Путеводитель 
1. [Запуск сайта WordPress](INSTRUCTIONS.md#1-Запуск-сайта-WordPress) 
2. [Настройка тестового сценария в JMeter](INSTRUCTIONS.md#2-Настройка-тестового-сценария-в-JMeter)
3. [Соединение Apache JMeter с Grafana](INSTRUCTIONS.md#3-Соединение-Apache-JMeter-с-Grafana)
4. [Настройка Jenkins для интеграции с JMeter](INSTRUCTIONS.md#4-Настройка-Jenkins-для-интеграции-с-JMeter) 
 
<br>
<br>

### 1. Запуск сайта WordPress
В процессе выполнения задач по нагрузочному тестированию базы данных была проведена настройка проекта WordPress, который позволяет создавать сайты без знания языков программирования.
  * Открыта необходимая папка с ранее склонированного проекта на GitHub в Visual Studio Code:
```bash
  # Переход в каталог WordPress и открытие папки wp в Visual Studio Code:
   cd ~/Documents/Performance_testingQA79/congenial-potato/wp && code .
```
  * Для совместимости плагинов в `docker-compose.yml` изменен образ контейнера на `image: wordpress:php8.0`.
    * Добавлена строка `platform: linux/x86_64`, чтобы указать Docker на создание образов для архитектуры x86_64 (необходима для работы на Apple M1 с архитектурой ARM).
    * Чтобы избежать предупреждения при запуске контейнеров, удалена строка `version: '3'`.
      * В версиях Docker Compose 2.0 и выше больше не требуется указывать версию файла, так как композитор теперь автоматически обрабатывает данные. Это упрощает работу и делает файлы конфигурации более понятными.
  * Запущены контейнеры с помощью Docker и проверено их состояние:
```bash
   # Запуск контейнеров Docker в фоновом режиме
   docker-compose up -d

   # Проверка запущенных контейнеров
   docker ps
```
  * В браузере по адресу `localhost:80`, указанном в `docker-compose.yml`, запущен сайт WordPress.
    * Выбран язык.
    * Заполнены все поля: название сайта, имя пользователя, email, придуман пароль, ☑️. 
    * На странице добавления комментария я убедился, что комментарий успешно добавляется.

   <br>
   
### 2. Настройка тестового сценария в JMeter
   * Скачан [JDBC-драйвер для MySQL](https://dev.mysql.com/downloads/connector/j/) и помещен в папку lib внутри каталога JMeter:
```bash
   mv ~/Downloads/mysql-connector-j-8.4.0.jar /Applications/apache-jmeter-5.6.3/lib
```
  * Скачанный файл [`wp_db_test.jmx`](wp_db_test.jmx) был перемещен в директорию для хранения тестов производительности и запущен с помощью JMeter для проведения нагрузочного тестирования базы данных:
```bash
   # Созданы папки для хранения проекта:
   mkdir -p ~/Documents/Performance_testingQA79/Load_testing_report/jmeter
             
   # Перемещение файла .jmx в директорию:
   mv ~/Downloads/wp_db_test.jmx ~/Documents/Performance_testingQA79/Load_testing_report/jmeter

   # Переход в директорию:
   cd ~/Documents/Performance_testingQA79/Load-testing_report/jmeter

   #Открытие теста в JMeter:
   jmeter -t wp_db_test.jmx 
```
  * Установлены графики в JMeter (Add => Listener => jp@gc - Назввание графика)
    * Hits per Second (HPS, запросов в секунду)
      * График Hits per Second (HPS) в Apache JMeter отображает количество HTTP-запросов, поступающих на сервер за одну секунду, и помогает оценить его производительность и нагрузку.
    * Transactions per Second (TPS, транзакций в секунду)
      *  Этот график является одним из ключевых индикаторов производительности приложения и позволяет оценить, насколько эффективно система справляется с нагрузкой.
    * Response Times Over Time (Время отклика с течением времени) 
      * Этот график отображает время отклика запросов по мере выполнения теста. Можно увидеть, как время отклика изменяется, например, в зависимости от увеличения нагрузки.
 
<br>

### 3. Соединение Apache JMeter с Grafana
#### Настройка JMeter для записи данных в InfluxDB
  * Открыт JMeter, добавлен Backend Listener к своему тесту:
    * Щелчок правой кнопкой мыши на Test Plan => Add => Listener => Backend Listener.
  * В свойствах Backend Listener выбрано `org.apache.jmeter.backend.influxdb.InfluxDBBackendListenerClient`, чтобы JMeter напрямую подключился к базе данных InfluxDB.
  * Установлены значения параметров:
    * **influxdb Url:** URL InfluxDB сервера и название базы данных, `http://localhost:8086/write?db=influx`.
    * **measurement:** Имя, по которому будут записываться метрики, `jmeter`.
#### Настройка Grafana
  * За основу взят [лекционный проект](https://github.com/netology-code/loadqa-homeworks/tree/main/2.Load%20environment/samples/telegraf) из задания «Подготовка стенда нагрузочного тестирования».
  * Запущена Grafana и открыт веб-интерфейс, доступный по адресу `http://localhost:3000`.
```bash
   # Запуск Docker-контейнер в фоне:
   docker compose up -d
```
  * В браузере по адресу `localhost:3000` введены логин и пароль от Grafana, указанные в `docker-compose.yml`.
  * Соединена Grafana с базой данных:
    * Путь: Configuration => Data sources => Add data source => InfluxDB.
    * В настройках InfluxDB указано `URL: http://influxdb:8086`, который взят из блоков `links` и `ports` в `docker-compose.yml`.
    * Проверка соединения с помощью кнопки **Test**.
#### Импорт Dashboard
  * Использован готовый Dashboard для анализа данных, найденный по запросу [Apache JMeter Dashboard](https://grafana.com/grafana/dashboards/5496-apache-jmeter-dashboard-by-ubikloadpack/).
  * Скопирован ID дашборда и импортирован в Grafana:
    * Путь: Dashboards => Browse => Import => Скопированный ID => Load.
#### Запуск теста в JMeter
  * Для формирования графика в Grafana запущен тест в JMeter.

<br>

### 4. Настройка Jenkins для интеграции с JMeter
CI/CD [Jenkins](https://www.jenkins.io/) — это мощный инструмент, который позволяет удаленно подключаться к различным компьютерам, тестовым стендам или стендам заказчиков для развертывания необходимых приложений и сервисов. В данном отчете будет описан процесс установки и настройки Jenkins с использованиемGeneric Java package (.war) LTS версии, а также основные этапы его запуска.
#### Подготовка и скачивание  
  * Скачан файл [`Generic Java package(.war)`](https://www.jenkins.io/download/) LTS версии для запуска Jenkins который будет работать в фоне
  * В официальной документации Jenkins был выбран способ установки через [WAR-file](https://www.jenkins.io/doc/book/installing/war-file/).
#### Настройка проекта
```bash
   # Создание директории для Jenkins:
   mkdir ~/Jenkins

   # Перемещение скачанного файла jenkins.war в созданную директорию:
   mv ~/Downloads/jenkins.war ~/Jenkins

   # Переход в директорию Jenkins:
   cd ~/Jenkins

   # Создание файла для хранения учетных данных Jenkins:
   nano ~/Jenkins/credentials.txt

   # В файле добавляются строки в следующем формате:
   username=имя_пользователя
   password=пароль

   # Создание скрипта для запуска Jenkins:
   nano start-jenkins.sh
```
  * В `start-jenkins.sh` добавлен следующий скрипт для чтения учетных данных и проверки, запущен ли Jenkins:
```bash
   #!/bin/bash

   export JENKINS_HOME=~/Jenkins

   # Чтение учетных данных из файла
   if [ -f "$JENKINS_HOME/credentials.txt" ]; then
       source $JENKINS_HOME/credentials.txt
   else
       echo "Файл с учетными данными не найден!"
       exit 1
   fi

   # Проверка, работает ли Jenkins уже
   if pgrep -f "jenkins.war" > /dev/null; then
       echo "Jenkins уже запущен."
   else
       echo "Запуск Jenkins..."
       java -jar ~/Jenkins/jenkins.war --httpPort=8080
   fi
```
```bash
   # Сделает скрипт исполняемым:
   chmod +x start-jenkins.sh

   # Устанавливает права доступа к файлу credentials.txt, позволяя только владельцу читать и записывать файл:
   chmod 600 credentials.txt

   # Открытие файла конфигурации терминала:
   nano ~/.zshrc

   # Добавление алиаса для запуска Jenkins:В ~/.zshrc добавляется строка:
   alias jenkins="~/Jenkins/start-jenkins.sh"

   # Перезагрузка файла конфигурации Zsh для применения изменений:
   source ~/.zshrc

   # Проверка установленной версии Java (рекомендуется использовать JDK 17 или JDK 21, более подробные рекомендации можно найти в документации Jenkins):
   java --version

   # Запуск Jenkins через созданный алиас:
   jenkins
```
  * Перешел по адресу `http://localhost:8080` и дожидался появления страницы разблокировки Jenkins. При регистрации Jenkins предлагает ввести ключ-пароль, с подсказкой, где его найти. Ключ-пароль также можно посмотреть в логах при запуске Jenkins или командой:
```bash
   # Вывод ключа-пароля для регистрации в Jenkins:
   cat ~/Jenkins/secrets/initialAdminPassword

   # Вывод логина и пароля для авторизации в Jenkins:
   cat ~/Jenkins/credentials.txt
```
#### Соединение с JMeter
  * Создание элемента (Item):
    * В Jenkins создан новый элемент (Item) с уникальным названием.
    * Выбрана опция «Создать задачу со свободной конфигурацией», что позволяет указать все необходимые параметры проекта.
  * Шаги сборки:
    * В раздел «Общие настройки» для конфигурации сборки добавлен шаг «Выполнить команду shell», который работает на macOS и Linux. Этот шаг позволяет выполнять скрипты и команды оболочки.  
  * Команды для выполнения:
    * В поле «Выполнить команды» вставлен путь к JMeter с командой, переходящей в папку `bin`, а затем выполняется сценарий для сборки отчета JMeter Dashboard в фоновом режиме.
  * Пример команды:
```bash
/Applications/apache-jmeter-5.6.3/bin/jmeter -n -t ~/Documents/Performance_testingQA79/Load_testing_report/jmeter/wp_db_test.jmx -l ~/Documents/Performance_testingQA79/Load_testing_report/jmeter/test_results.jtl -e -o ~/Documents/Performance_testingQA79/Load_testing_report/jmeter/report_output
```
<details>
  <summary>Каждый параметр имеет свою роль.</summary>

 Эта команда запускает JMeter в неинтерактивном режиме (`-n`), указывает файл теста (`-t`), задает файл для записи результатов (`-l`), генерирует отчет (`-e`) и определяет папку для сохранения выходных данных (`-o`):

```bash
jmeter -n -t <test JMX file> -l <test log file> -e -o <Path to output folder>
```

</details>

















