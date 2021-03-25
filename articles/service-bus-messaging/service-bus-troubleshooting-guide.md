---
title: Руководство по устранению неполадок в служебной шине Azure | Документация Майкрософт
description: Ознакомьтесь с советами и рекомендациями по устранению некоторых проблем, которые могут возникнуть при использовании служебной шины Azure.
ms.topic: article
ms.date: 03/03/2021
ms.openlocfilehash: b44587747a59acb3c0124c0a76b63de68d6d8ae7
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105031296"
---
# <a name="troubleshooting-guide-for-azure-service-bus"></a>Руководство по устранению неполадок в служебной шине Azure
В этой статье содержатся советы и рекомендации по устранению некоторых проблем, которые могут возникнуть при использовании служебной шины Azure. 

## <a name="connectivity-certificate-or-timeout-issues"></a>Проблемы с подключением, сертификатом или временем ожидания
Следующие шаги могут помочь при устранении неполадок с подключением, сертификатами и временем ожидания для всех служб в каталоге *. servicebus.windows.net. 

- Перейдите к или [wget](https://www.gnu.org/software/wget/) `https://<yournamespace>.servicebus.windows.net/` . Он помогает проверять наличие IP-фильтрации, виртуальной сети или цепочки сертификатов, которые являются общими при использовании пакета SDK для Java.

    Пример успешного сообщения:
    
    ```xml
    <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
    ```
    
    Пример сообщения об ошибке сбоя:

    ```xml
    <Error>
        <Code>400</Code>
        <Detail>
            Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
        </Detail>
    </Error>
    ```
- Выполните следующую команду, чтобы проверить, заблокирован ли порт в брандмауэре. Используются порты 443 (HTTPS), 5671 (AMQP) и 9354 (NET Messaging/SBMP). В зависимости от используемой библиотеки также используются другие порты. Ниже приведен пример команды, которая проверяет, заблокирован ли порт 5671. 

    ```powershell
    tnc <yournamespacename>.servicebus.windows.net -port 5671
    ```

    В Linux

    ```shell
    telnet <yournamespacename>.servicebus.windows.net 5671
    ```
- При наличии периодических проблем с подключением выполните следующую команду, чтобы проверить наличие пропущенных пакетов. Эта команда попытается установить 25 разных TCP-подключений каждые 1 секунду со службой. После этого можно проверить, сколько из них прошло успешное выполнение и завершилось сбоем, а также увидеть задержку подключения TCP. Это средство можно загрузить `psping` [отсюда](/sysinternals/downloads/psping).

    ```shell
    .\psping.exe -n 25 -i 1 -q <yournamespace>.servicebus.windows.net:5671 -nobanner     
    ```
    Аналогичные команды можно использовать, если вы используете другие средства, такие как `tnc` , `ping` и т. д. 
- Найдите трассировку сети, если предыдущие шаги не помогают и не анализируют их с помощью таких средств, как [Wireshark](https://www.wireshark.org/). При необходимости обратитесь в [Служба поддержки Майкрософт](https://support.microsoft.com/) . 
- Чтобы найти IP-адреса, которые нужно добавить в разрешенных для подключений, см. в разделе [какие IP-адреса нужно добавить в разрешенных](service-bus-faq.md#what-ip-addresses-do-i-need-to-add-to-allow-list). 


## <a name="issues-that-may-occur-with-service-upgradesrestarts"></a>Проблемы, которые могут возникнуть при обновлении или перезапуске службы

### <a name="symptoms"></a>Симптомы
- Запросы могут быть продолжены с немедленным регулированием.
- Может существовать перетаскивание входящих сообщений или запросов.
- Файл журнала может содержать сообщения об ошибках.
- Приложения могут быть отключены от службы в течение нескольких секунд.

### <a name="cause"></a>Причина
Обновление и перезапуск серверной службы могут вызвать эти проблемы в приложениях.

### <a name="resolution"></a>Решение
Если в коде приложения используется пакет SDK, то политика повторных попыток уже встроена и активна. Приложение будет повторно подключено без значительного влияния на приложение или рабочий процесс.

## <a name="unauthorized-access-send-claims-are-required"></a>Несанкционированный доступ: требуется отправка утверждений

### <a name="symptoms"></a>Симптомы 
Эта ошибка может возникать при попытке доступа к разделу служебной шины из Visual Studio на локальном компьютере с помощью назначенного пользователем управляемого удостоверения с разрешениями на отправку.

```bash
Service Bus Error: Unauthorized access. 'Send' claim\(s\) are required to perform this operation.
```

### <a name="cause"></a>Причина
Удостоверение не имеет разрешений на доступ к разделу служебной шины. 

### <a name="resolution"></a>Решение
Чтобы устранить эту ошибку, установите библиотеку [Microsoft. Azure. Services. AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication/) .  Дополнительные сведения см. в статье [Проверка подлинности локальной разработки](/dotnet/api/overview/azure/service-to-service-authentication#local-development-authentication). 

Сведения о назначении разрешений ролям см. в статье [Проверка подлинности управляемого удостоверения с Azure Active Directory для доступа к ресурсам служебной шины Azure](service-bus-managed-service-identity.md).

## <a name="service-bus-exception-put-token-failed"></a>Исключение служебной шины: ошибка при размещении токена

### <a name="symptoms"></a>Симптомы
При попытке отправить более 1000 сообщений с помощью одного и того же подключения служебной шины вы получите следующее сообщение об ошибке: 

`Microsoft.Azure.ServiceBus.ServiceBusException: Put token failed. status-code: 403, status-description: The maximum number of '1000' tokens per connection has been reached.` 

### <a name="cause"></a>Причина
Существует ограничение на количество токенов, используемых для отправки и получения сообщений с помощью одного соединения с пространством имен служебной шины. Это 1000. 

### <a name="resolution"></a>Решение
Откройте новое подключение к пространству имен служебной шины, чтобы отправить дополнительные сообщения.

## <a name="adding-virtual-network-rule-using-powershell-fails"></a>Не удается добавить правило виртуальной сети с помощью PowerShell

### <a name="symptoms"></a>Симптомы
Вы настроили две подсети из одной виртуальной сети в правиле виртуальной сети. При попытке удалить одну подсеть с помощью командлета [Remove-азсервицебусвиртуалнетворкруле](/powershell/module/az.servicebus/remove-azservicebusvirtualnetworkrule) она не удаляет подсеть из правила виртуальной сети. 

```azurepowershell-interactive
Remove-AzServiceBusVirtualNetworkRule -ResourceGroupName $resourceGroupName -Namespace $serviceBusName -SubnetId $subnetId
```

### <a name="cause"></a>Причина
Идентификатор Azure Resource Manager, указанный для подсети, может быть недопустимым. Это может произойти, если виртуальная сеть находится в другой группе ресурсов, отличной от той, которая имеет пространство имен служебной шины. Если вы не указали группу ресурсов виртуальной сети явным образом, команда CLI конструирует идентификатор Azure Resource Manager с помощью группы ресурсов пространства имен служебной шины. Поэтому он не может удалить подсеть из правила сети. 

### <a name="resolution"></a>Решение
Укажите полный идентификатор Azure Resource Manager подсети, включающий имя группы ресурсов, в которой находится виртуальная сеть. Пример:

```azurepowershell-interactive
Remove-AzServiceBusVirtualNetworkRule -ResourceGroupName myRG -Namespace myNamespace -SubnetId "/subscriptions/SubscriptionId/resourcegroups/ResourceGroup/myOtherRG/providers/Microsoft.Network/virtualNetworks/myVNet/subnets/mySubnet"
```

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи: 

- [Azure Resource Manager исключения](service-bus-resource-manager-exceptions.md). В нем перечислены исключения, созданные при взаимодействии с служебной шиной Azure с помощью Azure Resource Manager (через шаблоны или прямые вызовы).
- [Исключения обмена сообщениями](service-bus-messaging-exceptions.md). Он предоставляет список исключений, созданных платформа .NET Framework для служебной шины Azure.