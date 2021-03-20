---
title: Настройка защищенного хранилища для Azure Data Box
description: Узнайте, как использовать защищенное хранилище с Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.topic: how-to
ms.date: 07/10/2020
ms.author: alkohli
ms.subservice: pod
ms.openlocfilehash: 178ad169c1b576458f38b440ca79f4bb9eb012f5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92124917"
---
# <a name="use-customer-lockbox-for-azure-data-box-preview"></a>Использование защищенное хранилище для Azure Data Box (Предварительная версия)

Azure Data Box используется для перемещения данных клиентов в Azure и обратно. Существуют экземпляры, в которых служба поддержки Майкрософт может потребоваться доступ к данным клиента во время Поддержка. Защищенное хранилище можно использовать в качестве интерфейса для просмотра и утверждения или отклонения этих запросов на доступ к данным. 

В этой статье описывается, как инициировать и отслеживание запросов защищенное хранилище для импорта Data Box, а также для экспорта заказов. Статья относится как к Azure Data Box устройствам, так и к Azure Data Box Heavyным устройствам. 

## <a name="devops-workflow-for-data-access"></a>Рабочий процесс DevOps для доступа к данным

Группа поддержки и Data Boxных операций в корпорации Майкрософт обычно не имеет доступа к данным клиентов. Они пытаются устранить проблемы с помощью стандартных средств и телеметрии. <!--The only scenarios where there is a need to access customer data is when there is an issue with the data that needs to be fixed. For example, if the data is copied to a wrong folder or is in an incorrect format and is likely to result in an upload or download failure, then Microsoft will try to access your data in the Azure datacenter.--> 

Если проблемы не удается устранить, и требуется служба поддержки Майкрософт для изучения или исправления данных, они запрашивают повышенный доступ через портал JIT. Портал ЖИП проверяет уровень разрешений, предоставляет многофакторную проверку подлинности, а также включает утверждение от внутренних утверждающих Майкрософт. Например, утверждающее лицо может быть DevOps Manager. 

Если вы включили защищенное хранилище, то после того, как запрос на повышение прав доступа будет утвержден через портал JIT, корпорация Майкрософт также потребует явного согласия на доступ к данным. Доступ запрашивается и отслеживание осуществляется через службу защищенное хранилище на портале. 

Если защищенное хранилище не включено, ваше согласие не требуется для доступа к данным.


## <a name="prerequisites-for-access-request"></a>Необходимые условия для запроса на доступ

Перед тем как начать, убедитесь в следующем:

1. Вы создали порядок Azure Data Box в соответствии с инструкциями в:
    1. [Учебник. Azure Data Box заказов](data-box-deploy-ordered.md) для импорта заказов.
    1. [Учебник. Azure Data Box заказов](data-box-deploy-export-ordered.md) для экспорта заказов.

2. Вы настроили защищенное хранилище для Data Box. Это служба, которая является обязательной. 

    1. Защищенное хранилище в настоящее время находится на этапе предварительной версии для службы Data Box. Чтобы включить защищенное хранилище для Data Box в Организации, подпишитесь на [защищенное хранилище для общедоступной предварительной версии Azure](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_Kwz02N6XVCoKNpxIpqE_hUNzlTUUNYVkozOVlFNVRSWDVHRkkwTFQyViQlQCN0PWcu).
    2. Защищенное хранилище автоматически доступны для всех клиентов, имеющих план поддержки Azure с минимальным уровнем разработчика. <!--How do you enable Lockbox? change this for Azure Data Box, perhaps you need a different support plan When you have an eligible support plan, no action is required by you to enable Customer Lockbox. Customer Lockbox requests are initiated by a Microsoft engineer if this action is needed to progress a support ticket that is filed from somebody in your organization.-->

3. Запрос на обслуживание или обращение в службу поддержки уже открыты для этой проблемы. Дополнительные сведения о запросах в службу поддержки см. в разделе [файл a службы для Data Box](data-box-disk-contact-microsoft-support.md).


## <a name="track-approve-request-via-lockbox"></a>Трассировка, утверждение запроса через защищенное хранилище

Чтобы отслеживать и утверждать запрос на доступ к данным клиента, выполните следующие действия.

1. Корпорация Майкрософт обнаружит, что во время отправки или скачивания данных в центре Azure возникла ошибка. Например, на этапе **копирования данных** Data Box порядок останова. 

    Инженер службы поддержки подключается к Data Box через сеанс поддержки и пытается устранить проблему с помощью стандартных средств и телеметрии. Если Data Box диски заблокированы, а общие папки недоступны, инженер службы поддержки создает запрос на создание хранилища. 
 
2. При создании запроса, как правило, уведомление отправляется администратору подписки, но можно также настроить группу для уведомлений. 

3. Вы видите запрос на защищенное хранилище в портал Azure для вашего утверждения. 

    ![Запрос в портал Azure](./media/data-box-customer-lockbox/3-lockbox-request-azure-portal.png)

    Чтобы утвердить запрос на защищенное хранилище на портале, можно выбрать **утвердить**.

    ![Утвердить запрос](./media/data-box-customer-lockbox/4-lockbox-request-details-azure-portal.png)


    После утверждения запроса диски устройства разблокируются, а общие папки становятся доступными в сеансе поддержки.

4. Инженер службы поддержки решает проблему отправки, а затем отключает сеанс поддержки.

После устранения проблемы задание копирования данных будет выполнено до завершения.


## <a name="next-steps"></a>Дальнейшие действия

- [Защищенное хранилище для Microsoft Azure](../security/fundamentals/customer-lockbox-overview.md)

<!--- [Approve, audit support access requests to VMs using Customer Lockbox for Azure](https://azure.microsoft.com/blog/approve-audit-support-access-requests-to-vms-using-customer-lockbox-for-azure/)-->