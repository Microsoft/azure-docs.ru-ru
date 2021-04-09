---
title: Что такое Azure AD Connect с расширением Azure AD Connect Health. | Документы Майкрософт
description: Сведения о средствах, используемых для синхронизации и мониторинга вашей локальной среды в Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 01/08/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d8e1af1848405441088796d2e3b42e7b52eedba8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98065122"
---
# <a name="what-is-azure-ad-connect"></a>Что такое Azure AD Connect?

Azure AD Connect — это инструмент Майкрософт для достижения ваших целей гибридной идентификации.  Она предоставляет следующие возможности.
     
- [Синхронизация хэша паролей](whatis-phs.md). Метод входа, который синхронизирует хэш паролей локальных пользователей AD с Azure AD.
- [Сквозная аутентификация](how-to-connect-pta.md). Метод входа, который позволяет использовать один и тот же пароль в локальной среде и облаке, но не требует дополнительной инфраструктуры федеративной среды.
- [Интеграция федерации](how-to-connect-fed-whatis.md). Федерация — необязательная часть Azure AD Connect, которая позволяет настроить гибридную среду с помощью локальной инфраструктуры AD FS. Федерация также предоставляет возможности управления AD FS, такие как обновление сертификата и дополнительные развертывания сервера AD FS.
- [Синхронизация](how-to-connect-sync-whatis.md). Отвечает за создание пользователей, групп и других объектов.  Она также отвечает за согласованность сведений о пользователях и группах в локальной среде и в облаке.  Эта синхронизация также включает хэши паролей.
- [Мониторинг работоспособности.](whatis-azure-ad-connect.md#what-is-azure-ad-connect-health) Компонент Azure AD Connect Health реализует надежный мониторинг и предоставляет центральное расположение на портале Azure для просмотра связанных действий. 


![Что такое Azure AD Connect?](./media/whatis-hybrid-identity/arch.png)



## <a name="what-is-azure-ad-connect-health"></a>Что такое Azure AD Connect Health?

Azure Active Directory (Azure AD) Connect Health предоставляет надежный мониторинг локальной инфраструктуры идентификации. Это решение обеспечивает надежное подключение к службам Microsoft 365 и Microsoft Online Services.  Надежность достигается за счет мониторинга ключевых компонентов идентификации. С помощью этой службы пользователи могут с легкостью получить доступ к общим сведениям об этих компонентах.

Эти сведения можно найти на портале [Azure AD Connect Health](https://aka.ms/aadconnecthealth). На портале Azure AD Connect Health можно просматривать оповещения, отслеживать производительность, аналитику использования и другие сведения. Служба Azure AD Connect Health позволяет следить за состоянием работоспособности ключевых компонентов идентификации из одного места.

![Что такое Azure AD Connect Health](./media/whatis-hybrid-identity-health/aadconnecthealth2.png)

## <a name="why-use-azure-ad-connect"></a>Преимущества Azure AD Connect
Интеграция локальных каталогов с Azure AD помогает повысить продуктивность пользователей, так как используется единая идентификация для доступа к облачным и локальным ресурсам. Пользователи и организации получат следующие преимущества:

* Пользователи могут применить единое удостоверение для доступа к локальным приложениям и облачным службам, например Microsoft 365.
* Единое средство, обеспечивающее удобное развертывание для синхронизации и входа в систему.
* Предоставляет новые возможности для ваших сценариев. Azure AD Connect заменяет старые версии средств интеграции удостоверений, такие как DirSync и Azure AD Sync. Дополнительные сведения см. в статье [Сравнение инструментов интеграции каталогов гибридной идентификации](plan-hybrid-identity-design-considerations-tools-comparison.md).

## <a name="why-use-azure-ad-connect-health"></a>Зачем использовать Azure AD Connect Health?
Интеграция с Azure AD помогает повысить продуктивность ваших пользователей, предоставляя им единую идентификацию для доступа к облачным и локальным ресурсам. Обеспечение надежности среды, чтобы пользователи могли получать доступ к этим ресурсам, является непростой задачей.  Служба Azure AD Connect Health помогает отслеживать и получать полезные сведения о локальной инфраструктуре идентификации, таким образом обеспечивая надежность этой среды. Использовать службу так же просто, как установить агент на локальных серверах удостоверений.

Azure AD Connect Health для AD FS поддерживает AD FS 2.0 в Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 и Windows Server 2016. Поддерживается также мониторинг прокси-серверов AD FS и веб-приложений, которые обеспечивают проверку подлинности для доступа к экстрасети. Azure AD Connect Health для AD FS предоставляет следующий набор основных возможностей с помощью простой и быстрой установки агента работоспособности:

Основные преимущества и рекомендации

|Основные преимущества|Рекомендации|
|-----|-----|
|Повышенная безопасность|[Тенденции блокировки экстрасети](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs).</br>[Отчет о неудачных попытках входа в систему](how-to-connect-health-adfs-risky-ip.md).</br>[Соответствие требованиям конфиденциальности](reference-connect-health-user-privacy.md).|
|Оповещения обо [всех критических проблемах системы ADFS](how-to-connect-health-alert-catalog.md#alerts-for-active-directory-federation-services).|Конфигурация и доступность сервера.</br>[Производительность и возможность подключения](how-to-connect-health-adfs.md#performance-monitoring-for-ad-fs).</br>Регулярное обслуживание.|
|Простота развертывания и управления.|[Быстрая установка агента](how-to-connect-health-agent-install.md#install-the-agent-for-ad-fs).</br>Автообновление агента до последней версии.</br>Получение доступа к данным на портале за считаные минуты.|
Расширенные [метрики использования](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs).|Основные приложения.</br>Расположение в сети и TCP-подключение.</br>Количество запросов маркера на каждый сервер.|
|Удобство работы для пользователей|Панель мониторинга на портале Azure.</br>[Оповещения по электронной почте](how-to-connect-health-adfs.md#alerts-for-ad-fs).|


## <a name="license-requirements-for-using-azure-ad-connect"></a>Лицензионные требования для Azure AD Connect

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]

## <a name="license-requirements-for-using-azure-ad-connect-health"></a>Лицензионные требования для Azure AD Connect Health
[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-p1-license.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Оборудование и предварительные требования](how-to-connect-install-prerequisites.md) 
- [Стандартные параметры](how-to-connect-install-express.md)
- [Настраиваемые параметры](how-to-connect-install-custom.md)
- [Установка агентов Azure AD Connect Health](how-to-connect-health-agent-install.md)
