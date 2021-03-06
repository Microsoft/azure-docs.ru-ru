---
title: Использование пакетов DACPAC и пакета BACPAC базы данных SQL Azure SQL Server
description: Дополнительные сведения об использовании пакетов DACPAC и BACPAC в Azure SQL Server
keywords: SQL для пограничных вычислений, sqlpackage
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 40bd0eda16f9f96dd356eef900369ab25854e9f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93392254"
---
# <a name="sql-database-dacpac-and-bacpac-packages-in-sql-edge"></a>Пакеты DACPAC и BACPAC базы данных SQL в SQL Server ребра

SQL Azure для пограничных вычислений — это оптимизированное ядро реляционной СУБД, предназначенное для развертываний в Интернете вещей и пограничных средах. Он основан на последних версиях ядро СУБД Microsoft SQL, что обеспечивает ведущие в отрасли функции повышения производительности, безопасности и обработки запросов. Наряду с передовыми возможностями управления реляционными базами данных, которые предоставляются в SQL Server, SQL Azure для пограничных вычислений предоставляет встроенные функции аналитики в реальном времени и сложной обработки событий.

Azure SQL ребро предоставляет собственный механизм, позволяющий развертывать пакеты [DACPAC и пакета BACPAC базы данных SQL](/sql/relational-databases/data-tier-applications/data-tier-applications) во время выполнения или после развертывания SQL Server.

Пакеты DACPAC и пакета BACPAC базы данных SQL можно развернуть в SQL Server с помощью `MSSQL_PACKAGE` переменной среды. Переменную среды можно настроить с помощью любого из следующих элементов.  
- Расположение локальной папки в контейнере SQL, содержащем файлы DACPAC и BACPAC. Эту папку можно сопоставить с Томом узла с помощью точек подключения или контейнеров томов данных. 
- Путь к локальному файлу в сопоставлении контейнера SQL с пакетом DACPAC или BACPAC-файлом. Этот путь к файлу можно сопоставить с Томом узла с помощью точек подключения или контейнеров томов данных. 
- Путь к локальному файлу в сопоставлении контейнера SQL с ZIP-файлом, содержащим файлы DACPAC или BACPAC. Этот путь к файлу можно сопоставить с Томом узла с помощью точек подключения или контейнеров томов данных. 
- URL-адрес SAS большого двоичного объекта Azure, содержащий файлы DACPAC и BACPAC.
- URL-адрес SAS большого двоичного объекта Azure в файл DACPAC или BACPAC. 

## <a name="use-a-sql-database-dac-package-with-sql-edge"></a>Использование пакета DAC Базы данных SQL с SQL для пограничных вычислений

Чтобы развернуть (или импортировать) пакет DAC базы данных SQL `(*.dacpac)` или BACPAC-файл `(*.bacpac)` с помощью хранилища BLOB-объектов Azure и файла ZIP, выполните следующие действия. 

1. Создайте или извлеките пакет DAC или экспортируйте BACPAC-файл, используя описанный ниже механизм. 
    - Создайте или извлеките пакет DAC Базы данных SQL. Сведения о том, как создать пакет DAC для существующей базы данных SQL Server, см. в статье [Извлечение DAC из базы данных](/sql/relational-databases/data-tier-applications/extract-a-dac-from-a-database/).
    - Экспорт развернутого пакета DAC или базы данных. Сведения о создании BACPAC-файла для существующей базы данных SQL Server см. в разделе [Экспорт приложения уровня данных](/sql/relational-databases/data-tier-applications/export-a-data-tier-application/) .

2. Заархивировать `*.dacpac` `*.bacpac` файл или и передать его в учетную запись хранилища BLOB-объектов Azure. Дополнительные сведения о передаче файлов в хранилище BLOB-объектов Azure см. в статье [Передача, скачивание и составление списка больших двоичных объектов с помощью портала Azure](../storage/blobs/storage-quickstart-blobs-portal.md).

3. Создайте подписанный URL-адрес для ZIP-файла с помощью портала Azure. Дополнительные сведения см. в статье [Делегирование доступа с помощью подписанных URL-адресов (SAS)](../storage/common/storage-sas-overview.md).

4. Обновите конфигурацию модуля SQL для пограничных вычислений, включив в нее URI общего доступа для пакета DAC. Чтобы обновить модуль SQL для пограничных вычислений, выполните следующие действия.

    1. Перейдите к нужному развертыванию Центра Интернета вещей на портале Azure.

    2. В левой области щелкните **IoT Edge**.

    3. На странице **IoT Edge** найдите и выберите IoT Edge, где развернут модуль SQL для пограничных вычислений.

    4. На странице **Устройство IoT Edge** выберите **Set Module** (Настройка модуля).

    5. На странице **Установка модулей** щелкните модуль Azure SQL.

    6. На панели **обновление IOT Edge модуль** выберите **переменные среды**. Добавьте `MSSQL_PACKAGE` переменную среды и укажите URL-адрес SAS, созданный на шаге 3 выше, в качестве значения переменной среды. 

    7. Нажмите кнопку **Обновить**.

    8. На странице **Установка модулей** выберите **Проверка и создать**.

    9. На странице **Установка модулей** выберите **создать**.

5. После обновления модуля файлы пакета будут скачаны, распакованы и развернуты для экземпляра SQL Server.

При каждом перезапуске контейнера Azure SQL ребро SQL Server пытается загрузить сжатый пакет файла и оценить изменения. Если обнаруживается новая версия файла, изменения развертываются в базе данных SQL для пограничных вычислений.

## <a name="known-issue"></a>Известная проблема

Во время некоторых развертываний DACPAC или BACPAC пользователи могут столкнуться со временем ожидания команды, что приведет к сбою операции развертывания DACPAC. Если вы столкнулись с этой проблемой, используйте SQLPackage.exe (или клиентские средства SQL) для применения пакета DACPAC или BACPAC NIC. 

## <a name="next-steps"></a>Дальнейшие действия

- [Развертывание SQL Azure для пограничных вычислений с помощью портала Azure](deploy-portal.md)
- [Потоковая передача данных](stream-data.md)
- [Машинное обучение и ИИ с применением ONNX в SQL для пограничных вычислений](onnx-overview.md)