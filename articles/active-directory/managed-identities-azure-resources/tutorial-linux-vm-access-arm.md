---
title: Учебник`:` Получение доступа к Azure Resource Manager с помощью управляемого удостоверения — Azure AD
description: Из этого руководства вы узнаете, как получить доступ к Azure Resource Manager с помощью назначаемого системой управляемого удостоверения на виртуальной машине Linux.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: bryanla
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 653159c2e40d3375a422f0da14274f57130de1fe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93359686"
---
# <a name="use-a-linux-vm-system-assigned-managed-identity-to-access-azure-resource-manager"></a>Использование назначаемого системой управляемого удостоверения на виртуальной машине Linux для доступа к Azure Resource Manager

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом кратком руководстве показано, как использовать назначаемое системой управляемое удостоверение для доступа к API Azure Resource Manager с помощью виртуальной машины Linux. Управляемыми удостоверениями для ресурсов Azure автоматически управляет Azure. Они позволяют проходить проверку подлинности в службах, поддерживающих аутентификацию Azure Active Directory, без указания учетных данных в коде. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Предоставление виртуальной машине доступа к группе ресурсов в Azure Resource Manager. 
> * Получение маркера доступа с помощью удостоверения виртуальной машины и вызов Azure Resource Manager с его помощью. 

## <a name="prerequisites"></a>Предварительные требования

- Основные сведения об управляемых удостоверениях. См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](overview.md). 
- Учетная запись Azure. Зарегистрируйте [бесплатную учетную запись](https://azure.microsoft.com/free/).
- Кроме того, вам потребуется виртуальная машина Linux с включенными управляемыми удостоверениями, назначенными системой.
  - Если вам нужно создать виртуальную машину для работы с этим учебником, выполните инструкции из раздела [Создание виртуальной машины](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine)

## <a name="grant-access"></a>Предоставление доступа

С помощью управляемых удостоверений для ресурсов Azure код может получить маркеры доступа, чтобы пройти проверку подлинности и получить доступ к ресурсам, поддерживающим аутентификацию Azure AD. API Azure Resource Manager поддерживает аутентификацию Azure AD. Сначала нам необходимо предоставить этому удостоверению виртуальной машины доступ к ресурсу в Azure Resource Manager, в этом случае — к группе ресурсов, в которой находится виртуальная машина.  

1. Перейдите на вкладку **Группы ресурсов**.
2. Выберите определенную **группу ресурсов**, которую использовали для своей виртуальной машины.
3. Перейдите к элементу **Управление доступом (IAM)** на панели слева.
4. Щелкните **Добавить**, чтобы добавить новое назначение роли виртуальной машине. В поле **Роль** выберите **Читатель**.
5. В раскрывающемся списке далее в поле **Назначение доступа к** выберите ресурс **Виртуальная машина**.
6. Проверьте, чтобы в раскрывающемся списке **Подписка** была выбрана нужная подписка. В поле **Группа ресурсов** выберите **Все группы ресурсов**.
7. И наконец, в поле **Выбрать** выберите свою виртуальную машину Linux в раскрывающемся списке, а затем нажмите кнопку **Сохранить**.

    ![Замещающий текст](media/msi-tutorial-linux-vm-access-arm/msi-permission-linux.png)

## <a name="get-an-access-token-using-the-vms-system-assigned-managed-identity-and-use-it-to-call-resource-manager"></a>Получение маркера доступа с помощью назначаемого системой управляемого удостоверения виртуальной машины и вызов Resource Manager с использованием этого маркера

Для выполнения этих действий вам потребуется клиент SSH. Если вы используете Windows, можно использовать клиент SSH в [подсистеме Windows для Linux](/windows/wsl/about). Если вам нужна помощь в настройке ключей SSH-клиента, ознакомьтесь с разделом [Использование ключей SSH с Windows в Azure](../../virtual-machines/linux/ssh-from-windows.md) или [Как создать и использовать пару из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. На портале перейдите на виртуальную машину Linux и в разделе **Обзор** щелкните **Подключиться**.  
2. **Подключитесь** к виртуальной машине с помощью выбранного клиента SSH. 
3. В окне терминала выполните запрос к локальной конечной точке управляемых удостоверений для ресурсов Azure с помощью `curl`, чтобы получить маркер доступа для Azure Resource Manager.  
 
    Запрос `curl` маркера доступа приведен ниже.  
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' -H Metadata:true   
    ```
    
    > [!NOTE]
    > Значение параметра resource должно точно совпадать со значением, которое будет использоваться в Azure AD.    Если используется идентификатор ресурса Resource Manager, добавьте косую черту после универсального кода ресурса (URI). 
    
    Ответ включает маркер доступа, необходимый для доступа к Azure Resource Manager. 
    
    Ответ:  

    ```bash
    {"access_token":"eyJ0eXAiOi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
    ```
    
    Этот маркер доступа можно использовать для доступа к Azure Resource Manager, например для просмотра подробных сведений о группе ресурсов, к которой вы ранее предоставили доступ виртуальной машине. Замените значения \<SUBSCRIPTION ID\>, \<RESOURCE GROUP\> и \<ACCESS TOKEN\> созданными ранее значениями. 
    
    > [!NOTE]
    > В URL-адресе учитывается регистр знаков, поэтому должен использоваться тот же регистр, который использовался, когда вы присваивали имя группе ресурсов. Обязательно укажите прописную букву "G" в имени группы resourceGroup.  
    
    ```bash 
    curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>?api-version=2016-09-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```
    
    Ответ с определенными сведениями о группе ресурсов выглядит следующим образом: 
     
    ```bash
    {"id":"/subscriptions/98f51385-2edc-4b79-bed9-7718de4cb861/resourceGroups/DevTest","name":"DevTest","location":"westus","properties":{"provisioningState":"Succeeded"}} 
    ```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как использовать назначаемое системой управляемое удостоверение для доступа к API Azure Resource Manager.  Сведения об Azure Resource Manager см. здесь:

> [!div class="nextstepaction"]
>[Azure Resource Manager](../../azure-resource-manager/management/overview.md)
>[Создание и удаление управляемых удостоверений, назначаемых пользователем, а также получение их списка с помощью Azure PowerShell](how-to-manage-ua-identity-powershell.md)