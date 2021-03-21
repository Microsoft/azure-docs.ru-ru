---
title: Устранение неполадок с подключением между Synapse Studio и хранилищем
description: Устранение неполадок с подключением между Synapse Studio и хранилищем
author: saveenr
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: xujiang1
ms.reviewer: jrasnick
ms.openlocfilehash: d570b4a8df5d59cf8828985bee20852d6bc79b1e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98117067"
---
# <a name="troubleshoot-connectivity-between-azure-synapse-analytics-synapse-studio-and-storage"></a>Устранение неполадок с подключением между Azure Synapse Analytics Synapse Studio и хранилищем

В синапсе Studio можно просматривать ресурсы данных, расположенные в связанном хранилище. Это пошаговое руководством поможет решить проблемы с подключением при попытке доступа к ресурсам данных. 

## <a name="case-1-storage-account-lacks-proper-permissions"></a>#1 регистра: учетная запись хранения не имеет необходимых разрешений

Если учетная запись хранения не имеет необходимых разрешений, вы не сможете расширить структуру хранилища с помощью "Data" — > "Linked" (связанные) в синапсе Studio. См. снимок экрана с симптомами проблем ниже. 

Подробное сообщение об ошибке может отличаться, но общее значение сообщения об ошибке: "Этот запрос не имеет полномочий на выполнение этой операции".

В узле связанного хранилища:  
![Проблемы с подключением к хранилищу 1](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-1.png)

В узле контейнера хранилища:  
![Проблемы с подключением к хранилищу 1A](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-1a.png)

**Решение**. чтобы назначить учетную запись правильной роли, см. статью [Использование портал Azure для назначения роли Azure для доступа к данным BLOB-объектов и очередей](../../storage/common/storage-auth-aad-rbac-portal.md) .


## <a name="case-2-failed-to-send-the-request-to-storage-server"></a>#2 регистра: не удалось отправить запрос на сервер хранилища

При выборе стрелки для расширения структуры хранилища в разделе "данные" — > "связанный" в синапсе Studio может появиться ошибка "REQUEST_SEND_ERROR" на панели слева. См. снимок экрана проблемы ниже:

В узле связанного хранилища:  
![Проблемы с подключением к хранилищу 2](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-2.png)

В узле контейнера хранилища:  
![Проблемы с подключением к хранилищу 2A](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-2a.png)

Эта проблема может быть вызвана несколькими причинами.

### <a name="the-storage-resource-is-behind-a-vnet-and-a-storage-private-endpoint-needs-to-configure"></a>Ресурс хранилища находится за виртуальной сетью, и необходимо настроить закрытую конечную точку хранилища

**Решение**. в этом случае необходимо настроить закрытую конечную точку хранилища для учетной записи хранения. Сведения о настройке частной конечной точки хранилища для виртуальной сети см. в статье [использование портал Azure для назначения роли Azure для доступа к данным BLOB-объектов и очередей](../security/how-to-connect-to-workspace-from-restricted-network.md).

\<storage-account-name\>Чтобы проверить подключение после настройки частной конечной точки хранилища, можно использовать команду nslookup. DFS.Core.Windows.NET. Он должен вернуть строку, подобную: " \<storage-account-name\> . privatelink.DFS.Core.Windows.NET".

### <a name="the-storage-resource-is-not-behind-a-vnet-but-the-blob-service-azure-ad-endpoint-is-not-accessible-due-to-firewall-configured"></a>Ресурс хранилища не находится за виртуальной сетью, но конечная точка службы BLOB-объектов (Azure AD) недоступна, так как настроен брандмауэр

**Решение**. в этом случае необходимо открыть учетную запись хранения в портал Azure. В левой области навигации прокрутите вниз до раздела **Поддержка и устранение неполадок** и выберите **Проверка подключения** , чтобы проверить состояние подключения **службы BLOB-объектов (Azure AD)** . Если она недоступна, следуйте рекомендациям, приведенным в разделе "Конфигурация **брандмауэров и виртуальных сетей** " на странице учетной записи хранения. Дополнительные сведения о брандмауэрах хранилища см. в статье [Настройка брандмауэров службы хранилища Azure и виртуальных сетей](../../storage/common/storage-network-security.md).

### <a name="other-issues-to-check"></a>Другие проблемы, которые следует проверить 

* Ресурс хранилища, к которому вы обращаетесь, Azure Data Lake Storage 2-го поколения и находится за брандмауэром и виртуальной сетью (с настроенной частной конечной точкой хранилища) одновременно.
* Ресурс контейнера, к которому вы обращаетесь, был удален или не существует.
* Пересечение с клиентом. клиент рабочей области, который пользователь использовал для входа, не совпадает с клиентом учетной записи хранения. 


## <a name="next-steps"></a>Дальнейшие действия
Если предыдущие шаги не помогли устранить проблему, [Создайте запрос в службу поддержки](../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md).