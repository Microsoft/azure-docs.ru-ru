---
title: Интеграция NPS с проверкой подлинности RADIUS VPN-шлюза для MFA
description: Описывается интеграция аутентификации RADIUS шлюза Azure с NPS-сервером для обеспечения Многофакторной идентификации.
services: vpn-gateway
documentationcenter: na
author: ahmadnyasin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/16/2019
ms.author: genli
ms.openlocfilehash: 208e99f61694f5a81a98dbc649e2a6035f57891b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96018279"
---
# <a name="integrate-azure-vpn-gateway-radius-authentication-with-nps-server-for-multi-factor-authentication"></a>Интеграция аутентификации RADIUS шлюза Azure с NPS-сервером для обеспечения Многофакторной идентификации 

В статье описано, как интегрировать сервер политики сети (NPS-сервер) с аутентификацией RADIUS VPN-шлюза Azure, чтобы обеспечить Многофакторную идентификацию (MFA) для VPN-подключений типа "точка — сеть". 

## <a name="prerequisite"></a>Предварительное требование

Для использования MFA пользователи должны быть добавлены в службу Azure Active Directory (Azure AD), которая должна синхронизироваться с локальной или облачной средой. Кроме того, пользователь должен выполнить процесс автоматической регистрации для использования MFA.  Дополнительные сведения см. в разделе [Настройка учетной записи для двухфакторной проверки подлинности](../active-directory/user-help/multi-factor-authentication-end-user-first-time.md).

## <a name="detailed-steps"></a>Подробные инструкции

### <a name="step-1-create-a-virtual-network-gateway"></a>Шаг 1. Создание шлюза виртуальной сети

1. Войдите на [портал Azure](https://portal.azure.com).
2. В виртуальной сети, в которой будет размещен шлюз виртуальной сети, выберите **Подсети**, а затем выберите **Подсеть шлюза**, чтобы создать подсеть. 

    ![Рисунок: добавление подсети шлюза](./media/vpn-gateway-radiuis-mfa-nsp/gateway-subnet.png)
3. Создайте шлюз виртуальной сети, указав следующие параметры.

    - **Тип шлюза**. Выберите **VPN**.
    - **Тип VPN**. Выберите **На основе маршрута**.
    - **SKU**. Выберите номер SKU в соответствии с требованиями.
    - **Виртуальная сеть**. Выберите виртуальную сеть, в которой была создана подсеть шлюза.

        ![Рисунок: параметры шлюза виртуальной сети](./media/vpn-gateway-radiuis-mfa-nsp/create-vpn-gateway.png)


 
### <a name="step-2-configure-the-nps-for-azure-ad-mfa"></a>Шаг 2. Настройка NPS для Azure AD MFA

1. На сервере NPS [установите расширение NPS для Azure AD MFA](../active-directory/authentication/howto-mfa-nps-extension.md#install-the-nps-extension).
2. Откройте консоль NPS, щелкните правой кнопкой мыши **клиенты RADIUS** и выберите **создать**. Создайте клиент RADIUS, указав следующие параметры:

    - **Понятное имя**. Введите любое имя.
    - **Адрес (IP или DNS)**. Введите подсеть шлюза, созданную на шаге 1.
    - **Общий секрет**. Введите любой секретный ключ и запомните его для последующего использования.

      ![Изображение параметров клиента RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client1.png)

 
3.  На вкладке **Дополнительно** () укажите имя поставщика **RADIUS Standard** и убедитесь, что флажок **Дополнительные параметры** не установлен.

    ![Изображение с дополнительными параметрами клиента RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client2.png)

4. Перейдите в раздел **политики** политики  >  **сети**, дважды щелкните **подключения к политике сервера маршрутизации и удаленного доступа Майкрософт** , выберите **предоставить доступ**, а затем нажмите кнопку **ОК**.

### <a name="step-3-configure-the-virtual-network-gateway"></a>Шаг 3. Настройка шлюза виртуальной сети

1. Войдите на [портал Azure](https://portal.azure.com).
2. Откройте шлюз виртуальной сети, который вы создали. Убедитесь, что для него указан тип шлюза **VPN** и тип VPN **На основе маршрута**.
3. Последовательно выберите **пункты Конфигурация сайта**  >  **настроить сейчас** и укажите следующие параметры.

    - **Пул адресов**. Введите подсеть шлюза, созданную на шаге 1.
    - **Тип проверки подлинности**. Выберите **Проверка подлинности RADIUS**.
    - **IP-адрес сервера**. Введите IP-адрес NPS-сервера.

      ![Рисунок: параметры подключения типа "точка — сеть"](./media/vpn-gateway-radiuis-mfa-nsp/configure-p2s.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Многофакторная идентификация Azure AD](../active-directory/authentication/concept-mfa-howitworks.md)
- [Интеграция существующей инфраструктуры NPS с многофакторной идентификацией Azure AD](../active-directory/authentication/howto-mfa-nps-extension.md)