## Создание дашборда для мониторинга информации от датчиков и настройка алертов о событиях

Данный сценарий выполнен на serverless сервисах и не использует виртуальных машин.

В качестве примера, сценарий реализует мониторинг датчиков серверной комнаты,
однако используя такой же подход можно реализовать мониторинг любых устройств. Например:
 - Мониторинг сетевого оборудования
 - Мониторинг параметров работы станков
 - Мониторинг показателей качества воздуха
 - Мониторинг климатических установок
 и т.д.

Детальное описание сценария доступно в [документации](https://cloud.yandex.ru/docs/solutions/iot/monitoring).

#### Описание сценария
Существует серверная комната, в которой установлена система мониторинга,
собирающая следующие параметры:
 - Температура в комнате
 - Влажность в комнате
 - Датчик воды на полу (Есть вода /нет воды)
 - Детектор дыма (Есть дым /нет дыма)
 - Датчик открытия двери в помещение (Дверь открыта / дверь закрыта)
 - Датчик открытия дверцы серверной стойки (Дверца открыта / дверца закрыта)

Данные от датчиков приходят на контроллер, который их передает один раз в минуту на сервер по протоколу MQTT в следующем формате:

```
{
    "DeviceId":"e7a68b2d-464e-4222-88bd-c9e8d10a70cd",
    "TimeStamp":"2020-05-21T10:16:43Z",
    "Values":[
        {"Type":"Float","Name":"Humidity","Value":"12.456"},
        {"Type":"Float","Name":"Temperature","Value":"-23.456"},
        {"Type":"Bool","Name":"Water sensor","Value":"false"},
        {"Type":"Bool","Name":"Smoke sensor","Value":"false"},
        {"Type":"Bool","Name":"Room door sensor","Value":"true"},
        {"Type":"Bool","Name":"Rack door sensor","Value":"false"}
    ]
}
```

В рамках данного сценария создается дашборд и настраиваются алерты для каждого из показателей.
Сценарий реализован на базе эмулятора устройства, сделанного на базе Yandex Functions.


#### Функция эмулятора устройства

Код функции находится в файле [device-emulator.js](device-emulator.js).

Точка входа функции - device-emulator.handler
Функцию необходимо запускать с помощью триггера-таймера с [CRON выражением](https://cloud.yandex.ru/docs/functions/concepts/trigger/timer#cron-expression) `* * * * ? *` (вызов раз в 1 минуту).
В функции должен использоваться сервисный аккаунт, имеющий роль iot.devices.writer, для того, чтобы можно было отправлять данные в Yandex IoT Core.

Функция использует следующие переменные окружения:
- DEVICE_ID — уникальный ID нашего эмулируемого устройства
- HUMIDITY_SENSOR_VALUE — базовое значение показания датчика влажности
- TEMPERATURE_SENSOR_VALUE — базовое значение показания датчика температуры
- RACK_DOOR_SENSOR_VALUE — показания датчика открытия дверцы стойки
- ROOM_DOOR_SENSOR_VALUE — показания датчика открытия двери в серверную комнату
- SMOKE_SENSOR_VALUE — показания детектора дыма
- WATER_SENSOR_VALUE — показания детектора воды
- IOT_CORE_DEVICE_ID — DeviceID устройства в Yandex IoT Core


#### Функция отправки данных в Yandex Monitoring

Код функции находится в файле [monitoring.py](monitoring.py).
Функцию необходимо запускать с помощью триггера Yandex IoT Core, который будет вызывать ее при получении новой посылки от устройства.
В функции должен использоваться сервисный аккаунт, имеющий роль editor, для того, чтобы можно было отправлять данные в Yandex Monitoring.

Функция использует следующие переменные окружения:
 - VERBOSE_LOG — если True, то функция выводит в журнал информацию о своем выполнении
 - METRICS_FOLDER_ID — ID каталога Яндекс.Облака, в мониторинг которого будут отправлены данные от устройства.
