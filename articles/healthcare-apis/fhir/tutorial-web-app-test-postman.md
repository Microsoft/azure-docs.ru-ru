---
title: Руководство по веб-приложению. Тестирование подключения к Azure API для FHIR
description: В этом руководстве рассматривается пример развертывания простого веб-приложения. В этом разделе руководства описано, как тестировать подключение к серверу FHIR с помощью средства Postman.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: 1c64468a2e420734ca51a5b9308bb52e13712c51
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019946"
---
# <a name="testing-the-fhir-api"></a>Тестирование API для FHIR
На двух предыдущих шагах вы развернули Azure API для FHIR и зарегистрировали клиентское приложение. Теперь можно убедиться, что служба Azure API для FHIR правильно настроена для работы с клиентским приложением. 

## <a name="retrieve-capability-statement"></a>Получение отчета о возможностях
Для начала мы получим отчет о возможностях Azure API для FHIR. 
1. Откройте Postman.
1. Чтобы получить отчет о возможностях, выполните запрос GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/metadata. В примере на следующем рисунке сервер FHIR имеет имя **fhirserver**.

![Отчет о возможностях](media/tutorial-web-app/postman-capability-statement.png)

Теперь мы попробуем получить данные о пациенте. Для этого выполните запрос GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/Patient. Вы получите ошибку 401 Unauthorized (Не авторизовано). Эта ошибка возникает из-за того, что вы еще не подтвердили наличие прав на доступ к данным о пациентах.

## <a name="get-patient-from-fhir-server"></a>Получение данных о пациенте с сервера FHIR
![Ошибка при получении данных о пациенте](media/tutorial-web-app/postman-patient-authorization-failed.png)

Чтобы получить такой доступ, вам нужен маркер доступа.
1. В окне Postman выберите **Authorization** (Авторизация) и укажите в поле Type (Тип) значение **OAuth2.0**.
1. Выберите действие **Get New Access Token** (Получить новый маркер доступа).
1. Заполните поля и щелкните **Request Token** (Маркер запроса). Ниже указаны значения каждого поля для работы с этим руководством.

|Поле                |Значение                                                               |
|---------------------|--------------------------------------------------------------------|
|Имя токена           |Имя вашего токена.                                               |
|Тип предоставления разрешения           |Код авторизации                                                  |
|URL-адрес обратного вызова         |https://www.getpostman.com/oauth2/callback                          |
|URL-адрес аутентификации             |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/?resource=https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com|
|Access Token URL (URL-адрес маркера доступа)     |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/token|
|Идентификатор клиента            |Идентификатор клиента, скопированный на предыдущих шагах.             |
|Секрет клиента        |\<BLANK>                                                            |
|Область                |\<BLANK>                                                            |
|Состояние                |1 234                                                                |
|Аутентификация клиента|Отправьте учетные данные клиента в тексте запроса.                                     |

4. Войдите с правильными учетными данными и щелкните **Accept** (Принять).
1. Прокрутите страницу с результатом вниз и выберите **Use Token** (Использовать маркер).
1. Снова щелкните **Send** (Отправить) в верхней части страницы, но теперь вы получите другой результат: ![Success Patient](media/tutorial-web-app/postman-patient-authorization-success.png) (Успешное получение данных о пациенте).

## <a name="post-patient-into-fhir-server"></a>Отправка данных о пациенте на сервер FHIR
Теперь у вас есть доступ, а значит вы можете создать нового пациента. Ниже приведен простой пример данных, с которыми можно добавить пациента на сервер FHIR. Введите приведенный ниже код в раздел **Body** (Текст) в средстве Postman.

``` json
    {
        "resourceType": "Patient",
        "active": true,
        "name": [
            {
                "use": "official",
                "family": "Kirk",
                "given": [
                    "James",
                    "Tiberious"
                ]
            },
            {
                "use": "usual",
                "given": [
                    "Jim"
                ]
            }
        ],
        "gender": "male",
        "birthDate": "1960-12-25"
    }
```
Этот запрос POST создаст на сервере FHIR нового пациента с именем James Tiberious Kirk.
![Отправка пациента](media/tutorial-web-app/postman-post-patient.png)

Если вы снова выполните описанный выше шаг GET для получения данных о пациенте, в выходных данных вы увидите James Tiberious Kirk.

## <a name="troubleshooting-access-issues"></a>Устранение неполадок с доступом
Если во время выполнения какого-либо из этих шагов возникли проблемы, ознакомьтесь с предоставленной документацией по Azure Active Directory и Azure API для FHIR. 

* [Конфигурация удостоверений Azure Active Directory для Azure API для FHIR](azure-ad-hcapi.md) — в этом документе приводятся некоторые основные принципы работы Azure Active Directory и взаимодействие с Azure API для FHIR.
* [Проверка маркера доступа Azure API для FHIR](azure-ad-hcapi-token-validation.md) — это руководство содержит более конкретные сведения о проверке маркера доступа и действиях по устранению неполадок с получением доступа.

## <a name="next-steps"></a>Next Steps
Теперь, когда вы можете успешно подключаться к клиентскому приложению, можете перейти к созданию веб-приложения.

>[!div class="nextstepaction"]
>[Создание веб-приложения](tutorial-web-app-write-web-app.md)



