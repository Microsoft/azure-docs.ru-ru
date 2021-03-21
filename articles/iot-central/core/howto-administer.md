---
title: Изменение параметров приложения IoT Central Azure | Документация Майкрософт
description: Как администратор, как управлять приложением IoT Central Azure, изменяя имя приложения, URL-адрес, передачу изображения и удаление приложения.
author: viv-liu
ms.author: viviali
ms.date: 12/19/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 81cd2ca1cb47e6fdfb4858df930b73c1bd10118a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101091820"
---
# <a name="change-iot-central-application-settings"></a>Изменение параметров приложения IoT Central



В этой статье описывается, как администратор может управлять приложением, изменяя имя приложения и URL-адрес, отправку образа и удаляя приложение в приложении IoT Central Azure.

Чтобы получить доступ к разделу **Администрирование**, вам должна быть назначена роль **Администратор** приложения Azure IoT Central. Если вы создаете приложение Azure IoT Central, вам автоматически назначается роль **Администратор** для этого приложения.

## <a name="change-application-name-and-url"></a>Изменение имени приложения и URL-адреса

На странице **Параметры приложения** можно изменить имя и URL-адрес приложения, а затем выбрать **Сохранить**.

![Страница "Параметры приложения"](media/howto-administer/image0-a.png)

Если администратор создает пользовательскую тему для приложения, эта страница содержит параметр, позволяющий скрыть **имя приложения** в пользовательском интерфейсе. Этот параметр полезен, если эмблема приложения в пользовательской теме содержит имя приложения. Дополнительные сведения см. [в статье Настройка пользовательского интерфейса IOT Central Azure](./howto-customize-ui.md).

> [!Note]
> Если URL-адрес будет изменен, старый URL-адрес может быть занят другим клиентом Azure IoT Central. В этом случае он будет недоступен для использования. При изменении URL-адреса старый URL-адрес больше нельзя будет использовать, поэтому вам нужно уведомить пользователей о новом URL-адресе.

## <a name="delete-an-application"></a>Удаление приложения

Нажмите кнопку **Удалить**, чтобы окончательно удалить приложение IoT Central. Это действие окончательно удаляет все данные, связанные с приложением.

> [!Note]
> Чтобы удалить приложение, у вас также должны быть разрешения на удаление ресурсов в подписке Azure, которую вы выбрали при создании приложения. Дополнительные сведения см. в статье [назначение ролей Azure для управления доступом к ресурсам подписки Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="manage-programmatically"></a>Программное управление

Пакеты SDK IoT Central Azure Resource Manager доступны для Node, Python, C#, Ruby, Java и Go. Эти пакеты можно использовать для создания, перечисления, обновления или удаления IoT Central приложений. Пакеты содержат вспомогательные методы для управления проверкой подлинности и обработкой ошибок.

Примеры использования пакетов SDK для Azure Resource Manager можно найти по адресу [https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples](https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples) .

Дополнительные сведения см. в следующих репозиториях и пакетах GitHub:

| Язык | Хранилище | Пакет |
| ---------| ---------- | ------- |
| Узел | [https://github.com/Azure/azure-sdk-for-js](https://github.com/Azure/azure-sdk-for-js) | [https://www.npmjs.com/package/@azure/arm-iotcentral](https://www.npmjs.com/package/@azure/arm-iotcentral)
| Python |[https://github.com/Azure/azure-sdk-for-python](https://github.com/Azure/azure-sdk-for-python) | [https://pypi.org/project/azure-mgmt-iotcentral](https://pypi.org/project/azure-mgmt-iotcentral)
| C# | [https://github.com/Azure/azure-sdk-for-net](https://github.com/Azure/azure-sdk-for-net) | [https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral](https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral)
| Ruby | [https://github.com/Azure/azure-sdk-for-ruby](https://github.com/Azure/azure-sdk-for-ruby) | [https://rubygems.org/gems/azure_mgmt_iot_central](https://rubygems.org/gems/azure_mgmt_iot_central)
| Java | [https://github.com/Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java) | [https://search.maven.org/search?q=a:azure-mgmt-iotcentral](https://search.maven.org/search?q=a:azure-mgmt-iotcentral)
| Go | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как администрировать приложение Azure IoT Central, предлагаем следующий шаг: Узнайте, как [управлять пользователями и ролями](howto-manage-users-roles.md) в Azure IOT Central.
