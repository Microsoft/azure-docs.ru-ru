---
title: удаленный рабочий стол служб шлюза с Azure Active Directory
description: Рекомендации по архитектуре для достижения удаленный рабочий стол служб шлюза с Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4baaf2de6fbe4a56f64d449644b8594217dc432c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96172743"
---
# <a name="remote-desktop-gateway-services"></a>Службы шлюза удаленный рабочий стол

Развертывание Standard службы удаленных рабочих столов (RDS) включает различные [службы ролей удаленный рабочий стол](/windows-server/remote/remote-desktop-services/Desktop-hosting-logical-architecture) , работающие под Windows Server. В развертывании RDS с помощью прокси приложения Azure Active Directory (Azure AD) имеется постоянное исходящее подключение с сервера, на котором запущена служба соединителя. Другие развертывания оставляют открытыми входящие подключения через подсистему балансировки нагрузки. Этот шаблон проверки подлинности позволяет предлагать больше типов приложений, публикуя локальные приложения с помощью службы удаленных рабочих столов. Это также сокращает область атаки развертывания с помощью AD Application Proxy Azure.

## <a name="use-when"></a>Используется, если

Необходимо предоставить удаленный доступ и защитить развертывание службы удаленных рабочих столов с предварительной проверкой подлинности.

![Схема архитектуры](./media/authentication-patterns/rdp-auth.png)

## <a name="components-of-system"></a>Компоненты системы

* **Пользователь**: доступ к RDS, обслуживаемой прокси приложения.

* **Веб-браузер**. компонент, с которым взаимодействует пользователь, чтобы получить доступ к внешнему URL-адресу приложения.

* **Azure AD**: проверяет подлинность пользователя. 

* **Служба прокси приложения**. действует как обратный прокси-сервер для пересылки запроса от пользователя к RDS. Прокси приложения также может применять любые политики условного доступа. 

* **Службы удаленных рабочих столов**. действует как платформа для отдельных виртуализированных приложений, обеспечивая безопасный доступ к мобильным и удаленным рабочим столам, а также предоставление конечным пользователям возможности запуска приложений и настольных систем из облака. 

## <a name="implement-remote-desktop-gateway-services-with-azure-ad"></a>Реализация служб шлюза удаленный рабочий стол с помощью Azure AD

* [Публикация удаленного рабочего стола с помощью Azure AD Application Proxy](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md) 

* [Добавление локального приложения для удаленного доступа через прокси приложения в Azure AD](../manage-apps/application-proxy-add-on-premises-application.md)

