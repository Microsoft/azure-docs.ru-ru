---
title: Управление ресурсами в Azure Red Hat OpenShift | Документация Майкрософт
description: Управление проектами, шаблонами, потоками изображений в кластере Azure Red Hat OpenShift
services: openshift
keywords: проекты Red Hat openshift запрашивают самоподготовку
author: mjudeikis
ms.author: gwallace
ms.date: 07/19/2019
ms.topic: conceptual
ms.service: azure-redhat-openshift
ms.openlocfilehash: bf2cf5a0d41af15821035c615fe071c8580e125f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633110"
---
# <a name="manage-projects-templates-image-streams-in-an-azure-red-hat-openshift-cluster"></a>Управление проектами, шаблонами, потоками изображений в кластере Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3,11 будет прекращена 30 июня 2022. Поддержка для создания новых кластеров Azure Red Hat OpenShift 3,11 продолжится до 30 ноября 2020. После выхода из эксплуатации оставшиеся кластеры Azure Red Hat OpenShift 3,11 будут закрыты для предотвращения уязвимости системы безопасности.
> 
> Следуйте указаниям этого руководством, чтобы [создать кластер Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Если у вас есть определенные вопросы, [свяжитесь с нами](mailto:arofeedback@microsoft.com).

В платформе контейнера OpenShift проекты используются для группирования и изоляции связанных объектов. Администратор может предоставить разработчикам доступ к конкретным проектам, позволить им создавать собственные проекты и предоставлять им административные права для отдельных проектов.

## <a name="self-provisioning-projects"></a>Самостоятельная подготовка проектов

Вы можете позволить разработчикам создавать собственные проекты. Конечная точка API отвечает за подготовку проекта в соответствии с шаблоном, который называется Project-Request. Веб-консоль и `oc new-project` команда используют эту конечную точку, когда разработчик создает новый проект.

При отправке запроса проекта API заменяет следующие параметры в шаблоне:

| Параметр               | Описание                                    |
| ----------------------- | ---------------------------------------------- |
| PROJECT_NAME            | Имя проекта. Обязательный.             |
| PROJECT_DISPLAYNAME     | Отображаемое имя проекта. Значение может быть пустым. |
| PROJECT_DESCRIPTION     | Описание проекта. Значение может быть пустым.  |
| PROJECT_ADMIN_USER      | Имя пользователя для администрирования.       |
| PROJECT_REQUESTING_USER | Имя пользователя запрашивающего пользователя.           |

Доступ к API предоставляется разработчикам с помощью привязки роли кластера самоподготовки. По умолчанию эта функция доступна для всех разработчиков, прошедших проверку подлинности.

## <a name="modify-the-template-for-a-new-project"></a>Изменение шаблона для нового проекта 

1. Войдите в систему как пользователь с `customer-admin` правами доступа.

2. Измените шаблон запроса проекта по умолчанию.

   ```
   oc edit template project-request -n openshift
   ```

3. Удалите шаблон проекта по умолчанию из процесса обновления Azure Red Hat OpenShift (АТО), добавив следующую аннотацию: `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   Шаблон запроса проекта не будет обновлен процессом обновления АТО. Это позволяет клиентам настраивать шаблон и сохранять эти настройки при обновлении кластера.

## <a name="disable-the-self-provisioning-role"></a>Отключение роли самоподготовки

Вы можете предотвратить самостоятельную подготовку новых проектов для группы пользователей, прошедших проверку подлинности.

1. Войдите в систему как пользователь с `customer-admin` правами доступа.

2. Измените привязку роли кластера самоподготовки.

   ```
   oc edit clusterrolebinding.rbac.authorization.k8s.io self-provisioners
   ```

3. Удалите роль из процесса обновления АТО, добавив следующую аннотацию: `openshift.io/reconcile-protect: "true"` .

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

4. Измените привязку роли кластера, чтобы предотвратить `system:authenticated:oauth` Создание проектов:

   ```
   apiVersion: rbac.authorization.k8s.io/v1
   groupNames:
   - osa-customer-admins
   kind: ClusterRoleBinding
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
     labels:
       azure.openshift.io/owned-by-sync-pod: "true"
     name: self-provisioners
   roleRef:
     name: self-provisioner
   subjects:
   - kind: SystemGroup
     name: osa-customer-admins
   ```

## <a name="manage-default-templates-and-imagestreams"></a>Управление шаблонами по умолчанию и Имажестреамс

В Azure Red Hat OpenShift можно отключить обновления для всех шаблонов по умолчанию и потоков изображений внутри `openshift` пространства имен.
Чтобы отключить обновления для всех `Templates` и `ImageStreams` в `openshift` пространстве имен, выполните следующие действия.

1. Войдите в систему как пользователь с `customer-admin` правами доступа.

2. Изменить `openshift` пространство имен:

   ```
   oc edit namespace openshift
   ```

3. Удалите `openshift` пространство имен из процесса обновления АТО, добавив следующую аннотацию: `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   Любой отдельный объект в `openshift` пространстве имен можно удалить из процесса обновления, добавив `openshift.io/reconcile-protect: "true"` к нему заметку.

## <a name="next-steps"></a>Дальнейшие действия

Воспользуйтесь руководством.
> [!div class="nextstepaction"]
> [Создание кластера Azure Red Hat OpenShift](tutorial-create-cluster.md)
