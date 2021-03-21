---
title: Публикация объявления в лаборатории в Azure DevTest Labs | Документация Майкрософт
description: Узнайте, как опубликовать пользовательское объявление в существующей лаборатории, чтобы уведомить пользователей о последних изменениях или дополнениях к лабораторию в Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 72c6ed7ab691ece094673ff5a158ffa769a9f7a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88270773"
---
# <a name="post-an-announcement-to-a-lab-in-azure-devtest-labs"></a>Публикация объявления в лабораторию в Azure DevTest Labs

Администратор лаборатории может разместить произвольное объявление в существующей лаборатории, чтобы известить пользователей о свежих изменениях или дополнениях в этой лаборатории. Например, они будут полезны для извещения о таких событиях:

- доступность новых размеров виртуальных машин;
- непригодность некоторых образов на текущий момент;
- изменения в политиках лаборатории.

Опубликованное объявление отображается на странице с общими сведениями о лаборатории. Если его щелкнуть, можно получить дополнительные сведения.

Функция публикации объявлений предназначена для временных сообщений.  Когда объявление теряет актуальность, его можно легко отключить.

## <a name="steps-to-post-an-announcement-in-an-existing-lab"></a>Процедура публикации объявления в существующей лаборатории

1. Войдите на [портал Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. При необходимости щелкните **Все службы**, а затем выберите в списке **DevTest Labs**. Возможно, лаборатория уже отображается на панели мониторинга в разделе **Все ресурсы**.
1. Из списка лабораторий выберите ту, в которой вы намерены опубликовать объявление.
1. В области лаборатории **Обзор** выберите **Configuration and policies** (Конфигурация и политики).

    ![Кнопка Configuration and Policies (Конфигурация и политики)](./media/devtest-lab-announcements/devtestlab-config-and-policies.png)

1. В разделе **Параметры** в левой части экрана выберите **Объявление лаборатории**.

    ![Кнопка "Объявление лаборатории"](./media/devtest-lab-announcements/devtestlab-announcements.png)

1. Чтобы создать сообщение для пользователей, которое будет отображаться в этой лаборатории, установите для параметра **Включено** значение **Да**.

1. Введите **дату окончания срока действия**, чтобы указать время, когда объявление больше не будет отображаться пользователям. Если не ввести дату окончания срока действия, объявление будет отображаться, пока вы не отключите его.

   > [!NOTE]
   > После того как истечет срок действия объявления, оно больше не будет отображаться пользователям, но оно по-прежнему будет находиться в области **Lab announcement** (Объявление лаборатории). Вы можете редактировать его и повторно включить, чтобы оно снова стало активным.
   >
   >

1. Заполните поля **Название объявления** и **Текст объявления**.

   Название не может превышать 100 символов. Оно отображается для пользователей на странице общей информации о лаборатории. Когда пользователь выбирает название, ему предоставляется текст объявления.

   В тексте объявления поддерживается разметка Markdown. При вводе текста вы можете просмотреть сообщение в области предварительного просмотра в нижней части экрана.

    ![Экран объявления лаборатории для создания сообщения.](./media/devtest-lab-announcements/devtestlab-post-announcement.png)


1. Нажмите кнопку **Сохранить**, когда объявление будет готово к публикации.

Когда вы решите, что объявление больше не нужно показывать пользователям лаборатории, вернитесь на страницу **объявления лаборатории** и установите для параметра **Включено** значение **Нет**. Если вы указали дату окончания срока действия, объявления автоматически отключится в указанное время.

## <a name="steps-for-users-to-view-an-announcement"></a>Действия пользователя для просмотра объявления

1. На [портале Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040) выберите лабораторию.

1. Если в этой лаборатории опубликовано объявление, извещение об этом отображается в верхней части страницы с общей информацией о лаборатории. В качестве извещения используется название объявления, которое указывается при его создании.

    ![Объявление лаборатории на странице общей информации](./media/devtest-lab-announcements/devtestlab-user-announcement.png)

1. Пользователь может выбрать сообщение, чтобы просмотреть его полный текст.

    ![Дополнительные сведения об объявлении лаборатории](./media/devtest-lab-announcements/devtestlab-user-announcement-text.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager
Объявление можно указать как часть шаблона Azure Resource Manager, как показано в следующем примере:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string",
            "defaultValue": "devtestlabfromarm"
        },
        "regionId": {
            "type": "string",
            "defaultValue": "eastus"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-04-26-preview",
            "name": "[parameters('name')]",
            "type": "Microsoft.DevTestLab/labs",
            "location": "[parameters('regionId')]",
            "tags": {},
            "properties": {
                "labStorageType": "Premium",
                "announcement":
                {
                    "title": "Important! Three images are currently inaccessible. Click for more information.",
                    "markdown":"# Images are inaccessible\n\nThe following 3 images are currently not available for use: \n\n- image1\n- image2\n- image3\n\nI am working to fix the problem ASAP.",
                    "enabled": "Enabled",
                    "expirationDate":"2018-12-31T06:00:00+00:00",
                    "expired": "false"
                },
                "support": {
                    "markdown": "",
                    "enabled": "Enabled"
                }
            },
            "resources": [
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "LabVmsShutdown",
                    "location": "[parameters('regionId')]",
                    "type": "schedules",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ],
                    "properties": {
                        "status": "Enabled",
                        "timeZoneId": "Eastern Standard Time",
                        "dailyRecurrence": {
                            "time": "1900"
                        },
                        "taskType": "LabVmsShutdownTask",
                        "notificationSettings": {
                            "status": "Disabled",
                            "timeInMinutes": 30
                        }
                    }
                },
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "[concat('Dtl', parameters('name'))]",
                    "type": "virtualNetworks",
                    "location": "[parameters('regionId')]",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ]
                }
            ]
        }
    ]
}
```

Вы можете развернуть шаблон Azure Resource Manager одним из следующих способов:

- [Портал Azure](../azure-resource-manager/templates/deploy-portal.md)
- [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Azure CLI](../azure-resource-manager/templates/deploy-cli.md)
- [REST API](../azure-resource-manager/templates/deploy-rest.md)

## <a name="next-steps"></a>Дальнейшие действия
* Когда вы изменяете или создаете политику лаборатории, об этом можно сообщить пользователям с помощью объявления. Статья [Управление всеми политиками лаборатории в Azure DevTest Labs](devtest-lab-set-lab-policy.md) содержит сведения о применении ограничений и соглашений для всей подписки с помощью настраиваемых политик.
* Изучите [коллекцию шаблонов быстрого запуска DevTest Labs Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
