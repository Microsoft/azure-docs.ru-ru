---
title: Устранение проблем подключения агента серверов с поддержкой ARC в Azure
description: В этой статье рассказывается, как устранять неполадки и устранять проблемы с агентом подключенных компьютеров, которые возникают на серверах с поддержкой ARC в Azure при попытке подключения к службе.
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 36feb6a65ec52d99dfd664ae54cb099ea6a7e239
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90900674"
---
# <a name="troubleshoot-the-connected-machine-agent-connection-issues"></a>Устранение проблем с подключением агента подключенных компьютеров

Эта статья содержит сведения об устранении неполадок и способах устранения проблем, которые могут возникнуть при попытке настроить агент подключенного компьютера с поддержкой ARC в Azure для Windows или Linux. При настройке подключения к службе включаются как интерактивные, так и масштабируемые методы установки. Общие сведения см. в статье [Обзор серверов с поддержкой Arc](./overview.md).

## <a name="agent-verbose-log"></a>Подробный журнал агента

Прежде чем выполнять действия по устранению неполадок, описанные ниже в этой статье, необходимо получить подробный журнал. Он содержит выходные данные команд инструмента **азкмажент** при использовании аргумента Verbose (-v). Файлы журнала записываются в `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` для Windows, а Linux — в `/var/opt/azcmagent/log/azcmagent.log` .

### <a name="windows"></a>Windows

Ниже приведен пример команды для включения подробного ведения журнала с помощью агента подключенного компьютера для Windows при выполнении интерактивной установки.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

Ниже приведен пример команды для включения подробного ведения журнала с помощью агента подключенного компьютера для Windows при выполнении масштабируемой установки с использованием субъекта-службы.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
  --verbose
```

### <a name="linux"></a>Linux

Ниже приведен пример команды для включения подробного ведения журнала с помощью агента подключенного компьютера для Linux при выполнении интерактивной установки.

>[!NOTE]
>Для запуска **азкмажент** требуются права доступа *root* на компьютерах Linux.

```bash
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

Ниже приведен пример команды для включения подробного ведения журнала с помощью агента подключенного компьютера для Linux при выполнении масштабируемой установки с использованием субъекта-службы.

```bash
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
  --verbose
```

## <a name="agent-connection-issues-to-service"></a>Проблемы с подключением агента к службе

В следующей таблице перечислены некоторые известные ошибки и рекомендации по их устранению.

|Сообщение |Ошибка |Возможные причины |Решение |
|--------|------|---------------|---------|
|Не удалось получить поток устройства маркера авторизации |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` |Не удается получить доступ к `login.windows.net` конечной точке | Проверьте подключение к конечной точке. |
|Не удалось получить поток устройства маркера авторизации |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. |Прокси-сервер или брандмауэр блокирует доступ к `login.windows.net` конечной точке. | Проверьте подключение к конечной точке, которое не блокируется брандмауэром или прокси-сервером. |
|Не удалось получить поток устройства маркера авторизации  |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp lookup login.windows.net: no such host`. | Групповая политика *Конфигурация компьютера объекта \ административные шаблоны \ система \ пользователи \ удаление профилей пользователей старше указанного числа дней при перезагрузке системы* включена. | Убедитесь, что объект групповой политики включен и нацелен на затронутый компьютер. Дополнительные сведения см. в сноске <sup>[1](#footnote1)</sup> . |
|Не удалось получить маркер авторизации из имени субъекта-службы |`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` |Прокси-сервер или брандмауэр блокирует доступ к `login.windows.net` конечной точке. |Проверьте подключение к конечной точке, которое не блокируется брандмауэром или прокси-сервером. |
|Не удалось получить маркер авторизации из имени субъекта-службы |`Invalid client secret is provided` |Неправильный или недопустимый секрет субъекта-службы. |Проверьте секрет субъекта-службы. |
| Не удалось получить маркер авторизации из имени субъекта-службы |`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' was not found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` |Неверный субъект-служба или идентификатор клиента. |Проверьте субъект-службу и/или идентификатор клиента.|
|Получение ответа на ресурс ARM |`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` |Неверные учетные данные и разрешения |Убедитесь, что вы или субъект-служба является членом роли **адаптации компьютера, подключенного к Azure** . |
|Не удалось Азкмажентконнект ресурс ARM |`The subscription is not registered to use namespace 'Microsoft.HybridCompute'` |Поставщики ресурсов Azure не зарегистрированы. |Зарегистрируйте [поставщиков ресурсов](./agent-overview.md#register-azure-resource-providers). |
|Не удалось Азкмажентконнект ресурс ARM |`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` |Прокси-сервер или брандмауэр блокирует доступ к `management.azure.com` конечной точке. |Проверьте подключение к конечной точке, которое не блокируется брандмауэром или прокси-сервером. |

<a name="footnote1"></a><sup>1</sup> Если этот объект групповой политики включен и применяется к компьютерам с агентом подключенного компьютера, он удаляет профиль пользователя, связанный со встроенной учетной записью, указанной для службы *химдс* . В результате он также удаляет сертификат проверки подлинности, используемый для взаимодействия со службой, которая кэшируется в локальном хранилище сертификатов, в течение 30 дней. До 30-дневного ограничения предпринимается попытка продлить сертификат. Чтобы устранить эту проблему, выполните действия по [отмене регистрации компьютера](manage-agent.md#unregister-machine) , а затем повторно зарегистрируйте его в запущенной службе `azcmagent connect` .

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли здесь свою проблему или рекомендации не позволили ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы от экспертов Azure через [Microsoft Q&а](/answers/topics/azure-arc.html).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Служба поддержки Azure взаимодействует с сообществом Azure, предоставляя ответы, поддержку и советы экспертов.

* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.
