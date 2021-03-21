---
title: Диспетчер фортаникс конфиденциальных вычислений в управляемом приложении Azure
description: Узнайте, как развернуть Фортаникс конфиденциальный диспетчер вычислительных систем (CCM) в управляемом приложении на портал Azure.
author: JBCook
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.workload: infrastructure
ms.topic: how-to
ms.date: 02/03/2021
ms.author: jencook
ms.openlocfilehash: 757ce9b7502316bbc8a5b8f27ba672048b7bbace
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563427"
---
# <a name="fortanix-confidential-computing-manager-in-an-azure-managed-application"></a>Диспетчер фортаникс конфиденциальных вычислений в управляемом приложении Azure

В этой статье показано, как развернуть приложение, управляемое с помощью Фортаникс конфиденциального диспетчера вычислительных ресурсов, в портал Azure.

Фортаникс — сторонний поставщик программного обеспечения с продуктами и службами, созданными на основе инфраструктуры Azure. Существуют другие сторонние поставщики, предлагающие аналогичные конфиденциальные вычислительные службы в Azure.

> [!NOTE]
>Продукты, упоминаемые в этом документе, не контролируются корпорацией Майкрософт. Корпорация Майкрософт предоставляет эти сведения только для удобства, а ссылки на эти продукты сторонних производителей не подразумевают подтверждения корпорацией Майкрософт.

## <a name="prerequisites"></a>Предварительные условия

- Частный реестр DOCKER для отправки преобразованных образов приложений.
- Если у вас еще нет подписки Azure, [создайте учетную запись Azure](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/), прежде чем начинать работу.

## <a name="deploy-a-confidential-computing-manager-through-an-azure-managed-application"></a>Развертывание конфиденциального диспетчера вычислительных ресурсов с помощью управляемого приложения Azure

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com/).

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/azure-portal.png" alt-text="портал Azure.":::

2. В строке поиска найдите "Фортаникс конфиденциальные вычислительные системы", и вы увидите список Marketplace для Фортаникс CCM. Выберите **Фортаникс конфиденциальный диспетчер вычислительных ресурсов в Azure**.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/search-marketplace-listing.png" alt-text="Список Marketplace.":::

3. Откроется страница, на которой создается приложение, управляемое с помощью CCM. Выберите **создать**.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/create-managed-application.png" alt-text="Создать приложение.":::

4. Заполните все обязательные поля.
   1. В разделе сведения об управляемом приложении поле **управляемая группа ресурсов** будет иметь значение по умолчанию, которое пользователь может изменить при необходимости.
   2. В поле **регион** выберите либо **Восточная Австралия**, Юго- **Восточная Австралия**, **Восточная часть США**, **Западная часть США 2**, **Западная Европа**, **Северная Европа**, **Центральная Канада**, **Восточная Канада** или **Восточная часть США 2 (euap)**.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/required-fields.png" alt-text="Обязательные поля":::

   Выберите **проверить и создать** , чтобы создать управляемое с помощью CCM фортаникс приложение.

5. Просмотрите сведения и после завершения проверки установите флажок **я принимаю условия выше** , а затем щелкните **создать** , чтобы создать управляемое приложение.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/review-details.png" alt-text="Сведения о проверке.":::

6. Начнется развертывание Фортаникс CCM и будет уведомление о том, что развертывание выполняется.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/deployment-progress.png" alt-text="Ход развертывания.":::

7. После завершения развертывания нажмите кнопку **"переход к ресурсу** ", чтобы открыть страницу "Обзор" развернутого приложения на основе CCM, чтобы зарегистрировать узел вычислений.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/fortanix-resource.png" alt-text="Снимок экрана, на котором показано успешное развертывание в портал Azure.]":::

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/fortanix-overview.png" alt-text="Снимок экрана, показывающий общие сведения о ресурсе конфиденциальных вычислений в портал Azure.":::

## <a name="enroll-the-compute-node-in-fortanix-ccm"></a>Регистрация расчетного узла в Фортаникс CCM

1. В меню навигации слева выберите **Диспетчер конфиденциальных вычислений** . Войдите в Фортаникс CCM и создайте учетную запись, как показано на **рис. 9**.

    Дополнительные сведения о регистрации, входе и создании учетной записи в CCM см. в разделе [ccm начало работы](https://support.fortanix.com/hc/en-us/articles/360034373551-User-s-Guide-Logging-in).
    
    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/fortanix-login.png" alt-text="Снимок экрана, на котором показано входное Фортаникс конфиденциальное имя диспетчера вычислительных систем.":::
    
2. Чтобы получить токен Join из консоли управления CCM, сначала нажмите кнопку **Регистрация узла** . Затем в окне Регистрация узла нажмите кнопку **Копировать** , чтобы скопировать токен JOIN.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/get-join-token.png" alt-text="Снимок экрана, показывающий получение маркера объединения.":::

3. Теперь, чтобы зарегистрировать агент узла, выберите вкладку **Агент узла конфиденциальных вычислений** и нажмите кнопку **Добавить** , чтобы добавить агент узла CCM.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/add-node-agent.png" alt-text="Снимок экрана, показывающий Добавление агента узла.":::

4.  В форме агент узла CCM заполните все обязательные поля. Вставьте маркер объединения, скопированный на шаге 2 в **маркере Join**. Выберите **Проверка + отправить** для подтверждения.

    Дополнительные сведения о том, как зарегистрировать расчетный узел CCM, см. в разделе [Регистрация вычислений узла](https://support.fortanix.com/hc/en-us/articles/360043085652-User-s-Guide-Compute-Nodes).
    
    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/enroll-compute-node.png" alt-text="Снимок экрана, показывающий регистрацию расчетного узла.":::
    
5. После завершения проверки нажмите кнопку **Отправить** , чтобы завершить создание агента узла.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/node-agent-created.png" alt-text="Снимок экрана, показывающий, что создан агент узла.":::

6. Чтобы проверить состояние развертывания, перейдите на вкладку **Обзор** и выберите ссылку **управляемая группа ресурсов** .

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/node-enrolled.png" alt-text="Снимок экрана, на котором показано, что узел зарегистрирован.":::
    
    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/managed-resource-group.png" alt-text="Снимок экрана, на котором показана проверка состояния развертывания.":::

7. Теперь вы увидите, что состояние развертывания все еще выполняется и займет несколько минут для успешной регистрации агента узла.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/deployment-in-progress.png" alt-text="Снимок экрана, на котором показано выполняющееся развертывание.":::

8. Когда регистрация агента узла завершится успешно, состояние изменится на "успешно".

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/deployment-succeeded.png" alt-text="Снимок экрана, на котором показано, что развертывание прошло удачно.":::

9. Теперь в приложении, управляемом с помощью CCM, перейдите на страницы "вычисленные узлы", и вы увидите, что узел находится в **активном** состоянии и успешно зарегистрирован.

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager/node-active-state.png" alt-text="Снимок экрана, на котором показано, что узел успешно зарегистрирован.":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Пользователь также может удалить агент узла CCM с страницы агента конфиденциальных вычислительных узлов. Чтобы удалить агент узла, выберите агент узла и нажмите кнопку **Удалить** на верхней панели.

:::image type="content" source="media/how-to-fortanix-confidential-computing-manager/delete-node-agent.png" alt-text="Снимок экрана, на котором показано удаление агента узла.":::

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы зарегистрировали узел с помощью управляемого приложения Azure в диспетчере конфиденциальных компьютерных систем Фортаникс. Регистрация узла позволяет преобразовать образ приложения для выполнения на основе виртуальной машины с конфиденциальными компьютерами. Дополнительные сведения о виртуальных машинах конфиденциальных вычислений в Azure см. в [этой статье](virtual-machine-solutions.md).

Дополнительные сведения о конфиденциальных предложениях Azure см. в статье [конфиденциальные вычисления в Azure](overview.md).

Узнайте, как выполнять аналогичные задачи с помощью других сторонних предложений в Azure, например [Анжуна](https://azuremarketplace.microsoft.com/marketplace/apps/anjuna-5229812.aee-az-v1) [и.](https://sconedocs.github.io)

