---
title: Получение данных о погоде от партнеров по погоде
description: В этой статье описывается получение данных о погоде от партнеров.
author: sunasing
ms.topic: article
ms.date: 03/31/2020
ms.author: sunasing
ms.openlocfilehash: bb28c517e353af6b8c1ee0cad788ff41b971918c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96460872"
---
# <a name="get-weather-data-from-weather-partners"></a>Получение данных о погоде от партнеров по погоде

Azure Фармбеатс поможет вам перенести данные о погоде из поставщиков данных погоды с помощью инфраструктуры соединителя на основе DOCKER. С помощью этой платформы поставщики данных Weather реализуют DOCKER, который можно интегрировать с Фармбеатс. В настоящее время поддерживается следующий поставщик данных о погоде.

  ![Партнеры Фармбеатс](./media/get-sensor-data-from-sensor-partner/dtn-logo.png)
  
   [дтн](https://www.dtn.com/dtn-content-integration/)

Данные погоды можно использовать для создания ценных сведений и создания моделей AI или ML в Фармбеатс.

## <a name="before-you-start"></a>Перед началом работы

Чтобы получить данные о погоде, убедитесь, что [установлен фармбеатс](./install-azure-farmbeats.md). Интеграция с погодой поддерживается в версиях 1.2.11 и выше. 

## <a name="enable-weather-integration-with-farmbeats"></a>Включение интеграции погоды с Фармбеатс

Чтобы начать получать данные о погоде в Фармбеатс Датахуб:

1. Перейдите к Фармбеатс Датахуб Swagger `https://farmbeatswebsite-api.azurewebsites.net/swagger` .

2. Перейдите к API/партнер и выполните запрос POST. Используйте следующие входные полезные данные:

   ```json
   {  
 
     "dockerDetails": {  
       "credentials": { 
         "username": "<credentials to access private docker - not required for public docker>", 
         "password": "<credentials to access private docker – not required for public docker>"  
       },  
       "imageName" : "<docker image name",
       "imageTag" : "<docker image tag, default:latest>",
       "azureBatchVMDetails": {  
         "batchVMSKU" : "<VM SKU. Default is standard_d2_v2>",  
         "dedicatedComputerNodes" : 1,
         "nodeAgentSKUID": "<Node SKU. Default is batch.node.ubuntu 18.04>"  
       }
     },
     "partnerCredentials": {  
       "key1": "value1",  
       "key2": "value2"  
     },  
     "partnerType": "Weather",  
     "name": "<Name of the partner>",  
     "description": "<Description>",
     "properties": {  }  
   }  
   ```

   Например, чтобы получить данные о погоде из ДТН, используйте следующие полезные данные. Имя и описание можно изменить в соответствии с вашими предпочтениями.

   > [!NOTE]
   > Для следующего шага требуется ключ API. Чтобы получить ключ для подписки ДТН, обратитесь в ДТН.

   ```json
   {
 
     "dockerDetails": {
       "imageName": "dtnweather/dtn-farm-beats",
       "imageTag": "latest",
       "azureBatchVMDetails": {
         "batchVMSKU": "standard_d2_v2",
         "dedicatedComputerNodes": 1,
         "nodeAgentSKUID": "batch.node.ubuntu 18.04"
       }
     },
     "partnerCredentials": {
      "apikey": "<API key from DTN>"
      },
     "partnerType": "Weather",
     "name": "dtn-weather",
     "description": "DTN registered as a Weather Partner in FarmBeats"
   }  
   ```

   > [!NOTE]
   > Дополнительные сведения об объекте Partner см. в [приложении](get-weather-data-from-weather-partner.md#appendix) в этой статье.

   На предыдущем шаге будут подготавливаться ресурсы, позволяющие запускать DOCKER в среде Фармбеатс клиента.  

   Инициализация ресурсов занимает около 10 – 15 минут.

3. Проверьте состояние объекта/партнер, созданного на предыдущем шаге. Чтобы проверить состояние, выполните запрос GET к API/партнер и проверьте состояние объекта Partner. После того как Фармбеатс подготовит партнера к успешному подготовке, состояние изменится на **активно**.

4. В API/Жобтипе выполните запрос GET. Проверьте созданные ранее задания Weather в процессе добавления партнеров. В заданиях Weather поле **пипелиненаме** имеет следующий формат: **Partner-name_partner-type_job-Name**.

      Теперь у вашего экземпляра Фармбеатс есть активный партнер по погоде данных. Можно выполнять задания для запроса данных о погоде для определенного расположения (широты и долготы) и диапазона дат. Типы заданий будут иметь подробные сведения о параметрах, необходимых для выполнения заданий погоды.

      Например, для ДТН будут созданы следующие типы заданий:
   
      - **get_dtn_daily_observations**: получение ежедневных наблюдений за расположением и периодом времени.
      - **get_dtn_daily_forecasts**: получение ежедневных прогнозов по местоположению и периоду времени.
      - **get_dtn_hourly_observations**: получение почасовых наблюдений за расположение и период времени.
      - **get_dtn_hourly_forecasts**: получение почасовых прогнозов по местоположению и периоду времени.

6. Запишите идентификатор и параметры типов заданий.

7. Перейдите к API вызовом/Jobs и выполните запрос POST в/Жобс. Используйте следующие входные полезные данные:

   ```json
    {
         "typeId": "<id of the JobType>",
         "arguments": {
           "additionalProp1": {},
           "additionalProp2": {},
           "additionalProp3": {}
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
       }
   ```

   Например, чтобы запустить **get_dtn_daily_observations**, используйте следующие полезные данные:

   ```json
   { 
         "typeId": "<id of the JobType>",
         "arguments": {
               "latitude": 47.620422,
               "longitude": -122.349358,
               "days": 5
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
   }
   ```

8. На предыдущем шаге выполняются задания Weather, определенные в партнерской DOCKER и принимающие данные о погоде в Фармбеатс. Состояние задания можно проверить, сделав запрос GET на/Жобс.. В ответе найдите **currentState**. По завершении для **currentState** установлено значение " **успех**".

## <a name="query-ingested-weather-data"></a>Запрос полученных данных о погоде

После завершения заданий Weather можно запросить полученные данные о погоде для создания моделей или практичных ценных сведений с помощью интерфейсов API Фармбеатс Датахуб.

Для запроса данных о погоде с помощью REST API Фармбеатс:

1. В Фармбеатс Датахуб [Swagger](https://yourdatahub.azurewebsites.net/swagger)перейдите к API/веасердаталокатион и выполните запрос GET. Ответ содержит объекты/Веасердаталокатион, созданные для расположения (широты и долготы), указанное заданием. Запишите **идентификатор** и **веасердатамоделид** объектов.

2. Выполните запрос GET/{ID} в API/Веасердатамодел для **веасердатамоделид** , как было сделано ранее. Модель данных Weather отображает все метаданные и сведения о полученных данных о погоде. Например, в объекте модели данных Weather мера Weather содержит сведения о том, какие сведения о погоде поддерживаются и в каких типах и единицах. Пример:

   ```json
   {
       "name": "Temperature <name of the weather measure - this is what we will receive as part of the queried weather data>",
       "dataType": "Double <Data Type - eg. Double, Enum>",
       "type": "AmbientTemperature <Type of measure eg. AmbientTemperature, Wind speed etc.>",
       "unit": "Celsius <Unit of measure eg. Celsius, Percentage etc.>",
       "aggregationType": "None <either of None, Average, Maximum, Minimum, StandardDeviation, Sum, Total>",
       "description": "<Description of the measure>"
   }
   ```

   Запишите ответ вызова GET/{ID} для модели данных weather.

3. Перейдите к API телеметрии и выполните запрос POST. Используйте следующие входные полезные данные:

   ```json
   {
       "weatherDataLocationId": "<id from step 1 above>",
     "searchSpan": {
       "from": "2020-XX-XXT07:30:00Z",
       "to": "2020-XX-XXT07:45:00Z"
     }
   }
   ```

    В ответе отображаются данные о погоде, доступные для указанного диапазона времени:

   ```json
   {
     "timestamps": [
       "2020-XX-XXT07:30:00Z",
       "2020-XX-XXT07:45:00Z"
     ],
     "properties": [
       {
         "values": [
           "<id of the weatherDataLocation>",
           "<id of the weatherDataLocation>"
         ],
         "name": "Id",
         "type": "String"
       },
       {
         "values": [
           29.1,
           30.2
         ],
         "name": "Temperature <name of the WeatherMeasure as defined in the WeatherDataModel object>",
         "type": "Double <Data Type of the value - eg. Double>"
       }
     ]
   }
   ```

В предыдущем примере ответ показывает данные для двух меток времени. Он также показывает имя меры (температуру) и значения данных о погоде в двух метках времени. Чтобы интерпретировать тип и единицу сообщаемых значений, обратитесь к связанной модели данных погоды.

## <a name="troubleshoot-job-failures"></a>Устранение ошибок заданий

Чтобы устранить неполадки заданий, [Проверьте журналы заданий](troubleshoot-azure-farmbeats.md#weather-data-job-failures).


## <a name="appendix"></a>Приложение

|        Партнер   |  Сведения   |
| ------- | -------             |
|     Доккердетаилс — imageName         |          Имя образа DOCKER. Например, docker.io/mydockerimage (изображение в hub.docker.com) или myazureacr.azurecr.io/mydockerimage (образ в реестре контейнеров Azure) и т. д. Если реестр не указан, по умолчанию используется hub.docker.com.      |
|          Доккердетаилс — Имажетаг             |         Имя тега образа DOCKER. Значение по умолчанию — "latest".     |
|  Доккердетаилс — учетные данные      |  Учетные данные для доступа к частному DOCKER. Партнер предоставляет учетные данные.   |
|  Доккердетаилс-Азуребатчвмдетаилс-Батчвмску     |    Номер SKU виртуальной машины пакетной службы Azure. Дополнительные сведения см. в разделе [все доступные виртуальные машины Linux](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). <BR> <BR> Допустимые значения: "Small", "ExtraLarge", "Large", "A8", "A9", "Medium", "A5", "A6", "A7", "STANDARD_D1", "STANDARD_D2", "STANDARD_D3", "STANDARD_D4 ', ' STANDARD_D11 ', ' STANDARD_D12 ', ' STANDARD_D13 ', ' STANDARD_D14 ', ' A10 ', ' A11 ', ' STANDARD_D1_V2 ', ' STANDARD_D2_V2 ', ' STANDARD_D3_V2 ', ' STANDARD_D4_V2 ', ' STANDARD_D11_V2 ', ' STANDARD_D12_V2 ', ' STANDARD_D13_V2 ', ' STANDARD_D14_V2 ', ' STANDARD_G1 ' , "STANDARD_G5", "STANDARD_D5_V2", "BASIC_A1", "BASIC_A2", "BASIC_A3", "BASIC_A4", "STANDARD_A1", "STANDARD_A2", "STANDARD_A3", "STANDARD_A4", "STANDARD_A5", "STANDARD_A6", "STANDARD_A7", "STANDARD_A8", "STANDARD_A9", "STANDARD_A10", "STANDARD_A11", "STANDARD_D15_V2", "STANDARD_F1", "STANDARD_F2", "STANDARD_F4", "STANDARD_F8", "STANDARD_F16", "STANDARD_NV6", "STANDARD_NV12", "STANDARD_NV24", "STANDARD_NC6" , "STANDARD_H8", "STANDARD_H8m", "STANDARD_H16", "STANDARD_H16m", "STANDARD_H16mr", "STANDARD_H16r", "STANDARD_A1_V2", "STANDARD_A2_V2", "STANDARD_A4_V2", "STANDARD_A8_V2", "STANDARD_A2m_V2", "STANDARD_A4m_V2", "STANDARD_A8m_V2", "STANDARD_M64ms", "STANDARD_M128s" и "STANDARD_D2_V3". *Значение по умолчанию — "STANDARD_D2_V2".*  |
|    Доккердетаилс-Азуребатчвмдетаилс-Дедикатедкомпутернодес   |  Количество выделенных узлов компьютеров на пул пакетной службы. Значение по умолчанию — 1. |
|    Доккердетаилс-Азуребатчвмдетаилс-Нодеажентскуид          |    Идентификатор SKU агента узла пакетной службы Azure. Сейчас поддерживается только агент узла пакетной службы "пакет. Node. Ubuntu 18,04".    |
| Доккердетаилс — Партнеркредентиалс | Учетные данные для вызова API-партнера в DOCKER. Партнер предоставляет эти сведения на основе поддерживаемого механизма авторизации; Например, имя пользователя и пароль или ключи API. |
| партнертипе | "Погода". Другие типы партнеров в Фармбеатс: "датчик" и "образ".  |
|  name   |   Требуемое имя участника в системе Фармбеатс.   |
|  description; |  Описание.   |

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы запросили данные датчиков из экземпляра Azure Фармбеатс, Узнайте, как [создавать карты](generate-maps-in-azure-farmbeats.md#generate-maps) для ферм.