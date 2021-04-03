---
title: Написание кода для отслеживания запросов с помощью Azure Application Insights | Документация Майкрософт
description: Написание кода для отслеживания запросов с помощью Application Insights, что позволяет получить профили для ваших запросов.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.custom: devx-track-csharp
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: aaa1d6df9faa20b1a561bfccdfea682af7645c18
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88930253"
---
# <a name="write-code-to-track-requests-with-application-insights"></a>Написание кода для отслеживания запросов с помощью Azure Application Insights

Чтобы просмотреть профили для приложения на странице производительности, Application Insights нужно отслеживать запросы для вашего приложения. Application Insights может автоматически отслеживать запросы для приложений, построенных на уже инструментированных платформах. Например, ASP.net и ASP.Net Core. 

Но для других приложений, например, для рабочих ролей в облачной службе Azure и интерфейсов API без отслеживания состояния в Service Fabric, нужно добавить новый код, который сообщит Application Insights о начале и завершении запроса. После добавления кода телеметрия запросов отправляется в Application Insights. Вы можете просмотреть телеметрию на странице производительности. Для этих запросов собираются профили. 

Чтобы вручную отслеживать запросы, сделайте следующее:

  1. Добавьте следующий код в раннюю точку во времени существования приложения:  

        ```csharp
        using Microsoft.ApplicationInsights.Extensibility;
        ...
        // Replace with your own Application Insights instrumentation key.
        TelemetryConfiguration.Active.InstrumentationKey = "00000000-0000-0000-0000-000000000000";
        ```
      Дополнительные сведения об этой глобальной конфигурации ключа инструментирования см. в статье [Using Service Fabric with Application Insights](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/blob/dev/appinsights/ApplicationInsights.md) (Использование Service Fabric с Application Insights).  

  1. Для всех фрагментов кода, которые необходимо инструментировать, добавьте инструкцию  **using**`StartOperation<RequestTelemetry>`, как показано в следующем примере.

        ```csharp
        using Microsoft.ApplicationInsights;
        using Microsoft.ApplicationInsights.DataContracts;
        ...
        var client = new TelemetryClient();
        ...
        using (var operation = client.StartOperation<RequestTelemetry>("Insert_Your_Custom_Event_Unique_Name"))
        {
          // ... Code I want to profile.
        }
        ```

        Вызов `StartOperation<RequestTelemetry>` в другой области `StartOperation<RequestTelemetry>` не поддерживается. Вместо этого можно использовать `StartOperation<DependencyTelemetry>` во вложенной области. Пример:  
        
        ```csharp
        using (var getDetailsOperation = client.StartOperation<RequestTelemetry>("GetProductDetails"))
        {
        try
        {
          ProductDetail details = new ProductDetail() { Id = productId };
          getDetailsOperation.Telemetry.Properties["ProductId"] = productId.ToString();
        
          // By using DependencyTelemetry, 'GetProductPrice' is correctly linked as part of the 'GetProductDetails' request.
          using (var getPriceOperation = client.StartOperation<DependencyTelemetry>("GetProductPrice"))
          {
              double price = await _priceDataBase.GetAsync(productId);
              if (IsTooCheap(price))
              {
                  throw new PriceTooLowException(productId);
              }
              details.Price = price;
          }
        
          // Similarly, note how 'GetProductReviews' doesn't establish another RequestTelemetry.
          using (var getReviewsOperation = client.StartOperation<DependencyTelemetry>("GetProductReviews"))
          {
              details.Reviews = await _reviewDataBase.GetAsync(productId);
          }
        
          getDetailsOperation.Telemetry.Success = true;
          return details;
        }
        catch(Exception ex)
        {
          getDetailsOperation.Telemetry.Success = false;
        
          // This exception gets linked to the 'GetProductDetails' request telemetry.
          client.TrackException(ex);
          throw;
        }
        }
        ```
