---
title: Руководство по веб-приложению. Создание веб-приложения
description: В этом руководстве рассматривается пример развертывания простого веб-приложения. Из этого раздела руководства вы узнаете, как написать веб-приложение.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: bb9c206a17a11f0cf710ac4ee3ac7ccae0fa9eb9
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020776"
---
# <a name="write-azure-web-application-to-read-fhir-data"></a>Создание веб-приложения Azure для чтения данных FHIR
Вы уже можете установить подключение к серверу FHIR и отправить данные в запросе POST, а теперь создадите веб-приложение, которое будет считывать данные FHIR. На этом заключительном этапе руководства мы рассмотрим процессы создания и использования веб-приложения.

## <a name="create-web-application"></a>Создание веб-приложения
В Azure выберите действие **Создать ресурс** и вариант **Веб-приложение**. Обязательно присвойте веб-приложению то же имя, которое вы указали в URI перенаправления для клиентского приложения, или вернитесь к тому шагу и укажите новый URI перенаправления. 

![Создание веб-приложения](media/tutorial-web-app/create-web-app.png)

Когда веб-приложение станет доступно, щелкните **Перейти к ресурсу**. Справа, в разделе "Средства разработки", выберите **Редактор службы приложений (предварительная версия)** и **Перейти**. При нажатии кнопки "Перейти" откроется Редактор службы приложений. Щелкните правой кнопкой мыши серое пространство в разделе *Исследовать* и создайте новый файл с именем **index.html**.

Ниже приведен код, который можно поместить в файл **index.html**. Вам придется обновить в нем следующие элементы:
* **clientId** — обновите идентификатор клиентского приложения. Это будет тот же идентификатор, который вы получили при извлечении маркера.
* **authority** — укажите идентификатор клиента Azure AD.
* **FHIRendpoint** — для параметра FHIRendpoint укажите имя службы FHIR.
* **scopes** — укажите полный URL-адрес аудитории.

``` HTML

<!DOCTYPE html>
<html>

<head>
    <title>FHIR Patient browser sample app</title>
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.js"></script>
</head>

<body>
    <div class="leftContainer">
        <p id="WelcomeMessage">Welcome to the FHIR Patient browsing sample Application</p>
        <button id="SignIn" onclick="signIn()">Sign In</button>
    </div>

    <div id="patientTable">
    </div>

    <script>
        var msalConfig = {
            auth: {
                clientId: '<CLIENT-ID>',
                authority: "https://login.microsoftonline.com/<AZURE-AD-TENANT-ID>"
            },
            cache: {
                cacheLocation: "localStorage",
                storeAuthStateInCookie: true
            }
        }

        var FHIRConfig = {
            FHIRendpoint: "https://<FHIR-SERVER-NAME>.azurehealthcareapis.com"
        }
        var requestObj = {
            scopes: ["https://<FHIR-SERVER-NAME>.azurehealthcareapis.com/user_impersonation"]
        }

        function authRedirectCallBack(error, response) {
            if (error) {
                console.log(error);
            } else {
                if (response.tokenType === "access_token") {
                    callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, response.accessToken, FHIRCallback);
                }
            }
        }

        var myMSALObj = new Msal.UserAgentApplication(msalConfig);
        myMSALObj.handleRedirectCallback(authRedirectCallBack);

        function signIn() {
            myMSALObj.loginPopup(requestObj).then(function (loginResponse) {
                showWelcomeMessage();
                acquireTokenPopupAndCallFHIRServer();
            }).catch(function (error) {
                console.log(error);
            })
        }

        function showWelcomeMessage() {
            var divWelcome = document.getElementById('WelcomeMessage');
            divWelcome.innerHTML = "Welcome " + myMSALObj.getAccount().userName + " to FHIR Patient Browsing App";
            var loginbutton = document.getElementById('SignIn');
            loginbutton.innerHTML = 'Sign Out';
            loginbutton.setAttribute('onclick', 'signOut()')
        }

        function signOut() {
            myMSALObj.logout();
        }

        function acquireTokenPopupAndCallFHIRServer() {
            myMSALObj.acquireTokenSilent(requestObj).then(function (tokenResponse) {
                callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
            }).catch(function (error) {
                console.log(error);
                if (requiresInteraction(error.errorCode)) {
                    myMSALObj.acquireTokenPopup(requestObj).then(function (tokenResponse) {
                        callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
                    }).catch(function (error) {
                        console.log(error);
                    })
                }
            });
        }

        function callFHIRServer(theUrl, method, message, accessToken, callBack) {
            var xmlHttp = new XMLHttpRequest();
            xmlHttp.onreadystatechange = function () {
                if (this.readyState == 4 && this.status == 200)
                    callBack(JSON.parse(this.responseText));
            }
            xmlHttp.open(method, theUrl, true);
            xmlHttp.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
            xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
            xmlHttp.send(message);
        }

        function FHIRCallback(data) {
            patientListHtml = '<ol>';
            data.entry.forEach(function(e) {
                patientListHtml += '<li>' + e.resource.name[0].family + ', ' + e.resource.name[0].given + ' (' + e.resource.id + ')';
            });
            patientListHtml += '</ol>';
            document.getElementById("patientTable").innerHTML = patientListHtml;
        }
    </script>
</body>

</html>
```

На этом этапе можно вернуться к ресурсу веб-приложения и открыть URL-адрес на странице общих сведений. Войдите в систему, чтобы отобразились данные ранее созданного пациента James Tiberious Kirk.

## <a name="next-steps"></a>Next Steps
Итак, вы успешно развернули Azure API для FHIR, зарегистрировали общедоступное клиентское приложение, проверили доступ к нему и создали небольшое веб-приложение. На следующем шаге ознакомьтесь поддерживаемыми возможностями Azure API для FHIR.

>[!div class="nextstepaction"]
>[Поддерживаемые возможности](fhir-features-supported.md)





