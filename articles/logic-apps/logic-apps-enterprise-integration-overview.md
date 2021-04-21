---
title: Интеграция с корпоративными решениями B2B
description: Сведения о создании автоматизированных рабочих процессов B2B для корпоративной интеграции с помощью Azure Logic Apps и пакета интеграции Enterprise
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, logicappspm
ms.topic: overview
ms.date: 03/24/2021
ms.openlocfilehash: 09d253aae4c054db5efdc252f62986044e1d366b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771863"
---
# <a name="b2b-enterprise-integration-solutions-with-azure-logic-apps-and-enterprise-integration-pack"></a>Решения для корпоративной интеграции B2B с Azure Logic Apps и пакетом интеграции Enterprise

Для выполнения решений "бизнес-бизнес" (B2B) и беспрепятственного взаимодействия между организациями можно создавать автоматизированные масштабируемые рабочие процессы интеграции предприятия, используя пакет интеграции Enterprise (EIP) с [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Хотя организации используют разные протоколы и форматы, они могут обмениваться сообщениями в электронном формате. EIP преобразует различные форматы в формат, который могут обрабатывать системы вашей организации, и поддерживает стандартные отраслевые протоколы, включая [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md), [X12](logic-apps-enterprise-integration-x12.md) и [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md). Кроме того, вы можете повысить уровень безопасности для сообщений, используя и шифрование, и цифровые подписи. EIP поддерживает эти [соединители интеграции предприятия](../connectors/managed.md#enterprise-connectors) и следующие отраслевые стандарты:

* электронный обмен данными (EDI);
* интеграция корпоративных приложений (EAI).

Если вы знакомы с Microsoft BizTalk Server или службами BizTalk Azure, то EIP следует таким же концепциям, что делает эти компоненты простыми в использовании. Однако существенное отличие состоит в том, что EIP архитектурно основан на "учетных записях интеграции", которые упрощают хранение и управление артефактами, используемыми в коммуникациях B2B. Эти учетные записи представляют собой облачные контейнеры, в которых хранятся все ваши артефакты, такие как партнеры, соглашения, схемы, карты и сертификаты. 

## <a name="why-use-the-enterprise-integration-pack"></a>Для чего используется пакет интеграции Enterprise?

* С помощью EIP можно хранить все артефакты в одном месте — в своей учетной записи интеграции.

* Вы можете создавать рабочие процессы B2B и интегрировать их со сторонними приложениями "программного обеспечения как услуги" (SaaS), локальными приложениями и пользовательскими приложениями, используя Azure Logic Apps и соединители.

* Вы можете создать пользовательский код для приложения логики с помощью функций Azure.

## <a name="how-do-i-get-started"></a>С чего начать?

Прежде чем приступить к созданию рабочих процессов приложения логики B2B с помощью EIP, вам понадобятся следующие элементы:

* Подписка Azure. Если у вас еще нет подписки Azure, [зарегистрируйтесь для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/).

* [Учетная запись интеграции](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) с артефактами,которые вы хотите использовать.

* Для создания карт и схем можно использовать [Инструменты корпоративной интеграции Microsoft Azure Logic Apps для Visual Studio 2015 2.0](https://aka.ms/vsmapsandschemas) и Visual Studio 2015.

После создания учетной записи интеграции и добавления артефактов, можно приступить к созданию рабочих процессов B2B с помощью этих артефактов, создав приложение логики на портале Azure. Если вы новичок в приложениях логики, попробуйте [создать базовое приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md). Для работы с этими артефактами, сначала необходимо связать учетную запись интеграции с приложением логики. После этого приложение логики будет иметь доступ к учетной записи интеграции. Вы можете создавать, развертывать приложения логики, а также управлять ими с помощью Visual Studio или [PowerShell](/powershell/module/az.logicapp).

Ниже приведен список действий, необходимых для начала создания приложения логики B2B:

![Предварительные условия для создания приложений логики B2B](./media/logic-apps-enterprise-integration-overview/overview.png)  

## <a name="try-now"></a>Попробовать

[Развертывание полнофункционального примера приложения логики, которое отправляет и получает сообщения AS2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-as2-send-receive)

## <a name="next-steps"></a>Дальнейшие шаги

* [Создание торговых партнеров](logic-apps-enterprise-integration-partners.md)
* [Создание соглашений](../logic-apps/logic-apps-enterprise-integration-agreements.md)
* [Добавление схем](logic-apps-enterprise-integration-schemas.md)
* [Добавление карт](../logic-apps/logic-apps-enterprise-integration-maps.md)
* [Миграция из служб BizTalk](../logic-apps/logic-apps-move-from-mabs.md)
