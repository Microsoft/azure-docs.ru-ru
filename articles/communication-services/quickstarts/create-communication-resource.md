---
title: Краткое руководство. Создание ресурсов Служб коммуникации Azure и управление ими
titleSuffix: An Azure Communication Services quickstart
description: Из этого краткого руководства вы узнаете, как создавать ресурс Служб коммуникации Azure и управлять им.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
zone_pivot_groups: acs-plat-azp-azcli-net-ps
ms.openlocfilehash: aabb8bdf4105702aa623c45bc291770b05b8279e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726777"
---
# <a name="quickstart-create-and-manage-communication-services-resources"></a>Краткое руководство. Создание ресурсов Служб коммуникации Azure и управление ими

Приступите к работе со Службами коммуникации Azure, подготовив первый ресурс Служб коммуникации Azure. Ресурсы служб коммуникации можно подготовить к работе с помощью [портала Microsoft Azure](https://portal.azure.com) или с помощью клиентской библиотеки управления .NET. Клиентская библиотека управления и портал Microsoft Azure позволяют создавать, настраивать, обновлять и удалять ресурсы и интерфейсы с помощью [Azure Resource Manager](../../azure-resource-manager/management/overview.md), службы развертывания и управления Azure. Все функциональные возможности, доступные в пакетах SDK, доступны также на портале Azure. 


> [!WARNING]
> Обратите внимание: службы коммуникации Azure доступны в нескольких географических регионах, однако для получения номера телефона ресурс должен иметь расположение данных со значением "US". Следует также отметить, что в общедоступной предварительной версии вы не сможете передать коммуникационные ресурсы в другую подписку.

::: zone pivot="platform-azp"
[!INCLUDE [Azure portal](./includes/create-resource-azp.md)]
::: zone-end

::: zone pivot="platform-azcli"
[!INCLUDE [Azure CLI](./includes/create-resource-azcli.md)]
::: zone-end

::: zone pivot="platform-net"
[!INCLUDE [.NET](./includes/create-resource-net.md)]
::: zone-end

::: zone pivot="platform-powershell"
[!INCLUDE [PowerShell](./includes/create-resource-powershell.md)]
::: zone-end


## <a name="access-your-connection-strings-and-service-endpoints"></a>Доступ к строкам подключения и конечным точкам службы

Строки подключения позволяют пакетам SDK Служб коммуникации Azure подключаться к Azure и проходить проверку подлинности. Вы можете получить доступ к строкам подключения Служб коммуникации Azure и конечным точкам службы с портала Azure или же программным способом с помощью API Azure Resource Manager.

Перейдя к ресурсу Служб коммуникации, выберите **Ключи** в меню навигации и скопируйте значения **строки подключения** или **конечной точки**, чтобы использовать их в пакетах SDK Служб коммуникации. Обратите внимание, что у вас есть доступ к первичным и вторичным ключам. Это может быть полезно, если вы хотите предоставить временный доступ к ресурсам Служб коммуникации для сторонней или промежуточной среды.

:::image type="content" source="./media/key.png" alt-text="Снимок экрана со страницей &quot;Ключ&quot; Служб коммуникации.":::

Вы также можете получить доступ к основным сведениям, например информации о группе ресурсов или ключах для определенного ресурса, с помощью Azure CLI. 

Установите [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli) и используйте следующую команду, чтобы войти. Вам потребуется предоставить учетные данные для подключения к учетной записи Azure.
```azurecli
az login
```

Теперь у вас есть доступ к главной информации о ресурсах.
```azurecli
az communication list --resource-group "<resourceGroup>"

az communication list-key --name "<communicationName>" --resource-group "<resourceGroup>"
```

Если вы хотите выбрать конкретную подписку, можно также указать флаг ```--subscription``` и указать идентификатор подписки.
```
az communication list --resource-group  "resourceGroup>"  --subscription "<subscriptionID>"

az communication list-key --name "<communicationName>" --resource-group "resourceGroup>" --subscription "<subscriptionID>"
```

## <a name="store-your-connection-string"></a>Сохранение строки подключения

Пакеты SDK Служб коммуникации используют строки подключения для авторизации запросов, выполняемых в этих службах. Есть несколько вариантов для хранения строки подключения.

* Приложение, работающее на настольном компьютере или на устройстве, может хранить строку подключения в файле **app.config** или **web.config**. Добавьте строку подключения в раздел **AppSettings** в этих файлах.
* Приложение, работающее в Службе приложений Azure, может хранить строку подключения в [параметрах приложения Службы приложений Azure](../../app-service/configure-common.md). На портале добавьте строку подключения в раздел **Строки подключения** на вкладке "Параметры приложения".
* Строку подключения можно хранить в [Azure Key Vault](../../data-factory/store-credentials-in-key-vault.md).
* Если приложение выполняется локально, может потребоваться сохранить строку подключения в переменной среды.

### <a name="store-your-connection-string-in-an-environment-variable"></a>Сохранение строки подключения в переменной среды

Чтобы настроить переменную среды, откройте окно консоли и выберите операционную систему на следующих вкладках. Замените `<yourconnectionstring>` фактической строкой подключения.

#### <a name="windows"></a>[Windows](#tab/windows)

Откройте окно консоли и введите следующую команду:

```console
setx COMMUNICATION_SERVICES_CONNECTION_STRING "<yourconnectionstring>"
```

После добавления переменной среды может потребоваться перезапустить все запущенные приложения, которым может понадобиться считать переменную среды, в том числе окно консоли. Например, если вы используете Visual Studio в качестве редактора, перезапустите Visual Studio перед запуском примера.

#### <a name="macos"></a>[macOS](#tab/unix)

Измените **ZSHRC**-файл и добавьте переменную среды:

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

После добавления переменной среды запустите `source ~/.zshrc` из окна консоли, чтобы применить изменения. Если вы создали переменную среды с открытой интегрированной средой разработки, для доступа к переменной может потребоваться закрыть и повторно открыть редактор, интегрированную среду разработки или оболочку.

#### <a name="linux"></a>[Linux](#tab/linux)

Измените **.bash_profile** и добавьте переменную среды.

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

После добавления переменной среды запустите `source ~/.bash_profile` из окна консоли, чтобы применить изменения. Если вы создали переменную среды с открытой интегрированной средой разработки, для доступа к переменной может потребоваться закрыть и повторно открыть редактор, интегрированную среду разработки или оболочку.

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

Если ресурсу назначены номера телефонов, при удалении ресурса эти номера телефонов также будут автоматически удалены.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * создать ресурс Служб коммуникации;
> * настроить географию и теги ресурсов;
> * получить доступ к ключам для этого ресурса.
> * Удаление ресурса

> [!div class="nextstepaction"]
> [Создание первых маркеров доступа пользователя](access-tokens.md)
