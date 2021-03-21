---
title: Подключение ServiceNow с Соединитель управления ИТ-услугами
description: Узнайте, как подключиться к ServiceNow с помощью Соединитель управления ИТ-услугами (ITSMC) в Azure Monitor, чтобы централизованно отслеживать рабочие элементы ITSM и управлять ими.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 12/21/2020
ms.openlocfilehash: 51d16ae53e34dacfc41e3d31b01115f9c55ca9f7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102039552"
---
# <a name="connect-servicenow-with-it-service-management-connector"></a>Подключение ServiceNow с Соединитель управления ИТ-услугами

В этой статье показано, как настроить подключение между экземпляром ServiceNow и Соединитель управления ИТ-услугами (ITSMC) в Log Analytics, чтобы можно было централизованно управлять рабочими элементами управления ИТ-службами (ITSM).

## <a name="prerequisites"></a>Предварительные условия
Убедитесь, что выполнены следующие предварительные требования для подключения.

### <a name="itsmc-installation"></a>Установка ITSMC

Сведения об установке ITSMC см. [в разделе Добавление решения соединитель управления ИТ-услугами](./itsmc-definition.md#add-it-service-management-connector).

> [!NOTE]
> ITSMC поддерживает только официальное предложение SaaS (программное обеспечение как услуга) из ServiceNow. Частные развертывания ServiceNow не поддерживаются.

### <a name="oauth-setup"></a>Установка OAuth

Поддерживаемые версии ServiceNow включают Париж, Орландо, Нью Йорк, Мадрид, Лондон, Kingston), Джакарта, Стамбул, Хельсинки и Geneva.

Администраторы ServiceNow должны создать идентификатор клиента и секрет клиента для своего экземпляра ServiceNow. При необходимости ознакомьтесь со следующими сведениями.

- [Настройка OAuth для Париж](https://docs.servicenow.com/bundle/paris-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для Орландо](https://docs.servicenow.com/bundle/orlando-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии New York](https://docs.servicenow.com/bundle/newyork-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Madrid](https://docs.servicenow.com/bundle/madrid-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии London](https://docs.servicenow.com/bundle/london-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Kingston](https://docs.servicenow.com/bundle/kingston-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Jakarta](https://docs.servicenow.com/bundle/jakarta-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Istanbul](https://docs.servicenow.com/bundle/istanbul-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Helsinki](https://docs.servicenow.com/bundle/helsinki-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
- [Настройка OAuth для версии Geneva](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/security/task/t_SettingUpOAuth.html)

В рамках настройки OAuth рекомендуется:

1. [Создайте конечную точку для доступа клиентов к экземпляру](https://docs.servicenow.com/bundle/newyork-platform-administration/page/administer/security/task/t_CreateEndpointforExternalClients.html).

1. Обновите срок действия маркера обновления:

   1. В области **ServiceNow** найдите **System OAuth** и выберите **реестр приложений**. 
   1. Выберите имя определенного OAuth и измените срок **жизни маркера обновления** на **7 776 000 с** (90 дней). 
   1. Нажмите кнопку **Обновить**. 

1. Установите внутреннюю процедуру, чтобы убедиться, что подключение остается активным. Несколько дней до ожидаемого срока действия маркера обновления. выполните следующие операции:

   1. [Завершите процесс синхронизации вручную для конфигурации СОЕДИНИТЕЛЯ ITSM](./itsmc-resync-servicenow.md).

   1. Отозвать старый маркер обновления. Мы не рекомендуем хранить старые ключи в целях безопасности. 
   
      1. В области **ServiceNow** найдите **System OAuth** и выберите **Управление токенами**. 
      
      1. Выберите Старый токен из списка в соответствии с именем OAuth и датой окончания срока действия.

         ![Снимок экрана, на котором показан список маркеров для OAuth.](media/itsmc-connections-servicenow/snow-system-oauth.png)
      1. Выберите отменить **доступ**  >  **отозвать**.

## <a name="install-the-user-app-and-create-the-user-role"></a>Установка пользовательского приложения и создание роли пользователя

Используйте следующую процедуру, чтобы установить пользовательское приложение службы и создать для него роль пользователя "Интеграция". Эти учетные данные будут использоваться для создания подключения ServiceNow в Azure.

> [!NOTE]
> ITSMC поддерживает только официальное пользовательское приложение для интеграции с Microsoft Log Analytics, которое загружается из магазина ServiceNow. ITSMC не поддерживает прием кода на стороне ServiceNow или в любом приложении, которое не является частью официального решения ServiceNow. 

1. Посетите [магазин servicenow](https://store.servicenow.com/sn_appstore_store.do#!/store/application/ab0265b2dbd53200d36cdc50cf961980/1.0.1) и установите **пользовательское приложение для servicenow и ИНТЕГРАЦИЮ с Microsoft OMS** в экземпляр servicenow.
   
   >[!NOTE]
   >В рамках текущего перехода от Microsoft Operations Management Suite (OMS) к Azure Monitor OMS теперь называется Log Analytics.     
2. После установки перейдите на левую панель навигации экземпляра ServiceNow, а затем найдите и выберите **Microsoft OMS Integrator**.  
3. Выберите **Контрольный список установки**.

   Состояние отображается как  **не завершено** , так как роль пользователя еще не создана.

4. В текстовом поле рядом с полем **создать пользователя интеграции** введите имя пользователя, который может подключаться к ITSMC в Azure.
5. Введите пароль для этого пользователя и нажмите кнопку **ОК**.  

Созданный пользователь отобразится с назначенными ролями по умолчанию:

- personalize_choices;
- import_transformer;
- x_mioms_microsoft.user;
- itil;
- template_editor;
- view_changer.

После успешного создания пользователя состояние **Контрольный список установки** переместится на **завершенный** и отобразит список сведений о роли пользователя, созданной для приложения.

> [!NOTE]
> ITSMC может отсылать инциденты в ServiceNow без каких бы то ни было других модулей, установленных в вашем экземпляре ServiceNow. Если вы используете модуль используется в своем экземпляре ServiceNow и хотите создавать события или оповещения в ServiceNow с помощью соединителя, добавьте следующие роли для пользователя интеграции:
> 
> - evt_mgmt_integration;
> - evt_mgmt_operator.  

## <a name="create-a-connection"></a>Создание подключения
Используйте следующую процедуру для создания подключения ServiceNow.

> [!NOTE]
> Оповещения, отправляемые из Azure Monitor, могут создавать один из следующих элементов в ServiceNow: события, инциденты или оповещения. 

1. В портал Azure перейдите ко **всем ресурсам** и найдите **ServiceDesk (йоурворкспаценаме)**.

2. В разделе **Источники данных рабочей области** выберите **ITSM Connections (подключения**).

   ![Снимок экрана, на котором показан выбор источника данных.](media/itsmc-overview/add-new-itsm-connection.png)

3. В верхней части правой панели выберите **Добавить**.

4. Укажите сведения, как описано в следующей таблице, а затем нажмите кнопку **ОК**.

   | **Поле** | **Описание** |
   | --- | --- |
   | **Имя соединения**   | Введите имя экземпляра ServiceNow, который вы хотите подключить с помощью ITSMC. Это имя используется позже в Log Analytics при настройке рабочих элементов ITSM и просмотре подробной аналитики. |
   | **Тип партнера**   | Выберите **ServiceNow**. |
   | **URL-адрес сервера**   | Введите URL-адрес экземпляра ServiceNow, который вы хотите подключить к ITSMC. URL-адрес должен указывать на поддерживаемую версию SaaS с суффиксом *. servicenow.com* (например, `https://XXXXX.service-now.com/` ).|
   | **Имя пользователя**   | Введите имя пользователя интеграции, созданное в приложении ServiceNow, для поддержки подключения к ITSMC.|
   | **Пароль**   | Введите пароль, связанный с этим именем пользователя. **Примечание**. имя пользователя и пароль используются только для создания маркеров проверки подлинности. Они хранятся не в любом месте службы ITSMC.  |
   | **Идентификатор клиента**   | Введите идентификатор клиента, который вы хотите использовать для проверки подлинности OAuth2, созданной ранее. Дополнительные сведения о создании идентификатора клиента и секрета см. в разделе [Настройка OAuth](https://old.wiki/index.php/OAuth_Setup). |
   | **Секрет клиента**   | Введите секрет клиента, созданный для этого идентификатора.   |
   | **Область синхронизации данных (в днях)** | Введите число за последние дней, из которых должны быть данные. Ограничение составляет 120 дней. |
   | **Рабочие элементы для синхронизации**   | Выберите рабочие элементы ServiceNow, которые вы хотите синхронизировать с Log Analytics Azure, с помощью ITSMC. Выбранные значения импортируются в Log Analytics. Параметры — это инциденты и запросы на изменение.|
   | **Создание нового элемента конфигурации в продукте ITSM** | Выберите этот параметр, если вы хотите создать элементы конфигурации в продукте ITSM. Если этот параметр выбран, ITSMC создает элементы конфигурации (если они отсутствуют) в поддерживаемой системе ITSM. Она отключена по умолчанию. |

![Снимок экрана с окнами и параметрами для добавления подключения ServiceNow.](media/itsmc-connections-servicenow/itsm-connection-servicenow-connection-latest.png)

После успешного подключения и синхронизации:

- Выбранные рабочие элементы из экземпляра ServiceNow импортируются в Log Analytics. Сводку этих рабочих элементов можно просмотреть на плитке **соединитель управления ИТ-услугами** .

- Инциденты можно создавать на основе оповещений Log Analytics или записей журнала, а также из оповещений Azure в этом экземпляре ServiceNow.

> [!NOTE]
> Для ServiceNow предусмотрено ограничение скорости запросов в час. Чтобы настроить ограничение, определите **REST API ограничения скорости для входящего трафика** в экземпляре ServiceNow.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор соединитель ITSM](itsmc-overview.md)
* [Создание рабочих элементов ITSM из оповещений Azure](./itsmc-definition.md#create-itsm-work-items-from-azure-alerts)
* [Устранение неполадок в соединитель ITSM](./itsmc-resync-servicenow.md)
