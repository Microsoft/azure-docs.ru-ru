---
title: Завершение использования расширения управляемой виртуальной машины для управляемых удостоверений — Azure AD
description: Пошаговые инструкции по отмене использования расширения виртуальной машины и началу использования службы метаданных экземпляра Azure (IMDS) для проверки подлинности.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/03/2020
ms.author: barclayn
ROBOTS: NOINDEX
ms.openlocfilehash: dca5f9ed2911ae3042fb9871f849212ec18b1b58
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99539389"
---
# <a name="how-to-stop-using-the-virtual-machine-managed-identities-extension-and-start-using-the-azure-instance-metadata-service"></a>Как отключить использование расширения управляемых удостоверений виртуальной машины и начать использовать службу метаданных экземпляра Azure

## <a name="virtual-machine-extension-for-managed-identities"></a>Расширение виртуальной машины для управляемых удостоверений

Расширение виртуальной машины для управляемых удостоверений используется для запроса токенов для управляемого удостоверения на виртуальной машине. Рабочий процесс состоит из следующих шагов:

1. Во первых, Рабочая нагрузка в ресурсе вызывает локальную конечную точку `http://localhost/oauth2/token` для запроса маркера доступа.
2. Затем расширение виртуальной машины использует учетные данные для управляемого удостоверения, чтобы запросить маркер доступа из Azure AD. 
3. Маркер доступа возвращается вызывающему объекту и может использоваться для проверки подлинности в службах, поддерживающих проверку подлинности Azure AD, например Azure Key Vault или службы хранилища Azure.

Из-за нескольких ограничений, описанных в следующем разделе, расширение управляемой виртуальной машины для управляемых удостоверений было нерекомендуемым в пользу использования эквивалентной конечной точки в службе метаданных экземпляра Azure (IMDS).

### <a name="provision-the-extension"></a>Подготавливает расширение 

Если для виртуальной машины или масштабируемого набора виртуальных машин настроено управляемое удостоверение, при необходимости вы можете выбрать подготовку управляемых удостоверений для расширения виртуальной машины ресурсов Azure с помощью `-Type` параметра командлета [Set-азвмекстенсион](/powershell/module/az.compute/set-azvmextension) . Вы можете передать значение `ManagedIdentityExtensionForWindows` или `ManagedIdentityExtensionForLinux` , в зависимости от типа виртуальной машины, и присвоить ему имя с помощью `-Name` параметра. В параметре `-Settings` указан порт, используемый конечной точкой токена OAuth для получения токена:

```azurepowershell-interactive
$settings = @{ "port" = 50342 }
   Set-AzVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
```

Можно также использовать шаблон развертывания Azure Resource Manager для инициализации расширения виртуальной машины, добавив следующий код JSON в `resources` раздел шаблона (используйте `ManagedIdentityExtensionForLinux` для элементов name и Type для версии Linux).

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
    "apiVersion": "2018-06-01",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.ManagedIdentity",
        "type": "ManagedIdentityExtensionForWindows",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "port": 50342
        }
    }
}
```
    
    
При работе с масштабируемыми наборами виртуальных машин можно также подготавливать управляемые удостоверения для расширения масштабируемого набора виртуальных машин Azure Resources с помощью командлета [Add-азвмссекстенсион](/powershell/module/az.compute/add-azvmssextension) . Вы можете передать значение `ManagedIdentityExtensionForWindows` или `ManagedIdentityExtensionForLinux` , в зависимости от типа масштабируемого набора виртуальных машин, и присвоить ему имя с помощью `-Name` параметра. В параметре `-Settings` указан порт, используемый конечной точкой токена OAuth для получения токена:

   ```azurepowershell-interactive
   $setting = @{ "port" = 50342 }
   $vmss = Get-AzVmss
   Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Setting $settings 
   ```
Чтобы настроить расширение масштабируемого набора виртуальных машин с помощью шаблона развертывания Azure Resource Manager, добавьте следующий код JSON в `extensionpProfile` раздел шаблона (используйте `ManagedIdentityExtensionForLinux` для элементов name и Type для версии Linux).

```json
"extensionProfile": {
    "extensions": [
        {
            "name": "ManagedIdentityWindowsExtension",
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
                },
                "protectedSettings": {}
            }
        }
```

Подготовка расширения виртуальной машины может завершиться ошибкой из-за сбоев уточняющего запроса DNS. В этом случае перезапустите виртуальную машину и повторите попытку. 

### <a name="remove-the-extension"></a>Удаление расширения 
Чтобы удалить расширение, используйте `-n ManagedIdentityExtensionForWindows` или `-n ManagedIdentityExtensionForLinux` коммутатор (в зависимости от типа виртуальной машины) с помощью команды [AZ VM Extension Delete](/cli/azure/vm/)или [AZ vmss Extension Delete](/cli/azure/vmss) для масштабируемых наборов виртуальных машин с помощью Azure CLI или `Remove-AzVMExtension` PowerShell:

```azurecli-interactive
az vm identity --resource-group myResourceGroup --vm-name myVm -n ManagedIdentityExtensionForWindows
```

```azurecli-interactive
az vmss extension delete -n ManagedIdentityExtensionForWindows -g myResourceGroup -vmss-name myVMSS
```

```azurepowershell-interactive
Remove-AzVMExtension -ResourceGroupName myResourceGroup -Name "ManagedIdentityExtensionForWindows" -VMName myVM
```

### <a name="acquire-a-token-using-the-virtual-machine-extension"></a>Получение маркера с помощью расширения виртуальной машины

Ниже приведен пример запроса с использованием управляемых удостоверений для конечной точки расширения виртуальной машины для ресурсов Azure.

```
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Элемент | Описание |
| ------- | ----------- |
| `GET` | HTTP-команда, указывающая, что необходимо извлечь данные из конечной точки. В этом случае используется маркер доступа OAuth. | 
| `http://localhost:50342/oauth2/token` | Конечная точка управляемых удостоверений для ресурсов Azure, где 50342 — порт по умолчанию, который можно настроить. |
| `resource` | Параметр строки запроса, указывающий URI идентификатора приложения целевого ресурса. Он также отображается в утверждении (аудитории) `aud` выданного маркера. В этом примере запрашивается маркер для доступа к Azure Resource Manager, который имеет универсальный код ресурса (URI) идентификатора приложения `https://management.azure.com/`. |
| `Metadata` | Поле заголовка HTTP-запроса, необходимое управляемым удостоверениям для ресурсов Azure для устранения рисков от атак с подделкой запроса со стороны сервера (SSRF). Должно быть присвоено значение true (все в нижнем регистре).|
| `object_id` | (Необязательно.) Параметр строки запроса, указывающий object_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|
| `client_id` | (Необязательно.) Параметр строки запроса, указывающий client_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|


Пример ответа:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Элемент | Описание |
| ------- | ----------- |
| `access_token` | Запрашиваемый маркер доступа. При вызове защищенного REST API маркер внедряется в поле `Authorization` заголовка запроса в качестве маркера носителя, позволяя API выполнить проверку подлинности вызывающего объекта. | 
| `refresh_token` | Не используется управляемыми удостоверениями для ресурсов Azure. |
| `expires_in` | Количество секунд, в течение которых маркер доступа является действительным, прежде чем его срок действия истечет (с момента выдачи). Время выдачи можно найти в утверждении `iat` маркера. |
| `expires_on` | Период времени после истечения срока действия маркера доступа. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `exp` маркера). |
| `not_before` | Период времени, когда маркер доступа вступает в силу и может быть принят. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `nbf` маркера). |
| `resource` | Ресурс, для которого был запрошен маркер доступа, соответствует параметру строки запроса `resource`. |
| `token_type` | Тип маркера, который является маркером доступа носителя, что значит, что ресурс может предоставлять доступ носителю этого маркера. |


### <a name="troubleshoot-the-virtual-machine-extension"></a>Устранение неполадок с расширением виртуальной машины 

#### <a name="restart-the-virtual-machine-extension-after-a-failure"></a>Перезапустить расширение виртуальной машины после сбоя

При завершении работы расширения в Windows и некоторых версиях Linux вы можете перезапустить его вручную с помощью следующего командлета:

```azurepowershell-interactive
Set-AzVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Где: 
- Имя и тип расширения для Windows: `ManagedIdentityExtensionForWindows`
- Имя и тип расширения для Linux: `ManagedIdentityExtensionForLinux`

#### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Скрипт автоматизации завершается ошибкой при попытке экспорта схемы для расширения управляемых удостоверений для ресурсов Azure

Если управляемые удостоверения для ресурсов Azure включены на виртуальной машине, при попытке использовать функцию "скрипт автоматизации" для виртуальной машины или ее группы ресурсов отображается следующая ошибка:

![Ошибка экспорта скрипта автоматизации управляемых удостоверений для ресурсов Azure](./media/howto-migrate-vm-extension/automation-script-export-error.png)

В настоящее время расширение виртуальной машины "управляемые удостоверения для ресурсов Azure" не поддерживает возможность экспорта схемы в шаблон группы ресурсов. Поэтому созданный шаблон не содержит параметры конфигурации для включения управляемых удостоверений для ресурсов Azure в ресурсе. Эти разделы можно добавить вручную, следуя примерам в разделе [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью шаблонов](qs-configure-template-windows-vm.md).

Когда функция экспорта схемы становится доступной для расширения виртуальной машины "управляемые удостоверения для ресурсов Azure" (запланировано на прекращение в январе 2019), оно будет указано в списке [Экспорт групп ресурсов, содержащих расширения виртуальных машин](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

## <a name="limitations-of-the-virtual-machine-extension"></a>Ограничения расширения виртуальной машины 

Использование расширения виртуальной машины имеет несколько основных ограничений. 

 * Самое серьезное ограничение заключается в том, что учетные данные, используемые для запроса маркеров, хранятся на виртуальной машине. Злоумышленник, успешно нарушающий виртуальную машину, может захвату учетные данные. 
 * Кроме того, расширение виртуальной машины по-прежнему не поддерживается несколькими дистрибутивами Linux, и значительной стоимостью разработки для изменения, сборки и тестирования расширения в каждом из этих дистрибутивов. Сейчас поддерживаются только следующие дистрибутивы Linux: 
    * CoreOS Stable
    * CentOS 7.1; 
    * Red Hat 7.2 
    * Ubuntu 15.04. 
    * Ubuntu 16.04
 * Развертывание виртуальных машин с управляемыми удостоверениями влияет на производительность, так как расширение виртуальной машины также должно быть подготовлено. 
 * Наконец, расширение виртуальной машины может поддерживать только 32 назначаемых пользователем управляемых удостоверений на каждую виртуальную машину. 

## <a name="azure-instance-metadata-service"></a>Служба метаданных экземпляров Azure

[Служба метаданных экземпляров Azure (IMDS)](../../virtual-machines/windows/instance-metadata-service.md) — это конечная точка RESTful, которая предоставляет сведения о выполняющихся экземплярах виртуальных машин, которые можно использовать для управления виртуальными машинами и их настройки. Конечная точка доступна на хорошо известном IP-адресе, не поддерживающем маршрутизацию ( `169.254.169.254` ), доступ к которому можно получить только с помощью виртуальной машины.

Использование Azure IMDS для запроса маркеров имеет несколько преимуществ. 

1. Служба является внешней по отношению к виртуальной машине, поэтому учетные данные, используемые управляемыми удостоверениями, больше не находятся на виртуальной машине. Вместо этого они размещаются и защищаются на главном компьютере виртуальной машины Azure.   
2. Все операционные системы Windows и Linux, поддерживаемые в Azure IaaS, могут использовать управляемые удостоверения.
3. Развертывание выполняется быстрее и проще, так как расширение виртуальной машины больше не нужно подготавливать.
4. Конечная точка IMDS позволяет назначить одной виртуальной машине до 1000 управляемых удостоверений, назначенных пользователю.
5. Нет существенных изменений запросов, использующих IMDS, в отличие от тех, которые используют расширение виртуальной машины, поэтому достаточно просто перенести существующие развертывания, в которых в настоящее время используется расширение виртуальной машины.

По этим причинам служба IMDS Azure является нерекомендуемым способом запроса токенов, как только расширение виртуальной машины будет признано устаревшим. 


## <a name="next-steps"></a>Дальнейшие действия

* [Использование управляемых удостоверений для ресурсов Azure на виртуальной машине Azure для получения маркера доступа](how-to-use-vm-token.md)
* [Служба метаданных экземпляров Azure](../../virtual-machines/windows/instance-metadata-service.md)
