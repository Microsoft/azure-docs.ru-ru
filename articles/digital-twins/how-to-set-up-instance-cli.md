---
title: Настройка экземпляра и проверки подлинности (CLI)
titleSuffix: Azure Digital Twins
description: См. раздел Настройка экземпляра службы Digital двойников с помощью интерфейса командной строки.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 7108ee956c18fbcc150b56cbcb8ae1c69841dda4
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198593"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-cli"></a>Настройка экземпляра и проверки подлинности Azure Digital двойников (CLI)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

В этой статье описаны действия по **настройке нового экземпляра Azure Digital двойников**, включая создание экземпляра и настройку проверки подлинности. По завершении работы с этой статьей у вас будет готовый экземпляр Azure Digital двойников для начала программирования.

Эта версия этой статьи выполняет эти действия вручную, по одной с помощью интерфейса командной строки.
* Чтобы выполнить эти действия вручную с помощью портал Azure, см. версию портала этой статьи: инструкции по [*настройке экземпляра и проверки подлинности (портал)*](how-to-set-up-instance-portal.md).
* Чтобы выполнить автоматическую установку с помощью примера скрипта развертывания, см. статью [*как настроить экземпляр и проверку подлинности (в скрипте)*](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-cloud-shell-session"></a>Создание сеанса Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Создание экземпляра Digital двойников для Azure

В этом разделе вы **создадите новый экземпляр Azure Digital двойников** с помощью команды Cloud Shell. Необходимо указать:
* Группа ресурсов, в которой будет развернут экземпляр. Если у вас еще нет имеющейся группы ресурсов, вы можете создать ее сейчас с помощью следующей команды:
    ```azurecli-interactive
    az group create --location <region> --name <name-for-your-resource-group>
    ```
* Регион для развертывания. Чтобы узнать, какие регионы поддерживают Azure Digital двойников, посетите страницу [*продукты Azure, доступные по регионам*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* Имя экземпляра. Если в вашей подписке есть другой экземпляр Azure Digital двойников в регионе, где уже используется указанное имя, вам будет предложено выбрать другое имя.

Используйте следующие значения в следующей команде, чтобы создать экземпляр:

```azurecli-interactive
az dt create --dt-name <name-for-your-Azure-Digital-Twins-instance> -g <your-resource-group> -l <region>
```

### <a name="verify-success-and-collect-important-values"></a>Проверка успешности и получение важных значений

Если экземпляр успешно создан, результат в Cloud Shell выглядит примерно так, выводя сведения о созданном ресурсе:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/create-instance.png" alt-text="командное окно с успешным созданием группы ресурсов и экземпляра Azure Digital двойников":::

Обратите внимание на имя **узла**, **имени** и группы Azure Digital **двойников из выходных** данных. Это все важные значения, которые могут потребоваться при продолжении работы с вашим экземпляром Azure Digital двойников для настройки проверки подлинности и связанных ресурсов Azure. Если другие пользователи будут программироваться на экземпляре, вы должны использовать эти значения совместно с ними.

> [!TIP]
> Эти свойства и все свойства экземпляра можно просмотреть в любое время, запустив `az dt show --dt-name <your-Azure-Digital-Twins-instance>` .

Теперь у вас есть экземпляр Azure Digital двойников, готовый к работе. Далее вы получите соответствующие разрешения пользователя Azure для управления им.

## <a name="set-up-user-access-permissions"></a>Настройка разрешений доступа пользователей

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

Используйте следующую команду, чтобы назначить роль (она должна выполняться пользователем с [достаточными разрешениями](#prerequisites-permission-requirements) в подписке Azure). Команда требует передачи *имени участника-пользователя* в учетной записи Azure AD для пользователя, которому должна быть назначена роль. В большинстве случаев это будет соответствовать электронной почте пользователя в учетной записи Azure AD.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<Azure-AD-user-principal-name-of-user-to-assign>" --role "Azure Digital Twins Data Owner"
```

Результат этой команды выводит сведения о созданном назначении роли.

> [!NOTE]
> Если эта команда возвращает ошибку, сообщающую, что CLI **не может найти пользователя или субъект-службу в базе данных Graph**:
>
> Назначьте роль с помощью *идентификатора объекта* пользователя. Это может произойти для пользователей в личных [учетных записях Майкрософт (MSAS)](https://account.microsoft.com/account). 
>
> Используйте [страницу портал Azure Azure Active Directory пользователей](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) , чтобы выбрать учетную запись пользователя и открыть сведения о ней. Скопируйте *ObjectID* пользователя:
>
> :::image type="content" source="media/includes/user-id.png" alt-text="Представление страницы пользователя в портал Azure выделение идентификатора GUID в поле &quot;идентификатор объекта&quot;" lightbox="media/includes/user-id.png":::
>
> Затем повторите команду списка назначений ролей, используя *идентификатор объекта* пользователя для приведенного `assignee` выше параметра.

### <a name="verify-success"></a>Проверка успешного выполнения

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

Теперь у вас есть экземпляр Azure Digital двойников, который готов к работе и ему назначены разрешения для управления им.

## <a name="next-steps"></a>Дальнейшие действия

Вытестируйте отдельные REST API вызовы в экземпляре с помощью команд CLI Azure Digital двойников: 
* [AZ DT Справочник](/cli/azure/ext/azure-iot/dt)
* [*Практическое руководство. Использование CLI для Azure Digital Twins*](how-to-use-cli.md)

Или см. раздел как подключить клиентское приложение к экземпляру с помощью кода проверки подлинности:
* [*Пошаговое руководство. Написание кода проверки подлинности приложения*](how-to-authenticate-client.md)