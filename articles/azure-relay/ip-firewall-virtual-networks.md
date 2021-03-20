---
title: Настройка брандмауэра IP для пространства имен Azure Relay
description: Сведения о том, как с помощью правил брандмауэра разрешить подключения к пространствам имен Azure Relay только с определенных IP-адресов.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: ad8feed5df49dcc4503226a5fae50195bb9d48aa
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91999502"
---
# <a name="configure-ip-firewall-for-an-azure-relay-namespace"></a>Настройка брандмауэра для IP-адресов для пространства имен Azure Relay
По умолчанию пространства имен Azure Relay доступны из Интернета при условии, что запрос поступает с действительными данными для аутентификации и авторизации. С помощью брандмауэра для IP-адресов такой доступ можно дополнительно ограничить набором или диапазоном IPv4-адресов в нотации [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Эта возможность полезна в сценариях, в которых служба Azure Relay должна быть доступна только из определенных хорошо известных сайтов. Правила брандмауэра позволяют настроить правила приема трафика, поступающего с определенных IPv4-адресов. Например, при использовании Azure Relay в сочетании [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services) можно создать **правило брандмауэра**, разрешающее трафик только с IP-адресов в локальной инфраструктуре. 


> [!IMPORTANT]
> Эта функция в настоящее время находится на стадии предварительной версии. 


## <a name="enable-ip-firewall-rules"></a>Включение правил брандмауэра для IP-адресов
Правила брандмауэра для IP-адресов применяются на уровне пространства имен. Поэтому они действуют для всех клиентских подключений по любым поддерживаемым протоколам. Любые попытки подключения с IP-адреса, который не соответствует правилу разрешения IP-адресов для пространства имен, отклоняются. В ответе клиенту правило фильтрации IP-адресов не упоминается. Правила фильтрации IP-адресов применяются по порядку, поэтому первое правило, которое соответствует IP-адресу, определяет действие (принять или отклонить).

### <a name="use-azure-portal"></a>Использование портала Azure
В этом разделе показано, как использовать портал Azure для создания правил брандмауэра для IP-адресов на уровне пространства имен. 

1. Перейдите к **пространству имен Azure Relay** на [портале Azure](https://portal.azure.com).
2. В меню слева выберите **Сеть**. Если выбрать **Все сети** в разделе **Разрешить доступ из**, пространство имен Azure Relay будет принимать подключения с любого IP-адреса. Такое значение параметра равноценно правилу, которое принимает диапазон IP-адресов 0.0.0.0/0. 

    ![На снимке экрана показана страница "сеть" с выбранным параметром "все сети".](./media/ip-firewall/all-networks-selected.png)
1. Чтобы ограничить доступ к определенным сетям и IP-адресам, выберите вариант **Выбранные сети**. В разделе **Брандмауэр** выполните следующие шаги:
    1. Выберите вариант **Добавить IP-адрес клиента**, чтобы предоставить текущему IP-адресу клиента доступ к пространству имен. 
    2. В качестве **диапазона адресов** введите определенный IPv4-адрес или диапазон IPv4-адресов в нотации CIDR. 

        ![Брандмауэр — выбран вариант "Все сети"](./media/ip-firewall/selected-networks-trusted-access-disabled.png)
3. На панели инструментов нажмите **Сохранить**, чтобы сохранить параметры. Подождите несколько минут, чтобы подтверждение появилось в уведомлениях портала.


### <a name="use-resource-manager-template"></a>Использование шаблона Resource Manager
Следующий шаблон Resource Manager позволяет добавить правило фильтрации IP-адресов в существующее пространство имен Azure Relay.

Этот шаблон принимает один параметр **ipMask**, значение которого определяет один IPv4-адрес или блок IP-адресов в нотации CIDR. Например, значение 70.37.104.0/24 в нотации CIDR представляет 256 IPv4-адресов в диапазоне от 70.37.104.0 до 70.37.104.255. Число 24 обозначает количество значимых битов префикса для адресов этого диапазона.

> [!NOTE]
> Хотя запрещающие правила отсутствуют, в шаблоне Azure Resource Manager для действия по умолчанию установлено значение **Разрешить**, которое не ограничивает подключения.
> При создании правил виртуальной сети или брандмауэров необходимо изменить значение параметра ***defaultAction***.
> 
> из
> ```json
> "defaultAction": "Allow"
> ```
> значение
> ```json
> "defaultAction": "Deny"
> ```
>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "relayNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Relay namespace"
        }
      },
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location for Namespace"
        }
      }
    },
    "variables": {
      "namespaceNetworkRuleSetName": "[concat(parameters('relayNamespaceName'), concat('/', 'default'))]"
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('relayNamespaceName')]",
        "type": "Microsoft.Relay/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
        },
        "properties": { }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.Relay/namespaces/networkrulesets",
        "dependsOn": [
          "[concat('Microsoft.Relay/namespaces/', parameters('relayNamespaceName'))]"
        ],
        "properties": {
          "ipRules": 
          [
            {
                "ipMask":"10.1.1.1",
                "action":"Allow"
            },
            {
                "ipMask":"11.0.0.0/24",
                "action":"Allow"
            }
          ],
          "virtualNetworkRules": [],
          "trustedServiceAccessEnabled": false,
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

Инструкции по развертыванию шаблона см. в статье [Развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).



## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о других возможностях, связанных с безопасностью сети, см. в [этой статье](network-security.md).


<!-- Links -->

[express-route]:  ../expressroute/expressroute-faqs.md#supported-services
