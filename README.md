# Домашнее задание к занятию "10.02. Системы мониторинга"

## Обязательные задания

1. Опишите основные плюсы и минусы pull и push систем мониторинга.

pull:
 - более высокий уровень контроля за метриками
 - более высокий уровень безопасности
 - нет необходимости открывать порт вовне
 минусы: потребляет больше ресурсов, менее производительный
 
 push:
  - возможность отправки метрик в разные источники
  - работает за NAT
  - менее загружен ненужными алертами
  минусы: флуд, могут придти ненужные данные

2. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus - pull(push с доп компонентой)
    - TICK - push
    - Zabbix - push (pull с доп компонентой)
    - VictoriaMetrics - push/pull
    - Nagios - pull

3. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, 
используя технологии docker и docker-compose.(по инструкции ./sandbox up )

В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

    - curl http://localhost:8086/ping
    
>   *   Trying 127.0.0.1:8086...
>   * TCP_NODELAY set
>   * Connected to localhost (127.0.0.1) port 8086 (#0)
>    GET /ping HTTP/1.1
>    Host: localhost:8086
>    User-Agent: curl/7.68.0
>     Accept: */*
>    
>   * Mark bundle as not supporting multiuse
>   < HTTP/1.1 204 No Content
>   < Content-Type: application/json
>   < Request-Id: 9386547f-a80f-11eb-8044-000000000000
>   < X-Influxdb-Version: 1.3.5
>   < Date: Wed, 28 Apr 2021 10:50:41 GMT
>   < 
>   * Connection #0 to host localhost left intact
    
    - curl http://localhost:8888

>   <!DOCTYPE html><html><head><meta http-equiv="Content-type" content="text/html; charset=utf-8"><title>Chronograf</title><link rel="icon shortcut" 
>   href="/favicon.fa749080.ico"><link rel="stylesheet" href="/src.9cea3e4e.css"></head><body> <div id="react-root" data-basepath=""></div> <script 
>   src="/src.a969287c.js"></script> </body></html>21:17:59 ~ sergey@Intel8086:~/git/sandbox (master=)

    - curl http://localhost:9092/kapacitor/v1/ping
    
>   *   Trying 127.0.0.1:9092...
>   * TCP_NODELAY set
>   * Connected to localhost (127.0.0.1) port 9092 (#0)
>    GET /kapacitor/v1/ping HTTP/1.1
>     Host: localhost:9092
>     User-Agent: curl/7.68.0
>     Accept: */*
>    
>   * Mark bundle as not supporting multiuse
>   < HTTP/1.1 204 No Content
>   < Content-Type: application/json; charset=utf-8
>   < Request-Id: 9af33711-a80f-11eb-8033-000000000000
>   < X-Kapacitor-Version: 1.3.3
>   < Date: Wed, 28 Apr 2021 10:50:54 GMT
>   < 
>   * Connection #0 to host localhost left intact

А также скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

<img width="1533" alt="HW_mon2" src="https://user-images.githubusercontent.com/99620296/204638308-09b7d01e-f7c0-4018-b688-09454a89d560.png">


P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`

4. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs).
    - Добавьте в конфигурацию telegraf плагин - [disk](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/disk):
    ```
    [[inputs.disk]]
      ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
    ```
    - Так же добавьте в конфигурацию telegraf плагин - [mem](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/mem):
    ```
    [[inputs.mem]]
    ```
    - После настройки перезапустите telegraf.
 
    - Перейдите в веб-интерфейс Chronograf (`http://localhost:8888`) и откройте вкладку `Data explorer`.
    - Нажмите на кнопку `Add a query`
    - Изучите вывод интерфейса и выберите БД `telegraf.autogen`
    - В `measurments` выберите mem->host->telegraf_container_id , а в `fields` выберите used_percent. 
    Внизу появится график утилизации оперативной памяти в контейнере telegraf.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. 
    Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.
    - Приведите скриншот с отображением
    метрик утилизации места на диске (disk->host->telegraf_container_id) из веб-интерфейса.  
    
    <img width="1836" alt="HW_mon21" src="https://user-images.githubusercontent.com/99620296/204638502-109ac6ec-ccd4-4695-a0e3-7b3f6b8d4a87.png">


5. Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
режима privileged:
```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

<img width="359" alt="HW_mon3" src="https://user-images.githubusercontent.com/99620296/204638573-5a8bc680-f2d4-4614-b3c0-05e96d7883ec.png">

После настройки перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

В веб-интерфейсе откройте вкладку `Dashboards`. Попробуйте создать свой dashboard с отображением:

    - утилизации ЦПУ
    - количества использованного RAM
    - утилизации пространства на дисках
    - количество поднятых контейнеров
    - аптайм
    - ...
    - фантазируйте)
    
    ---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
