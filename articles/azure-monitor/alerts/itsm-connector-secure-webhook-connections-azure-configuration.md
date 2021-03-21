---
title: Соединитель управления ИТ-услугами — безопасный экспорт в Azure Monitor в конфигурациях Azure
description: В этой статье показано, как настроить Azure для подключения продуктов и служб ITSM к безопасному экспорту в Azure Monitor для централизованного мониторинга рабочих элементов ITSM и управления ими.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 01/03/2021
ms.openlocfilehash: 8eb9430e3d280c52cf84c61f0a44cb12152ac054
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102037546"
---
# <a name="configure-azure-to-connect-itsm-tools-using-secure-export"></a>Настройка Azure для подключения средств ITSM с помощью безопасного экспорта

В этой статье содержатся сведения о настройке Azure с целью использования "безопасного экспорта".
Чтобы использовать "безопасный экспорт", выполните следующие действия.

1. [Зарегистрируйте приложение в Azure AD.](./itsm-connector-secure-webhook-connections-azure-configuration.md#register-with-azure-active-directory)
1. [Определение субъекта-службы.](./itsm-connector-secure-webhook-connections-azure-configuration.md#define-service-principal)
1. [Создайте безопасную группу действий веб-перехватчика.](./itsm-connector-secure-webhook-connections-azure-configuration.md#create-a-secure-webhook-action-group)
1. Настройте среду партнера.
    Безопасный экспорт поддерживает подключения со следующими инструментами ITSM:
    * [ServiceNow](./itsmc-secure-webhook-connections-servicenow.md)
    * [Хеликс BMC](./itsmc-secure-webhook-connections-bmc.md)

## <a name="register-with-azure-active-directory"></a>Регистрация в Azure Active Directory

Чтобы зарегистрировать приложение в Azure AD, выполните следующие действия.

1. Выполните инструкции, указанные в статье [Регистрация приложения с помощью платформы удостоверений Майкрософт](../../active-directory/develop/quickstart-register-app.md).
2. В Azure AD выберите **открыть приложение**.
3. Выберите **задать** для **URI идентификатора приложения**.

   [![Снимок экрана с параметром для задания U R I для приложения I D.](media/itsm-connector-secure-webhook-connections-azure-configuration/azure-ad.png)](media/itsm-connector-secure-webhook-connections-azure-configuration/azure-ad-expand.png#lightbox)
4. Щелкните **Сохранить**.

## <a name="define-service-principal"></a>Определение субъекта-службы

Служба группы действий — это приложение первого производителя, поэтому оно имеет разрешение на получение маркеров проверки подлинности из приложения AAD для проверки подлинности с помощью службы.
В качестве дополнительного шага можно определить роль приложения в манифесте созданного приложения, что позволит дополнительно ограничить доступ таким образом, чтобы сообщения могли быть отправлены только определенным приложениям с этой конкретной ролью. Эта роль должна быть назначена субъекту-службе группы действий (требуются права администратора клиента).

Этот шаг можно выполнить с помощью одних и тех же [команд PowerShell](../alerts/action-groups.md#secure-webhook-powershell-script).

## <a name="create-a-secure-webhook-action-group"></a>Создание безопасной группы действий веб-перехватчика

После регистрации приложения в Azure AD вы можете создавать рабочие элементы в средстве ITSM на основе оповещений Azure с помощью действия "безопасное веб-перехватчик" в группах действий.

Группы действий предоставляют модульный и многократно используемый способ активации действий для оповещений Azure. Группы действий можно использовать с оповещениями метрик, оповещениями журнала действий и оповещениями Log Analytics Azure в портал Azure.
Дополнительные сведения о группах действия см. в статье [Создание групп действий и управление ими на портале Azure](../alerts/action-groups.md).

Чтобы добавить веб-перехватчик к действию, выполните следующие инструкции по безопасному веб-перехватчику:

1. На [портале Azure](https://portal.azure.com/) найдите и выберите область **Монитор**. В области **Монитор** объединены все параметры мониторинга и данные.
2. Выберите **оповещения**  >  **Управление действиями**.
3. Выберите [Add action group](../alerts/action-groups.md#create-an-action-group-by-using-the-azure-portal) (Добавить группу действий) и заполните поля.
4. Введите имя в поля **Action group name** (Имя группы действий) и **Short name** (Короткое имя). Короткое имя используется вместо полного имени группы действий при отправке уведомлений с помощью этой группы.
5. Выберите **безопасный веб-перехватчик**.
6. Выберите следующие сведения:
   1. Выберите идентификатор объекта Azure Active Directory зарегистрированного экземпляра.
   2. Для универсального кода ресурса вставьте URL-адрес обработчика, скопированный из [среды средства ITSM](#configure-the-itsm-tool-environment).
   3. Установите для **параметра включить общую схему предупреждений** значение **Да**. 

   На следующем рисунке показана конфигурация примера безопасного действия веб-перехватчика.

   ![Снимок экрана, на котором показана защита действия веб-перехватчика.](media/itsm-connector-secure-webhook-connections-azure-configuration/secure-webhook.png)

## <a name="configure-the-itsm-tool-environment"></a>Настройка среды средства ITSM

Конфигурация состоит из двух этапов:

1. Получите универсальный код ресурса (URI) для определения безопасного экспорта.
2. Определения в соответствии с потоком средства ITSM.

## <a name="next-steps"></a>Дальнейшие действия

* [Конфигурация безопасного экспорта ServiceNow](./itsmc-secure-webhook-connections-servicenow.md)
* [Конфигурация безопасного экспорта BMC](./itsmc-secure-webhook-connections-bmc.md)
