---
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 10/26/2018
ms.author: raynew
ms.openlocfilehash: c7826b09ef063d572a98fb344f6862cc8310aa86
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87495984"
---
1. Войдите на [портал Azure](https://portal.azure.com) > **Службы восстановления**.
2. Последовательно выберите **Создать ресурс** > **Monitoring + Management** (Мониторинг и управление) > **Backup and Site Recovery**.
3. В поле **Имя** укажите понятное имя для идентификации хранилища. Если у вас есть несколько подписок, выберите нужную.
4. [Создайте новую группу ресурсов](../articles/azure-resource-manager/templates/deploy-portal.md) или выберите существующую. Укажите регион Azure. 
5. Чтобы быстро получить доступ к хранилищу на панели мониторинга, щелкните **Закрепить на панели мониторинга** > **Создать**.

   ![Снимок экрана с вариантами создания хранилища Служб восстановления](./media/site-recovery-create-vault/new-vault-settings.png)

   Новое хранилище появится в колонке **Панель мониторинга** > **Все ресурсы** и на основной странице **Хранилища служб восстановления**.
