---
title: Руководство разработчика хранилища ключей Azure
description: Разработчики могут использовать хранилище ключей Azure для управления криптографическими ключами в среде Microsoft Azure.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 550084ccbb1df24fe0cbc0a4630efe19e6685e5c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101709909"
---
# <a name="azure-key-vault-developers-guide"></a>Руководство разработчика хранилища ключей Azure

Хранилище Key Vault обеспечивает безопасный доступ к конфиденциальной информации из приложений:

- Ключи, секреты и сертификаты защищаются без необходимости написания кода, и вы можете легко использовать их из своих приложений.
- Вы можете предоставить клиентам возможность самостоятельно и управлять собственными ключами, секретами и сертификатами, чтобы вы могли сосредоточиться на предоставлении основных компонентов программного обеспечения. Таким образом ваши приложения не будут нести ответственность или потенциальную ответственность за ключи клиента, секреты и сертификаты клиентов.
- Приложение может использовать ключи для подписывания и шифрования, но позволяет управлять ключами вне приложения. Дополнительные сведения о ключах см. в разделе [о ключах](../keys/about-keys.md) .
- Вы можете управлять учетными данными, такими как пароли, ключи доступа и токены SAS, сохранив их в Key Vault в качестве секретов, см. [сведения о секретах](../secrets/about-secrets.md) .
- Управление сертификатами. Дополнительные сведения см. в разделе [About Certificates](../certificates/about-certificates.md) .

Дополнительные сведения о хранилище ключей Azure см. в статье [Что такое хранилище ключей Azure?](overview.md).

## <a name="public-previews"></a>Общедоступные предварительные версии

Периодически мы выпускаем общедоступные предварительные версии нового компонента Key Vault. Испытайте общедоступные функции предварительной версии и сообщите нам о том, что вы думаете azurekeyvault@microsoft.com с помощью нашего адреса электронной почты для отзывов.

## <a name="creating-and-managing-key-vaults"></a>Создание хранилищ ключей и управление ими

Управление Key Vault, аналогичное другим службам Azure, выполняется с помощью службы Azure Resource Manager. Azure Resource Manager — это служба развертывания и управления для Azure. Она обеспечивает уровень управления для создания, обновления и удаления ресурсов в учетной записи Azure. Дополнительные сведения см. в разделе [Azure Resource Manager](../../azure-resource-manager/management/overview.md)

Доступом к слою управления управляет служба [управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). В Key Vault, уровень управления, также называемый плоскостью управления или управления, позволяет создавать и администрировать хранилища ключей и его атрибуты, включая политики доступа, но не ключи, секреты и сертификаты, управляемые на плоскости данных. Вы можете использовать предварительно определенную `Key Vault Contributor` роль для предоставления доступа к управлению Key Vault.     

**API и пакеты SDK для управления хранилищем ключей:**

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Ссылки](/cli/azure/keyvault)<br>[Краткое руководство](quick-create-cli.md)|[Ссылки](/powershell/module/az.keyvault)<br>[Краткое руководство](quick-create-powershell.md)|[Ссылки](/rest/api/keyvault/)|[Ссылки](/azure/templates/microsoft.keyvault/vaults)<br>[Краткое руководство](./vault-create-template.md)|[Ссылки](/dotnet/api/microsoft.azure.management.keyvault)|[Ссылки](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)|[Ссылки](/java/api/com.microsoft.azure.management.keyvault)|[Ссылки](/javascript/api/@azure/arm-keyvault)|

См. раздел [клиентские библиотеки](client-libraries.md) для пакетов установки и исходный код.

Дополнительные сведения о Key Vault плоскости управления см. в разделе [Key Vault Management плоскость](./secure-your-key-vault.md#management-plane-and-azure-rbac) .

## <a name="authenticate-to-key-vault-in-code"></a>Проверка подлинности в коде Key Vault

Key Vault использует проверку подлинности Azure AD, которая требует наличия субъекта безопасности Azure AD для предоставления доступа. Субъект безопасности Azure AD может быть пользователем, субъектом-службой приложения, [управляемым удостоверением для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md)или группой участников безопасности любого типа.

### <a name="authentication-best-practices"></a>Рекомендации по проверке подлинности

Рекомендуется использовать управляемое удостоверение для приложений, развернутых в Azure. Если вы используете службы Azure, которые не поддерживают управляемое удостоверение или если приложения развертываются локально, [субъект-служба с сертификатом](../../active-directory/develop/howto-create-service-principal-portal.md) является возможной альтернативой. В этом случае сертификат должен храниться в Key Vault и поворачиваться часто. Субъект-службу с секретом можно использовать для сред разработки и тестирования, а также локально или в Cloud Shell с помощью субъекта пользователя.

Рекомендуемые субъекты безопасности на среду:
- **Рабочая среда**:
  - Управляемое удостоверение или субъект-служба с сертификатом
- **Среды тестирования и разработки**:
  - Управляемое удостоверение, субъект-служба с сертификатом или субъектом-службой с секретом
- **Локальная разработка**:
  - Субъект-пользователь или субъект-служба с секретом

Описанные выше сценарии проверки подлинности поддерживаются **клиентской библиотекой удостоверений Azure** и интегрированы с пакетами sdk для Key Vault. Библиотеку удостоверений Azure можно использовать в разных средах и платформах без изменения кода. Удостоверение Azure также автоматически получает маркер проверки подлинности из входа в Azure с помощью Azure CLI, Visual Studio, Visual Studio Code и других. 

Дополнительные сведения о либарари Azure Identity Client см. в следующих статьях:

**Клиентские библиотеки Azure Identity**

| .NET | Python | Java | JavaScript |
|--|--|--|--|
|[Пакет SDK для Azure Identity .NET](/dotnet/api/overview/azure/identity-readme)|[Пакет SDK для удостоверений Azure Python](/python/api/overview/azure/identity-readme)|[Пакет SDK для Azure Identity Java](/java/api/overview/azure/identity-readme)|[Azure Identity SDK JavaScript](/javascript/api/overview/azure/identity-readme)|     

>[!Note]
> [Библиотека проверки подлинности приложения](/dotnet/api/overview/azure/service-to-service-authentication) , которая была рекомендована для Key Vault пакета SDK для .NET версии 3, которая в настоящее время прекращена. Следуйте указаниям в [статье AppAuthentication to Azure. Руководство по миграции удостоверений](/dotnet/api/overview/azure/app-auth-migration) для миграции на Key Vault пакет SDK для .NET версии 4.

Руководства по проверке подлинности в Key Vault в приложениях см. в следующих статьях:
- [Проверка подлинности Key Vault в приложении, размещенном на виртуальной машине в .NET](./tutorial-net-virtual-machine.md)
- [Аутентификация в Key Vault в приложении, размещенном на виртуальной машине в Python](./tutorial-python-virtual-machine.md)
- [Проверка подлинности в Key Vault со службой приложений](./tutorial-net-create-vault-azure-web-app.md)

## <a name="manage-keys-certificates-and-secrets"></a>Управление ключами, сертификатами и секретами

Доступом к ключам, секретам и сертификатам управляет плоскость данных. Управление доступом к плоскости данных можно выполнить с помощью политик доступа к локальному хранилищу или Azure RBAC (Предварительная версия).

**API ключей и пакеты SDK**

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Ссылки](/cli/azure/keyvault/key)<br>[Краткое руководство](../keys/quick-create-cli.md)|[Ссылки](/powershell/module/az.keyvault/)<br>[Краткое руководство](../keys/quick-create-powershell.md)|[Ссылки](/rest/api/keyvault/#key-operations)|[Ссылки](/azure/templates/microsoft.keyvault/vaults/keys)<br>[Краткое руководство](../keys/quick-create-template.md)|[Ссылки](/dotnet/api/azure.security.keyvault.keys)<br>[Краткое руководство](../keys/quick-create-net.md)|[Ссылки](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)<br>[Краткое руководство](../keys/quick-create-python.md)|[Ссылки](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-keys/4.2.0/index.html)<br>[Краткое руководство](../keys/quick-create-java.md)|[Ссылки](/javascript/api/@azure/keyvault-keys/)<br>[Краткое руководство](../keys/quick-create-node.md)|

**API-интерфейсы и пакеты SDK для сертификатов**

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Ссылки](/cli/azure/keyvault/certificate)<br>[Краткое руководство](../certificates/quick-create-cli.md)|[Ссылки](/powershell/module/az.keyvault)<br>[Краткое руководство](../certificates/quick-create-powershell.md)|[Ссылки](/rest/api/keyvault/#certificate-operations)|Н/Д|[Ссылки](/dotnet/api/azure.security.keyvault.certificates)<br>[Краткое руководство](../certificates/quick-create-net.md)|[Ссылки](/python/api/overview/azure/keyvault-certificates-readme)<br>[Краткое руководство](../certificates/quick-create-python.md)|[Ссылки](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-certificates/4.1.0/index.html)<br>[Краткое руководство](../certificates/quick-create-java.md)|[Ссылки](/javascript/api/@azure/keyvault-certificates/)<br>[Краткое руководство](../certificates/quick-create-node.md)|

**Секреты API и пакеты SDK**

| Azure CLI | PowerShell | REST API | Resource Manager | .NET | Python | Java | JavaScript |  
|--|--|--|--|--|--|--|--|
|[Ссылки](/cli/azure/keyvault/secret)<br>[Краткое руководство](../secrets/quick-create-cli.md)|[Ссылки](/powershell/module/az.keyvault/)<br>[Краткое руководство](../secrets/quick-create-powershell.md)|[Ссылки](/rest/api/keyvault/#secret-operations)|[Ссылки](/azure/templates/microsoft.keyvault/vaults/secrets)<br>[Краткое руководство](../secrets/quick-create-template.md)|[Ссылки](/dotnet/api/azure.security.keyvault.secrets)<br>[Краткое руководство](../secrets/quick-create-net.md)|[Ссылки](/python/api/overview/azure/keyvault-secrets-readme)<br>[Краткое руководство](../secrets/quick-create-python.md)|[Ссылки](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-secrets/4.2.0/index.html)<br>[Краткое руководство](../secrets/quick-create-java.md)|[Ссылки](/javascript/api/@azure/keyvault-secrets/)<br>[Краткое руководство](../secrets/quick-create-node.md)|

См. раздел [клиентские библиотеки](client-libraries.md) для пакетов установки и исходный код.

Дополнительные сведения о Key Vault безопасности плоскости данных см. в статьях [Key Vault плоскость данных и политики доступа](./secure-your-key-vault.md#data-plane-and-access-policies) , [Key Vault плоскость данных и Azure RBAC (Предварительная версия)](./secure-your-key-vault.md#data-plane-and-azure-rbac-preview) .

### <a name="code-examples"></a>Примеры кода

Полные примеры использования хранилища ключей с приложениями см. в следующих документах:

- [Примеры кода Azure Key Vault](https://azure.microsoft.com/resources/samples/?service=key-vault) — примеры кода для Azure Key Vault. 

## <a name="how-tos"></a>Практические руководства

В следующих статьях приводятся рекомендации по решению конкретных задач при работе с Azure Key Vault:

- [Доступ к хранилищу ключей под защитой брандмауэра](access-behind-firewall.md). Для доступа к хранилищу ключей необходимо, чтобы клиентское приложение имело доступ к нескольким конечным точкам, требуемым для различных функций.
- Сведения о развертывании сертификатов на виртуальных машинах из Key Vault — [Windows](../../virtual-machines/extensions/key-vault-windows.md), [Linux](../../virtual-machines/extensions/key-vault-linux.md) — облачное приложение, работающее на виртуальной машине в Azure, нуждается в сертификате. Как получить этот сертификат для виртуальной машины?
- [Развертывание сертификата веб-приложения Azure через Key Vault](../../app-service/configure-ssl-certificate.md#import-a-certificate-from-key-vault)
- Назначение политики доступа (портал[CLI](assign-access-policy-cli.md)  |  [PowerShell](assign-access-policy-powershell.md)  |  [](assign-access-policy-portal.md)). 
- В разделе [Как использовать обратимое удаление в Key Vault с помощью интерфейса командной строки](./key-vault-recovery.md) описывается использование и жизненный цикл хранилища ключей, а также различных объектов хранилища ключей с возможностью обратимого удаления.
- [Передача безопасных значений (например, паролей) во время развертывания](../../azure-resource-manager/templates/key-vault-parameter.md) — Если в процессе развертывания в качестве параметра необходимо передать безопасное значение (например, пароль), его можно сохранить как секретный код в хранилище ключей Azure и вставить ссылку на это значение в другие шаблоны Resource Manager.

## <a name="integrated-with-key-vault"></a>Интеграция с хранилищем ключей

Эти статьи посвящены другим сценариям и службам, использующим Key Vault или интегрирующимся с ним.

- [Шифрование неактивных](../../security/fundamentals/encryption-atrest.md) данных позволяет кодировать (шифровать) данные при сохранении. Ключи шифрования данных часто шифруются с помощью ключа шифрования ключа в Azure Key Vault для дальнейшего ограничения доступа.
- [Azure Information Protection](/azure/information-protection/plan-implement-tenant-key) позволяет управлять собственным ключом клиента. Например, вместо того, чтобы вашим ключом клиента управляла корпорация Майкрософт (по умолчанию), вы можете сами управлять им в соответствии с определенными нормами своей организации. Сценарий с использованием собственного ключа клиента называется BYOK.
- [Служба "Частная связь Azure](private-link-service.md) " позволяет получать доступ к службам Azure (например, Azure Key Vault, службе хранилища azure и Azure Cosmos DB) и службам клиентов и партнеров Azure, размещенных в частной конечной точке в виртуальной сети.
- Key Vault интеграция с [сеткой событий](../../event-grid/event-schema-key-vault.md)  позволяет пользователям получать уведомления при изменении состояния секрета, хранящегося в хранилище ключей. Вы можете распространить новую версию секретов в приложения или поворачивать с течением срока действия секретов, чтобы предотвратить сбои.
- Вы можете защитить секреты [Azure Devops](/azure/devops/pipelines/release/azure-key-vault) от нежелательного доступа в Key Vault.
- [Использование секрета, хранящегося в Key Vault в модулях данных для подключения к службе хранилища Azure](./integrate-databricks-blob-storage.md)
- Настройка и запуск поставщика Azure Key Vault для [драйвера хранилища секретов CSI](./key-vault-integrate-kubernetes.md) в Kubernetes

## <a name="key-vault-overviews-and-concepts"></a>Основные сведения о Key Vault

- [Общие сведения о функции обратимого удаления в Azure Key Vault](soft-delete-overview.md). Описание функции, которая позволяет восстанавливать случайно или намеренно удаленные объекты.
- [Руководство по регулированию Azure Key Vault](overview-throttling.md). Описание основных механизмов регулирования числа запросов и подходов к созданию приложения.
- [Системы безопасности и географические ограничения Azure Key Vault](overview-security-worlds.md). Описание взаимосвязей между регионами и областями безопасности.

## <a name="social"></a>Социальные сети

- [Блог хранилища ключей](/archive/blogs/kv/)
- [Форум хранилища ключей](https://aka.ms/kvforum)