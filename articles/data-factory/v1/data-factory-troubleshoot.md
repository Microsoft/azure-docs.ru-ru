---
title: Устранение неполадок фабрики данных Azure
description: Узнайте, как устранять неполадки при использовании фабрики данных Azure.
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
robots: noindex
ms.openlocfilehash: 70c1aecd358b52a7b5b62ba7cb8f3badffef2998
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104779925"
---
# <a name="troubleshoot-data-factory-issues"></a>Устранение неполадок Фабрики данных
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. 

В этой статье приводятся советы по устранению неполадок, возникающих при использовании фабрики данных Azure. В этой статье перечислены не все возможные проблемы использования службы, однако рассматриваются некоторые вопросы и приводятся общие рекомендации по устранению неполадок.   

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="troubleshooting-tips"></a>Советы по устранению неполадок
### <a name="error-the-subscription-is-not-registered-to-use-namespace-microsoftdatafactory"></a>Ошибка "Подписка не зарегистрирована для использования пространства имен Microsoft.DataFactory"
Если эта ошибка возникает, поставщик ресурсов фабрики данных Azure не был зарегистрирован на компьютере. Выполните следующие действия.

1. Запустите Azure PowerShell.
2. Войдите в свою учетную запись Azure с помощью следующей команды:

   ```powershell
   Connect-AzAccount
   ```

3. Выполните следующую команду, чтобы зарегистрировать поставщик фабрики данных Azure:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
   ```

### <a name="problem-unauthorized-error-when-running-a-data-factory-cmdlet"></a>Проблема: ошибка авторизации при выполнении командлета фабрики данных
Скорее всего, для Azure PowerShell используется неправильная учетная запись или подписка Azure. Чтобы выбрать правильную учетную запись и подписку Azure для Azure PowerShell, используйте такие командлеты:

1. Connect-AzAccount — используйте правильный идентификатор пользователя и пароль.
2. Get-AzSubscription — Просмотр всех подписок для учетной записи.
3. Select-AzSubscription &lt; имя подписки &gt; — выберите подходящую подписку. Используйте подписку, которая использовалась для создания фабрики данных на портале Azure.

### <a name="problem-fail-to-launch-data-management-gateway-express-setup-from-azure-portal"></a>Проблема: не удается запустить экспресс-установку шлюза управления данными с портала Azure
Для экспресс-установки шлюза управления данными требуется Internet Explorer или другой веб-браузер, совместимый с Microsoft ClickOnce. Если не удается запустить экспресс-установку, выполните одно из следующих действий:

* Используйте Internet Explorer или другой веб-браузер, совместимый с Microsoft ClickOnce.

    Если вы используете Chrome, перейдите в [веб-магазин Chrome](https://chrome.google.com/webstore/), выполните поиск по ключевому слову "ClickOnce", выберите одно из расширений ClickOnce и установите его.

    То же самое (установку надстройки) сделайте и в случае с браузером Firefox. Нажмите кнопку "Открыть меню" на панели инструментов (три горизонтальные линии в правом верхнем углу), нажмите кнопку "Надстройки", введите "ClickOnce" в строку поиска, выберите одно из расширений ClickOnce и установите его.
* Щелкните ссылку **Установка вручную** в той же колонке на портале. Этот подход используется для скачивания файла установки и его запуска вручную. После успешного завершения установки откроется диалоговое окно настройки шлюза управления данными. Скопируйте **ключ** на экране портала и используйте его в диспетчере конфигурации, чтобы вручную зарегистрировать шлюз в службе.  

### <a name="problem-fail-to-connect-to-sql-server"></a>Проблема: не удалось подключиться к SQL Server
Запустите **диспетчер конфигурации шлюза управления данными** на компьютере шлюза и используйте вкладку **Устранение неполадок**, чтобы проверить подключение к SQL Server с компьютера шлюза. Советы по устранению неполадок, связанных со шлюзом или подключением, см. в разделе [Устранение неполадок в работе шлюза](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).   

### <a name="problem-input-slices-are-in-waiting-state-forever"></a>Проблема. неограниченное состояние входных срезов
Срезы могут находиться в состоянии **ожидания** по разным причинам. Одна из распространенных причин — для свойства **external** не задано значение **true**. Все наборы данных, созданные вне фабрики данных Azure, должны быть помечены свойством **external** . Это свойство указывает на то, что данные являются внешними и не поддерживаются какими-либо конвейерами в фабрике данных. После того как данные станут доступны в соответствующем хранилище, срезы данных помечаются флагом **Ready** (готово).

Пример использования свойства **external** приведен ниже. При необходимости можно указать **externalData***, если параметру External присвоить значение true.

Дополнительные сведения об этом свойстве см. в статье [Наборы данных](data-factory-create-datasets.md).

```json
{
  "name": "CustomerTable",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "MyLinkedService",
    "typeProperties": {
      "folderPath": "MyContainer/MySubFolder/",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      }
    }
  }
}
```

Чтобы устранить эту ошибку, добавьте свойство **external** и дополнительный раздел **externalData** в определение JSON входной таблицы и повторно создайте эту таблицу.

### <a name="problem-hybrid-copy-operation-fails"></a>Проблема: сбой гибридной операции копирования
Действия по устранению неполадок с копированием в локальное хранилище данных и из него с помощью шлюза управления данными см. в статье [Устранение неполадок в работе шлюза](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).

### <a name="problem-on-demand-hdinsight-provisioning-fails"></a>Проблема: сбой подготовки HDInsight по запросу
При использовании связанной службы типа HDInsightOnDemandLinkedService нужно задать имя linkedServiceName, указывающее на хранилище BLOB-объектов Azure. Фабрика данных использует это хранилище для хранения журналов и вспомогательных файлов для кластера HDInsight по запросу.  Иногда подготовка кластера HDInsight по запросу завершается следующей ошибкой:

```
Failed to create cluster. Exception: Unable to complete the cluster create operation. Operation failed with code '400'. Cluster left behind state: 'Error'. Message: 'StorageAccountNotColocated'.
```

Обычно эта ошибка указывает на то, что расположение учетной записи хранения, указанной в linkedServiceName, не совпадает с расположением центра обработки данных, в котором происходит подготовка HDInsight. Например, если ваша фабрика данных Azure находится в западной части США, а хранилище Azure расположено в восточной части США, то подготовка по запросу в западной части США выполнена не будет.

Есть еще одно свойство JSON, additionalLinkedServiceNames, где можно указать дополнительные учетные записи хранения в HDInsight по запросу. Эти дополнительные связанные учетные записи хранения должны находиться в том же расположении, что и кластер HDInsight, иначе происходит сбой с той же ошибкой.

### <a name="problem-custom-net-activity-fails"></a>Проблема: сбой настраиваемого действия .NET
Подробные действия см. в разделе [Отладка конвейера с помощью настраиваемого действия](data-factory-use-custom-activities.md#troubleshoot-failures).

## <a name="use-azure-portal-to-troubleshoot"></a>Устранение неполадок с помощью портала Azure
### <a name="using-portal-blades"></a>Использование колонок на портале
Действия см. в статье [Мониторинг конвейера](data-factory-monitor-manage-pipelines.md).

### <a name="using-monitor-and-manage-app"></a>Использование приложения по мониторингу и управлению
Сведения см. в статье [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью приложения по мониторингу и управлению](data-factory-monitor-manage-app.md).

## <a name="use-azure-powershell-to-troubleshoot"></a>Использование Azure PowerShell для устранения неполадок
### <a name="use-azure-powershell-to-troubleshoot-an-error"></a>Использование Azure PowerShell для устранения ошибок
Сведения см. в разделе [Мониторинг конвейеров фабрики данных с помощью Azure PowerShell](data-factory-monitor-manage-pipelines.md).

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: /previous-versions/azure/dn834987(v=azure.100)
[cmdlet-reference]: https://go.microsoft.com/fwlink/?LinkId=517456
[json-scripting-reference]: /previous-versions/azure/dn835050(v=azure.100)

[azure-portal]: https://portal.azure.com/

[image-data-factory-troubleshoot-with-error-link]: ./media/data-factory-troubleshoot/DataFactoryWithErrorLink.png

[image-data-factory-troubleshoot-datasets-with-errors-blade]: ./media/data-factory-troubleshoot/DatasetsWithErrorsBlade.png

[image-data-factory-troubleshoot-table-blade-with-problem-slices]: ./media/data-factory-troubleshoot/TableBladeWithProblemSlices.png

[image-data-factory-troubleshoot-activity-run-with-error]: ./media/data-factory-troubleshoot/ActivityRunDetailsWithError.png

[image-data-factory-troubleshoot-dataslice-blade-with-active-runs]: ./media/data-factory-troubleshoot/DataSliceBladeWithActivityRuns.png

[image-data-factory-troubleshoot-walkthrough2-with-errors-link]: ./media/data-factory-troubleshoot/Walkthrough2WithErrorsLink.png

[image-data-factory-troubleshoot-walkthrough2-datasets-with-errors]: ./media/data-factory-troubleshoot/Walkthrough2DataSetsWithErrors.png

[image-data-factory-troubleshoot-walkthrough2-table-with-problem-slices]: ./media/data-factory-troubleshoot/Walkthrough2TableProblemSlices.png

[image-data-factory-troubleshoot-walkthrough2-slice-activity-runs]: ./media/data-factory-troubleshoot/Walkthrough2DataSliceActivityRuns.png

[image-data-factory-troubleshoot-activity-run-details]: ./media/data-factory-troubleshoot/Walkthrough2ActivityRunDetails.png