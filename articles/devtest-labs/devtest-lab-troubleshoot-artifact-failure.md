---
title: Диагностика сбоев артефактов на виртуальной машине Azure DevTest Labs
description: DevTest Labs предоставляют сведения, которые можно использовать для диагностики сбоя артефакта. В этой статье показано, как устранять неполадки артефактов.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 440ce6a537ac8d6a21ae8010bfbb3c38a82bf01e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85480819"
---
# <a name="diagnose-artifact-failures-in-the-lab"></a>Диагностика сбоев артефактов в лаборатории 
После создания артефакта можно проверить, работает ли он. Журналы артефактов в Azure DevTest Labs предоставляют сведения, которые можно использовать для диагностики сбоя артефакта. Есть два способа просмотреть данные журнала артефактов для виртуальной машины Windows:

* На портале Azure
* в виртуальной машине.

> [!NOTE]
> Чтобы можно было правильно идентифицировать и объяснить сбой, важно правильно структурировать артефакт. Сведения о том, как правильно создать артефакт, см. в статье [Создание пользовательских артефактов для виртуальной машины DevTest Labs](devtest-lab-artifact-author.md). Правильно структурированный артефакт можно изучить на примере этого артефакта [тестовых типов параметров](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-test-paramtypes).

## <a name="troubleshoot-artifact-failures-by-using-the-azure-portal"></a>Устранение неполадок при сбоях артефактов с помощью портала Azure

1. На портале Azure в списке ресурсов выберите свою лабораторию.
2. Выберите виртуальную машину Windows, содержащую анализируемый артефакт.
3. На левой панели в разделе **ОБЩИЕ** выберите **Артефакты**. Появится список артефактов, связанных с этой виртуальной машиной. В нем указаны имя и состояние артефакта.

   ![Состояние артефакта](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifacts-failure-new.png)

4. Выберите артефакт, отображаемый состояние **Сбой**. Артефакт откроется и отобразится сообщение расширения, содержащее сведения о сбое артефакта.

   ![Сообщение об ошибке артефакта](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifact-error.png)


## <a name="troubleshoot-artifact-failures-from-within-the-virtual-machine"></a>Устранение неполадок при сбоях артефактов из виртуальной машины

1. Войдите в виртуальную машину, содержащую артефакт, для которого необходимо выполнить диагностику.
2. Перейдите к расположению C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\\*1.9*\Status, где *1.9* — это номер версии расширения пользовательских сценариев Azure.

   ![Файл состояния](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifact-error-vm-status-new.png)

3. Откройте файл **состояния**.

Инструкции по поиску файлов журналов на виртуальной машине **Linux** см. в следующей статье: [Использование расширения пользовательских сценариев Azure версии 2 с виртуальными машинами Linux](../virtual-machines/extensions/custom-script-linux.md#troubleshooting) .


## <a name="related-blog-posts"></a>Связанные записи в блогах
* [Join a VM to existing AD Domain using ARM template in Azure Dev Test Lab](https://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs) (Присоединение виртуальной машины к имеющемуся домену AD с помощью шаблона ARM в Azure DevTest Labs)

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как [добавить в лабораторию репозиторий Git](devtest-lab-add-artifact-repo.md).

