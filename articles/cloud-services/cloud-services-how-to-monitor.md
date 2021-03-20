---
title: Мониторинг облачной службы Azure (классическая модель) | Документация Майкрософт
description: Сведения о задачах и механизмах для мониторинга облачной службы Azure.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 60f320f86860cca482cdf25c7d93f84dae8c4e5f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98743378"
---
# <a name="introduction-to-cloud-service-classic-monitoring"></a>Общие сведения о мониторинге облачной службы (классическая модель)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Вы можете отслеживать для облачных служб ключевые показатели производительности. Каждая роль облачной службы собирает минимальный набор данных: использование ЦП, сетевых и дисковых ресурсов. Если к роли облачной службы применено расширение `Microsoft.Azure.Diagnostics`, эта роль может собирать дополнительные элементы данных. В этой статье содержатся базовые сведения о системе диагностике Azure для облачных служб.

Базовый уровень мониторинга включает сбор данных от счетчиков производительности для экземпляра роли с интервалом в 3 минуты. Базовые данные мониторинга не сохраняются в учетной записи хранения, то есть не требуют дополнительных затрат.

Расширенный уровень мониторинга включает сбор дополнительных метрик с интервалами в 5 минут, 1 час и 12 часов. Статистические данные хранятся в таблицах в учетной записи хранения и удаляются по прошествии 10 дней. Учетная запись хранения настраивается отдельно для каждой роли. Этот параметр можно указать в строке подключения в файлах [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) и [.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg).


## <a name="basic-monitoring"></a>Базовый мониторинг

Как мы уже упоминали во введении, облачная служба автоматически собирает основные данные мониторинга из узла виртуальной машины. Сюда входят процент использования ЦП и данные операций сетевого ввода-вывода и дискового чтения/записи. Собранные данные мониторинга автоматически отображаются на портале Azure на странице общих сведений и метрик для облачной службы. 

Для базового мониторинга не требуется учетная запись хранения. 

![Плитки базового мониторинга для облачной службы](media/cloud-services-how-to-monitor/basic-tiles.png)

## <a name="advanced-monitoring"></a>Расширенный мониторинг

Расширенный мониторинг включает использование расширения **система диагностики Azure** (и, при необходимости, пакета SDK Application Insights) для роли, которую требуется отслеживать. Расширение диагностики использует файл конфигурации (отдельный для каждой роли) с именем **diagnostics.wadcfgx**, в котором можно настроить собираемые метрики диагностики. Расширение диагностики Azure собирает и хранит данные в учетной записи хранения Azure. Эти параметры настраиваются в **WADCFGX-**, [CSDEF-](cloud-services-model-and-package.md#servicedefinitioncsdef) и [CSCFG-](cloud-services-model-and-package.md#serviceconfigurationcscfg) файлах. Это означает, что расширенный мониторинг потребует дополнительных расходов.

При создании каждой роли Visual Studio добавляет в нее расширение системы диагностики Azure. Расширение диагностики может собирать данные следующих типов:

* настраиваемые счетчики производительности;
* Журналы приложений
* Журналы событий Windows
* источник событий .NET;
* Журналы IIS
* Трассировка событий Windows на основе манифестов
* Аварийные дампы
* пользовательские журналы ошибок.

> [!IMPORTANT]
> Хотя все эти данные объединяются в учетную запись хранения, портал **не предоставляет собственный** способ построения диаграммы данных. Настоятельно рекомендуем интегрировать в приложение другую службу, например Application Insights.

## <a name="setup-diagnostics-extension"></a>Настройка расширения системы диагностики

Если у вас еще нет **классической** учетной записи хранения, [создайте ее](../storage/common/storage-account-create.md). Для создания учетной записи хранения обязательно используйте **классическую модель развертывания**.

Затем перейдите к ресурсу **Учетная запись хранения (классика)**. Выберите **Параметры**  >  **ключи доступа** и скопируйте значение **первичной строки подключения** . Это значение вы примените для облачной службы. 

Чтобы включить расширенную диагностику, необходимо внести изменения в два файла конфигурации: **ServiceDefinition.csdef** и **ServiceConfiguration.cscfg**.

### <a name="servicedefinitioncsdef"></a>ServiceDefinition.csdef

Добавьте в файл **ServiceDefinition.csdef** новый параметр `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` для каждой роли, которая использует расширенную диагностику. Visual Studio автоматически добавляет это значение в файл при создании нового проекта. Если же оно отсутствует, добавьте его прямо сейчас. 

```xml
<ServiceDefinition name="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
```

Значение определяет новый параметр, который должен быть добавлен в каждый файл **ServiceConfiguration.cscfg**. 

Скорее всего, у вас есть два **CSCFG**-файла. Один из них (с именем **ServiceConfiguration.cloud.cscfg**) используется для развертывания в Azure, а другой (с именем **ServiceConfiguration.local.cscfg**) — для локальных развертываний в эмулированной среде. Откройте для редактирования каждый файл **.cscfg**. Добавьте в них параметр с именем `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString`. Задайте значение **основной строки подключения** классической учетной записи хранения. Если вы хотите использовать локальное хранилище на компьютере разработки, используйте значение `UseDevelopmentStorage=true`.

```xml
<ServiceConfiguration serviceName="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2015-04.2.6">
  <Role name="WorkerRoleWithSBQueue1">
    <Instances count="1" />
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=mystorage;AccountKey=KWwkdfmskOIS240jnBOeeXVGHT9QgKS4kIQ3wWVKzOYkfjdsjfkjdsaf+sddfwwfw+sdffsdafda/w==" />
      
      <!-- or use the local development machine for storage
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
      -->
```

## <a name="use-application-insights"></a>Использование Application Insights

Когда вы публикуете облачную службу из Visual Studio, вы можете настроить для нее отправку данных диагностики в Application Insights. Можно отправить данные в уже существующий ресурс Application Insights или создать новый ресурс Azure в рамках процесса публикации. Application Insights может отслеживать для облачной службы статистику доступности, производительности, сбоев и использования. Также в Application Insights можно добавлять пользовательские диаграммы, чтобы контролировать по ним самые важные данные. Чтобы получить данные об экземпляре роли, добавьте в проект облачной службы пакет SDK для Application Insights. Дополнительные сведения об интеграции с Application Insights см. в статье [Application Insights для облачных служб Azure](../azure-monitor/app/cloudservices.md).

Обратите внимание, что в Application Insights могут отображаться счетчики производительности (и другие параметры), настроенные в расширении службы диагностики Azure, но по-настоящему широкие возможности вы получите только при интеграции пакета SDK для Application Insights в рабочие роли и веб-роли.


## <a name="next-steps"></a>Дальнейшие действия

- [Подробнее об Application Insights для облачных служб Azure](../azure-monitor/app/cloudservices.md)
- [Настройка счетчиков производительности](diagnostics-performance-counters.md)




