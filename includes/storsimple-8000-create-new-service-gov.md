---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: c3f913aa3de1b723cd4eae70ff8e578a8abbec12
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95560160"
---
#### <a name="to-create-a-new-service"></a>Создание новой службы
1. Войдите на [портал Microsoft Azure для государственных организаций](https://portal.azure.us/) с данными своей учетной записи Майкрософт.
2. На портале для государственных организаций щелкните **+** , а затем в Marketplace выберите **Просмотреть все**. Найдите _физическое устройство StorSimple_. Выберите **Серия физического устройства StorSimple** и щелкните **Создать**. Либо на портале для государственных организаций щелкните **+** , а затем в разделе **Хранилище** выберите **Серия физического устройства StorSimple**.
3. В колонке **диспетчера устройств StorSimple** выполните следующие действия.
   
   1. В поле **Имя ресурса** укажите уникальное имя для службы. Здесь необходимо указать понятное имя, пригодное для идентификации службы. Имя может содержать от 2 до 50 символов, среди которых могут быть буквы, цифры и дефисы. Имя должно начинаться и заканчиваться буквой или цифрой.
   2. Выберите **Подписка** в раскрывающемся списке. Подписка привязана к учетной записи для выставления счетов. Это поле отсутствует, если у вас имеется только одна подписка.
   3. **Группа ресурсов** — **создайте** группу ресурсов или выберите **имеющуюся**. Дополнительные сведения см. в статье [Рекомендации по группам ресурсов Azure](../articles/azure-resource-manager/management/manage-resource-groups-portal.md).
   4. Введите **Местоположение** для службы. Расположение означает географический регион, в котором планируется развертывание устройства. Выберите **Айова, США** или **Вирджиния, США**.
   5. Установите флажок **Создать новую учетную запись хранения** для автоматического создания учетной записи хранения вместе со службой. Укажите имя для этой учетной записи хранения. Если требуется хранить данные в другом расположении, снимите этот флажок.
   6. Установите флажок **Закрепить на панели мониторинга**, если вам нужен быстрый доступ к этой службе с панели мониторинга.
   7. Щелкните **Создать**, чтобы создать диспетчер устройств StorSimple. Создание службы займет несколько минут. После успешного создания службы вы получите уведомление и откроется новая колонка службы.