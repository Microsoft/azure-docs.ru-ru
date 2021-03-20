---
title: включить файл
description: включить файл
services: service-bus-messaging
author: clemensv
ms.service: service-bus-messaging
ms.topic: include
ms.date: 11/24/2020
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: ca483d0b71bde945a7e46da785dd6a76b3a8f177
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98693411"
---
Параметр протокола AMQP-over-WebSockets выполняется через порт TCP 443, как и HTTP/REST API, но в противном случае он функционально идентичен обычному AMQP. Этот параметр имеет несколько более высокие начальные задержки при подключении из-за дополнительных обращений к подтверждению и немного больше ресурсов, чем компромисс для совместного использования HTTPS-порта. Если выбран этот режим, для обмена данными достаточно TCP-порта 443. Следующие параметры позволяют выбрать режим AMQP или AMQP WebSockets.

| Язык | Параметр   |
| -------- | ----- |
| .NET     | Свойство [сервицебусконнектион. TransportType](/dotnet/api/microsoft.azure.servicebus.servicebusconnection.transporttype) с [TransportType. AMQP](/dotnet/api/microsoft.azure.servicebus.transporttype) или [TransportType. амкпвебсоккетс](/dotnet/api/microsoft.azure.servicebus.transporttype) |
| Java     | [com. Microsoft. Azure. servicebus. ClientSettings](/java/api/com.microsoft.azure.servicebus.clientsettings.clientsettings) с [com. Microsoft. Azure. servicebus. примитивы. TransportType. AMQP](/java/api/com.microsoft.azure.servicebus.primitives.transporttype) или [com.Microsoft.Azure.servicebus.Primitives.TransportType.AMQP_WEB_SOCKETS](/java/api/com.microsoft.azure.servicebus.primitives.transporttype) |
| Узел  | [Сервицебусклиентоптионс](/javascript/api/@azure/service-bus/servicebusclientoptions) имеет `webSocket` аргумент конструктора. |
| Python | [ServiceBusClient.transport_type](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-servicebus/latest/azure.servicebus.html#azure.servicebus.ServiceBusClient) с [TransportType. AMQP](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-servicebus/latest/azure.servicebus.html#azure.servicebus.TransportType) или [TransportType. амкповервебсоккет](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-servicebus/latest/azure.servicebus.html#azure.servicebus.TransportType) |