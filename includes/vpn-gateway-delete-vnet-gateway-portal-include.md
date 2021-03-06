---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/10/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 91dc6ba0bcd455aefe9de2efdff2feb20938152d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100376397"
---
### <a name="step-1-navigate-to-the-virtual-network-gateway"></a>Шаг 1. Переход к шлюзу виртуальной сети

1. На [портале Azure](https://portal.azure.com) выберите **Все ресурсы**. 
2. Чтобы открыть страницу шлюза виртуальной сети, перейдите к шлюзу виртуальной сети и щелкните его, чтобы выбрать. 

### <a name="step-2-delete-connections"></a>Шаг 2. Удаление подключений

1. На странице шлюза виртуальной сети щелкните **Подключения**, чтобы просмотреть все подключения к шлюзу.
2. В строке с именем подключения щелкните **…** и в раскрывающемся списке выберите **Удалить**.
3. Нажмите кнопку **Да**, чтобы подтвердить удаление подключения. Если имеется несколько подключений, удалите их все.

### <a name="step-3-delete-the-virtual-network-gateway"></a>Шаг 3. Удаление шлюза виртуальной сети

Учтите: если кроме конфигурации типа "сеть — сеть" в этой виртуальной сети есть конфигурация типа "точка — сеть", удаление шлюза виртуальной сети приведет к автоматическому отключению всех клиентов P2S без предупреждения.

1. На странице шлюза виртуальной сети щелкните **Обзор**.
2. На странице **Обзор** щелкните **Удалить**, чтобы удалить этот шлюз.
