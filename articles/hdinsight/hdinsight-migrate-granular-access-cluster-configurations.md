---
title: Детализация конфигураций кластера Azure HDInsight с доступом на основе ролей
description: Изучите изменения, необходимые в процессе миграции, чтобы детально получить доступ к конфигурациям кластера HDInsight на основе ролей.
author: tylerfox
ms.author: tyfox
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/20/2020
ms.openlocfilehash: a30768f4904c9e5be2edc020f12260cf3a54c889
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102425895"
---
# <a name="migrate-to-granular-role-based-access-for-cluster-configurations"></a>Переход на детализированный доступ на основе ролей для конфигураций кластера

Мы представляем некоторые важные изменения для поддержки более детализированного доступа на основе ролей для получения конфиденциальной информации. В рамках этих изменений некоторые действия могут потребоваться **3 сентября 2019,** если вы используете одну из [затронутых сущностей или сценариев](#am-i-affected-by-these-changes).

## <a name="what-is-changing"></a>Что изменяется?

Ранее секреты можно было получить через API HDInsight, используя для этого пользователи кластера, обладающие [ролями](../role-based-access-control/rbac-and-directory-admin-roles.md)владельца, участника или читателя, так как они были доступны любому пользователю с `*/read` разрешением. Секреты определяются как значения, которые можно использовать для получения более повышения уровня доступа, чем разрешено для роли пользователя. К ним относятся такие значения, как учетные данные HTTP для шлюза кластера, ключи учетной записи хранения и учетные данные базы данных.

Начиная с 3 сентября 2019, доступ к этим секретам потребует `Microsoft.HDInsight/clusters/configurations/action` разрешения, то есть пользователи с ролью читателя больше не могут получить к ним доступ. Роли, имеющие это разрешение, — участник, владелец и новая роль оператора кластера HDInsight (подробнее см. ниже).

Мы также представляем новую роль [оператора кластера HDInsight](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator) , которая сможет получать секреты без предоставления административных разрешений участнику или владельцу. Подведение итогов.

| Роль                                  | Раньше назывался .                                                                                       | Переход вперед       |
|---------------------------------------|--------------------------------------------------------------------------------------------------|-----------|
| Читатель                                | — Доступ на чтение, включая секреты.                                                                   | — Доступ на чтение, **исключение** секретов |           |   |   |
| Оператор кластера HDInsight<br>(Новая роль) | Н/Д                                                                                              | — Доступ на чтение и запись, включая секреты         |   |   |
| Участник                           | — Доступ на чтение и запись, включая секреты.<br>— Создание всех типов ресурсов Azure и управление ими.<br>— Выполнение действий скрипта.     | Без изменения. |
| Владелец                                 | — Доступ на чтение и запись, включая секреты.<br>— Полный доступ ко всем ресурсам<br>— Делегирование доступа другим пользователям.<br>— Выполнение действий скрипта. | Без изменения. |

Сведения о том, как добавить назначение роли оператора кластера HDInsight пользователю, чтобы предоставить им доступ для чтения и записи к секретам кластера, см. в разделе ниже, [добавьте назначение роли "оператор кластера hdinsight" пользователю](#add-the-hdinsight-cluster-operator-role-assignment-to-a-user).

## <a name="am-i-affected-by-these-changes"></a>Я затронул эти изменения?

Затрагиваются следующие сущности и сценарии:

- [API](#api): пользователи, использующие `/configurations` `/configurations/{configurationName}` конечные точки или.
- [Средства Azure HDInsight для Visual Studio Code](#azure-hdinsight-tools-for-visual-studio-code) версии 1.1.1 или ниже.
- [Azure Toolkit for IntelliJ](#azure-toolkit-for-intellij) версии 3.20.0 или ниже.
- [Средства Azure Data Lake и Stream Analytics для Visual Studio](#azure-data-lake-and-stream-analytics-tools-for-visual-studio) ниже версии 2.3.9000.1.
- [Azure Toolkit for Eclipse](#azure-toolkit-for-eclipse) версии 3.15.0 или ниже.
- [Пакет SDK для .NET](#sdk-for-net)
    - [версии 1. x или 2. x](#versions-1x-and-2x): пользователи, использующие `GetClusterConfigurations` `GetConnectivitySettings` методы, `ConfigureHttpSettings` , `EnableHttp` или `DisableHttp` из класса конфигуратионсоператионсекстенсионс.
    - [версии 3. x и выше](#versions-3x-and-up): пользователи, использующие `Get` методы,, `Update` `EnableHttp` или `DisableHttp` из `ConfigurationsOperationsExtensions` класса.
- [Пакет SDK для Python](#sdk-for-python): пользователи, `get` использующие `update` методы или из `ConfigurationsOperations` класса.
- [Пакет SDK для Java](#sdk-for-java): пользователи, `update` использующие `get` методы или из `ConfigurationsInner` класса.
- [Пакет SDK для Go](#sdk-for-go): пользователи, `Get` использующие `Update` методы или из `ConfigurationsClient` структуры.
- [AZ. HDInsight PowerShell](#azhdinsight-powershell) ниже версии 2.0.0.
См. следующие разделы (или используйте приведенные выше ссылки), чтобы просмотреть шаги миграции для вашего сценария.

### <a name="api"></a>API

Следующие API будут изменены или устарели:

- [**Get/конфигуратионс/{конфигуратионнаме}**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (удаление конфиденциальной информации)
    - Ранее использовался для получения индивидуальных типов конфигурации (включая секреты).
    - Начиная с 3 сентября 2019, этот вызов API теперь будет возвращать отдельные типы конфигурации с опущенными секретами. Чтобы получить все конфигурации, включая секреты, используйте новый вызов POST/конфигуратионс. Чтобы получить только параметры шлюза, используйте новый вызов POST/Жетгатевайсеттингс.
- [**Get/конфигуратионс**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (не рекомендуется)
    - Ранее использовалось для получения всех конфигураций (включая секреты)
    - Начиная с 3 сентября 2019, этот вызов API будет устаревшим и больше не будет поддерживаться. Чтобы получить все конфигурации, пересылаемые вперед, используйте новый вызов POST/конфигуратионс. Чтобы получить конфигурации с опущенными конфиденциальными параметрами, используйте вызов GET/Конфигуратионс/{конфигуратионнаме}.
- [**POST/конфигуратионс/{конфигуратионнаме}**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings) (не рекомендуется)
    - Ранее использовалось для обновления учетных данных шлюза.
    - Начиная с 3 сентября 2019, этот вызов API будет устаревшим и более не поддерживаться. Вместо этого используйте новый POST/Упдатегатевайсеттингс.

Добавлены следующие API-интерфейсы замены:</span>

- [**ОПУБЛИКОВАТЬ/конфигуратионс**](/rest/api/hdinsight/hdinsight-cluster#list-configurations)
    - Используйте этот API для получения всех конфигураций, включая секреты.
- [**ОПУБЛИКОВАТЬ/Жетгатевайсеттингс**](/rest/api/hdinsight/hdinsight-cluster#get-gateway-settings)
    - Используйте этот API для получения параметров шлюза.
- [**ОПУБЛИКОВАТЬ/Упдатегатевайсеттингс**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings)
    - Используйте этот API для обновления параметров шлюза (имя пользователя и/или пароль).

### <a name="azure-hdinsight-tools-for-visual-studio-code"></a>Средства Azure HDInsight для Visual Studio Code

Если вы используете версию 1.1.1 или ниже, обновите до [последней версии средств Azure HDInsight для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=mshdinsight.azure-hdinsight&ssr=false) , чтобы избежать перерывов.

### <a name="azure-toolkit-for-intellij"></a>Набор средств Azure для IntelliJ

Если вы используете версию 3.20.0 или ниже, обновите [последнюю версию подключаемого модуля Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij) , чтобы избежать перерывов.

### <a name="azure-data-lake-and-stream-analytics-tools-for-visual-studio"></a>Средства Azure Data Lake и Stream Analytics для Visual Studio

Обновите [Azure Data Lake и средства Stream Analytics для Visual Studio](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools&ssr=false#overview) до версии 2.3.9000.1 или более поздней, чтобы избежать перерывов в работе.  Справку по обновлению см. в нашей документации, а также в [статье обновление средств Data Lake для Visual Studio](./hadoop/apache-hadoop-visual-studio-tools-get-started.md#update-data-lake-tools-for-visual-studio).

### <a name="azure-toolkit-for-eclipse"></a>Набор средств Azure для Eclipse

Если вы используете версию 3.15.0 или ниже, обновите ее до [последней версии Azure Toolkit for Eclipse](https://marketplace.eclipse.org/content/azure-toolkit-eclipse) во избежание перерывов.

### <a name="sdk-for-net"></a>Пакет SDK для .NET

#### <a name="versions-1x-and-2x"></a>Версии 1. x и 2. x

Обновите [версию 2.1.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/2.1.0) SDK HDInsight для .NET. При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде:

- `ClusterOperationsExtensions.GetClusterConfigurations`**больше не будет возвращать конфиденциальные параметры** , такие как ключи хранилища (основной сайт) или учетные данные HTTP (шлюз).
    - Для получения всех конфигураций, включая конфиденциальные параметры, используйте `ClusterOperationsExtensions.ListConfigurations` Переход вперед.  Обратите внимание, что пользователи с ролью "читатель" не смогут использовать этот метод. Это позволяет детально контролировать, какие пользователи могут получать доступ к конфиденциальной информации для кластера.
    - Чтобы получить только учетные данные шлюза HTTP, используйте `ClusterOperationsExtensions.GetGatewaySettings` .

- `ClusterOperationsExtensions.GetConnectivitySettings` теперь является нерекомендуемым и заменено на `ClusterOperationsExtensions.GetGatewaySettings` .

- `ClusterOperationsExtensions.ConfigureHttpSettings` теперь является нерекомендуемым и заменено на `ClusterOperationsExtensions.UpdateGatewaySettings` .

- `ConfigurationsOperationsExtensions.EnableHttp` и `DisableHttp` теперь являются устаревшими. Протокол HTTP теперь всегда включен, поэтому эти методы больше не нужны.

#### <a name="versions-3x-and-up"></a>Версии 3. x и выше

Обновите пакет SDK HDInsight для .NET до [версии 5.0.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/5.0.0) или более поздней. При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде:

- [`ConfigurationOperationsExtensions.Get`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.get)**больше не будет возвращать конфиденциальные параметры** , такие как ключи хранилища (основной сайт) или учетные данные HTTP (шлюз).
    - Для получения всех конфигураций, включая конфиденциальные параметры, используйте [`ConfigurationOperationsExtensions.List`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.list) Переход вперед.Обратите внимание, что пользователи с ролью "читатель" не смогут использовать этот метод. Это позволяет детально контролировать, какие пользователи могут получать доступ к конфиденциальной информации для кластера. 
    - Чтобы получить только учетные данные шлюза HTTP, используйте [`ClusterOperationsExtensions.GetGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.getgatewaysettings) . 
- [`ConfigurationsOperationsExtensions.Update`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.update) теперь является нерекомендуемым и заменено на [`ClusterOperationsExtensions.UpdateGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.updategatewaysettings) . 
- [`ConfigurationsOperationsExtensions.EnableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.enablehttp) и [`DisableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.disablehttp) теперь являются устаревшими. Протокол HTTP теперь всегда включен, поэтому эти методы больше не нужны.

### <a name="sdk-for-python"></a>Пакет SDK для Python

Обновление до [версии 1.0.0](https://pypi.org/project/azure-mgmt-hdinsight/1.0.0/) или более поздней версии пакета SDK для HDInsight для Python. При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде:

- [`ConfigurationsOperations.get`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#get-resource-group-name--cluster-name--configuration-name--custom-headers-none--raw-false----operation-config-)**больше не будет возвращать конфиденциальные параметры** , такие как ключи хранилища (основной сайт) или учетные данные HTTP (шлюз).
    - Для получения всех конфигураций, включая конфиденциальные параметры, используйте [`ConfigurationsOperations.list`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#list-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-) Переход вперед.Обратите внимание, что пользователи с ролью "читатель" не смогут использовать этот метод. Это позволяет детально контролировать, какие пользователи могут получать доступ к конфиденциальной информации для кластера. 
    - Чтобы получить только учетные данные шлюза HTTP, используйте [`ClusterOperations.get_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#get-gateway-settings-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-) .
- [`ConfigurationsOperations.update`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#update-resource-group-name--cluster-name--configuration-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-) теперь является нерекомендуемым и заменено на [`ClusterOperations.update_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#update-gateway-settings-resource-group-name--cluster-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-) .

### <a name="sdk-for-java"></a>Пакет SDK для Java

Обновление до [версии 1.0.0](https://search.maven.org/artifact/com.microsoft.azure.hdinsight.v2018_06_01_preview/azure-mgmt-hdinsight/1.0.0/jar) или более поздней версии пакета SDK для HDInsight для Java. При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде:

- `ConfigurationsInner.get`**больше не будет возвращать конфиденциальные параметры** , такие как ключи хранилища (основной сайт) или учетные данные HTTP (шлюз).
- `ConfigurationsInner.update` теперь является устаревшим.

### <a name="sdk-for-go"></a>Пакет SDK для Go

Обновление до [версии 27.1.0](https://github.com/Azure/azure-sdk-for-go/tree/master/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight) или более поздней версии пакета SDK HDInsight для Go. При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде:

- [`ConfigurationsClient.get`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Get)**больше не будет возвращать конфиденциальные параметры** , такие как ключи хранилища (основной сайт) или учетные данные HTTP (шлюз).
    - Для получения всех конфигураций, включая конфиденциальные параметры, используйте [`ConfigurationsClient.list`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.List) Переход вперед.Обратите внимание, что пользователи с ролью "читатель" не смогут использовать этот метод. Это позволяет детально контролировать, какие пользователи могут получать доступ к конфиденциальной информации для кластера. 
    - Чтобы получить только учетные данные шлюза HTTP, используйте [`ClustersClient.get_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.GetGatewaySettings) .
- [`ConfigurationsClient.update`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Update) теперь является нерекомендуемым и заменено на [`ClustersClient.update_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.UpdateGatewaySettings) .

### <a name="azhdinsight-powershell"></a>AZ. HDInsight PowerShell
Обновите команду [AZ PowerShell версии 2.0.0](https://www.powershellgallery.com/packages/Az) или более поздней, чтобы избежать перерывов.  При использовании метода, затрагиваемого этими изменениями, могут потребоваться минимальные изменения в коде.
- `Grant-AzHDInsightHttpServicesAccess` теперь является нерекомендуемым и заменено новым `Set-AzHDInsightGatewayCredential` командлетом.
- `Get-AzHDInsightJobOutput` был обновлен для поддержки детализированного доступа к ключу хранилища на основе ролей.
    - На пользователей с ролями оператора, участника или владельца кластера HDInsight это не повлияет.
    - Пользователям с ролью читателя потребуется `DefaultStorageAccountKey` явно указать параметр.
- `Revoke-AzHDInsightHttpServicesAccess` теперь является устаревшим. Протокол HTTP теперь всегда включен, поэтому этот командлет больше не нужен.
 См [. раздел AZ. ](https://github.com/Azure/azure-powershell/blob/master/documentation/migration-guides/Az.2.0.0-migration-guide.md#azhdinsight) Дополнительные сведения см. в разделе с руководством по миграции HDInsight.

## <a name="add-the-hdinsight-cluster-operator-role-assignment-to-a-user"></a>Добавление назначения роли оператора кластера HDInsight пользователю

Пользователь с ролью [владельца](../role-based-access-control/built-in-roles.md#owner) может назначить роль [оператора кластера hdinsight](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator) пользователям, которым требуется доступ на чтение и запись к конфиденциальным значениям конфигурации кластера hdinsight (например, учетные данные шлюза кластера и ключи учетной записи хранения).

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Самый простой способ добавить это назначение роли — использовать `az role assignment create` команду в Azure CLI.

> [!NOTE]
> Эта команда должна выполняться пользователем с ролью владельца, так как только они могут предоставлять эти разрешения. `--assignee`— Это имя субъекта-службы или адрес электронной почты пользователя, которому необходимо назначить роль оператора кластера HDInsight. Если возникает ошибка "недостаточно разрешений", см. раздел часто задаваемые вопросы ниже.

#### <a name="grant-role-at-the-resource-cluster-level"></a>Предоставление роли на уровне ресурсов (кластера)

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee <user@domain.com> --scope /subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HDInsight/clusters/<ClusterName>
```

#### <a name="grant-role-at-the-resource-group-level"></a>Предоставление роли на уровне группы ресурсов

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com -g <ResourceGroupName>
```

#### <a name="grant-role-at-the-subscription-level"></a>Предоставление роли на уровне подписки

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com
```

### <a name="using-the-azure-portal"></a>Использование портала Azure

Можно также использовать портал Azure для добавления назначения роли оператора кластера HDInsight пользователю. См. документацию по [назначению ролей Azure с помощью портал Azure](../role-based-access-control/role-assignments-portal.md).

## <a name="faq"></a>Вопросы и ответы

### <a name="why-am-i-seeing-a-403-forbidden-response-after-updating-my-api-requests-andor-tool"></a>Почему после обновления запросов API и (или) средства отображается ответ 403 Forbidden (запрещено)?

Конфигурации кластера теперь находятся за детальным управлением доступом на основе ролей и должны иметь `Microsoft.HDInsight/clusters/configurations/*` разрешение на доступ к ним. Чтобы получить это разрешение, назначьте пользователю или субъекту-службе, пытающимся получить доступ к конфигурациям, роль "оператор кластера HDInsight", "участник" или "владелец".

### <a name="why-do-i-see-insufficient-privileges-to-complete-the-operation-when-running-the-azure-cli-command-to-assign-the-hdinsight-cluster-operator-role-to-another-user-or-service-principal"></a>Почему я вижу «недостаточно прав для выполнения операции» при выполнении команды Azure CLI для назначения роли оператора кластера HDInsight другому пользователю или субъекту-службе?

Кроме роли владельца, пользователь или субъект-служба, которые выполняют команду, должны иметь достаточные разрешения Azure AD для поиска идентификаторов объектов уполномоченного. Это сообщение указывает на недостаточные разрешения Azure AD. Попробуйте заменить `-–assignee` аргумент на `–assignee-object-id` и укажите идентификатор объекта уполномоченного в качестве параметра вместо имени (или идентификатора участника в случае управляемого удостоверения). Дополнительные сведения см. в разделе необязательные параметры раздела [AZ Role назначений](/cli/azure/role/assignment#az-role-assignment-create) .

Если это не поможет, обратитесь к администратору Azure AD, чтобы получить правильные разрешения.

### <a name="what-will-happen-if-i-take-no-action"></a>Что произойдет, если не предпринимать никаких действий?

Начиная с 3 сентября 2019 г `GET /configurations` `POST /configurations/gateway` . вызовы больше не будут возвращать никаких сведений, и `GET /configurations/{configurationName}` вызов больше не будет возвращать конфиденциальные параметры, такие как ключи учетной записи хранения или пароль кластера. То же самое относится к соответствующим методам SDK и командлетам PowerShell.

Если вы используете более раннюю версию одного из средств для Visual Studio, VSCode, IntelliJ или Eclipse, упомянутых выше, они больше не будут работать до обновления.

Более подробные сведения см. в соответствующем разделе этого документа для вашего сценария.