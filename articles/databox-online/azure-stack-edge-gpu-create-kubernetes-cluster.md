---
title: Создание кластера Kubernetes и управление им на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: В этой статье описывается создание кластера Kubernetes и управление им на устройстве с интерфейсом GPU Pro на Azure Stack с помощью интерфейса Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/08/2021
ms.author: alkohli
ms.openlocfilehash: ddbf3c99845f4c38f511260f4dec274903a9d586
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102631967"
---
# <a name="connect-to-and-manage-a-kubernetes-cluster-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Подключение к кластеру Kubernetes и управление им через kubectl на устройстве с графическим процессором Azure Stack. Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

На устройстве Azure Stack пограничной Pro создается кластер Kubernetes при настройке роли вычислений. После создания кластера Kubernetes можно подключиться к кластеру и управлять им локально с клиентского компьютера с помощью собственного средства, такого как *kubectl*.

В этой статье описывается, как подключиться к кластеру Kubernetes на устройстве с Azure Stack пограничным Pro, а затем управлять им с помощью *kubectl*. 


## <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем.

1. У вас есть доступ к устройству Azure Stack пограничной Pro.

2. Устройство Azure Stack Edge Pro активировано, как описано в статье [Активация Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

3. Вы включили на устройстве роль вычислений. Кластер Kubernetes также был создан на устройстве при настройке вычислений на устройстве в соответствии с инструкциями, приведенными в разделе [Настройка вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

4. У вас есть доступ к клиентской системе Windows с PowerShell 5,0 или более поздней версии для доступа к устройству. Также можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device) . 

5. У вас есть конечная точка API Kubernetes со страницы **устройства** локального веб-интерфейса. Дополнительные сведения см. в инструкциях в статье [Получение KUBERNETES API в конечной точке](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints) .


## <a name="connect-to-powershell-interface"></a>Подключение к интерфейсу PowerShell

После создания кластера Kubernetes можно получить доступ к этому кластеру для создания пространств имен и пользователей и назначения пользователям пространств имен. Для этого потребуется подключиться к интерфейсу PowerShell устройства. Выполните следующие действия в клиенте Windows с PowerShell.

[!INCLUDE [Connect to admin runspace](../../includes/azure-stack-edge-gateway-connect-minishell.md)]


## <a name="configure-cluster-access-via-kubernetes-rbac"></a>Настройка доступа к кластеру через Kubernetes RBAC

После создания кластера Kubernetes можно использовать *kubectl* через cmdline для доступа к кластеру. 

При таком подходе создается пространство имен и пользователь. Затем пользователь связывается с пространством имен. Кроме того, необходимо получить файл *конфигурации* , позволяющий использовать клиент Kubernetes для непосредственного взаимодействия с созданным кластером Kubernetes без подключения к интерфейсу PowerShell устройства Azure Stack ребр Pro.

1. Создание пространства имен. Тип:

    `New-HcsKubernetesNamespace -Namespace <string>` 

    > [!NOTE]
    > Для пространства имен и имен пользователей применяются [соглашения об именовании поддоменов DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) .

    Пример выходных данных:

    `[10.100.10.10]: PS> New-HcsKubernetesNamespace -Namespace "myasetest1"`

2. Создайте пользователя и получите файл конфигурации. Тип:

    `New-HcsKubernetesUser -UserName <string>`

    > [!NOTE]
    > Вы не можете использовать *асеусер* в качестве имени пользователя, так как оно зарезервировано для пользователя по умолчанию, связанного с пространством имен IOT, для Azure Stack пограничной Pro.

    Ниже приведен пример выходных данных файла конфигурации.
   
    ```powershell
    [10.100.10.10]: PS> New-HcsKubernetesUser -UserName "aseuser1"
        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01ERXlPVEUyTlRFeE4xb1hEVE13TURFeU5qRTJOVEV4TjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTXNpCkdZUHB0U1VWcDhKZEdVcHE1MVBURWhsZm8wWkk3YXFBdUlrOHZWdUFCZHJQK1FBSzFxcEN1di93NjIwbUtpZ0QKak1aT3Q4QkREREppWHF6UDZRZm5Oc0U2VXBHMnh0YnYrcTZHV2R5K0t6WkxMbXlwWGY3VjlzZEJnejVKVDNvYQpIdzFja2NTUklHSlV3UWxTbklNaHJUS3JUNDZFUUp3d282TmlNUzZMZDZieVk3WkUrTGg3OS9aNEhLanhTRmhMClc5ZG8veThZR3FXUDZmZTFmMmVmSkhUeGtwR05HZE1UVjNuOFlCZ0pSRzdrNjh0N2MrZ1NhbUlVWVJpTUNSNFAKYlFxcFpscWYvV2REZEJHOFh6aDJ0M1l4SkVIMm00T0Z1cSsvUitMYm95aHdKbmNMdVJ5OEpNZWlwTEQ3UlN0QwpZTDNNR0EzN2JieTRyYm4zVzg4Q0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNbzFwWlBtQzV1cmRPZUJhSWQ4eEQzRkxCMG8KTlErbXBXMWpDd0ZtY3h6dUtlWmRsNXc2N0tuS2JTcDR0TXo1cXg3bUtSc0UxcnBoWkh2VHlKUXg1ZFk2ZE1Kdgp5d2FQZjBpT05TNlU2cC9INE12U1dJaEtJZ1FuTnE1dDh4TjJCNnZpQW41RmZoRkx6WEQrUlZGSm42cnovWkZnCmV6MHpxTkNKYmcvelFucFROcmQ2cnFFRHpoSVFZOVdYVWQycFh3ZXRqUXJpMkpZamh4NmtEcTVoRkZTM0FLUnIKOWlQTVQxaWNkR1NUMFVvM1hIZ1k2ck45WGp3MHFrY2I0Sy83UlVVWlRvS3dKamROR3lNTnpad000L2puR0p5SwpQTE9ycU5Ddlkvb0lkVEM5eVZVY3VRbXVlR0VqT20xUnN1RDFHYVE0RTZwakppVWJpMVdrajJ1bFhOWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
            server: https://10.128.47.90:6443
            name: kubernetes
        contexts:
        - context:
            cluster: kubernetes
            user: aseuser1
            name: aseuser1@kubernetes
        current-context: aseuser1@kubernetes
        kind: Config
        preferences: {}
        users:
        - name: aseuser1
            user:
            client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwVENDQWJtZ0F3SUJBZ0lJWlFXcjY2cGFWSm93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBeE1qa3hOalV4TVRkYUZ3MHlNVEF4TWpneU1qVTJNVGRhTUJNeApFVEFQQmdOVkJBTVRDR0Z6WlhWelpYSXhNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDCkFRRUF4R3RDaXJ2cVhGYm5wVmtaYlVPRWQ3cXg2UzNVZ092MlhHRHNKM2VYWXN0bUxQVjMrQnlBcWwyN211L1AKaWdmaWt6MG9QSW1iYmtvcVJkamlYckxFZnk0N3dHcEhzdUhUOHNLY2tHTnJleFE2bXhaZ29xaU1nL2FuMUpMdwpiOFEvVnlQaWdVdUt6eVBseEhUZmlmSVM5MzR1VnZVZUc0dzlMRjAyZ2s2Nitpc0ZtanhsVmhseWRMNlc2UmZTCjl0OGpNMEFkdEpJL0xNbE13RHJJRVdFKzM4WDVNelJhQkJYNnlzNDFWSkZxekcwdW14dHdxN2pGOXp1UTE4ekIKalRZaDl3OWVKcDJwS2Fvak5tNE9SSDh4SzVSaUhocjJ2anFJWXkxRDd2WDh0b0U1K05HNmxHZjh5L1NvQnNRbQpmOG9vL1k3SEZmQXVGdlN6WUc1RUlQTFM4UUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEClZSMGxCQXd3Q2dZSUt3WUJCUVVIQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNR1BxY0YzS1BCbHZ0K24KN1NOcGE3anhWYkhZVGxyNTgwVWxzek93WEwwVnVPUUlCYmN2djIzZk9HNkhDZlQ0bWxBU0JRWVNZcmpLMjJTTwpTWld4cjNQUDlhVzNHajkxc0ttSnc1ZUF1WFhQbUJpK1RWQzBvY0ZLaEQvZ0o1aC93YnBaVndpVjVyRWE5Kzc2CnhNcFAzRld6dG5tT1hPaEl6UFNlR3B4YWpwQXd3ZXd4QU0yb0xGRFZFcy9XTFFMODJZM3NFcE93NVNaSVJJNXMKUHhMUTVnV1ZPM2x2SXcwZ3IrdkJlanZSOUZKaWVuTWFRdGdjSVgyRmpDaDBRMHVYRkdsTVNXWEljbjRLRTR0TApQSFFMalRSVUwyVnRXcW1YZ1RBM3RzN01DcGNRTFdPZFJUYkpSejZCbkc1aXVwcDdOSlFvYW9YcWpNVk5DVDZCCllYMEd0Skk9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBeEd0Q2lydnFYRmJucFZrWmJVT0VkN3F4NlMzVWdPdjJYR0RzSjNlWFlzdG1MUFYzCitCeUFxbDI3bXUvUGlnZmlrejBvUEltYmJrb3FSZGppWHJMRWZ5NDd3R3BIc3VIVDhzS2NrR05yZXhRNm14WmcKb3FpTWcvYW4xSkx3YjhRL1Z5UGlnVXVLenlQbHhIVGZpZklTOTM0dVZ2VWVHNHc5TEYwMmdrNjYraXNGbWp4bApWaGx5ZEw2VzZSZlM5dDhqTTBBZHRKSS9MTWxNd0RySUVXRSszOFg1TXpSYUJCWDZ5czQxVkpGcXpHMHVteHR3CnE3akY5enVRMTh6QmpUWWg5dzllSnAycEthb2pObTRPUkg4eEs1UmlIaHIydmpxSVl5MUQ3dlg4dG9FNStORzYKbEdmOHkvU29Cc1FtZjhvby9ZN0hGZkF1RnZTellHNUVJUExTOFFJREFRQUJBb0lCQUVVSUVXM2kxMTQycU5raQo5RjNEWWZZV1pscTJZYjRoc0FjTmhWSGxwUTN5d0dsQ3FEUktDQ3BZSVF3MkJqSFR6WnpEM0xWU0E0K0NmMUxuCkE4QVdnaHJVcStsWE1QVzhpcG9DTGJaTlNzUUord0x3bld2dFl0MHFQaGZtd0p2M1UrK1RUQkwyOHNVVUw3ZVkKLzh0aWlhbno3ZU5mNklIMENyZmgxcnQ3WWhsemtRd1hBVHNScVJja0dMaTgrdGN5WnVzdGFhbENUSzBGRTdCaQpBUGE5a2w1SG56eCs4TTcvNWladHkwTUIxYWpWMnlGblBkUmlKSFVCb1AxVVV0QUthYjVZU0RvNllkZ2pIUTRHCjNWN1l1YWZobnVFMXA0VVIvUkloVVdjRlVVaTFBOFpZMFdnd1BDTmhnMWpQZU5vb2Y1UHpRbEY1OTRBREVwUUYKOFR2bG92RUNnWUVBOWZZbUxyY0tlQ0JiMTFoQVhoTi91Z1RTbU5xNnpFL1pPSWl6M0xwckdjRDhvWDdCVW9GcgplelkxbktSS2tkczE5OERnVjlQZUhuNzllQTRoMjM5RkIwNFFhMUJBdUVMRzRsdHJ3VlNxaFBENUR6YkcrSEhSCnJtYThVMEpUSmVVS0tJVjRUUGxlTzFtK2tjbkRJVXY1ckpwZDVXU3RvcUhXdk9RZkEvRUF0VlVDZ1lFQXpHOTcKTitCZVVvbFNiREttVUNGdTdPZGhYSXJYR3RnSEorZ2JOMDlnSHRURG5PY0IxZ1NzNkpZa1FPQU9qbWFxK05lRAp5SUF1NytheWlFRmpyT2tzTGhkSTREUXNkWFZveFFGVko1V1JwWlk3UTVRaFZpYUR2enR4NDlzSDlKSkplM2U0Cnl3NWdpNGkxKy90MnY2eWRKcWdNQ0xxOHlEdFRrcE9PSitkbkp5MENnWUJwZ3lpcURaZU9KTU9CUTdpSkl2QSsKQ21lVmJ1K0hTaEd6TU9HSHBPamc2V3IybEh1Mk94S3lqblM5TjdWTmtLNDhGQitwVFpnUm1RUi9CZ0Q4T2tLUQplYXFOZnFYazViQ1AxZ3dKcVpwazRVTFdoZmNoQ1NLY0lESlZ2VFFTSTRrU0RQK29kYWs0Nkt6WnVhWGRtTXdJCmdVZ2FhZkFhdmpaeVhhSDRmT0NDNlFLQmdHVXJCaDh3dVh5KzJEc1RGWnF4OE9McjNoS2Q0clUyRXRSODJIc1cKbk1xbEgraVZxU0x3VFdFTWJBUnUzTVU3cVlCYnBxdWlRNWdVNG1UcmR4Z3FpK0tEUTEwd2RJL3IrbDBEdTlCTApCRGlkajlaeGg4M0tZWWhSTXBzLzJULys1TDVsRU4zcnozczl2RkZtcisxS3pycENqeklDdDBtZmtrd0hHV0pGCjhaWkJBb0dBVXB3aUIrcWlHbkpxU1FtZHNSZFVabGFBaTRpbGhaa01RYTRHem95ZFQ3OTVHTm44ZThBRjd3WHMKTGpyYjdEV1FwakdCMnZpUlkySUZBVmIyKzZsdDlwOVJRMTZnSmxpNU5ZRXVvQWRoWXBsVWdBZGFHWHNGNHdabwo3SHFHTHBGdmUxVU5Gb0dQdkxpWUNrUFVYdGduQ3dNb0R2SEpKNzVYMXl6ckh6cmxUS1k9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==  
        
    [10.100.10.10]: PS>
    ```
    

3. Файл конфигурации отображается в виде обычного текста. Скопируйте этот файл и сохраните его как файл *конфигурации* . 

    > [!IMPORTANT]
    > Не сохраняйте файл конфигурации в формате *txt* , сохраните файл без расширения.

4. Файл конфигурации должен находиться в `.kube` папке профиля пользователя на локальном компьютере. Скопируйте файл в эту папку в профиле пользователя.

    ![Расположение файла конфигурации на клиенте](media/azure-stack-edge-gpu-create-kubernetes-cluster/location-config-file.png)

5. Свяжите пространство имен с созданным пользователем. Тип:

    `Grant-HcsKubernetesNamespaceAccess -Namespace <string> -UserName <string>`

    Пример выходных данных:

    `[10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace "myasetest1" -UserName "aseuser1"`

    После создания файла конфигурации физический доступ к кластеру не требуется. Если клиент может проверить связь с IP-адресом устройства Azure Stack пограничной Pro, вы сможете направить кластер с помощью команд *kubectl* .

6. Запустите новый сеанс PowerShell на клиенте. Вам не нужно подключаться к интерфейсу устройства. Теперь можно установить `kubectl` на клиенте с помощью следующей команды:

    ```powershell
    PS C:\windows\system32> curl https://storage.googleapis.com/kubernetes-release/release/v1.15.2/bin/windows/amd64/kubectl.exe -O kubectl.exe
    
    PS C:\windows\system32>
    ```
    Например, если на главном узле Kubernetes был запущен v 1.15.2, установите на клиенте v 1.15.2.

    > [!IMPORTANT]
    > Загрузите клиент, версия которого отличается от версии главного узла не более чем на единицу. Версия клиента, но может привести к тому, что основная версия будет вынимать до одной дополнительной версии. Например, мастер v 1.3 должен работать с узлами v 1.1, v 1.2 и v 1.3 и работать с клиентами версии 1.2, v 1.3 и v 1.4. Дополнительные сведения о версии клиента Kubernetes см. в разделе [Политика поддержки отклонения версий и версий Kubernetes](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-version-skew). Дополнительные сведения о версии Kubernetes Server на Azure Stack пограничных Pro см. в статьях получение версии сервера Kubernetes.<!-- insert link-->
    > Иногда `kubectl` предварительно устанавливается в системе, если вы используете DOCKER для Windows или другие средства. Важно скачать определенную версию `kubectl` , как указано в этом разделе, чтобы работать с этим кластером kubernetes. 

    Установка займет несколько минут.

7. Убедитесь, что установленная версия загружена. Необходимо указать абсолютный путь, по которому `kubectl.exe` был установлен в системе.
    
    ```powershell
    PS C:\Users\myuser> C:\windows\system32\kubectl.exe version
    Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
    Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
    PS C:\Users\myuser>
    ``` 
    
    Дополнительные сведения о `kubectl` командах, используемых для управления кластером Kubernetes, см. в [обзоре kubectl](https://kubernetes.io/docs/reference/kubectl/overview/).

8. Добавьте запись DNS в файл hosts в системе. 

    1. Запустите Блокнот от имени администратора и откройте `hosts` файл, расположенный по адресу `C:\windows\system32\drivers\etc\hosts` . 
    2. Используйте сведения, сохраненные на странице **устройства** в локальном пользовательском интерфейсе на предыдущем шаге, чтобы создать запись в файле hosts. 

        Например, скопируйте эту конечную точку, `https://compute.asedevice.microsoftdatabox.com/[10.100.10.10]` чтобы создать следующую запись с IP-адресом устройства и DNS-доменом: 

        `10.100.10.10 compute.asedevice.microsoftdatabox.com`

9. Чтобы убедиться, что можно подключиться к Kubernetes Pod, введите:
    
    ```powershell
    PS C:\Users\myuser> kubectl get pods -n "myasetest1"
    No resources found.
    PS C:\Users\myuser>
    ```
Теперь вы можете развертывать приложения в пространстве имен, а затем просматривать эти приложения и их журналы.

> [!IMPORTANT]   
> Существует множество команд, которые не могут быть запущены, например команды, для которых требуется административный доступ. Вы можете выполнять только операции, разрешенные в пространстве имен.


## <a name="remove-kubernetes-cluster"></a>Удаление кластера Kubernetes

Чтобы удалить кластер Kubernetes, необходимо удалить конфигурацию IoT Edge.

Подробные инструкции см. в разделе [Управление конфигурацией IOT Edge](azure-stack-edge-gpu-manage-compute.md#manage-iot-edge-configuration).
   

## <a name="next-steps"></a>Следующие шаги

- [Развертывание приложения без отслеживания состояния на Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-stateless-application-kubernetes.md).
