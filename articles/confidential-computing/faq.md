---
title: Вопросы и ответы о конфиденциальных вычислениях Azure
description: Получите ответы на часто задаваемые вопросы о конфиденциальных вычислениях Azure.
author: JBCook
ms.topic: troubleshooting
ms.workload: infrastructure
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.date: 4/17/2020
ms.author: jencook
ms.openlocfilehash: a5ecd3827bbdc12b098684f1feda2df652f11940
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102551918"
---
# <a name="frequently-asked-questions-for-azure-confidential-computing"></a>Вопросы и ответы о конфиденциальных вычислениях Azure

В этой статье содержатся ответы на некоторые наиболее распространенные вопросы о выполнении [рабочих нагрузок для конфиденциальных вычислений на виртуальных машинах Azure](overview.md).

Если эта статья не помогла устранить вашу проблему с Azure, посетите [форумы по Azure на сайтах MSDN и Stack Overflow](https://azure.microsoft.com/support/forums/). Описание своей проблемы можно опубликовать на этих форумах или [в Twitter (@AzureSupport)](https://twitter.com/AzureSupport). Можно также отправить запрос в службу поддержки Azure. Чтобы отправить такой запрос, на странице [поддержки Azure](https://azure.microsoft.com/support/options/) щелкните "Получить поддержку".

## <a name="confidential-computing-virtual-machines"></a>Виртуальные машины конфиденциальных вычислений <a id="vm-faq"></a>

**Как развернуть виртуальные машины серии DCsv2 в Azure?**

Вот некоторые способы развертывания виртуальных машин DCsv2:
   - с помощью [шаблона Azure Resource Manager](../virtual-machines/windows/template-description.md);
   - на [портале Azure](https://portal.azure.com/#create/hub);
   - с помощью шаблона решения [Azure confidential computing (Virtual Machine)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview) (Конфиденциальные вычисления Azure (виртуальная машина) из Marketplace. Шаблон решения Marketplace предназначен для использования в конкретных поддерживаемых сценариях (с определенными настройками регионов, образов, уровней доступности, возможностей шифрования дисков). 

**Конфиденциальные вычисления Azure поддерживаются для всех образов ОС?**

Нет. Виртуальные машины могут быть развернуты только на компьютерах 2-го поколения с Ubuntu Server 18.04, Ubuntu Server 16.04, Windows Server 2019 Datacenter и Windows Server 2016 Datacenter. Узнайте больше о виртуальных машинах 2-го поколения для [Linux](../virtual-machines/generation-2.md) и [Windows](../virtual-machines/generation-2.md).

**Виртуальные машины DCsv2 неактивны (выделены серым цветом) на портале, и я не могу выбрать нужную.**

В соответствии со сведениями во всплывающем информационном окне, появляющемся рядом с виртуальной машиной, можно предпринять различные действия.
   -    **UnsupportedGeneration**. Измените поколение образа виртуальной машины на "Gen2".
   -    **NotAvailableForSubscription**. Регион пока недоступен для вашей подписки. Выберите доступный регион.
   -    **InsufficientQuota**. [Создайте запрос в службу поддержки для увеличения квоты](../azure-portal/supportability/per-vm-quota-requests.md). У бесплатных пробных подписок нет квоты на виртуальные машины конфиденциальных вычислений. 

**Не удается найти виртуальные машины DCsv2 в средстве выбора размера на портале.**

Убедитесь, что выбран [доступный регион](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines). Также убедитесь, что в средстве выбора размера выбрано "Очистить все фильтры". 

**Можно ли включить ускоренную сеть с помощью конфиденциальных вычислений в Azure?**

 Нет. Ускоренная сеть не поддерживается на виртуальных машинах DC-Series и DCsv2-Series. Ускоренная сеть не может быть включена для любых конфиденциальных вычислительных ресурсов развертывания виртуальных машин или развертывания кластера службы Azure Kubernetes, работающих на конфиденциальных вычислениях.

**Можно ли использовать выделенный узел Azure на этих компьютерах?**

Да. Выделенный узел Azure поддерживает виртуальные машины серии DCsv2. Выделенный узел Azure предоставляет физический сервер с одним клиентом для выполнения виртуальных машин. Обычно пользователи используют выделенный узел Azure для обеспечения соответствия требованиям физической безопасности, целостности данных и мониторинга. 

**Выводится следующее сообщение об ошибке развертывания шаблона Azure Resource Manager: "Operation could not be completed as it results in exceeding approved standard DcsV2 Family Cores Quota" (Операция не может быть выполнена, так как она приводит к превышению квоты на ядра для семейства Standard DcsV2)** .

[Создайте запрос в службу поддержки для увеличения квоты](../azure-portal/supportability/per-vm-quota-requests.md). У бесплатных пробных подписок нет квоты на виртуальные машины конфиденциальных вычислений. 

**В чем разница между виртуальными машинами серий DCsv2 и DC?**

Виртуальные машины серии DC работают на старых 6-ядерных процессорах Intel с Intel SGX и имеют меньший объем общей памяти, меньший объем памяти кэша страниц анклава (Enclave Page Cache, EPC), доступны только в двух регионах (Восточная часть США и Западная Европа с размерами Standard_DC2s и Standard_DC4s). Выпуск этих виртуальных машин в общий доступ не планируется, и их не рекомендуется использовать в рабочей среде. Чтобы развернуть эти виртуальные машины, используйте экземпляр [виртуальной машины конфиденциальных вычислений серии DC [предварительная версия]](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.confidentialcompute?tab=Overview) в Marketplace.

**Виртуальные машины DCsv2 доступны повсеместно?**

Нет. Сейчас эти виртуальные машины доступны только в некоторых регионах. Список последних доступных регионов приведен на [странице продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines). 

**На этих компьютерах ОТКЛЮЧЕНа технология Hyper-Threading?**

Технология Hyper-Threading отключена для всех кластеров с конфиденциальными вычислениями в Azure.

**Как установить пакет SDK Open Enclave на виртуальных машинах DCsv2?**
   
Инструкции по установке пакета SDK OE на локальном компьютере или в Azure см. на странице [GitHub по пакету SDK Open Enclave](https://github.com/openenclave/openenclave).
     
Кроме того, на странице GitHub по пакету SDK Open Enclave можно найти инструкции по установке в конкретных ОС.
   - [Установка пакета SDK OE в Windows](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Windows.md)
   - [Установка пакета SDK OE в Ubuntu 18.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_18.04.md)
   - [Установка пакета OE SDK в Ubuntu 16.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_16.04.md)