---
title: Службы данных с поддержкой Arc Azure — известные проблемы
description: Последние известные проблемы
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 03/02/2021
ms.topic: conceptual
ms.openlocfilehash: 8100d9e12f107e0c4598876c46453b46c6ee4d0e
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102122006"
---
# <a name="known-issues---azure-arc-enabled-data-services-preview"></a>Известные проблемы: службы данных с поддержкой Arc Azure (Предварительная версия)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="february-2021"></a>Февраль 2021 года

- Подключенный режим кластера отключен

## <a name="introduced-prior-to-february-2021"></a>Представлено до февраля 2021

### <a name="data-controller"></a>Контроллер данных

- В службе Azure Kubernetes Service (AKS) версия Kubernetes 1.19. x не поддерживается.
- В Kubernetes 1,19 `containerd` не поддерживается.
- Ресурс контроллера данных в Azure сейчас является ресурсом Azure. Любые обновления, такие как DELETE, не передаются обратно в кластер kubernetes.
- Имена экземпляров не могут быть длиннее 13 символов
- Нет обновления на месте для контроллера данных ARC в Azure или экземпляров базы данных.
- Образы контейнеров служб данных с поддержкой Arc не подписаны.  Возможно, потребуется настроить узлы Kubernetes так, чтобы разрешить извлечь неподписанные образы контейнера.  Например, если в качестве среды выполнения контейнера используется DOCKER, можно задать переменную среды DOCKER_CONTENT_TRUST = 0 и перезапустить.  Другие среды выполнения контейнеров имеют аналогичные параметры, например в [OpenShift](https://docs.openshift.com/container-platform/4.5/openshift_images/image-configuration.html#images-configuration-file_image-configuration).
- Не удается создать на основе портал Azure службы "Дуга Azure", управляемые экземпляры SQL или группы серверов PostgreSQL.
- Пока вы используете NFS, `allowRunAsRoot` `true` перед созданием контроллера данных ARC в Azure необходимо задать в файле профиля развертывания.
- Только проверка подлинности входа SQL и PostgreSQL.  Отсутствует поддержка Azure Active Directory или Active Directory.
- Создание контроллера данных в OpenShift требует ослабления ограничений безопасности.  Дополнительные сведения см. в документации.
- Если вы используете подсистему Azure Kubernetes Service (AKS) в центре Azure Stack с контроллером данных Arc и экземплярами базы данных Azure, обновление до более новой версии Kubernetes не поддерживается. Удалите контроллер данных ARC в Azure и все экземпляры базы данных перед обновлением кластера Kubernetes.
- Кластеры AKS, охватывающие [несколько зон доступности](../../aks/availability-zones.md) , в настоящее время не поддерживаются для служб данных, поддерживающих службу "Дуга Azure". Чтобы избежать этой проблемы, при создании кластера AKS в портал Azure при выборе региона, в котором доступны зоны, удалите все зоны из элемента управления выбором. См. следующее изображение:

   :::image type="content" source="media/release-notes/aks-zone-selector.png" alt-text="Снимите флажки для каждой зоны, чтобы указать значение нет.":::

### <a name="postgresql"></a>PostgreSQL

- Функция "PostgreSQL" в службе "Дуга Azure" возвращает неточное сообщение об ошибке, когда оно не может восстановиться на относительную точку во времени. Например, если вы указали точку во времени для восстановления, которая старше, чем резервная копия, восстановление завершится с сообщением об ошибке следующего вида: `ERROR: (404). Reason: Not found. HTTP response body: {"code":404, "internalStatus":"NOT_FOUND", "reason":"Failed to restore backup for server...}`
В этом случае перезапустите команду, указав момент времени в диапазоне дат, для которого имеются резервные копии. Чтобы определить этот диапазон, необходимо составить список резервных копий и просмотреть даты их съемки.
- Восстановление на момент времени поддерживается только в разных группах серверов. Целевым сервером операции восстановления на момент времени не может быть сервер, с которого была выполнена архивация. Она должна быть другой группой серверов. Однако полное восстановление поддерживается для одной и той же группы серверов.
- При выполнении полного восстановления требуется идентификатор резервного копирования. По умолчанию, если вы не указываете идентификатор резервной копии, будет использоваться последняя резервная копия. Это не работает в этом выпуске.

## <a name="next-steps"></a>Следующие шаги

> **Хотите попробовать?**  
> Быстро Начните работу с помощью [Azure Arc JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) в AKS, AWS эластичной Kubernetes службы (ЕКС), Google Cloud Kubernetes Engine (гке) или на виртуальной машине Azure.

- [Установка клиентских средств](install-client-tools.md)
- [Создание контроллера данных Azure Arc](create-data-controller.md) (сначала необходимо установить клиентские средства)
- [Создание управляемого экземпляра SQL Azure в Azure Arc](create-sql-managed-instance.md) (сначала требуется создать контроллер данных Azure Arc)
- [Создание группы серверов с уровнем "Гипермасштабирование" Базы данных Azure для PostgreSQL в Azure Arc ](create-postgresql-hyperscale-server-group.md) (сначала необходимо создать контроллер данных Azure Arc)
- [Поставщики ресурсов для служб Azure](../../azure-resource-manager/management/azure-services-resource-providers.md)
- [Заметки о выпуске — службы данных с поддержкой ARC в Azure (Предварительная версия)](release-notes.md)
