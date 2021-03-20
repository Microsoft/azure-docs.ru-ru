---
title: Управление создателем карт Microsoft Azure (Предварительная версия)
description: В этой статье вы узнаете, как управлять создателем карт Microsoft Azure (Предварительная версия).
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/16/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: d26df4287032bc59cc58dd1d832d9d5a9c40afcd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100559127"
---
# <a name="manage-azure-maps-creator-preview"></a>Управление создателем Azure Maps (Предварительная версия) 

> [!IMPORTANT]
> Службы Создателя Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Служба Creator в Azure Maps позволяет создавать частные данные схем помещений. С помощью API Azure Maps и модуля схем помещений можно разрабатывать интерактивные и динамические веб-приложения для схем помещений. В настоящее время служба Creator доступна только в США при использовании ценовой категории S1.

В этой статье описаны действия, необходимые для создания и удаления ресурса Creator в учетной записи Azure Maps.

## <a name="create-creator-preview-resource"></a>Создание ресурса создателя (Предварительная версия)

1. Войдите на [портал Azure](https://portal.azure.com)

2. Выберите свою учетную запись службы Azure Maps. Если вы не видите учетную запись Azure Maps в разделе **Последние ресурсы**, перейдите в меню портала Azure. Щелкните **Все ресурсы**. Найдите и выберите свою учетную запись службы Azure Maps.

    ![Домашняя страница портала Azure Maps](./media/how-to-manage-creator/select-maps-account.png)

3. На странице учетной записи Azure Maps выберите элемент **Обзор** в разделе **Creator**. Выберите  **создать**  , чтобы создать ресурс создателя Azure Maps.

    ![Страница создания ресурса Creator в Azure Maps](./media/how-to-manage-creator/creator-blade-settings.png)

4. Введите имя и расположение для ресурса Creator. В настоящее время Creator поддерживается только в США. Выберите **Review + create** (Просмотреть и создать).

   ![Страница для ввода сведений об учетной записи Creator](./media/how-to-manage-creator/creator-creation-dialog.png)

5. Проверьте параметры и нажмите кнопку **создать**.

    ![Страница подтверждения параметров учетной записи Creator](./media/how-to-manage-creator/creator-create-dialog.png)

6. По завершении развертывания вы увидите страницу с сообщением об успешном выполнении или ошибке.

   ![Страница состояния развертывания ресурса](./media/how-to-manage-creator/creator-resource-created.png)

7. Выберите **Перейти к ресурсу**. На странице представления ресурса Creator отображается состояние ресурса Creator и выбранный демографический регион.

    ![Страница состояния ресурса Creator](./media/how-to-manage-creator/creator-resource-view.png)

   >[!NOTE]
   >На странице ресурсов Creator (создатель ресурса) можно вернуться к учетной записи Azure Maps, к которой он принадлежит, выбрав учетную запись Azure Maps.

## <a name="delete-creator-preview-resource"></a>Удалить ресурс создателя (Предварительная версия)

Чтобы удалить ресурс Creator, перейдите к учетной записи Azure Maps. В разделе **Creator** выберите **Обзор**. Нажмите кнопку **Удалить**.

>[!WARNING]
>При удалении ресурса Creator учетной записи Azure Maps также будут удалены наборы данных, наборы фрагментов и наборы состояний функции, созданные с помощью служб Creator.

![Страница Creator с кнопкой "Удалить"](./media/how-to-manage-creator/creator-delete.png)

Нажмите кнопку **Удалить** и введите имя создателя, чтобы подтвердить удаление. После удаления ресурса вы увидите страницу подтверждения, как показано на рисунке ниже:

![Страница Creator с сообщением о подтверждении удаления](./media/how-to-manage-creator/creator-confirm-delete.png)

## <a name="authentication"></a>Аутентификация

Создатель (Предварительная версия) наследует параметры управления доступом Azure Maps (IAM). Все вызовы API для доступа к данным должны отправляться с соблюдением правил проверки подлинности и авторизации.

Данные об использовании Creator записываются в диаграммы использования и журнал действий Azure Maps.  Дополнительные сведения см. в статье [Управление аутентификацией в Azure Maps](./how-to-manage-authentication.md).

## <a name="access-to-creator-services"></a>Доступ к службам Creator

Службы Creator (Предварительная версия) и службы, использующие данные, размещенные в создателе (например, служба визуализации), доступны по географическому URL-адресу. Географический URL-адрес определяется расположением, выбранным во время создания. Например, если создатель создан в США географическое расположение, все вызовы службы преобразования должны быть отправлены в `us.atlas.microsoft.com/conversion/convert` .

Кроме того, все данные, импортированные в Creator, нужно передавать в то же географическое расположение, где находится ресурс Creator. Например, если Creator подготовлен к работе в США, все необработанные данные нужно отправлять по адресу `us.atlas.microsoft.com/mapData/upload`.

## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о службах Creator (Предварительная версия) для сопоставления помещений:

> [!div class="nextstepaction"]
> [Data — Upload Preview](creator-indoor-maps.md#upload-a-drawing-package) (Данные — предварительная версия отправки)

> [!div class="nextstepaction"]
> [Преобразование данных](creator-indoor-maps.md#convert-a-drawing-package)

> [!div class="nextstepaction"]
> [Набор данных](creator-indoor-maps.md#datasets)

> [!div class="nextstepaction"]
> [Набор фрагментов](creator-indoor-maps.md#tilesets)

> [!div class="nextstepaction"]
> [Набор состояний функции](creator-indoor-maps.md#feature-statesets)

Узнайте, как использовать службы Creator (Предварительная версия) для отображения карт в приложении:

> [!div class="nextstepaction"]
> [Руководство по использованию Creator в Azure Maps](tutorial-creator-indoor-maps.md)

> [!div class="nextstepaction"]
> [Динамические стили для схем помещений](indoor-map-dynamic-styling.md)

> [!div class="nextstepaction"]
> [Использование модуля схем помещений](how-to-use-indoor-module.md)