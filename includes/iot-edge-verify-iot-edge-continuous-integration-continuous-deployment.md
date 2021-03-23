---
author: v-tcassi
ms.service: iot-edge
ms.topic: include
ms.date: 08/26/2020
ms.author: v-tcassi
ms.openlocfilehash: b5450e4846c3c49c89830ae65c50a95ee0c8d6eb
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104803345"
---
## <a name="verify-iot-edge-cicd-with-the-build-and-release-pipelines"></a>Проверка CI/CD для IoT Edge с использованием конвейеров сборки и выпуска

Чтобы запустить задание сборки, можно создать фиксацию в репозитории исходного кода или активировать задание вручную. В этом разделе вы вручную запускаете конвейер CI/CD для проверки правильности его работы. Затем вы проверите успешность развертывания.

1. В левой части меню выберите **конвейеры** и откройте конвейер сборки, созданный в начале этой статьи.

2. Вы можете активировать задание сборки в конвейере сборки, нажав кнопку **запустить конвейер** в правом верхнем углу.

    ![Запуск конвейера сборки вручную с помощью кнопки "запустить конвейер"](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/manual-trigger.png)

3. Проверьте параметры **запуска конвейера** . Затем выберите **выполнить**.

    ![Укажите параметры запуска конвейера и выберите выполнить.](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/run-pipeline-settings.png)

4. Выберите **Задание агента 1** , чтобы просмотреть ход выполнения. Чтобы проверить журналы выходных данных задания, выберите задание. 

    ![Проверка выходных данных журнала задания](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/view-job-run.png)

5. Если конвейер сборки успешно завершен, он активирует выпуск на этапе **разработки** . В успешном выпуске **dev** создается IOT Edge развертывание для целевых устройств IOT Edge.

    ![Выпуск для разработки](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/pending-approval.png)

6. Щелкните этап **разработки** , чтобы просмотреть журналы выпуска.

    ![Журналы выпуска](./media/iot-edge-verify-iot-edge-continuous-integration-continuous-deployment/release-logs.png)

7. Если происходит сбой конвейера, начните с просмотра журналов. Чтобы просмотреть журналы, перейдите к сводке выполнения конвейера и выберите задание и задачу. В случае сбоя определенной задачи Проверьте журналы этой задачи. Подробные инструкции по настройке и использованию журналов см. [в статье анализ журналов для диагностики проблем конвейера](/azure/devops/pipelines/troubleshooting/review-logs).
