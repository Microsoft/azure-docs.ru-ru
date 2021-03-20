---
title: Интерфейсы API и инструменты для разработчиков
description: Сведения об API-интерфейсах и средствах, доступных для разработки решений с помощью пакетной службы Azure.
ms.topic: conceptual
ms.date: 05/22/2020
ms.custom: seodec18
ms.openlocfilehash: 502eb08631223215933b75dca882c12c02d17bd9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89146408"
---
# <a name="overview-of-batch-apis-and-tools"></a>Общие сведения об API-интерфейсах и средствах пакетной службы

Обычно обработка параллельных рабочих нагрузок, использующих пакетную службу Azure, выполняется программным способом с помощью API-интерфейсов пакетной службы. Ваше клиентское приложение или служба могут использовать API-интерфейсы пакетной службы для взаимодействия с пакетной службой. С помощью API-интерфейсов пакетной службы можно создавать пулы вычислительных узлов виртуальные машины или облачные службы, а также управлять этими ресурсами. Вы можете запланировать выполнение заданий и задач на этих узлах.

Вы можете эффективно обрабатывать крупномасштабные рабочие нагрузки в своей организации или предоставлять внешние интерфейсы служб клиентам, чтобы они могли выполнять задания и задачи (по требованию или по расписанию) на одном, сотнях или тысячах узлов. Кроме того, пакетную службу Azure можно использовать как часть более крупного рабочего процесса под управлением таких средств, как [фабрика данных Azure](../data-factory/transform-data-using-dotnet-custom-activity.md?toc=%2fazure%2fbatch%2ftoc.json).

> [!TIP]
> Дополнительные сведения о функциях и рабочих процессах, используемых в пакетной службе Azure, см. в статье [Рабочий процесс и ресурсы пакетной службы](batch-service-workflow-features.md).

## <a name="azure-accounts-for-batch-development"></a>Учетные записи Azure для разработки с помощью пакетной службы

При разработке решений с использованием пакетной службы требуются следующие учетные записи в вашей подписке Azure:

- **Учетная запись пакетной службы** — это ресурсы пакетной службы Azure, в том числе пулы, вычислительные узлы, задания и задачи, связанные с [учетной записью пакетной службы](accounts.md) Azure. Когда приложение отправляет запрос к пакетной службе, выполняется проверка подлинности запроса с использованием имени учетной записи пакетной службы Azure, URL-адреса учетной записи и ключа доступа или токена Azure Active Directory. Вы можете [создать учетную запись пакетной службы](batch-account-create-portal.md) на портале Azure или программным способом.
- **Учетная запись хранения.** В пакетную службу встроена поддержка работы с файлами в [службе хранилища Azure](../storage/index.yml). При работе с пакетной службой хранилище BLOB-объектов Azure используется преимущественно не только для промежуточного хранения файлов программ и данных (запускаются и обрабатываются задачами соответственно), но и для хранения выходных данных (результаты выполнения задач). Каждая учетная запись пакетной службы обычно связана с соответствующей учетной записью хранения.

## <a name="service-level-and-management-level-apis"></a>Интерфейсы API уровня обслуживания и управления

Пакетная служба Azure имеет два набора API: один для уровня обслуживания и один для уровня управления. Их названия часто похожи, но они возвращают разные результаты.

В журнале действий будут записываться только действия из API управления. API уровня обслуживания обходят уровень управления ресурсами Azure (management.azure.com) и не регистрируются в журнале.

Например, [API уровня обслуживания для удаления пула](/rest/api/batchservice/pool/delete) работает непосредственно с учетной записью пакетной службы: `DELETE {batchUrl}/pools/{poolId}`

В отличие от вышеуказанного программного интерфейса, [API управления пакетной службы для удаления пула](/rest/api/batchmanagement/pool/delete) работает со слоем management.azure.com: `DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Batch/batchAccounts/{accountName}/pools/{poolName}`

## <a name="batch-service-apis"></a>API-интерфейсы пакетной службы

Приложения и службы могут напрямую вызывать REST API или использовать следующие клиентские библиотеки для выполнения рабочих нагрузок пакетной службы Azure и управления ими.

| API | Справочник по API | Скачивание | Учебник | Примеры кода | Дополнительные сведения |
| --- | --- | --- | --- | --- | --- |
| **Пакетная служба (REST)** |[REST API Azure — документация](/rest/api/batchservice/) |Недоступно |- |- | [Поддерживаемые версии](/rest/api/batchservice/batch-service-rest-api-versioning) |
| **Пакетная служба (.NET)** |[Пакет Azure SDK для .NET — документация](/dotnet/api/overview/azure/batch) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Batch/) |[Руководство](tutorial-parallel-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) | [Заметки о выпуске](https://aka.ms/batch-net-dataplane-changelog) |
| **Пакетная служба Python** |[Пакет Azure SDK для Python — документация](/python/api/overview/azure/batch/client) |[PyPI](https://pypi.org/project/azure-batch/) |[Руководство](tutorial-parallel-python.md)|[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Python/Batch) | [Файл сведений](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/batch/azure-batch/README.md) |
| **Node.js для пакетной службы** |[Пакет Azure SDK для JavaScript — документация](/javascript/api/overview/azure/batch/client) |[npm](https://www.npmjs.com/package/azure-batch) |[Руководство](batch-nodejs-get-started.md) |- | [Файл сведений](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/batch) |
| **Java для пакетной службы** |[Пакет Azure SDK для Java — документация](/java/api/overview/azure/batch) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Java) | [Файл сведений](https://github.com/Azure/azure-batch-sdk-for-java)|

## <a name="batch-management-apis"></a>API-интерфейсы для управления пакетной службой

API-интерфейсы Azure Resource Manager для пакетной службы предоставляют программный доступ к учетным записям пакетной службы. Применяя эти API, можно программно управлять учетными записями пакетной службы, квотами, пакетами приложений и другими ресурсами через поставщик Microsoft.Batch.  

| API | Справочник по API | Скачивание | Учебник | Примеры кода |
| --- | --- | --- | --- | --- |
| **REST для управления пакетной службой** |[REST API Azure — документация](/rest/api/batchmanagement/) |- |- |[GitHub](https://github.com/Azure-Samples/batch-dotnet-manage-batch-accounts) |
| **Библиотека .NET для управления пакетной службой** |[Пакет Azure SDK для .NET — документация](/dotnet/api/overview/azure/batch/management) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/) | [Руководство](batch-management-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) |
| **Python для управления пакетной службой** |[Пакет Azure SDK для Python — документация](/python/api/overview/azure/batch/management) |[PyPI](https://pypi.org/project/azure-mgmt-batch/) |- |- |
| **Node.js для управления пакетной службой** |[Пакет Azure SDK для JavaScript — документация](/javascript/api/overview/azure/batch/management) |[npm](https://www.npmjs.com/package/azure-arm-batch) |- |- | 
| **Java для управления пакетной службой** |[Пакет Azure SDK для Java — документация](/java/api/overview/azure/batch/management) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |- |

## <a name="batch-command-line-tools"></a>Программы командной строки пакетной службы

Эти программы командной строки обеспечивают ту же функциональность, что и API-интерфейсы пакетной службы и службы управления пакетной службой. 

- [Командлеты PowerShell для пакетной службы.](/powershell/module/az.batch/) Командлеты пакетной службы Azure в модуле [Azure PowerShell](/powershell/azure/) позволяют управлять ресурсами пакетной службы с помощью PowerShell.
- [Azure CLI.](/cli/azure) Это кроссплатформенный набор средств, который обеспечивает взаимодействие с разными службами Azure, включая пакетную службу и службу управления пакетной службой, с помощью команд оболочки. Дополнительные сведения об использовании Azure CLI с пакетной службой см. в статье [Управление ресурсами пакетной службы с помощью Azure CLI](batch-cli-get-started.md).

## <a name="other-tools-for-application-development"></a>Другие средства для разработки приложений

Эти дополнительные средства можно использовать для создания и отладки приложений и служб пакетной службы.

- [Портал Azure](https://portal.azure.com/): На портале Azure можно создавать, отслеживать и удалять пулы, задания и задачи пакетной службы. Во время выполнения заданий можно просмотреть сведения о состоянии этих и других ресурсов, а также скачать файлы из вычислительных узлов в пулах. Например, при устранении неполадок можно скачать файл `stderr.txt` задачи, завершившейся сбоем. Кроме того, можно скачать файлы удаленного рабочего стола, которые можно использовать для входа на вычислительные узлы.
- [Azure Batch Explorer.](https://azure.github.io/BatchExplorer/) Batch Explorer (ранее BatchLabs) — это бесплатное автономное клиентское средство с множеством функций для создания, отладки и мониторинга приложений в пакетной службе Azure. Скачайте [пакет установки](https://azure.github.io/BatchExplorer/) для Mac, Linux или Windows.
- [Azure Batch Shipyard](https://github.com/Azure/batch-shipyard). Batch Shipyard — это средство, способствующее подготовке, выполнению и отслеживанию контейнерной пакетной обработки и высокопроизводительных рабочих нагрузок в пакетной службе Azure.
- [Обозреватель службы хранилища.](https://azure.microsoft.com/features/storage-explorer/) Строго говоря, этот обозреватель не является средством пакетной службы Azure, но это полезный инструмент для разработки и отладки соответствующих решений.

## <a name="additional-resources"></a>Дополнительные ресурсы

- Дополнительные сведения о регистрации событий приложения пакетной службы см. в статье [Метрики, оповещения и журналы пакетной службы для диагностики и мониторинга](batch-diagnostics.md).
- Справочные сведения о событиях, которые происходят в пакетной службе, см. в статье [Пакетная аналитика](batch-analytics.md).
- Сведения о переменных среды для вычислительных узлов см. в статье [Переменные среды выполнения пакетной службы Azure](batch-compute-node-environment-variables.md).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее о [рабочем процессе и основных ресурсах пакетной службы](batch-service-workflow-features.md), таких как пулы, узлы, задания и задачи.
- Сведения об использовании C# и библиотеки .NET для пакетной службы при обработке простой рабочей нагрузки с помощью стандартного рабочего процесса пакетной службы см. в статье [Начало работы с библиотекой пакетной службы Azure для .NET](tutorial-parallel-dotnet.md). Кроме того, доступны [версия Python](tutorial-parallel-python.md) и [руководство Node.js](batch-nodejs-get-started.md).
- Скачайте [примеры кода с GitHub](https://github.com/Azure-Samples/azure-batch-samples), чтобы узнать, как C# и Python взаимодействуют с пакетной службой для планирования и обработки примеров рабочих нагрузок.
