---
title: Получение конечных точек для регистрации приложения Azure AD
titleSuffix: Microsoft identity platform
description: Узнайте, как найти конечные точки проверки подлинности для настраиваемого приложения, разрабатываемого или регистрируемого в Azure AD.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: ryanwi
ROBOTS: NOINDEX
ms.openlocfilehash: 778523869715916bf1e4c32af59ea7a0081df91b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100103317"
---
# <a name="how-to-discover-endpoints"></a>Обнаружение конечных точек

Конечные точки проверки подлинности для приложения можно найти на [портале Azure](https://portal.azure.com).

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений**, а затем в верхнем меню выберите **конечные точки** .

    Отобразится страница **конечные точки** , в которой отображаются конечные точки проверки подлинности для вашего клиента.
    
    Используйте конечную точку, совпадающую с используемым протоколом проверки подлинности, в сочетании с **идентификатором приложения (клиента)** для создания запроса проверки подлинности, относящегося к приложению.

В **национальных облаках** (например, в Azure AD для Китая, Германии и США) есть собственный портал регистрации приложений и конечные точки проверки подлинности Azure AD. Дополнительные сведения см. в [обзоре национальных облаков](authentication-national-cloud.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о конечных точках в различных средах Azure см. в [обзоре национальных облаков](authentication-national-cloud.md).
