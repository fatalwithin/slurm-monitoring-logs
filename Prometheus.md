# Основы  

Систематизация знаний. Нюансы настройки Prometheus. Лучшие паттерны использования.  

Прометей - OpenSource. Изначально разрабатывался в SoundCloud.  
Сделан на основе Boardman от Google. Разработан примерно в 2021-2013 году.

Почему Прометей стал стандартом на сегодня?  
  - Изначально ориентирован на работу в динамических окружениях - Service Discovery, Labels  
  - HA/Scalability - отказоустойчивость и масштабируемость  
  - включает Time Series Database, что вывело на качественно новый уровень скорость обработки данных  
  - собственный язык запросов к БД PromQL, который разрабатывался именно для решения задач мониторинга  

В Заббиксе в последних версиях тоже добавили поддержку Prometheus а также тоже перешли на Time Series DB.  

## Архитектура Прометея.  

Архитектура состоит из 3 частей: скрейпинга, rules&alerts, и service discovery.  

**Источники данных:**  

  - экспортеры - либо готовые, либо самописные.
  - клиентские библиотеки, встраиваемые в приложение.  

**Скрейпинг** - механизм сбора метрик из источников данных в базу Прометея.
Источники данных можно задавать статически, но рекомендуется динамически определять таргеты для скрейпинга через Service Discovery.  

**Rules & Alerts:**  
Правила позволяют сохранять кастомные PromQL-запросы в виде отдельной метрики (примерно как хранимые процедуры в SQL).  
Алерты - проверяются условия срабатывания алертов и срабатывает вебхук. Далее через механизм Alertmanager производится роутинг и доставка алертов в точку назначения - почта, мессенджер, СМС и т.д.  

Для визуализации используется чаще всего Grafana.  

### Немного из зарубежных источников про архитектуру Прометея

#### What is Prometheus?

Prometheus is an open-source **monitoring & alerting tool.** It was originally built by SoundCloud and now it is 100% open-source as a Cloud Native Computing Foundation graduated project. It has become highly popular in monitoring container & microservice environments.

#### Prometheus Architecture

#### Some Terminologies

- **Target** - It is what Prometheus monitors. It can be your. aplications, servers, etc.
- **Metric** - For our targets, we would like to monitor particular things. Like for example, if we have a server (target) we would want to monitor the number of errors on the HTTP endpoints exposed (metric).

[![Prometheus Server](https://res.cloudinary.com/practicaldev/image/fetch/s--vnRRV-CF--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/c8gjqrwqn4nn0kexhge5.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--vnRRV-CF--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/c8gjqrwqn4nn0kexhge5.png)
Here we see the main component of Prometheus, i.e, the server. It consists of three parts:

1. **Time Series Database (TSDB)** - Stores the metrics data. It also ingest it (append only), compacts and allows querying efficiently.
2. **Scrape Engine** - Pulls the metrics (description above) from our target resources and sends them to the TSDB. (Prometheus pulls are called scrapes).
3. **Server** - Used to make queries for the data stored in TSDB. This is also used to display the metrics in a dashboard using **Grafana/Prometheus UI**.![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--5NEZoh7m--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/9j0rfxtc913tc22zsurt.png)

#### More about Metrics

The metrics are defined with `TYPE` & `HELP` attributes to increase readability.

- `HELP` - It provides us with the description about the metric.

  ```
  TYPE
  ```

  Even tho Prometheus offers 4 core metric types to keep things simple, it allows us to create tags within those metric types for more specific use cases. The 4 core metric types are:

  - **Counter** - As the name suggests, it is used to maintain a count of the metrics. This can be, let's say, number of requests, errors, etc. Note: Do not use this type if the value of your metric can decrease.
  - **Gauge** - It is best suited for metrics that can go up &. down, like CPU usage.
  - **Histogram** - A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.
  - **Summary** - Similar to a histogram, a summary samples observations (usually things like request durations and response sizes). While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.

#### How does it work?

#### How does it get the data from Targets?

The Data Retrieval Worker pulls the data from the HTTP endpoints of the targets on path `/metrics`. Here we notice 2 things:

1. The endpoints should expose the path `/metrics`.
2. The data provided by the endpoint should be in the correct format that Prometheus understands.![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s---yN78_-I--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/kwlxobqhlmcu7dk3lsmm.png)

**Q.** How do we make sure that the target services expose /metric & that data is in correct format?
**A.** Some of them expose the endpoint by default. Ones that do not, need a component to do so. This component is known as an **Exporter**. An Exporter does the following:

1. Fetch data from the target
2. Convert data into a format that Prometheus understands
3. Expose the /metrics endpoint (This can now be retrieved by the Data Retrieval Worker) For different types of services, like APIs, Databases, Storage, HTTP, etc, Prometheus has a [list of Exporters](https://prometheus.io/docs/instrumenting/exporters/) you can use.

#### Monitoring Personal Services

Let's say you want to monitor an application you have written in Java, you can use [Client Libraries](https://prometheus.io/docs/instrumenting/clientlibs/) for that. It lets you expose application metrics via an HTTP endpoint `/metrics` on your application’s instance which can then be used to send data to the Metrics Server. In the official documentation, a list of various libraries has been provided, with information on how to create your own.

#### How is it different?

As mentioned above, Prometheus. uses a **pull mechanism** to get data from targets. But mostly, other monitoring systems use a **push mechanism** (we'll see what that is in a bit). How is this different and what makes Prometheus so special?

**Q.** What do you mean by push mechanism?
**A.** Instead of the server of the monitoring tool making requests to get the data, the servers of the application push the data to a database instead.

**Q.** Why is Prometheus better?
**A.** You can just get the data from the endpoint of the target, by multiple Prometheus instances. Also note that this way Prometheus can also monitor whether an application is responsive or not, rather than waiting for the target to push data.
(Checkout the [official comparison](https://prometheus.io/docs/introduction/comparison/) documentation)

**NOTE:** But what happens if the targets don't give us enough time to make a pull request? For this, Prometheus uses the **Pushgateway**. Using this, these services can now push their data to the Data Retrieval Worker instead of it pulling data like it usually does. Using this, you get the best out of both the ways!
[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--y1oZ6iKo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/8mcrr03yjo37sytjklul.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--y1oZ6iKo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/8mcrr03yjo37sytjklul.png)

#### How to use it?

Now that we know how Prometheus works, lets take a look into how we actually use it. So we mentioned about targets, metrics and all sorts of things. Where do we define those? Answer, in a **config (yaml)** file.

**Q.** When you define what targets you want to collect data from in the file, how does Prometheus find these targets
**A.** Using the Service Discovery. It also discovers services automatically based on the application running.
[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--Ug3-w2Sh--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/brzuwuq6f1r0qrqq3pji.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--Ug3-w2Sh--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/brzuwuq6f1r0qrqq3pji.png)

#### Default Configuration File

(Check the [official documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/) for configuration)

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```



- `global` - `scrape_interval` defines how often Prometheus is going to collect data from the targets mentioned in the file. This can of course be overridden.

  ```
  rule_files
  ```

  This allows us to set rules for metrics & alerts. These files can be reloaded at runtime by sending

  ```yaml
  SIGHUP
  ```

  to the Prometheus process. The

  ```yaml
  evaluation_interval
  ```

  defines how often these rules are evaluated. Prometheus supports 2 types of such rules:

  - **Recording Rules** - If you are performing some frequent operations, they can be precomputed and saved in as a new set of time series. This makes the monitoring system a bit faster.
  - **Alerting Rules** - This lets you define conditions to send alerts to external services, for example, when a particular condition is triggered.

- `scrape_configs` - Here we define the services/targets that we need Prometheus to monitor. In this example file, the `job_name` is `prometheus`. Meaning that it is monitoring the target as the Prometheus server itself. In short, it will get data from the `/metrics` endpoint exposed by the Prometheus server. Here, the target by default is `localhost:9090` which is where Prometheus will expect the metrics to be, at `/metrics`.

#### How does Alerting work?

Prometheus has an **Alermanager** that can be used to send alerts to you via Emails, mailing lists, etc. As mentioned above, Prometheus server uses the **Alerting Rules** to send alerts.

#### Where is the metrics data stored?

Prometheus stores it on disk, this can be a local database or remote. The data is stored in a time-series format so that one cannot write data directly.

#### How to get the data?

Prometheus lets use get the metrics data using the **PromQLQuery Language**. You can use a Web UI to request data from Prometheus server via PromQL.
[![https://prometheus.io/docs/introduction/overview/](https://res.cloudinary.com/practicaldev/image/fetch/s--bCRrIeC9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/43homdxpz9mrzzuqvz9n.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--bCRrIeC9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/43homdxpz9mrzzuqvz9n.png)

#### Running it Locally

Let's take the example of the configuration file (`config.yml`) above that monitors the Prometheus server running on our machine.
(Checkout the [README.md](https://github.com/prometheus/prometheus) file for more information)

```bash
$ mkdir -p $GOPATH/src/github.com/prometheus
$ cd $GOPATH/src/github.com/prometheus
$ git clone https://github.com/prometheus/prometheus.git
$ cd prometheus
$ make build
$ ./prometheus --config.file=your_config.yml$ mkdir -p $GOPATH/src/github.com/prometheus
$ cd $GOPATH/src/github.com/prometheus
$ git clone https://github.com/prometheus/prometheus.git
$ cd prometheus
$ make build
$ ./prometheus --config.file=config.yml
```



Running it on `localhost:9090`, you'll get the following Prometheus UI Dashboard that you can now configure:
[![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--Iwz8bOm0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/snmh0uf0s1solypnk47f.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--Iwz8bOm0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/snmh0uf0s1solypnk47f.png)

# Exposition  

## Общие сведения о процессе экспозиции

Процесс предоставления данных для Prometheus называется **exposition**. <u>Prometheus собирает данные по HTTP протоколу, в формате plain text c кодировкой UTF-8.</u>

Начиная с версии 0.4.0 это единственный способ, а до этого была еще возможность предоставлять данные в виде **Protobuf**.

Для комментариев используется #, с одним исключением: после # не следует HELP или TYPE – это зарезервированные токены. В случае HELP ожидается, как минимум, название метрики. После название может содержать любое количество произвольных символов с описанием метрики.TYPE ожидает 2 токена: название метрики и тип счетчика – и должен быть объявлен до первого использования метрики.

Метрики, в общем, имеют следующий формат:

```
metric_name{label_name="label_value",label_name="label_value"} value timestamp
```

Источники данных для Prometheus:

1. Для экспозиции уже существующих метрик из разнообразных источников, таких как nginx, Linux или Postgres, используются exporters. **Exporter** – это стороннее приложение, которое собирает метрики и отдает их в понятном для prometheus виде. На данный момент часть exporters поддерживаются командой Prometheus, часть поддерживается open source сообществом. Полный список exporters представлен в [документации ](https://prometheus.io/docs/instrumenting/exporters/)и постоянно пополняется.

2. Если же Вы не нашли подходящего именно Вам exporter, то всегда можно написать его самостоятельно. Для этого есть готовые библиотеки, которые существенно облегчают процесс написания exporter. Библиотеки есть для большинства современных языков программирования.

3. Также можно встроить процесс экспозиции прямо в свое приложение. Для этого есть готовые библиотеки, которые существенно облегчают этот процесс. Эти библиотеки также имеются для многих языков программирования.

## Node Exporter

Собирает базовые системные метрики по умолчанию - CPU, жесткий диск, память, сеть.

Может производить экспозицию произвольных метрик из файла.

Умеет также предоставлять информацию о состоянии systemd сервисов.

Работает только под Linux. Для Windows нужен WMI-экспортер.

Не имеет собственного конфиг-файла, вся настройка с помощью ключей. Лучшая практика - не запускать из командной строки напрямую, а сделать systemd - сервис.

Порт по умолчанию - 9100. При переходе на страницу будет единственная ссылка, по которой будут показаны все доступные метрики.

Перед каждой метрикой - 2 поля: #HELP и #TYPE.

**#HELP** - описание метрики

**#TYPE** - тип метрики

В конце списка будут добавленные метрики из файлов.

## Типы метрик 

Всего 4 типа метрик.

### Counter

A *counter* is a cumulative metric that represents a single [monotonically increasing counter](https://en.wikipedia.org/wiki/Monotonic_function) whose value can only increase or be reset to zero on restart. For example, you can use a counter to represent the number of requests served, tasks completed, or errors.

Do not use a counter to expose a value that can decrease. For example, do not use a counter for the number of currently running processes; instead use a gauge.

Client library usage documentation for counters:

- [Go](https://godoc.org/github.com/prometheus/client_golang/prometheus#Counter)
- [Java](https://github.com/prometheus/client_java#counter)
- [Python](https://github.com/prometheus/client_python#counter)
- [Ruby](https://github.com/prometheus/client_ruby#counter)

### Gauge

A *gauge* is a metric that represents a single numerical value that can arbitrarily go up and down.

Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of concurrent requests.

Client library usage documentation for gauges:

- [Go](https://godoc.org/github.com/prometheus/client_golang/prometheus#Gauge)
- [Java](https://github.com/prometheus/client_java#gauge)
- [Python](https://github.com/prometheus/client_python#gauge)
- [Ruby](https://github.com/prometheus/client_ruby#gauge)

### Histogram

A *histogram* samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.

A histogram with a base metric name of `<basename>` exposes multiple time series during a scrape:

- cumulative counters for the observation buckets, exposed as `<basename>_bucket{le="<upper inclusive bound>"}`
- the **total sum** of all observed values, exposed as `<basename>_sum`
- the **count** of events that have been observed, exposed as `<basename>_count` (identical to `<basename>_bucket{le="+Inf"}` above)

Use the [`histogram_quantile()` function](https://prometheus.io/docs/prometheus/latest/querying/functions/#histogram_quantile) to calculate quantiles from histograms or even aggregations of histograms. A histogram is also suitable to calculate an [Apdex score](https://en.wikipedia.org/wiki/Apdex). When operating on buckets, remember that the histogram is [cumulative](https://en.wikipedia.org/wiki/Histogram#Cumulative_histogram). See [histograms and summaries](https://prometheus.io/docs/practices/histograms) for details of histogram usage and differences to [summaries](https://prometheus.io/docs/concepts/metric_types/#summary).

Client library usage documentation for histograms:

- [Go](https://godoc.org/github.com/prometheus/client_golang/prometheus#Histogram)
- [Java](https://github.com/prometheus/client_java#histogram)
- [Python](https://github.com/prometheus/client_python#histogram)
- [Ruby](https://github.com/prometheus/client_ruby#histogram)

### Summary

Similar to a *histogram*, a *summary* samples observations (usually things like request durations and response sizes). While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.

A summary with a base metric name of `<basename>` exposes multiple time series during a scrape:

- streaming **φ-quantiles** (0 ≤ φ ≤ 1) of observed events, exposed as `<basename>{quantile="<φ>"}`
- the **total sum** of all observed values, exposed as `<basename>_sum`
- the **count** of events that have been observed, exposed as `<basename>_count`

See [histograms and summaries](https://prometheus.io/docs/practices/histograms) for detailed explanations of φ-quantiles, summary usage, and differences to [histograms](https://prometheus.io/docs/concepts/metric_types/#histogram).

Client library usage documentation for summaries:

- [Go](https://godoc.org/github.com/prometheus/client_golang/prometheus#Summary)
- [Java](https://github.com/prometheus/client_java#summary)
- [Python](https://github.com/prometheus/client_python#summary)
- [Ruby](https://github.com/prometheus/client_ruby#summary)

## **Установка Node Exporter**

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и команды выполняются с root привилегиями. 

1. Скачиваем архив с Node Exporter и распаковываем его.

Для установки используется версия: 0.18.1, последняя стабильная версия на момент подготовки курса.

```bash
# cd /tmp
# wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
# tar xvfz node_exporter-0.18.1.linux-amd64.tar.gz
# cd node_exporter-0.18.1.linux-amd64
# mv node_exporter /usr/local/bin/
```

Список версий можно посмотреть на странице [github](https://github.com/prometheus/node_exporter/releases).

2. Создадим файл со списком ключей для запуска Node Exporter.

Node Exporter не имеет конфигурационного файла и настраивается дополнительными ключами. Для того, чтобы каждый раз не перечитывать настройки systemd, список ключей вынесен в файл: /etc/sysconfig/node_exporter.

```
cat <<EOF > /etc/sysconfig/node_exporter
OPTIONS=""
EOF
```

3. Создаем systemd сервис.

```bash
cat <<EOF > /usr/lib/systemd/system/node_exporter.service 
[Unit]
Description=Prometheus Node exporter for machine metrics
Documentation=https://github.com/prometheus/node_exporter

[Service]
Restart=always
User=root
EnvironmentFile=/etc/sysconfig/node_exporter
ExecStart=/usr/local/bin/node_exporter \$OPTIONS
ExecReload=/bin/kill -HUP \$MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
EOF
```

С помощью директивы: EnvironmentFile все переменные из файла, указанного в этой директиве, добавляются в env, это позволяет нам передавать список ключей для Node Exporter через переменную `$OPTIONS` без изменения службы.

NB! Обратите внимание, Node Exporter требует root привилегий.

4. Обновляем список служб systemd, запускаем Node exporter и добавляем в автозапуск.

```bash
# systemctl daemon-reload
# systemctl start node_exporter
# systemctl enable node_exporter
```

5. Проверяем работу.

По **умолчанию**, Node Exporter слушает порт **9100.**

```bash
# curl -I http://localhost:9100
```

Ответ должен быть примерно таким:

```bash
HTTP/1.1 200 OK
Date: Fri, 08 Nov 2019 14:01:07 GMT
Content-Length: 150
Content-Type: text/html; charset=utf-8
```

### Ключи запуска Node Exporter

Далее приведен список наиболее востребованных ключей Node Exporter

```
--log.level
```

Default: info

Данный ключ устанавливает уровень логирования. Возможные уровни логирования: debug, info, warn, error.

```
--log.format
```

Default: logfmt

Данный ключ устанавливает формат логов. Доступные форматы: logfmt и json.

```
--web.listen-address
```

Default: ":9100"

Данный ключ устанавливает адрес и порт, по которому будет доступен Node Exporter.

```
--web.telemetry-path
```

Default: "/metrics"

Данный ключ устанавливает адрес, по которому доступны результаты экспозиции.

```
--web.disable-exporter-metrics
```

Default: -

Данный ключ устанавливает список метрик, которые будут исключены из экспозиции.

Например, для исключения всех метрик, имя которых начинается с go_, значение ключа будет: go_*. Допускается использовать перечисление нескольких метрик, с запятой в качестве разделителя.

Полный список ключей можно просмотреть с помощью команды help.

```bash
# node_exporter --help
```

### Добавление метрик из файла

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

1. Создадим файл, который будет содержать метрики.

```bash
# mkdir /tmp/node_exporter/
cat <<EOF > /tmp/node_exporter/node_exporter_custom_metric.prom
#TYPE slurm_demo_metric counter
slurm_demo_metric 100
EOF
```

2. Настраиваем Node Exporter для чтения метрик из файла.

С этой целью для ключа `collector.textfile.directory` укажем в качестве значения путь каталога, содержащий файлы с метриками:

```bash
cat <<EOF > /etc/sysconfig/node_exporter
OPTIONS="--collector.textfile.directory=/tmp/node_exporter"
EOF
```

В указанном каталоге Node Exporter читает все файлы по маске *.prom

3. Перезапустим службу Node Exporter:

```bash
# systemctl restart node_exporter
```

4. Проверим, что в экспозиции появилась новая метрика:

```bash
# curl -si http://localhost:9100/metrics | grep slurm
```

Ответ должен быть таким:

```bash
# HELP slurm_demo_metric Metric read from /tmp/node_exporter/node_exporter_custom_metric.prom
# TYPE slurm_demo_metric counter
slurm_demo_metric 100
```



## Blackbox Exporter

Этот экспортер - тестирование черного ящика, с угла зрения пользователя, поэтому его надо запускать отдельно. 

Настройка разделена на 2 части: что мониторить - предоставляет Prometheus в качестве параметров OWL и остальная настройка - на стороне самого экспортера.  

Возможные протоколы проверок от экспортера:

ICMP - скорость DNS запроса, время пинга, версия протоколов и результат проверки

TCP - скорость DNS запроса, установка подключения, результат, плюс по регекспу проверить нужный текст ответа от сервера

DNS - исключительно для проверки DNS-сервера

HTTP - скорость стадий запроса, наличие или отсутствие паттернов в ответе

Blackbox Exporter доступен по порту 9115, в отличие от Node Exporter - у него больше опций не веб-странице. 

Раздел Configuration - текущая конфигурация.

Раздел Metrics - метрики, которые относятся к работе самого экспортера.

Раздел Debug - тестовые проверки для демонстрации возможности экспортера.

Раздел Probe - тестовая проверка. Отличается тем, что в ней не выводится дебаг-информация.

## **Установка Blackbox Exporter**

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями. 

1. Скачиваем архив с Blackbox Exporter и распаковываем его.

Для установки используется версия: 0.18, последняя стабильная версия на момент подготовки курса. 

```bash
cd /tmp
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.18.0/blackbox_exporter-0.18.0.linux-amd64.tar.gz tar xvfz blackbox_exporter-0.18.0.linux-amd64.tar.gz
cd blackbox_exporter-0.18.0.linux-amd64
mv blackbox_exporter /usr/local/bin/
```

Список версий можно посмотреть на странице [github](https://github.com/prometheus/blackbox_exporter/releases).

2. Создаем каталог для конфигурационных файлов и копируем конфигурационные файл по умолчанию.

```bash
mkdir /etc/blackbox_exporter
cp blackbox.yml /etc/blackbox_exporter/
```

3. Создадим файл со списком ключей для запуска Blackbox Exporter.

 Для того, чтобы каждый раз не перечитывать настройки systemd, список ключей вынесен в файл: `/etc/sysconfig/blackbox_exporter`

```bash
cat <<EOF > /etc/sysconfig/blackbox_exporter
OPTIONS="--config.file=/etc/blackbox_exporter/blackbox.yml"
EOF
```

!NB Поскольку Blackbox Exporter имеет конфигурационный файл и по умолчанию ищет его в том же каталоге, где и бинарный файл, то через ключ `--config.file` мы задаем путь до конфигурационного файла.

4. Создаем systemd сервис:

```bash
cat <<EOF > /usr/lib/systemd/system/blackbox_exporter.service 
[Unit]
Description=Prometheus exporter for machine metrics
Documentation=https://github.com/prometheus/blackbox_exporter

[Service]
Restart=always
User=root
EnvironmentFile=/etc/sysconfig/blackbox_exporter
ExecStart=/usr/local/bin/blackbox_exporter \$OPTIONS
ExecReload=/bin/kill -HUP \$MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
EOF
```

С помощью директивы: EnvironmentFile все переменные из файла, указанного в этой директиве, добавляются в env. Это позволяет передавать список ключей для Node Exporter через переменную `$OPTIONS` без изменения службы.

!NB Обратите внимание, Blackbox Exporter **требует root привилегий** для выполнения некоторых проверок.

5. Обновляем список служб systemd и запускаем Blackbox Exporter.

```bash
systemctl daemon-reload
systemctl start blackbox_exporter
systemctl enable blackbox_exporter
```

6. Проверяем работу.

По **умолчанию**, Blackbox Exporter слушает порт **9115**

```bash
curl -I http://localhost:9115
```

Ответ должен быть примерно таким:

```bash
HTTP/1.1 200 OK
Content-Type: text/html
Date: Tue, 05 Nov 2019 07:09:10 GMT
Content-Length: 544
```

NB! Особенности использования Blackbox Exporter:

1. По умолчанию, все проверки выполняются по IPv6. Рекомендуется изменить значение на IPv4. Для этого надо использовать директиву: `preferred_ip_protocol: ip4` в настройках каждого модуля.

2. По ссылке /metrics Blackbox Exporter отдает данные о своей работе, такие как: потребление cpu, потребление ram и т.д. Результаты проверки доступны по url /probe.

3. Для получения подробностей по проверке в параметры запроса нужно добавить: `debug=true`

## Настройка Blackbox Exporter

### Общая структура конфигурационного файла.

1. Конфигурационный файл имеет формат yaml.

2. По умолчанию, Blackbox Exporter ищет конфигурационный файл с именем blackbox.yml, в том же каталоге, что и бинарный файл.

3. Общая структура конфигурационного файла:

```yaml
module: 
  icmp_slurm:
    prober: icmp
      icmp:
        preferred_ip_protocol: ip4
      
```

icmp_slurm – имя, по которому будет запускаться проверка. Это позволяет осуществить несколько проверок одного типа, с разными настройками.

Например, с использованием ipv4 и ipv6 протоколов:

```yaml
module: 
  icmp_v4_slurm:
    prober: icmp
      icmp:
        preferred_ip_protocol: ip4
  icmp_v6_slurm:
    prober: icmp
      icmp:
        preferred_ip_protocol: ip6
      
```

prober: задает, какой протокол будет использован для проверки. Возможные значения: icmp | dns | tcp | http.

Далее идут настройки, специфичные для каждого протокола.

### ICMP протокол

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

Данная проверка предназначена для проверки доступности хостов по протоколу ICMP.

1. Добавляем в конфигурационный файл проверку с использованием протокола ICMP и именем icmp_slurm

```bash
cat <<EOF >> /etc/blackbox_exporter/blackbox.yml
  icmp_slurm:
    prober: icmp
    timeout: 2s
    icmp:
      preferred_ip_protocol: ip4

EOF
```

NB! Для преобразования имен в ip используется dns, без учета файла hosts.

2. Чтобы применить изменения, перезапускаем Blackbox Exporter:

```bash
systemctl restart blackbox_exporter
```

3. Проверяем работу:

```bash
curl -is "http://localhost:9115/probe?module=icmp_slurm&target=slurm.io" | grep probe_success
```

В запросе, в качестве параметра **module**, передается имя проверки, а в качестве параметра **target** – какой ресурс проверять. С помощью grep фильтруем результат, чтобы получить только результат проверки.

Результат должен быть таким:

```bash
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

4. Полный список параметров для проверки по ICMP протоколу:

```
timeout
```

Default: scrape_timeout

Время, после которого проверка будет считаться неудачной. NB! Если значение не задано, используется scrape_timeout, который передал Prometheus.

```
preferred_ip_protocol
```

Default: ip6

Какой протокол используется для проверки. Допустимые значения: ip4| ip6.

```
source_ip_address
```

Default: -

Если на сервере несколько IP адресов, можно указать, с какого ip будет проводиться проверка.

```
dont_fragment
```

Default: false

Разрешен ли бит фрагментации пакетов. !NB Работает только с linux и IPv4.

```
payload_size
```

Default: -

Размер пакета, который отправляется при выполнении проверки.

### DNS протокол

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

Данная проверка предназначена для тестирования DNS серверов и наличия на них определенных записей.

1. Добавляем в конфигурационный файл проверку dns с именем dns_slurm:

```bash
cat <<EOF >> /etc/blackbox_exporter/blackbox.yml
  dns_slurm:
    prober: dns
    timeout: 2s
    dns:
      query_name: slurm.io
      preferred_ip_protocol: ip4
EOF
```

В данной конфигурации будет проверяться, может ли DNS сервер разрешить имя slurm.io.

2. Чтобы применить изменения, перезапускаем Blackbox Exporter:

```
systemctl restart blackbox_exporter
```

3. Проверяем работу:

```bash
curl -is "http://localhost:9115/probe?module=dns_slurm&target=8.8.8.8" | grep probe_success
```

В запросе, в качестве параметра module, передаётся имя проверки, а в качестве параметра target – на какой DNS сервер будет отправлен запрос. С помощью grep фильтруем результат, чтобы получить только результат проверки.

Результат должен быть таким:

```no-highlight
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

4. Полный список параметров для проверки по dns протоколу:

```
timeout
```

Default: scrape_timeout

Время, после которого проверка будет считаться неудачной. NB! Если значение не задано, используется scrape_timeout, который передал Prometheus.

```
preferred_ip_protocol
```

Default: ip6

Какой протокол используется для проверки. Допустимые значения: ip4| ip6.

```
source_ip_address
```

Default: -

Если на сервере несколько IP адресов, можно указать, с какого ip будет проводиться проверка.

```
transport_protocol
```

Default: udp

Протокол, по которому будет производиться проверка. Возможные значения: udp, tcp.

```
query_name:
```

Default: -

Запрос, который будет отправлен на DNS сервер.

```
query_type
```

Default: "ANY"

Тип записи, который будет запрашиваться. По умолчанию, запрашиваются все типы записей.

### TCP протокол

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

Данная проверка предназначена для тестирования сервисов с использованием TCP протокола.

1. Добавляем в конфигурационный файл проверку по TCP протоколу с именем tcp_slurm:

```bash
cat <<EOF >> /etc/blackbox_exporter/blackbox.yml
  tcp_slurm:
    prober: tcp
    timeout: 2s
    tcp:
      query_response: 
        - expect: "^SSH-2.0-"
      preferred_ip_protocol: ip4
EOF
```

В данной конфигурации будет проверяться, присутствует ли в ответе строка: SSH-2.0-

2. Чтобы применить изменения, перезапускаем Blackbox Exporter:

```bash
systemctl restart blackbox_exporter
```

3. Проверяем работу:

```bash
curl -is "http://localhost:9115/probe?module=tcp_slurm&target=127.0.0.1:22" | grep probe_success
```

В запросе, в качестве параметра module, передаётся имя проверки, а в качестве параметра target – IP адрес хоста и порт, для которых будет выполнена проверка. С помощью grep фильтруем результат, чтобы получить только результат проверки.

Результат должен быть таким:

```no-highlight
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

4. Полный список параметров для проверки по TCP протоколу:

```
timeout
```

Default: scrape_timeout

Время, после которого проверка будет считаться неудачной. NB! Если значение не задано, используется scrape_timeout, который передал Prometheus.

```
preferred_ip_protocol
```

Default: ip6

Какой протокол используется для проверки. Допустимые значения: ip4| ip6.

```
query_response
```

- expect - проверка на наличие строки в ответе. 
- send - позволяет задать, какой запрос будет отправлен на сервер.
- starttls - задает, будет ли использоваться tls при подключении, по умолчанию – false.

```
source_ip_address
```

Default: -

Если на сервере несколько IP адресов, можно указать, с какого ip будет проводиться проверка.

```
tls
```

Default: false

Использовать ли tls после подключения.

```
tls_config
```

Настройки для tls. Возможны следующие **настройки для tls**:

- **insecure_skip_verify** – проверять ли валидность сертификата. Значение по умолчанию – false.
- **ca_file** – путь к файлу с корневыми сертификатами.
- **cert_file** – путь к файлу с клиентским сертификатом.
- **key_file** – путь к файлу с клиентским ключом.
- **server_name** – строка для проверки имени сервера.

### HTTP протокол

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

Данная проверка предназначена для проверки сервисов с использованием HTTP протокола.

1. Добавляем в конфигурационный файл проверку по HTTP протоколу с именем проверки http_slurm:

```bash
cat <<EOF >> /etc/blackbox_exporter/blackbox.yml
  http_slurm:
    prober: http
    timeout: 2s
    http:
      preferred_ip_protocol: ip4
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: [200]
      fail_if_not_ssl: true
      method: GET
      fail_if_body_not_matches_regexp:
        - "Prometheus"
EOF
```

В данной конфигурации для проверки используется протокол ip v4, проверяется версия HTTP, код ответа, метод GET. Также проверка закончится с ошибкой, если не используется https или если в ответе отсутствует слово: Prometheus.

2. Чтобы применить изменения, перезапускаем Blackbox Exporter:

```
systemctl restart blackbox_exporter
```

3. Проверка.

```bash
curl -is "http://localhost:9115/probe?module=http_slurm&target=http://prometheus.io" | grep probe_success
```

В запросе, в качестве параметра module, передаётся имя проверки, а в качестве параметра target – адрес веб сайта, который проверяется. С помощью grep фильтруем результат, чтобы получить только результат проверки.

NB! Адрес сайта может передаваться вместе со схемой. Если схема не указана, то будет использоваться http.

Результат должен быть таким:

```bash
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

4. Полный список параметров для проверки по HTTP протоколу:

```
timeout
```

Default: scrape_timeout

Время, после которого проверка будет считаться неудачной. NB! Если значение не задано, используется scrape_timeout, который передал Prometheus.

```
preferred_ip_protocol
```

Default: ip6

Какой протокол используется для проверки. Допустимые значения: ip4| ip6.

```
source_ip_address
```

Default: -

Если на сервере несколько IP адресов, можно указать, с какого ip будет проводиться проверка.

```
valid_status_codes
```

default: 200

Проверка считается неудачной, если код не соответствует заданному.

```
valid_http_versions
```

Проверка считается неудачной, если версия HTTP не соответствует строке.

```
method
```

Default: GET

Тип запроса. Возможные значения: GET | POST.

```
headers
```

Список заголовков, которые передаются во время проверки.

```
no_follow_redirects
```

Default: false

Следовать ли редиректам при проверке.

```
fail_if_ssl
```

Deafult: false

Проверка считается неуспешной, если соединение установлено по https.

```
fail_if_not_ssl
```

Deafult: false

Проверка считается неуспешной, если соединение установлено без использования https.

```
fail_if_body_matches_regexp
```

Проверка считается неуспешной, если в body присутствует строка удовлетворяющая регулярному выражению.

```
fail_if_body_not_matches_regexp
```

Проверка считается неуспешной, если в body отсутствует строка удовлетворяющая регулярному выражению.

```
fail_if_header_matches
```

Проверка считается неуспешной, если в ответе присутствует заголовок, значение которого удовлетворяет регулярному выражению.

- header – имя заголовка, который проверяется.
- regexp – регулярное выражение, которое проверяется в значении заголовка.
- allow_missing – разрешить отсутствие заголовка. По умолчанию: false.

```
fail_if_header_not_matches
```

Проверка считается неуспешной, если в ответе отсутствует заголовок, значение которого удовлетворяет регулярному выражению.

- header – имя заголовка, который проверяется.
- regexp – регулярное выражение, которое проверяется в значении заголовка.
- allow_missing – разрешить отсутствие заголовка. По умолчанию: false.

```
basic_auth
```

- username – имя пользователя, которое используется для авторизации на проверяемом сайте.
- password – пароль, который используется для авторизации на проверяемом сайте.

```
bearer_token
```

Токен для bearer авторизации.

```
bearer_token_file
```

Файл, который содержит токен для bearer авторизации.

```
proxy_url
```

Адрес proxy сервера, если проверку необходимо выполнить через proxy сервер.

```
body
```

Body, которое передается вместе с запросом.

```
tls_config
```

- **insecure_skip_verify** – проверять ли валидность сертификата. Значение по умолчанию: false
- **ca_file** – путь к файлу с корневыми сертификатами.
- **cert_file** – путь к файлу с клиентским сертификатом.
- **key_file** – путь к файлу с клиентским ключом.
- **server_name** – строка для проверки имени сервера.

### **Настройка со стороны Prometheus**

Prometheus пока еще не установлен. В данном шаге приведена настройка Prometheus для работы с Blackbox Exporter. К ней необходимо будет вернуться после установки Prometheus, описанной в одной из следующих глав.

Пример настроек Prometheus для произведения скрайпинга с Blackbox Exporter:

```bash
scrape_configs:
  - job_name: blackbox
    metrics_path: /probe
    params:
      module: [http_slurm]
    static_configs:
      - targets:
        - "http://www.prometheus.io"
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
```

Настройка скрайпинга для Blackbox Exporter сильно отличается от настройки для большинства exporters. Далее, разберем конфиг по порядку.

```bash
params:
  module: [http_slurm]
```

Для работы Blackbox, в качестве параметров запроса, надо передавать, какой модуль должен использоваться для проверки. Эта часть конфигурационного файла добавляет к запросу параметр `module=http_slurm`. Здесь можно было бы передать и второй параметр, `target`. Но в этом случае для каждого URL придется делать отдельную конфигурацию. Поэтому рекомендуется делать это через переопределение labels.

```bash
static_configs:
  - targets:
    - "http://www.prometheus.io"
```

В отличие от большинства exporters, в секции задается не адрес и порт exporter, а адрес проверяемого ресурса.

А дальше идет `relabel_configs`, в котором происходит вся магия. Разберем по шагам, что происходит в relabel_configs.

```bash
relabel_configs:
  - source_labels: [__address__]
    target_label: __param_target
```

В source label `__address__` содержится адрес exporter – тот, который задается в targets. Данная часть конфига копирует значение из `__address__` (по сути, из targets) в `__param_target`. Это преобразование добавляет второй параметр, target, к запросу.

```bash
relabel_configs:  
  - source_labels: [__param_target]
    target_label: instance
```

В этой части мы значение из label `__param_target` записываем в instance. Это преобразование нужно, чтобы проверка каждого ресурса в label instance в качестве значения имела адрес этого ресурса.

```bash
relabel_configs:        
  - target_label: __address__
    replacement: 127.0.0.1:9115
```

Так как в targets у нас указаны адреса проверяемых ресурсов, то нужно сказать, как подключаться к exporter. Для этого в label: `__address__` записываем адрес Blackbox.

## Custom Exporter

Кастомный экспортер нужен тогда, когда не хватает возможностей готовых. Для кастомных экспортеров существуют библиотеки на большинстве популярных ЯП. 

Метрики производительности приложения логично получать из самого приложения (инструментирование кода).

**Общие рекомендации по написанию exporter.**

Есть ряд рекомендаций, которые пригодятся при написании exporter. Они не обязательны к исполнению, но их стоит придерживаться, так как они составлены на основе опыта по решению реальных проблем.

1. Выбор имени для метрик:

- Имена метрик должны быть в нижнем регистре.
- Для именования допускаются только символы: [a-zA-Z0-9:_].
- `_sum`, `_count`, `_bucket` и `_total` суффиксы, которые используются для **Summaries**, **Histograms** и **Counters**. В других случаях использовать их не стоит.
- `process_` и `scrape_` являются зарезервированными префиксами.
- Из названия метрики должно быть понятно, что эта метрика собирает. 
- В названии метрики должны содержаться единицы измерения.

Пример плохого именования:

```bash
http_request_duration
```

Из названия не ясны ни единицы, в которых производится измерение, ни к какому приложению метрика относится.

Пример хорошего именования этой же метрики:

```bash
nginx_http_request_duration_seconds_bucket
```

- Если есть число удачных запросов, число неудачных запросов и общее количество запросов, то лучше сделать 2 метрики: число удачных запросов и общее число запросов. В этом случае число неудачных запросов легко обработать. 

2. Выбор labels:

- Ряд слов, использовать которые в имени метки не запрещено, но и не рекомендовано: `region, zone, cluster, az, datacenter, customer, dc, owner, stage, service, environment, job, instance`.
- Метку `le` стоит использовать только с Histograms, а метку `quantile` стоит использовать только с Summaries.
- Метки необходимо выбирать таким образом, чтобы сумма значений метрики имела значение. Например, read/write показатели лучше хранить как отдельные метрики, а не разделять с использованием labels. 

3. Рекомендации по написанию exporters:

- Получение данных должно производиться, только когда Prometheus производит scraping.
- Не надо выставлять временные метки для метрик, это задача Prometheus.
- Выбирать порт для exporter стоит с учетом уже занятых портов. Список уже использованных портов размещен на [github](https://github.com/prometheus/prometheus/wiki/Default-port-allocations).

### Пример exporter на языке go

Практика выполняется на monitoring сервере, доступ к нему осуществляется по **IP** адресу и выполняются с root привилегиями.

Ниже приведен пример кода для простейшей реализации exporter. Exporter написан на языке go. Для реализации exporter используются готовые библиотеки prometheus. Exporter возвращает стандартные метрики, связанные с потреблением ресурсов самим exporter; данные метрики являются частью функционала библиотеки. Также присутствует собственная метка с типом данных Counter. Данная метрика увеличивает свое значение каждые 2 секунды.

1. Код приложения необходимо скопировать на сервер в файл slurm_exporter.go:

```go
package main

import (
        "net/http"
        "time"
        log "github.com/Sirupsen/logrus"

        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promhttp"
        "github.com/prometheus/client_golang/prometheus/promauto"
) 

func recordMetrics() {
        go func() {
                for {
                        opsProcessed.Inc()
                        time.Sleep(2 * time.Second)
                }
        }()
}

var (
        opsProcessed = promauto.NewCounter(prometheus.CounterOpts{
                Name: "slurm_example_exporter_processed_ops_total",
                Help: "The total number of processed events",
        })
)

func main() {
        recordMetrics()

        http.Handle("/metrics", promhttp.Handler())
        log.Info("Beginning to serve on port :2112")
        log.Fatal(http.ListenAndServe(":2112", nil))
}
```

2. Устанавливаем go на сервер:

```bash
yum install -y go
```

3. Устанавливаем необходимые зависимости:

```
go get -d -u github.com/prometheus/client_golang/prometheus
go get -d -u github.com/Sirupsen/logrus
```

Первая зависимость – это библиотека prometheus. Вторая библиотека – для реализации логирования.

4. Запускаем exporter:

```bash
go run slurm_exporter.go
```

5. Проверка работоспособности.

Откройте еще одно подключение к серверу monitoring и выполните в новой консоли:

```bash
curl http://localhost:2112/metrics
```

В результате вы должны получить полный список метрик, который предоставляет exporter.

## Application Library

Это самый эффективный способ мониторинга:

- проще и лучше всего сделать бизнес-метрики
- самый надежный источник данных для приложения - само приложение
- экономия потребляемых ресурсов

Надо не увлечься и не превращать систему мониторинга в систему трассировки - задача мониторинга узнать о том, что есть проблема, а не решать саму проблему! То есть не пытаться замониторить каждый возможный запрос, а вместо этого дать алерт, работате в целом система или нет.

Далее уже - делать трейсинг запросов по необходимости уже в системе трассировки.

# Prometheus  



# PromQL  



# Alerting  



# Визуализация данных  



# Advanced usage of Prometheus  

High Availability, федерация.  


# Prometheus в Kubernetes  

