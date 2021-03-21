---
title: Устранение неполадок с помощью журналов Аналитики Службы хранилища
description: Выявление и устранение проблем с задержкой с помощью аналитических журналов службы хранилища Azure и оптимизация клиентского приложения.
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/21/2019
ms.service: storage
ms.subservice: common
services: storage
tags: ''
ms.openlocfilehash: 1e6033f9a8f4cecd2429eca67a3d58e54d7ae1f6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99221114"
---
# <a name="troubleshoot-latency-using-storage-analytics-logs"></a>Устранение неполадок с помощью журналов Аналитики Службы хранилища

Диагностика и устранение неполадок — это ключевой навык создания и поддержки клиентских приложений с помощью службы хранилища Azure.

Из-за распределенного характера приложения Azure диагностика и устранение неполадок как ошибок, так и проблем с производительностью могут быть более сложными, чем в традиционных средах.

Ниже показано, как определить и устранить проблемы задержки с помощью аналитических журналов службы хранилища Azure и оптимизировать клиентское приложение.

## <a name="recommended-steps"></a>Рекомендуемые действия

1. Скачайте [журналы Аналитика службы хранилища](./manage-storage-analytics-logs.md#download-storage-logging-log-data).

2. Используйте следующий сценарий PowerShell для преобразования журналов необработанного формата в табличный формат:

   ```Powershell
   $Columns = 
        (   "version-number",
            "request-start-time",
            "operation-type",
            "request-status",
            "http-status-code",
            "end-to-end-latency-in-ms",
            "server-latency-in-ms",
            "authentication-type",
            "requester-account-name",
            "owner-account-name",
            "service-type",
            "request-url",
            "requested-object-key",
            "request-id-header",
            "operation-count",
            "requester-ip-address",
            "request-version-header",
            "request-header-size",
            "request-packet-size",
            "response-header-size",
            "response-packet-size",
            "request-content-length",
            "request-md5",
            "server-md5",
            "etag-identifier",
            "last-modified-time",
            "conditions-used",
            "user-agent-header",
            "referrer-header",
            "client-request-id"
        )

   $logs = Import-Csv “REPLACE THIS WITH FILE PATH” -Delimiter ";" -Header $Columns

   $logs | Out-GridView -Title "Storage Analytic Log Parser"
   ```

3. Скрипт запустит окно графического пользовательского интерфейса, в котором можно отфильтровать данные по столбцам, как показано ниже.

   ![Окно анализатора аналитических журналов хранилища](media/troubleshoot-latency-storage-analytics-logs/storage-analytic-log-parser-window.png)
 
4. Ограничьте записи журнала, основанные на "Operation-Type", и найдите запись журнала, созданную в течение интервала времени проблемы.

   ![Записи журнала типов операций](media/troubleshoot-latency-storage-analytics-logs/operation-type.png)

5. Во время возникновения проблемы важны следующие значения:

   * Operation-Type = BLOB
   * Request-Status = Саснетворкеррор
   * Сквозная задержка в МС = 8453
   * Задержка сервера в МС = 391

   Сквозная задержка вычисляется с помощью следующего уравнения:

   * Сквозная задержка = Server-Latency + задержка клиента

   Вычислите задержку клиента с помощью записи журнала:

   * Задержка клиента = сквозная задержка — Server-Latency

        Пример: 8453 – 391 = 8062ms

   В следующей таблице приведены сведения о OperationType и результатах состоянии RequestStatus высокой задержки.

   | Тип большого двоичного объекта |Состоянии RequestStatus =<br>Успешно|Состоянии RequestStatus =<br>ЖЕСТКИХ NetworkError|Рекомендация|
   |---|---|---|---|
   |GetBlob|Да|Нет|[**Операция с большим двоичным объектом:** Состоянии RequestStatus = успешное завершение](#getblob-operation-requeststatus--success)|
   |GetBlob|Нет|Да|[**Операция с большим двоичным объектом:** Состоянии RequestStatus = (SAS) NetworkError](#getblob-operation-requeststatus--sasnetworkerror)|
   |PutBlob|Да|Нет|[**Операция размещения:** Состоянии RequestStatus = успешное завершение](#put-operation-requeststatus--success)|
   |PutBlob|Нет|Да|[**Операция размещения:** Состоянии RequestStatus = (SAS) NetworkError](#put-operation-requeststatus--sasnetworkerror)|

## <a name="status-results"></a>Результаты состояния

### <a name="getblob-operation-requeststatus--success"></a>Операция BLOB: состоянии RequestStatus = Success

Проверьте следующие значения, как упоминалось на шаге 5 раздела "Рекомендуемые действия".

* Сквозная задержка
* Server-Latency
* Client-Latency

В **операции большого двоичного объекта** с **состоянии RequestStatus = Success**, если **Максимальное время** тратится на **задержку клиента**, это означает, что служба хранилища Azure тратит большой объем времени на запись данных клиенту. Эта задержка указывает на ошибку Client-Side.

**Рекомендация.**

* Исследуйте код в клиенте.
* Используйте Wireshark, анализатор сообщений Майкрософт или Tcping, чтобы исследовать проблемы с сетевым подключением от клиента. 

### <a name="getblob-operation-requeststatus--sasnetworkerror"></a>Операция с большим двоичным объектом: состоянии RequestStatus = (SAS) NetworkError

Проверьте следующие значения, как упоминалось на шаге 5 раздела "Рекомендуемые действия".

* Сквозная задержка
* Server-Latency
* Client-Latency

В **операции большого двоичного объекта** с **состоянии REQUESTSTATUS = (SAS) NetworkError**, если **Максимальное время** тратится на **задержку клиента**, наиболее распространенной проблемой является отключение клиента до истечения времени ожидания в службе хранилища.

**Рекомендация.**

* Изучите код клиента, чтобы понять, когда и почему клиент отключается от службы хранилища.
* Используйте Wireshark, анализатор сообщений Майкрософт или Tcping, чтобы исследовать проблемы с сетевым подключением от клиента. 

### <a name="put-operation-requeststatus--success"></a>Операция размещения: состоянии RequestStatus = Success

Проверьте следующие значения, как упоминалось на шаге 5 раздела "Рекомендуемые действия".

* Сквозная задержка
* Server-Latency
* Client-Latency

В **операции размещения** с **состоянии RequestStatus = Success**, если **Максимальное время** тратится на **задержку клиента**, это означает, что клиенту требуется больше времени для отправки данных в службу хранилища Azure. Эта задержка указывает на ошибку Client-Side.

**Рекомендация.**

* Исследуйте код в клиенте.
* Используйте Wireshark, анализатор сообщений Майкрософт или Tcping, чтобы исследовать проблемы с сетевым подключением от клиента. 

### <a name="put-operation-requeststatus--sasnetworkerror"></a>Операция размещения: состоянии RequestStatus = (SAS) NetworkError

Проверьте следующие значения, как упоминалось на шаге 5 раздела "Рекомендуемые действия".

* Сквозная задержка
* Server-Latency
* Client-Latency

В **операции PutBlob** с **состоянии REQUESTSTATUS = (SAS) NetworkError**, если **Максимальное время** тратится на **задержку клиента**, наиболее распространенной проблемой является то, что клиент отключается до истечения времени ожидания в службе хранилища.

**Рекомендация.**

* Изучите код клиента, чтобы понять, когда и почему клиент отключается от службы хранилища.
* Используйте Wireshark, анализатор сообщений Майкрософт или Tcping, чтобы исследовать проблемы с сетевым подключением от клиента.