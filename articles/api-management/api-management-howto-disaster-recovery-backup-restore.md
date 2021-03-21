---
title: Реализация аварийного восстановления с помощью резервного копирования и восстановления в управлении API
titleSuffix: Azure API Management
description: Использование архивации и восстановления для выполнения аварийного восстановления в службе управления API Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/05/2020
ms.author: apimpm
ms.openlocfilehash: 223d119786d99eac611ece597fc0e8de4fcaf6bd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98762407"
---
# <a name="how-to-implement-disaster-recovery-using-service-backup-and-restore-in-azure-api-management"></a>Реализация аварийного восстановления с помощью функций резервного копирования и восстановления службы в Azure API Management

Опубликовав интерфейсы API и управляя ими с помощью службы управления API Azure, вы получаете возможности по обеспечению отказоустойчивости и организации инфраструктуры, которые в противном случае вам пришлось бы проектировать и внедрять вручную. Платформа Azure устраняет большую часть потенциальных сбоев при относительно небольших затратах.

Чтобы обеспечить восстановление в случае проблем с доступностью в регионе, в котором размещена ваша служба управления API, будьте готовы в любой момент создать ее в другом регионе. В зависимости от цели времени восстановления может потребоваться сохранение резервной службы в одном или нескольких регионах. Вы также можете попытаться сохранить конфигурацию и содержимое в синхронизации с активной службой в соответствии с целевой точкой восстановления. Функции резервного копирования и восстановления службы предоставляют необходимые стандартные блоки для реализации стратегии аварийного восстановления.

Операции резервного копирования и восстановления также можно использовать для репликации конфигурации службы управления API между операционными средами, например для разработки и промежуточного хранения. Помните, что данные среды выполнения, такие как пользователи и подписки, также будут копироваться, что может быть не всегда желательным.

В этом руководстве показано, как автоматизировать операции резервного копирования и восстановления, а также как обеспечить успешную проверку подлинности запросов на резервное копирование и восстановление, Azure Resource Manager.

> [!IMPORTANT]
> Операция восстановления не изменяет конфигурацию настраиваемого узла целевой службы. Мы рекомендуем использовать одно и то же настраиваемое имя узла и сертификат TLS как для активных, так и для резервных служб, чтобы после завершения операции восстановления трафик можно было перенаправлять в резервный экземпляр с помощью простого изменения DNS CNAME.
>
> Операция резервного копирования не захватывает предварительно агрегированные данные журнала, используемые в отчетах, отображаемых в колонке аналитики в портал Azure.

> [!WARNING]
> Срок действия каждой резервной копии истекает через 30 дней. Если вы попытаетесь восстановить резервную копию по истечении 30 дней, восстановление завершится сбоем и будет отображено сообщение `Cannot restore: backup expired`.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="authenticating-azure-resource-manager-requests"></a>Проверка подлинности запросов к диспетчеру ресурсов Azure

> [!IMPORTANT]
> Интерфейс REST API для резервного копирования и восстановления использует диспетчер ресурсов Azure и применяет механизм проверки подлинности, отличный от интерфейсов REST API, используемых для управления объектами службы управления API. В этом разделе описываются действия, необходимые для проверки подлинности запросов к диспетчеру ресурсов Azure. Дополнительные сведения см. в [справочнике REST API Azure](/rest/api/index).

Все задачи, которые выполняются в ресурсах с помощью Azure Resource Manager, необходимо аутентифицировать в Azure Active Directory. Для этого:

-   добавьте приложение в клиент Azure Active Directory;
-   настройте разрешения для добавленного приложения;
-   получите маркер для проверки подлинности запросов к диспетчеру ресурсов Azure.

### <a name="create-an-azure-active-directory-application"></a>Создание приложения Azure Active Directory

1. Войдите на [портал Azure](https://portal.azure.com).
2. Используя подписку, включающую экземпляр службы управления API, перейдите на вкладку **Регистрация приложений** в **Azure Active Directory** (Azure Active Directory > Управление/Регистрация приложений).

    > [!NOTE]
    > Если этот каталог не отображается в вашей учетной записи, обратитесь к администратору подписки Azure, чтобы получить необходимые разрешения для учетной записи.

3. Щелкните **Регистрация нового приложения**.

    Справа появится окно **Создание**. Здесь вам нужно ввести информацию о приложении AAD.

4. Введите имя приложения.
5. Выберите тип приложения **Машинный код**.
6. Введите в поле **URI перенаправления** любой URL-адрес, например `http://resources`. Это поле является обязательным, но введенное значение впоследствии не используется. Установите флажок, чтобы сохранить приложение.
7. Нажмите кнопку **Создать**.

### <a name="add-an-application"></a>Добавление приложения

1. После создания приложения щелкните **разрешения API**.
2. Щелкните **+ Добавить разрешение**.
4. Нажмите кнопку **выбрать API-интерфейсы Майкрософт**.
5. Выберите **Управление службами Azure**.
6. Нажмите кнопку **Выбрать**.

    ![Добавление разрешений](./media/api-management-howto-disaster-recovery-backup-restore/add-app.png)

7. Щелкните **Делегированные разрешения** рядом с только что добавленным приложением и установите флажок **Access Azure Service Management (preview)** (Доступ к управлению службами Azure (предварительная версия)).
8. Нажмите кнопку **Выбрать**.
9. Нажмите кнопку **Предоставить разрешения**.

### <a name="configuring-your-app"></a>Настройка приложения

Прежде чем вызывать интерфейсы API, выполняющие резервное копирование и восстановление, необходимо получить маркер. В следующем примере для получения токена используется пакет NuGet [Microsoft. IdentityModel. Clients. ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) .

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System;

namespace GetTokenResourceManagerRequests
{
    class Program
    {
        static void Main(string[] args)
        {
            var authenticationContext = new AuthenticationContext("https://login.microsoftonline.com/{tenant id}");
            var result = authenticationContext.AcquireTokenAsync("https://management.azure.com/", "{application id}", new Uri("{redirect uri}"), new PlatformParameters(PromptBehavior.Auto)).Result;

            if (result == null) {
                throw new InvalidOperationException("Failed to obtain the JWT token");
            }

            Console.WriteLine(result.AccessToken);

            Console.ReadLine();
        }
    }
}
```

Замените `{tenant id}`, `{application id}` и `{redirect uri}`, следуя приведенным инструкциям.

1. Замените `{tenant id}` идентификатором клиента для созданного приложения Azure Active Directory. Чтобы получить доступ к идентификатору, щелкните **Регистрация приложений**  ->  **конечные точки**.

    ![Конечные точки][api-management-endpoint]

2. Замените `{application id}` значением, которое представлено на странице **Параметры**.
3. Замените значение `{redirect uri}` на значение с вкладки **URI перенаправления** приложения Azure Active Directory.

    Когда все значения будут указаны, этот пример кода должен вернуть примерно такой маркер:

    ![Токен][api-management-arm-token]

    > [!NOTE]
    > Срок действия маркера может истечь после определенного периода. Выполнить образец кода снова, чтобы создать новый маркер.

## <a name="calling-the-backup-and-restore-operations"></a>Вызов операций резервного копирования и восстановления

Интерфейсы REST API: [служба управления API — резервное копирование](/rest/api/apimanagement/2019-12-01/apimanagementservice/backup) и [служба управления API — восстановление](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore).

Прежде чем выполнять операции резервного копирования и восстановления, описанные в следующих разделах, задайте заголовок запроса авторизации для вызова REST.

```csharp
request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + token);
```

### <a name="back-up-an-api-management-service"></a><a name="step1"> </a>Архивация службы управления API

Чтобы выполнить архивацию службы управления API, отправьте следующий HTTP-запрос:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/backup?api-version={api-version}
```

где:

-   `subscriptionId` — идентификатор подписки, содержащей службу управления API, для которой вы пытаетесь выполнить резервное копирование;
-   `resourceGroupName` — имя группы ресурсов службы управления API Azure
-   `serviceName` — имя службы управления API, резервное копирование которой вы выполняете, на момент ее создания;
-   `api-version` -заменить на `2019-12-01`

В тексте запроса укажите целевую учетную запись хранения Azure, ключ доступа, имя контейнера BLOB-объектов и имя резервной копии:

```json
{
    "storageAccount": "{storage account name for the backup}",
    "accessKey": "{access key for the account}",
    "containerName": "{backup container name}",
    "backupName": "{backup blob name}"
}
```

Для заголовка запроса `Content-Type` установите значение `application/json`.

Резервное копирование — это длительная операция, которая занимает несколько минут. Если запрос выполнен успешно и запущен процесс резервного копирования, вы получите код состояния ответа `202 Accepted` с заголовком `Location`. Чтобы отслеживать состояние операции, отправляйте запросы GET на URL-адрес, указанный в заголовке `Location` . Пока резервное копирование выполняется, вы получаете код состояния "202 Accepted". Код ответа `200 OK` указывает на то, что резервное копирование успешно завершено.

### <a name="restore-an-api-management-service"></a><a name="step2"> </a>Восстановление службы управления API

Чтобы восстановить службу управления API из ранее созданной резервной копии, отправьте следующий HTTP-запрос:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/restore?api-version={api-version}
```

где:

-   `subscriptionId` — идентификатор подписки, включающей в себя службу управления API, которую нужно восстановить из резервной копии;
-   `resourceGroupName` — имя группы ресурсов, включающей в себя службу управления API Azure, в которую нужно восстановить резервную копию;
-   `serviceName` — имя восстанавливаемой службы управления API на момент ее создания;
-   `api-version` -заменить на `api-version=2019-12-01`

В тексте запроса укажите расположение файла резервной копии, то есть добавьте целевую учетную запись хранения Azure, ключ доступа, имя контейнера больших двоичных объектов и имя резервной копии:

```json
{
    "storageAccount": "{storage account name for the backup}",
    "accessKey": "{access key for the account}",
    "containerName": "{backup container name}",
    "backupName": "{backup blob name}"
}
```

Для заголовка запроса `Content-Type` установите значение `application/json`.

Восстановление — это длительная операция, которая может занять до 30 минут и более. Если запрос выполнен успешно и процесс восстановления инициирован, вы получите код состояния ответа `202 Accepted` с заголовком `Location`. Чтобы отслеживать состояние операции, отправляйте запросы GET на URL-адрес, указанный в заголовке `Location` . Пока восстановление выполняется, вы получаете код состояния "202 Accepted". Код ответа `200 OK` указывает на то, что восстановление успешно завершено.

> [!IMPORTANT]
> **Номер SKU** восстанавливаемой службы **должен совпадать** с номером SKU, сохраненным в резервной копии.
>
> **Изменения**, внесенные в конфигурацию службы (например, интерфейсы API, политики, внешний вид портала разработчика) во время восстановления, **могут быть перезаписаны**.

<!-- Dummy comment added to suppress markdown lint warning -->

> [!NOTE]
> Операции резервного копирования и восстановления также можно выполнять с помощью команд PowerShell [_BACKUP-азапиманажемент_](/powershell/module/az.apimanagement/backup-azapimanagement) и [_RESTORE-азапиманажемент_](/powershell/module/az.apimanagement/restore-azapimanagement) соответственно.

## <a name="constraints-when-making-backup-or-restore-request"></a>Ограничения при выполнении запроса на резервное копирование или восстановление

-   **Контейнер**, указанный в теле запроса, **должен существовать**.
-   Во время резервного копирования **следует избегать изменений в службе** , таких как обновление номера SKU или переход на более раннюю версию, изменение имени домена и многое другое.
-   Восстановление **резервной копии гарантируется только в течение 30 дней** с момента ее создания.
-   **Изменения** , внесенные в конфигурацию службы (например, API, политики и внешний вид портала разработчика) во время операции резервного копирования, **могут быть исключены из резервной копии и будут утеряны**.
-   Если учетная запись хранения Azure включена в [брандмауэре][azure-storage-ip-firewall] , клиент должен **Разрешить** набор [IP-адресов плоскости управления Azure API Management][control-plane-ip-address] в учетной записи хранения для резервного копирования или восстановления из для работы. Учетная запись хранения Azure может находиться в любом регионе Azure, Кроме того, в котором находится служба управления API. Например, если служба управления API находится в западной части США, учетная запись хранения Azure может находиться в западной части США 2, а клиент должен открыть панель управления IP-13.64.39.16 (IP-адрес плоскости управления API в западной части США) в брандмауэре. Это связано с тем, что запросы к службе хранилища Azure не Снатед на общедоступный IP-адрес из вычислений (плоскость управления API управления Azure) в одном регионе Azure. Запрос на хранение в разных регионах будет Снатед на общедоступный IP-адрес.
-   [Общий доступ к ресурсам в разных источниках (CORS)](/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) **не** должен включаться в службе BLOB-объектов в учетной записи хранения Azure.
-   **Номер SKU** восстанавливаемой службы **должен совпадать** с номером SKU, сохраненным в резервной копии.

## <a name="what-is-not-backed-up"></a>Что резервное копирование не выполнено
-   **Данные об использовании**, применяемые при создании аналитических отчетов, **не включаются** в резервную копию. Для периодического извлечения аналитических отчетов с целью их дальнейшего помещения на хранение используйте [API REST Azure API Management][azure api management rest api] .
-   Сертификаты [TLS/SSL пользовательского домена](configure-custom-domain.md) .
-   [Пользовательский сертификат ЦС](api-management-howto-ca-certificates.md), который включает промежуточные или корневые сертификаты, загруженные клиентом.
-   Параметры интеграции [виртуальной сети](api-management-using-with-vnet.md) .
-   Конфигурация [управляемого удостоверения](api-management-howto-use-managed-service-identity.md) .
-   [Диагностика Azure Monitor](api-management-howto-use-azure-monitor.md) Настройка.
-   [Протоколы и параметры шифров](api-management-howto-manage-protocols-ciphers.md) .
-   Содержимое [портала разработчика](api-management-howto-developer-portal.md#is-the-portals-content-saved-with-the-backuprestore-functionality-in-api-management) .

Частота резервного копирования службы влияет на целевую точку восстановления. Чтобы максимально снизить этот показатель, мы советуем настроить регулярное резервное копирование, а также выполнять резервное копирование по требованию после внесения изменений в службу управления API.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы ознакомиться с другими способами резервного копирования и восстановления, ознакомьтесь со следующими ресурсами.

-   [Репликация учетных записей управления API Azure](https://www.returngis.net/en/2015/06/replicate-azure-api-management-accounts/)
-   [Automating API Management Backup and Restore with Logic Apps](https://github.com/Azure/api-management-samples/tree/master/tutorials/automating-apim-backup-restore-with-logic-apps) (Автоматизация управления API резервного копирования и восстановления с помощью Logic Apps)
-   [Управление API Azure: резервное копирование и восстановление конфигурации](/archive/blogs/stuartleeks/azure-api-management-backing-up-and-restoring-configuration) 
     _Подход, описанный в Стюарт, не соответствует официальному руководству, но интересно._

[backup an api management service]: #step1
[restore an api management service]: #step2
[azure api management rest api]: /rest/api/apimanagement/apimanagementrest/api-management-rest
[api-management-add-aad-application]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-add-aad-application.png
[api-management-aad-permissions]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-permissions.png
[api-management-aad-permissions-add]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-permissions-add.png
[api-management-aad-delegated-permissions]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-delegated-permissions.png
[api-management-aad-default-directory]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-default-directory.png
[api-management-aad-resources]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-resources.png
[api-management-arm-token]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-arm-token.png
[api-management-endpoint]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-endpoint.png
[control-plane-ip-address]: api-management-using-with-vnet.md#control-plane-ips
[azure-storage-ip-firewall]: ../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range
