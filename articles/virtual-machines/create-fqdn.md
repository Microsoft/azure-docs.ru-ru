---
title: Создание полного доменного имени для виртуальной машины в портал Azure
description: Узнайте, как создать полное доменное имя (FQDN) для виртуальной машины в портал Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 1/12/2020
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: aadf4000da4ab88a3c3b4dee37d2179eb49d39d7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98132070"
---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-linux-vm"></a>Создание полного доменного имени на портале Azure для виртуальной машины Linux

При создании виртуальной машины на [портале Azure](https://portal.azure.com) для нее автоматически создается ресурс общедоступного IP-адреса. Этот IP-адрес используется для удаленного доступа к данной виртуальной машине. Несмотря на то что портал не создает [полное доменное имя](https://en.wikipedia.org/wiki/Fully_qualified_domain_name), его можно добавить после создания виртуальной машины. В этой статье показан процесс создания DNS-имени или полного доменного имени. 

## <a name="create-a-fqdn"></a>Создание полного доменного имени
Для работы с руководством требуется виртуальная машина. При необходимости на портале можно создать виртуальную машину [Linux](./linux/quick-create-portal.md) или [Windows](./windows/quick-create-portal.md) . Когда виртуальная машина будет готова, выполните следующие действия:


1. Выберите виртуальную машину на портале. 
1. В меню слева выберите **Конфигурация** .
1. В поле **Метка DNS-имени** введите префикс, который вы хотите использовать.
1. Щелкните **Сохранить** в верхней части страницы.
1. Вернитесь в колонку обзор виртуальной машины, выбрав **Обзор** в меню слева. 
1. Убедитесь, что *DNS-имя* отображается правильно. 

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете управлять DNS с помощью [зон Azure DNS](../dns/dns-getstarted-portal.md).

