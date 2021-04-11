---
title: Руководство по развертыванию в существующей виртуальной сети с помощью Azure CLI — служба выделенных устройств HSM Azure | Документация Майкрософт
description: В руководстве описано развертывание выделенного устройства HSM с помощью CLI в существующей виртуальной сети
services: dedicated-hsm
documentationcenter: na
author: msmbaldwin
manager: rkarlin
editor: ''
ms.service: key-vault
ms.topic: tutorial
ms.custom: mvc, seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/25/2021
ms.author: keithp
ms.openlocfilehash: fa1c01c2d9da19ec1f60878de83a509b7cf561e8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105606833"
---
# <a name="tutorial-deploying-hsms-into-an-existing-virtual-network-using-the-azure-cli"></a>Руководство по Развертывание устройств HSM в существующей виртуальной сети с помощью Azure CLI

Служба выделенных устройств HSM Azure предоставляет физическое устройство в исключительное пользование клиента со всеми возможностями административного контроля и под полную ответственность за управление устройством. Так как используется физическое устройство, корпорации Майкрософт приходится контролировать передачу устройства, чтобы убедиться в его эффективном использовании. Поэтому в обычной подписке Azure служба выделенных устройств HSM недоступна для подготовки ресурсов. Клиентам Azure, которым требуется доступ к службе выделенных устройств HSM, сначала следует связаться со своим менеджером Майкрософт по работе с клиентами и запросить регистрацию для использования службы выделенных устройств HSM. Только после этого можно будет подготовить службу к работе. 

В этом руководстве демонстрируется стандартный процесс подготовки к работе. При этом подразумевается следующее:

- у клиента есть виртуальная сеть;
- у клиента есть виртуальная машина;
- клиенту нужно добавить ресурсы HSM в существующую среду.

Стандартная высокодоступная архитектура с развертыванием в нескольких регионах может выглядеть следующим образом:

![Развертывание в нескольких регионах](media/tutorial-deploy-hsm-cli/high-availability-architecture.png)

В этом руководстве описана интеграция пары устройств HSM и требуемого шлюза ExpressRoute (см. Subnet 1 (Подсеть 1) на схеме выше) в существующую виртуальную сеть (см. VNET 1 (Виртуальная сеть 1) на схеме выше).  Все другие ресурсы — это стандартные ресурсы Azure. Аналогичный процесс интеграции можно использовать для устройств HSM в подсети 4 виртуальной сети 3 (см. представленный выше рисунок).

## <a name="prerequisites"></a>Предварительные требования

Служба выделенных устройств HSM Azure сейчас недоступна на портале Azure. Все взаимодействие со службой будут осуществляться из командной строки или с помощью PowerShell. В этом руководстве используется интерфейс командной строки (CLI) в Azure Cloud Shell. Если вы не знакомы с Azure CLI, см. статью [Начало работы с Azure CLI](/cli/azure/get-started-with-azure-cli).

Предполагается, что:

- Вы зарегистрировали службу выделенных устройств HSM Azure.
- Вы получили разрешение на использование службы. Если это не так, свяжитесь со своим менеджером Майкрософт по работе с клиентами, чтобы выяснить этот вопрос.
- Вы создали группу ресурсов, в которую будут добавлены ресурсы, развернутые в рамках этого руководства.
- Вы создали необходимую виртуальную сеть, подсеть и виртуальные машины, как показано на схеме выше, и теперь хотите интегрировать два устройства HSM в эту среду.

Для выполнения всех приведенных ниже инструкций требуется, чтобы вы открыли портал Azure и запустили Cloud Shell (щелкните значок "\>\_" справа в верхнем углу окна портала).

## <a name="provisioning-a-dedicated-hsm"></a>Подготовка выделенного устройства HSM к работе

Подготовка устройств HSM к работе и их интеграция в существующую виртуальную сеть через шлюз ExpressRoute будет проверяться с помощью SSH. Эта проверка помогает обеспечить базовую доступность устройства HSM для последующей настройки.

### <a name="validating-feature-registration"></a>Проверка регистрации функции

Как указано выше, перед подготовкой к работе необходимо зарегистрировать службу выделенных устройств HSM в своей подписке. Чтобы проверить, зарегистрирована ли служба, выполните приведенную ниже команду в Cloud Shell на портале Azure.

```azurecli
az feature show \
   --namespace Microsoft.HardwareSecurityModules \
   --name AzureDedicatedHSM
```

Эти команды должны вернуть состояние Registered (как показано ниже). Если они не возвращают состояние Registered, вам нужно зарегистрировать службу. Для этого свяжитесь со своим менеджером Майкрософт по работе с клиентами.

![Состояние подписки](media/tutorial-deploy-hsm-cli/subscription-status.png)

### <a name="creating-hsm-resources"></a>Создание ресурсов HSM

Перед созданием ресурсов HSM необходимо обеспечить наличие некоторых необходимых ресурсов. Нужно создать виртуальную сеть с диапазонами подсетей для вычислительных ресурсов, устройств HSM и шлюза. Ниже приведены примеры команд по созданию такой виртуальной сети.

```azurecli
az network vnet create \
  --name myHSM-vnet \
  --resource-group myRG \
  --address-prefix 10.2.0.0/16 \
  --subnet-name compute \
  --subnet-prefix 10.2.0.0/24
```

```azurecli
az network vnet subnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name hsmsubnet \
  --address-prefixes 10.2.1.0/24 \
  --delegations Microsoft.HardwareSecurityModules/dedicatedHSMs
```

```azurecli
az network vnet subnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name GatewaySubnet \
  --address-prefixes 10.2.255.0/26
```

>[!NOTE]
>Самым важным моментом при создании виртуальной сети, на который нужно обратить внимание, является то, что параметр delegations для подсети устройства HSM должен иметь значение Microsoft.HardwareSecurityModules/dedicatedHSMs.  Иначе подготовить устройство HSM к работе не удастся.

После настройки сети используйте приведенные ниже команды Azure CLI для подготовки устройств HSM.

1. Для подготовки первого устройства HSM используйте команду [az dedicated-hsm create](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_create). Имя HSM — hsm1. Замените подписку:

   ```azurecli
   az dedicated-hsm create --location westus --name hsm1 --resource-group myRG --network-profile-network-interfaces \
        /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet
   ```

   Развертывание занимает 25–30 минут. Большая часть этого времени уходит на развертывание устройств HSM.

1. Чтобы просмотреть сведения о текущем устройстве HSM, выполните команду [az dedicated-hsm show](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_show):

   ```azurecli
   az dedicated-hsm show --resource group myRG --name hsm1
   ```

1. Подготовьте второе устройство HSM с помощью следующей команды:

   ```azurecli
   az dedicated-hsm create --location westus --name hsm2 --resource-group myRG --network-profile-network-interfaces \
        /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet
   ```

1. Выполните команду [az dedicated-hsm list](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_list), чтобы просмотреть сведения о текущих устройствах HSM:

   ```azurecli
   az dedicated-hsm list --resource-group myRG
   ```

Есть и другие полезные команды. Команда [az dedicated-hsm update](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_update) позволяет обновить HSM:

```azurecli
az dedicated-hsm update --resource-group myRG –name hsm1
```

Удалить HSM можно с помощью команды [az dedicated-hsm delete](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_delete):

```azurecli
az dedicated-hsm delete --resource-group myRG –name hsm1
```

## <a name="verifying-the-deployment"></a>Проверка развертывания

Чтобы убедиться, что устройства подготовлены к работе, и просмотреть их параметры, запустите указанный ниже набор команд. Убедитесь, что группа ресурсов настроена соответствующим образом и имя ресурса совпадает с указанным в файле параметров.

```azurecli
subid=$(az account show --query id --output tsv)
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM1
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM2
```

Должны отобразиться примерно такие выходные данные:

```json
{
    "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSMl",
    "identity": null,
    "kind": null,
    "location": "westus",
    "managedBy": null,
    "name": "HSM1",
    "plan": null,
    "properties": {
        "networkProfile": {
            "networkInterfaces": [
            {
            "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.Network/networkInterfaces/HSMl_HSMnic", "privatelpAddress": "10.0.2.5",
            "resourceGroup": "HSM-RG"
            }
            L
            "subnet": {
                "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.Network/virtualNetworks/demo-vnet/subnets/hsmsubnet", "resourceGroup": "HSM-RG"
            }
        },
        "provisioningState": "Succeeded",
        "stampld": "stampl",
        "statusMessage": "The Dedicated HSM device is provisioned successfully and ready to use."
    },
    "resourceGroup": "HSM-RG",
    "sku": {
        "capacity": null,
        "family": null,
        "model": null,
        "name": "SafeNet Luna Network HSM A790",
        "size": null,
        "tier": null
    },
    "tags": {
        "Environment": "prod",
        "resourceType": "Hsm"
    },
    "type": "Microsoft.HardwareSecurityModules/dedicatedHSMs"
}
```

Теперь вы можете просмотреть ресурсы с помощью [обозревателя ресурсов Azure](https://resources.azure.com/).   В обозревателе ресурсов разверните раздел "Подписки" слева, подписку для службы выделенных устройств HSM, раздел "Группы ресурсов", используемую группу ресурсов и выберите элемент "Ресурсы".

## <a name="testing-the-deployment"></a>Проверка развертывания

Чтобы проверить развертывание, подключитесь к виртуальной машине, которая осуществляет доступ к устройству HSM, а затем подключитесь непосредственно к устройству HSM. Это позволит убедиться, что устройство HSM доступно.
Для подключения к виртуальной машине используется средство SSH. Используйте приведенную ниже команду, заменив имя администратора и имя DNS значениями, указанными в вашем файле параметров.

`ssh adminuser@hsmlinuxvm.westus.cloudapp.azure.com`

В приведенной выше команде вместо имени DNS можно также использовать IP-адрес виртуальной машины. Если команда выполнена успешно, отобразится запрос пароля, который необходимо ввести. После подключения к виртуальной машине можно подключиться к устройству HSM с помощью частного IP-адреса. Его можно узнать на портале ресурса сетевого интерфейса, связанного с HSM.

![Список компонентов](media/tutorial-deploy-hsm-cli/resources.png)

>[!NOTE]
>Обратите внимание на флажок "Показать скрытые типы", который нужно установить, чтобы отобразились ресурсы HSM.

Как видно на приведенном выше снимке экрана, можно щелкнуть HSM1_HSMnic или HSM2_HSMnic, чтобы отобразился соответствующий частный IP-адрес. Или же можно узнать правильный IP-адрес с помощью использовавшейся ранее команды `az resource show`. 

Подставьте правильный IP-адрес в следующую команду:

`ssh tenantadmin@10.0.2.4`

При успешном выполнении отобразится запрос пароля. По умолчанию установлен пароль PASSWORD. По запросу устройства HSM нужно будет изменить пароль на более надежный и воспользоваться механизмом для хранения пароля и предотвращения его утечки, используемым в вашей организации.

>[!IMPORTANT]
>Если вы забудете пароль, настройки устройства HSM нужно будет сбросить, что приведет к потере ключей.

Когда вы подключитесь к устройству HSM с помощью SSH, выполните приведенную ниже команду, чтобы проверить работоспособность устройства.

`hsm show`

Выходные данные должны быть аналогичны показанным на приведенном ниже снимке экрана.

![Снимок экрана: выходные данные в окне PowerShell.](media/tutorial-deploy-hsm-cli/hsm-show-output.png)

На этом этапе вы выделили все ресурсы для высокодоступного развертывания двух устройств HSM, а также убедились, что они доступны и работают. Дальнейшие операции по конфигурации или проверке требуют взаимодействия с самим устройством HSM. Для этого выполните инструкции, изложенные в разделе 7 руководства администратора Thales Luna HSM 7, где описаны действия по инициализации устройства HSM и созданию разделов. Вся документация и ПО будут доступны для скачивания на веб-сайте Thales, после того как вы зарегистрируетесь на [портале поддержки клиентов Thales](https://supportportal.thalesgroup.com/csm) и получите идентификатор клиента. Скачайте клиентское программное обеспечение версии 7.2, чтобы получить все необходимые компоненты.

## <a name="delete-or-clean-up-resources"></a>Удаление или очистка ресурсов

После того как вы завершили работу с устройством HSM, его можно удалить как ресурс и вернуть в пул свободных устройств. Очевидная проблема при этом — возможное наличие конфиденциальных данных клиента на устройстве. Лучший способ очистить устройство — 3 раза ввести неправильный пароль администратора HSM. (Примечание. Речь не об администраторе устройства, а собственно об администраторе HSM.) Устройство нельзя удалить как ресурс Azure, если оно не очищено. Это сделано для защиты материала ключа.

> [!NOTE]
> При возникновении проблем с конфигурацией устройств Thales обратитесь в [службу поддержки клиентов Thales](https://supportportal.thalesgroup.com/csm).

Если вы закончили работу со всеми ресурсами в этой группе ресурсов, их можно удалить с помощью приведенной ниже команды.

```azurecli
az group delete \
   --resource-group myRG \
   --name HSMdeploy \
   --verbose
```

## <a name="next-steps"></a>Дальнейшие действия

Выполнив инструкции, приведенные в этом руководстве, вы подготовили к работе ресурсы службы выделенных устройств HSM, развернули виртуальную сеть с необходимыми устройствами HSM и дополнительными сетевыми компонентами, которые обеспечивают взаимодействие с устройствами HSM.  Теперь вы можете добавить в это развертывание дополнительные ресурсы в соответствии с требованиями вашей архитектуры развертывания. Дополнительные сведения о планировании развертывания см. в базовой документации.
Рекомендуем использовать схему с двумя устройствами HSM в основном регионе, чтобы обеспечить доступность на уровне стойки, и двумя устройствами HSM в другом регионе, чтобы обеспечить доступность на уровне региона. 

* [Обеспечение высокого уровня доступности](high-availability.md)
* [Физическая безопасность](physical-security.md)
* [Сеть](networking.md)
* [Возможности поддержки](supportability.md)
* [Мониторинг](monitoring.md)
