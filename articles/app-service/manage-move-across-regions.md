---
title: Перемещение приложения в другой регион
description: Узнайте, как перемещать ресурсы службы приложений из одного региона в другой.
ms.topic: how-to
ms.date: 02/27/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 87a2da31802a8b6858e875c23ef1dbd2d6d006bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86524860"
---
# <a name="move-an-app-service-resource-to-another-region"></a>Перемещение ресурса службы приложений в другой регион

В этой статье описывается, как переместить ресурсы службы приложений в другой регион Azure. Вы можете переместить ресурсы в другой регион по ряду причин. Например, чтобы использовать преимущества нового региона Azure для развертывания компонентов или служб, доступных только в конкретных регионах, в соответствии с требованиями к внутренней политике и управлению, а также в ответ на требования к планированию загрузки.

Ресурсы службы приложений зависят от региона и не могут быть перемещены между регионами. Необходимо создать копию существующих ресурсов службы приложений в целевом регионе, а затем переместить содержимое в новое приложение. Если исходное приложение использует личный домен, его можно перенести в [новое приложение в целевом регионе](manage-custom-dns-migrate-domain.md) по завершении работы.

Чтобы упростить копирование приложения, можно [клонировать отдельное приложение службы приложений](app-service-web-app-cloning.md) в план службы приложений в другом регионе, но у него есть [ограничения](app-service-web-app-cloning.md#current-restrictions), особенно те, которые не поддерживают приложения Linux.

## <a name="prerequisites"></a>Предварительные требования

- Убедитесь, что приложение службы приложений находится в регионе Azure, из которого вы хотите переместить.
- Убедитесь, что целевой регион поддерживает службу приложений и любую связанную службу, ресурсы которой требуется переместить.
<!-- - Domain bindings, certificates, and managed identities can't replicated using the **Export template** method. You must create them manually. -->

## <a name="prepare"></a>Подготовка

Найдите все ресурсы службы приложений, которые вы используете в данный момент. Пример:

- Приложения службы приложений
- [Планы службы приложений](overview-hosting-plans.md)
- [Слоты развертывания](deploy-staging-slots.md)
- [Пользовательские домены, приобретенные в Azure](manage-custom-dns-buy-domain.md)
- [SSL-сертификаты](configure-ssl-certificate.md)
- [Интеграция виртуальной сети Azure](web-sites-integrate-with-vnet.md)
- [Гибридные подключения](app-service-hybrid-connections.md).
- [Управляемые удостоверения](overview-managed-identity.md)
- [Параметры резервного копирования](manage-backup.md)

Некоторые ресурсы, например импортированные сертификаты или гибридные подключения, содержат интеграцию с другими службами Azure. Сведения о том, как переместить эти ресурсы в регионах, см. в документации по соответствующим службам.

## <a name="move"></a>Переместить

1. [Создайте резервную копию исходного приложения](manage-backup.md).
1. [Создайте приложение в новом плане службы приложений в целевом регионе](app-service-plan-manage.md#create-an-app-service-plan).
2. [Восстановление резервной копии в целевом приложении](web-sites-restore.md)
2. Если вы используете личный домен, [привяжите его к целевому приложению](manage-custom-dns-migrate-domain.md#bind-the-domain-name-preemptively) с помощью `awverify.` и [включите домен в целевом приложении](manage-custom-dns-migrate-domain.md#enable-the-domain-for-your-app).
3. Настройте все остальное в целевом приложении так, чтобы оно совпадало с исходным приложением, и проверьте конфигурацию.
4. Когда личный домен будет готов к переходу к целевому приложению, следует [сопоставить доменное имя](manage-custom-dns-migrate-domain.md#remap-the-active-dns-name).

<!-- 1. Login to the [Azure portal](https://portal.azure.com) > **Resource Groups**.
2. Locate the Resource Group that contains the source App Service resources and click on it.
3. Select > **Settings** > **Export template**.
4. Choose **Deploy** in the **Export template** blade.
5. Click **TEMPLATE** > **Edit template** to open the template in the online editor.
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"identity": {` to find any managed identity definition. The following is an example if you have a user-assigned managed identity.
    ```json
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/<group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<identity-name>": {
                "principalId": "00000000-0000-0000-0000-000000000000",
                "clientId": "00000000-0000-0000-0000-000000000000"
            }
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/sites/hostNameBindings` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/sites/hostNameBindings",
        "apiVersion": "2018-11-01",
        "name": "[concat(parameters('sites_webapp_name'), '/', parameters('sites_webapp_name'), '.azurewebsites.net')]",
        "location": "West Europe",
        "dependsOn": [
            "[resourceId('Microsoft.Web/sites', parameters('sites_webapp_name'))]"
        ],
        "properties": {
            "siteName": "<app-name>",
            "hostNameType": "Verified"
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/certificates` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/certificates",
        "apiVersion": "2018-11-01",
        "name": "[parameters('certificates_test2_cephaslin_com_name')]",
        "location": "West Europe",
        "properties": {
            "hostNames": [
                "[parameters('certificates_test2_cephaslin_com_name')]"
            ],
            "password": "[parameters('certificates_test2_cephaslin_com_password')]"
        }
    },
    ```
7. Delete the entire JSON block. Click **Save** in the online editor.
8. Click **BASICS** > **Create new** to create a new resource group. Type the group name and click **OK**.
9. In **BASICS** > **Location**, select the region you want.   -->

## <a name="clean-up-source-resources"></a>Очистка исходных ресурсов

Удаление исходного приложения и плана службы приложений. [На план службы приложений на небесплатный уровень взимается плата, даже если в нем не выполняется ни одно приложение.](app-service-plan-manage.md#delete-an-app-service-plan)

## <a name="next-steps"></a>Дальнейшие действия

[Клонирование приложений службы приложений Azure с помощью PowerShell](app-service-web-app-cloning.md)