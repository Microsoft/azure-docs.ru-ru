---
title: План подготовки к работе с центром безопасности Azure | Документация Майкрософт
description: Этот документ содержит план подготовки для эффективного использования центра безопасности Azure.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: fece670cc-df70-445d-9773-b32cbaba8d4a
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/03/2018
ms.author: memildin
ms.openlocfilehash: e4f905b7a08564eadb20d65d651c137ef3741376
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102100327"
---
# <a name="azure-security-center-readiness-roadmap"></a>План готовности центра безопасности Azure
Этот документ содержит план подготовки, который поможет вам приступить к работе с центром безопасности Azure.

## <a name="understanding-security-center"></a>Основные сведения о центре безопасности
Центр безопасности Azure предоставляет возможности унифицированного управления безопасностью и расширенной защиты от угроз для рабочих нагрузок в Azure, в локальной среде и в других облаках. 

Используйте следующие ресурсы для начала работы с центром безопасности.

Статьи
- [Введение в Центр безопасности Azure](security-center-introduction.md)
- [Краткое руководство по центру безопасности Azure](security-center-get-started.md)

Видео
- [Introduction to Azure Security Center](https://azure.microsoft.com/resources/videos/introduction-to-azure-security-center/) (Ознакомительное видео о центре безопасности Azure)
- [Overview of Security Center Prevention, Detection and Response Capabilities](https://azure.microsoft.com/resources/videos/azurecon-2015-new-azure-security-center-helps-you-prevent-detect-and-respond-to-threats/) (Видеообзор возможностей предотвращения, обнаружения и ответных действий центра безопасности)

## <a name="planning-and-operations"></a>Планирование и эксплуатация

Чтобы воспользоваться всеми преимуществами центра безопасности, важно понять, каким образом разные люди или группы в организации используют эту службу. Это позволит обеспечить соответствие требованиям к безопасной эксплуатации, мониторингу, управлению и реагированию на инциденты.

Следующие ресурсы пригодятся вам в процессе планирования и эксплуатации.

- [Руководство по планированию использования центра безопасности Azure и работе в нем](security-center-planning-and-operations-guide.md)


### <a name="onboarding-computers-to-security-center"></a>Подключение компьютеров к центру безопасности
Центр безопасности автоматически обнаруживает все подписки или рабочие области Azure, не защищенные защитником Azure. Сюда входят подписки Azure, использующие бесплатный центр безопасности и рабочие области, на которых не включено решение для обеспечения безопасности.

Следующие ресурсы пригодятся вам в процессе подключения.

- [Подключение компьютеров, которые не относятся к Azure](quickstart-onboard-machines.md)
- [Azure Security Center Hybrid - Overview](https://youtu.be/NMa4L_M597k) (Видеообзор гибридных ресурсов центра безопасности Azure)

## <a name="mitigating-security-issues-using-security-center"></a>Устранение проблем безопасности с помощью центра безопасности
Центр безопасности автоматически собирает, анализирует и объединяет данные журналов, поступающие от ресурсов Azure, сети и подключенных решений партнеров, таких как брандмауэры и решения для защиты конечных точек, для выявления реальных угроз и сокращения ложных срабатываний.

Следующие ресурсы помогут вам управлять предупреждениями безопасности и защищать ресурсы.

Статьи    
- [Защита сети в центре безопасности Azure.](./security-center-network-recommendations.md)
- [Защита службы SQL Azure и данных в центре безопасности Azure](./security-center-remediate-recommendations.md)


Видеоролик    
- [Устранение проблем безопасности с помощью центра безопасности Azure](https://channel9.msdn.com/Blogs/Azure-Security-Videos/Mitigating-Security-Issues-using-Azure-Security-Center)

### <a name="security-center-for-incident-response"></a>Реагирование на инциденты с помощью центра безопасности
Чтобы минимизировать затраты и риски, важно составить план реагирования до обнаружения атаки. Центр безопасности Azure можно использовать на разных этапах реагирования на инциденты.

Ознакомьтесь со следующими ресурсами, что узнать, как центр безопасности можно интегрировать в процесс реагирования на инциденты.

Видео    
* [Azure Security Center in Incident Response](https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Security-Center-in-Incident-Response) (Реагирование на инциденты с помощью центра безопасности)
* [Respond quickly to threats with next-generation security operation, and investigation](https://youtu.be/e8iFCz5RM4g) (Быстрая реакция на угрозы с помощью современных операций и анализа безопасности)

Статьи    
* [Использование центра безопасности Azure для реагирования на инциденты](./tutorial-security-incident.md)
* [Использование службы автоматизации для реагирования на триггеры центра безопасности](workflow-automation.md)

## <a name="advanced-cloud-defense"></a>Расширенная защита облака

Для виртуальных машин Azure можно использовать дополнительные возможности защиты облака в центре безопасности. Эти возможности включают только JIT-доступ к виртуальной машине и адаптивные элементы управления приложением.

В следующих ресурсах объясняется, как использовать эти возможности в центре безопасности.

Видео    
* [Azure Security Center — Just-in-Time VM Access](https://youtu.be/UOQb2FcdQnU) (Центр безопасности Azure — JIT-доступ к виртуальной машине)
* [Центр безопасности Azure — адаптивные элементы управления приложениями](https://youtu.be/wWWekI1Y9ck)

Статьи    
* [Manage virtual machine access using Just-in-time](./security-center-just-in-time.md) (Управление доступом к виртуальным машинам с помощью JIT)
* [Адаптивные элементы управления приложениями в центре безопасности Azure](./security-center-adaptive-application.md)

## <a name="hands-on-activities"></a>Практические упражнения

* [Security Center hands-on lab](https://www.microsoft.com/handsonlabs/SelfPacedLabs/?storyGuid=78871abf-6f35-4aa0-840f-d801f5cdbd72) (Практические занятия по работе з центром безопасности)
* [Web Application Firewall (WAF) recommendation playbook in Security Center](https://gallery.technet.microsoft.com/ASC-Playbook-Protect-38bd47ff) (Сборник рекомендаций для брандмауэра веб-приложений в центре безопасности)
* [Azure Security Center Playbook: Security Alerts](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046) (Сборник схем для центра безопасности Azure: оповещения безопасности)

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Документация по центру безопасности](./index.yml)
* [Security Center REST API Documentation Page](/previous-versions/azure/reference/mt704034(v=azure.100)) (Документация по REST API центра безопасности)
* [Azure Security Center frequently asked questions (FAQ)](./faq-general.md) (Центр безопасности Azure: часто задаваемые вопросы)
* [Цены на центр безопасности](https://azure.microsoft.com/pricing/details/security-center/)
* [Рекомендации по обеспечению безопасности удостоверений](../security/fundamentals/identity-management-best-practices.md)
* [Рекомендации по обеспечению сетевой безопасности](../security/fundamentals/network-best-practices.md)
* [Рекомендации для PaaS](../security/fundamentals/paas-deployments.md)
* [Соответствие](https://www.microsoft.com/trustcenter/compliance/due-diligence-checklist)
* [Клиенты службы анализа журналов теперь могут использовать центр безопасности Azure для защиты своих рабочих нагрузок в гибридном облаке](/archive/blogs/msoms/oms-customers-can-now-use-azure-security-center-to-protect-their-hybrid-cloud-workloads)

## <a name="community-resources"></a>Ресурсы сообщества

* [UserVoice центра безопасности](https://feedback.azure.com/forums/347535-azure-security-center)
* [Вопрос.&страницу для центра безопасности](/answers/topics/azure-security-center.html)