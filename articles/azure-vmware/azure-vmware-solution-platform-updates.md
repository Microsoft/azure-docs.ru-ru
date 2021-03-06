---
title: Обновления платформы для решения VMware для Azure
description: Сведения об обновлениях платформы в решении VMware для Azure.
ms.topic: reference
ms.date: 03/24/2021
ms.openlocfilehash: da6317d49edd3f40e1a8f2518f91fe353bbae285
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105045218"
---
# <a name="platform-updates-for-azure-vmware-solution"></a>Обновления платформы для решения VMware для Azure

Решение Azure VMware будет применять важные обновления, начиная с 2021 марта. Вы получите уведомление с помощью службы работоспособности служб Azure, которая включает временную шкалу обслуживания. Дополнительные сведения см. в статье [обновления частного облака для решения VMware для Azure и](concepts-upgrades.md)обновление.

## <a name="march-24-2021"></a>24 марта 2021 г.
Все новые частные облака решения Azure VMware развертываются с помощью VMware vCenter версии 6.7 U3l и НСКС-T версии 3.1.1. Все существующие частные облака будут обновлены и обновлены **до июня 2021** до указанных выше выпусков.

Вы получите сообщение электронной почты с плановой датой и временем обслуживания. Вы можете изменить расписание обновления. В письме также содержатся сведения о обновленном компоненте, его возможностях в рабочих нагрузках, доступе к частному облаку и других службах Azure.  Через час до обновления вы получите уведомление, а затем снова после его завершения.

## <a name="march-15-2021"></a>15 марта 2021 г. 

- Служба решения Azure VMware будет выполнять обслуживание **до 19 марта 2021,** чтобы обновить сервер vCenter в частном облаке, чтобы VCenter Server 6,7 обновление 3l версии.

- В это время VMware vCenter будет недоступен.  Таким образом, вы не сможете управлять виртуальными машинами ("отменить", "запустить", "создать", "Удалить") или "масштабировать частное облако" (Добавление и удаление серверов и кластеров). Однако высокая доступность VMware (HA) будет продолжать работать для защиты существующих виртуальных машин. 
 
Дополнительные сведения об этой версии vCenter см. в разделе [VMware vCenter Server 6,7 обновление 3l. заметки о выпуске](https://docs.vmware.com/en/VMware-vSphere/6.7/rn/vsphere-vcenter-server-67u3l-release-notes.html).

## <a name="march-4-2021"></a>4 марта 2021 г.

- Решение Azure VMware применит [VMware ESXi 6,7, Patch Release ESXi670-202011002](https://docs.vmware.com/en/VMware-vSphere/6.7/rn/esxi670-202011002.html) к существующим частным версиям **до 15 марта 2021**.

- Документированные обходные пути для стека vSphere, согласно [вмса-2021-0002](https://www.vmware.com/security/advisories/VMSA-2021-0002.html), будут также применены **до 15 марта 2021**.

>[!NOTE]
>Это не нарушает работу и не влияет на службы и рабочие нагрузки Azure VMware. Во время обслуживания различные оповещения VMware, такие как _потеря сетевого подключения к двпортс_ и _потеря избыточности канала исходящей связи в двпортс_, появляются в vCenter и очищаются автоматически по мере выполнения обслуживания.

## <a name="post-update"></a>После обновления
После завершения появится новая версия компонентов VMware. Если вы заметили проблемы или можете получить какие-либо вопросы, обратитесь в службу поддержки, отправив запрос на поддержку.