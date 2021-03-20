---
title: Экспорт конфигурации подготовки и откат к известному работоспособному состоянию для аварийного восстановления
description: Узнайте, как экспортировать конфигурацию подготовки и вернуться к известному работоспособному состоянию для аварийного восстановления.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 03/19/2020
ms.author: kenwith
ms.openlocfilehash: c6af42c78bda66c4b397cbb99b26af7d6a5c7f07
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99256379"
---
# <a name="how-to-export-provisioning-configuration-and-roll-back-to-a-known-good-state"></a>Руководство. Экспорт конфигурации подготовки и откат к известному работоспособному состоянию

В этой статье вы узнаете, как выполнять следующие задачи.

- Экспортируйте и импортируйте конфигурацию подготовки из портал Azure
- Экспорт и импорт конфигурации подготовки с помощью API Microsoft Graph

## <a name="export-and-import-your-provisioning-configuration-from-the-azure-portal"></a>Экспортируйте и импортируйте конфигурацию подготовки из портал Azure

### <a name="export-your-provisioning-configuration"></a>Экспорт конфигурации подготовки

Чтобы экспортировать конфигурацию, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) в области навигации слева выберите **Azure Active Directory**.
1. В области **Azure Active Directory** выберите **корпоративные приложения** и выберите свое приложение.
1. В области навигации слева выберите **Подготовка**. На странице Настройка подготовки щелкните **сопоставления атрибутов**, затем **Отображение дополнительных параметров** и, наконец, **Проверьте схему**. Откроется редактор схемы.
1. Щелкните скачать на панели команд в верхней части страницы, чтобы скачать схему.

### <a name="disaster-recovery---roll-back-to-a-known-good-state"></a>Аварийное восстановление — откат к известному работоспособному состоянию

Экспорт и сохранение конфигурации позволяет выполнить откат к предыдущей версии конфигурации. Мы рекомендуем экспортировать конфигурацию подготовки и сохранить ее для дальнейшего использования в любое время при внесении изменений в сопоставления атрибутов или фильтры области. Все, что нужно сделать, — открыть JSON-файл, скачанный на приведенных выше шагах, скопировать все содержимое JSON-файла, заменить все содержимое полезных данных JSON в редакторе схемы, а затем сохранить. Если активен цикл подготовки, он будет завершен, а следующий цикл будет использовать обновленную схему. Следующий цикл также будет начальным циклом, который повторно оценивает каждого пользователя и группу на основе новой конфигурации. При откате к предыдущей конфигурации учитывайте следующее.

- Пользователи будут снова оцениваться, чтобы определить, должны ли они находиться в области. Если фильтры области изменены, пользователь не находится в области, что больше не будет отключено. Хотя в большинстве случаев это желаемое поведение, в некоторых случаях может возникнуть необходимость предотвратить это и использовать функцию « [пропустить удаление из области действия](./skip-out-of-scope-deletions.md) ». 
- При изменении конфигурации подготовки служба перезапускается и запускается [начальный цикл](./how-provisioning-works.md#provisioning-cycles-initial-and-incremental).

## <a name="export-and-import-your-provisioning-configuration-by-using-the-microsoft-graph-api"></a>Экспорт и импорт конфигурации подготовки с помощью API Microsoft Graph

Вы можете использовать Microsoft Graph API и обозреватель Microsoft Graph, чтобы экспортировать сопоставления атрибутов подготовки пользователей и схемы в JSON-файл и импортировать их обратно в Azure AD. Вы также можете использовать описанные здесь действия, чтобы создать резервную копию конфигурации подготовки.

### <a name="step-1-retrieve-your-provisioning-app-service-principal-id-object-id"></a>Шаг 1. Получение идентификатора субъекта-службы приложения подготовки (идентификатор объекта)

1. Запустите [портал Azure](https://portal.azure.com)и перейдите к разделу Properties приложения подготовки. Например, если вы хотите экспортировать сопоставление *приложения Workday в службу подготовки пользователей Active Directory* , перейдите в раздел свойств этого приложения.
1. В разделе "Свойства" вашего приложения подготовки скопируйте значение GUID, связанное с полем *Идентификатор объекта*. Это значение также называется **ServicePrincipalId** приложения и будет использоваться в операциях проводника Microsoft Graph.

   ![Идентификатор субъекта-службы приложения Workday](./media/export-import-provisioning-configuration/wd_export_01.png)

### <a name="step-2-sign-into-microsoft-graph-explorer"></a>Шаг 2. вход в Microsoft Graph Explorer

1. Запустите [обозреватель Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer)
1. Нажмите кнопку "Вход с помощью учетной записи Майкрософт" и войдите, используя учетные данные глобального администратора Azure AD или администратора приложения.

    ![Microsoft Graph вход](./media/export-import-provisioning-configuration/wd_export_02.png)

1. После выполнения входа вы увидите данные учетной записи пользователя на левой панели.

### <a name="step-3-retrieve-the-provisioning-job-id-of-the-provisioning-app"></a>Шаг 3. Получение идентификатора задания подготовки для приложения подготовки

В обозревателе Microsoft Graph выполните следующий запрос GET, заменив [servicePrincipalId] на атрибут **ServicePrincipalId**, извлеченный на [шаге 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs
```

Вы получите ответ, похожий на показанный ниже. Скопируйте атрибут идентификатора из ответа. Это значение представляет собой **ProvisioningJobId** и будет использоваться для получения метаданных базовой схемы.

   [![Идентификатор задания подготовки](./media/export-import-provisioning-configuration/wd_export_03.png)](./media/export-import-provisioning-configuration/wd_export_03.png#lightbox)

### <a name="step-4-download-the-provisioning-schema"></a>Шаг 4. скачивание схемы подготовки

В обозревателе Microsoft Graph выполните следующий запрос GET, заменив [servicePrincipalId] и [ProvisioningJobId] на значения ServicePrincipalId и ProvisioningJobId, полученные на предыдущих шагах.

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Скопируйте объект JSON из ответа и сохраните его в файл, чтобы создать резервную копию схемы.

### <a name="step-5-import-the-provisioning-schema"></a>Шаг 5. Импорт схемы подготовки

> [!CAUTION]
> Выполните этот шаг только в том случае, если необходимо изменить схему конфигурации, которую нельзя изменить с помощью портала Azure, или если требуется восстановить конфигурацию из ранее сохраненной резервной копии с допустимой и рабочей схемой.

В обозревателе Microsoft Graph настройте следующий запрос PUT, заменив [servicePrincipalId] и [ProvisioningJobId] на значения ServicePrincipalId и ProvisioningJobId, полученные на предыдущих шагах.

```http
    PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

На вкладке текста запроса скопируйте содержимое файла схемы JSON.

   [![Текст запроса](./media/export-import-provisioning-configuration/wd_export_04.png)](./media/export-import-provisioning-configuration/wd_export_04.png#lightbox)

На вкладке заголовков запроса добавьте атрибут заголовка Content-Type со значением "application/json".

   [![Заголовки запроса](./media/export-import-provisioning-configuration/wd_export_05.png)](./media/export-import-provisioning-configuration/wd_export_05.png#lightbox)

Выберите **выполнить запрос** , чтобы импортировать новую схему.