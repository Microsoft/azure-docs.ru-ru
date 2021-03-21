---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 02/09/2021
ms.author: alkohli
ms.openlocfilehash: 71f18cf8448060385ea38be9b2719b1ed545c5d2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100545424"
---
> [!NOTE] 
> После создания контейнера томов вы не сможете изменить связанные с ним параметры шифрования и учетные данные учетной записи хранения.

#### <a name="to-modify-a-volume-container"></a>Изменение контейнера томов

1. Перейдите в службу StorSimple диспетчер устройств и перейдите в раздел **управление > контейнеры томов**.

2. В таблице со списком контейнеров томов выберите тот, который требуется изменить. На странице **Устройства** выберите устройство, дважды щелкните его, затем перейдите на вкладку **Контейнеры томов**.

3. В таблице со списком контейнеров томов выберите тот, который требуется изменить. На панели команд в открывшейся колонке щелкните **Изменить**.

    ![Изменение контейнера томов](./media/storsimple-8000-modify-volume-container/modify-volume-container-01.png)

4. В колонке **изменение контейнера томов** выполните следующие действия.
   
   1. После указания имени, ключа шифрования и учетной записи хранения, связанных с контейнером томов, изменить эти значения невозможно. Измените соответствующий параметр пропускной способности.<!--STEPS NEED WORK. Updated screen doesn't show alternative to Unlimited or subsequent steps if they customize bandwidth. Can we talk them through this (briefly)?-->
      
       ![Изменение параметра пропускной способности](./media/storsimple-8000-modify-volume-container/modify-volume-container-02.png)<!--New graphic based on: modify-volume-container-bw-setting.png-->

   1.  Нажмите кнопку **OK**.<!--If they choose Custom, do they still click OK, or are there more steps?-->

5. На следующей странице диалогового окна **изменения контейнера томов** выполните следующие действия.<!--This step happens only if they choose Custom bandwidth? Are the steps similar to those in "Add volume container," step 3f, above?"-->
   
   1. Выберите из раскрывающегося списка шаблон пропускной способности.
   1. Проверьте параметры расписания для указанного шаблона.
   1. Нажмите кнопку **Сохранить** и подтвердите изменения.
      
       ![Подтверждение изменений](./media/storsimple-8000-modify-volume-container/modify-volume-container-03.png)

      Колонка **Контейнеры томов** обновится, чтобы отразить изменения.
