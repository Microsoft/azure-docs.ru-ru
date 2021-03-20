---
title: Создание пользовательской пробы с помощью классической модели развертывания (шлюз приложений Azure)
description: Узнайте, как создавать пользовательские проверки для шлюза приложений с помощью PowerShell в классической модели развертывания.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 13441899eeb5ca2b7c60977ab2858fe40a398d1a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397864"
---
# <a name="create-a-custom-probe-for-azure-application-gateway-classic-by-using-powershell"></a>Создание пользовательской проверки для шлюза приложений (классического) Azure с помощью PowerShell

> [!div class="op_single_selector"]
> * [Портал Azure](application-gateway-create-probe-portal.md)
> * [PowerShell и диспетчер ресурсов Azure](application-gateway-create-probe-ps.md)
> * [Классическая оболочка Azure](application-gateway-create-probe-classic-ps.md)

Следуя инструкциям этой статьи вы добавите пользовательскую пробу в имеющийся шлюз приложений с помощью PowerShell. Пользовательские пробы полезны в приложениях с конкретной страницей проверки работоспособности или приложениях, не предоставляющих успешный ответ веб-приложению по умолчанию.

> [!IMPORTANT]
> В Azure предлагаются две модели развертывания для создания ресурсов и работы с ними: [модель развертывания с помощью Resource Manager и классическая модель](../azure-resource-manager/management/deployment-models.md). В этой статье рассматривается использование классической модели развертывания. Для большинства новых развертываний Майкрософт рекомендует использовать модель диспетчера ресурсов. Узнайте, как [выполнить эти действия с помощью модели Resource Manager](application-gateway-create-probe-ps.md).

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-an-application-gateway"></a>Создание шлюза приложений

Создание шлюза приложений:

1. Создание ресурса шлюза приложений.
2. Создайте XML-файл конфигурации или объект конфигурации.
3. Применить конфигурацию к созданному ресурсу шлюза приложений.

### <a name="create-an-application-gateway-resource-with-a-custom-probe"></a>Создание ресурса шлюза приложений с пользовательской пробой

Для создания шлюза используйте командлет `New-AzureApplicationGateway`, подставив в него свои значения. Выставление счетов для шлюза начинается не на данном этапе, а позднее, после успешного запуска шлюза.

В следующем примере создается шлюз приложений с использованием виртуальной сети testvnet1 и подсети subnet-1.

```powershell
New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")
```

Чтобы проверить создание шлюза, используйте командлет `Get-AzureApplicationGateway`.

```powershell
Get-AzureApplicationGateway AppGwTest
```

> [!NOTE]
> Значение по умолчанию для *InstanceCount* — 2, максимальное значение — 10. Значение *GatewaySize* (Размер шлюза) по умолчанию — Medium (Средний). Можно выбрать значения Small, Medium или Large.
> 
> 

*Параметры virtualips* и *dnsName* отображаются пустыми, так как шлюз еще не запущен. Эти значения будут определены после запуска шлюза.

### <a name="configure-an-application-gateway-by-using-xml"></a>Настройка шлюза приложений с помощью XML-файла

В следующем примере все параметры шлюза приложений настраиваются и применяются к ресурсу шлюза приложений при помощи XML-файла.  

Скопируйте следующий текст в Блокнот.

```xml
<ApplicationGatewayConfiguration xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
<FrontendIPConfigurations>
    <FrontendIPConfiguration>
        <Name>fip1</Name>
        <Type>Private</Type>
    </FrontendIPConfiguration>
</FrontendIPConfigurations>
<FrontendPorts>
    <FrontendPort>
        <Name>port1</Name>
        <Port>80</Port>
    </FrontendPort>
</FrontendPorts>
<Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
    </Probes>
    <BackendAddressPools>
    <BackendAddressPool>
        <Name>pool1</Name>
        <IPAddresses>
            <IPAddress>1.1.1.1</IPAddress>
            <IPAddress>2.2.2.2</IPAddress>
        </IPAddresses>
    </BackendAddressPool>
</BackendAddressPools>
<BackendHttpSettingsList>
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
</BackendHttpSettingsList>
<HttpListeners>
    <HttpListener>
        <Name>listener1</Name>
        <FrontendIP>fip1</FrontendIP>
    <FrontendPort>port1</FrontendPort>
        <Protocol>Http</Protocol>
    </HttpListener>
</HttpListeners>
<HttpLoadBalancingRules>
    <HttpLoadBalancingRule>
        <Name>lbrule1</Name>
        <Type>basic</Type>
        <BackendHttpSettings>setting1</BackendHttpSettings>
        <Listener>listener1</Listener>
        <BackendAddressPool>pool1</BackendAddressPool>
    </HttpLoadBalancingRule>
</HttpLoadBalancingRules>
</ApplicationGatewayConfiguration>
```

Измените значения в скобках для элементов конфигурации. Сохраните файл с расширением XML.

В приведенном ниже примере показано, как использовать файл конфигурации, чтобы настроить шлюз приложений для балансировки нагрузки HTTP-трафика на общем порте 80 и направления сетевого трафика на внутренний порт 80 между двумя IP-адресами с помощью пользовательской проверки.

> [!IMPORTANT]
> В элементе протокола HTTP или HTTPS учитывается регистр.

\<Probe\>Для настройки пользовательских зондов добавляется новый элемент конфигурации.

Используются следующие параметры конфигурации:

|Параметр|Описание|
|---|---|
|**имя**; |Имя пользовательской пробы. |
| **Протокол** | Используемый протокол (возможные значения: HTTP или HTTPS).|
| **Host** и **Path** | Полный путь URL-адреса, который вызывается шлюзом приложений для определения работоспособности экземпляра. Например, если у вас есть веб-сайт http: \/ /contoso.com/, то пользовательская пробная проверка может быть настроена на "http: \/ /contoso.com/Path/custompath.htm" для проверки на наличие успешного HTTP-ответа.|
| **Интервал** | Задает интервал между пробами в секундах.|
| **Timeout** | Определяет время ожидания для проверки ответа HTTP.|
| **UnhealthyThreshold** | Количество неудачных ответов HTTP, по достижении которого серверный экземпляр считается *неработоспособным*.|

Имя проверки указывается в конфигурации \<BackendHttpSettings\> для назначения пула внутренних серверов, который использует параметры пользовательской проверки.

## <a name="add-a-custom-probe-to-an-existing-application-gateway"></a>Добавление пользовательской пробы в имеющийся шлюз приложений

Изменение текущей конфигурации шлюза приложений состоит из трех шагов: получение XML-файла конфигурации, внесение изменений для пользовательской проверки и настройка шлюза приложений с использованием новых параметров XML.

1. Получите XML-файл с помощью командлета `Get-AzureApplicationGatewayConfig`. Он экспортирует XML-файл конфигурации, который нужно изменить, чтобы добавить параметры пробы.

   ```powershell
   Get-AzureApplicationGatewayConfig -Name "<application gateway name>" -Exporttofile "<path to file>"
   ```

1. Откройте XML-файл в текстовом редакторе. Добавьте раздел `<probe>` после `<frontendport>`.

   ```xml
   <Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
   </Probes>
   ```

   В разделе backendHttpSettings XML-файла добавьте имя пробы, как показано в следующем примере.

   ```xml
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
   ```

   Сохраните XML-файл.

1. Обновите конфигурацию шлюза приложения с помощью нового XML-файла, используя командлет `Set-AzureApplicationGatewayConfig`. Он обновит шлюз приложений с учетом новой конфигурации.

```powershell
Set-AzureApplicationGatewayConfig -Name "<application gateway name>" -Configfile "<path to file>"
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите настроить протокол TLS, который ранее назывался разгрузкой SSL (SSL), см. раздел [Настройка шлюза приложений для разгрузки TLS](./tutorial-ssl-powershell.md).

Указания по настройке шлюза приложений для использования с внутренним балансировщиком нагрузки см. в статье [Создание шлюза приложений с внутренней подсистемой балансировщика нагрузки (ILB)](./application-gateway-ilb-arm.md).