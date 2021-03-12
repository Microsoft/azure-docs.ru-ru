---
title: Руководство. Веб-приложение — настройка Azure API для FHIR
description: В этом руководстве рассматривается пример развертывания простого веб-приложения. В этом первом руководстве приведены предварительные требования и описано развертывание Azure API для FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.custom: devx-track-js
ms.openlocfilehash: cb183b5c8aff018d4dc73819b938b24ad0daa934
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020014"
---
# <a name="deploy-javascript-app-to-read-data-from-fhir-service"></a>Развертывание приложения JavaScript для чтения данных из службы FHIR
В этом руководстве показано, как развернуть небольшое приложение JavaScript, которое считывает данные из службы FHIR. Последовательность действий будет следующей:
1. Развертывание сервера FHIR.
1. Регистрация общедоступного клиентского приложения
1. Проверка доступа к приложению.
1. Создание веб-приложения, считывающего данные FHIR.

## <a name="prerequisites"></a>Предварительные требования
Перед запуском этого набора руководств вам понадобится следующее:
1. Подписка Azure
1. Клиент Azure Active Directory.
1. Установленное приложение [Postman](https://www.getpostman.com/).

> [!NOTE]
> В этом руководстве предполагается, что служба FHIR, приложение Azure AD и пользователи Azure AD находятся в одном арендаторе Azure AD. Если в вашем случае это не так, вы все равно можете следовать инструкциям этого руководства, но дополнительно выполнить некоторые действия, руководствуясь документами, на которые приведены ссылки.

## <a name="deploy-azure-api-for-fhir"></a>Развертывание Azure API для FHIR
Первым шагом в этом руководстве является правильная настройка Azure API для FHIR.

1. Разверните [Azure API для FHIR](fhir-paas-portal-quickstart.md).
1. Развернув Azure API для FHIR, настройте параметры [CORS](configure-cross-origin-resource-sharing.md). Для этого перейдите в Azure API для FHIR и выберите CORS. 
    1. Задайте для параметра **Origins** (Источники) значение *.
    1. Задайте для параметра **Headers** (Заголовки) значение *.
    1. В разделе **Methods** (Методы) установите **Выбрать все**.
    1. Задайте для параметра **Max age** (Максимальный возраст) значение **600**.

## <a name="next-steps"></a>Next Steps
После развертывания Azure API для FHIR вы можете зарегистрировать общедоступное клиентское приложение.

>[!div class="nextstepaction"]
>[Регистрация общедоступного клиентского приложения](tutorial-web-app-public-app-reg.md)
