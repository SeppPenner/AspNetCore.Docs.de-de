---
title: Factorybezogene Middlewareaktivierung in ASP.NET Core
author: guardrex
description: Informationen zur Verwendung von stark typisierter Middleware mit einer factorybezogenen Aktivierungsimplementierung von ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/31/2019
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: b4d71c2c7f09acb58b73e84080e8574d77f8b326
ms.sourcegitcommit: dd9c73db7853d87b566eef136d2162f648a43b85
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "65086995"
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>Factorybezogene Middlewareaktivierung in ASP.NET Core

Von [Luke Latham](https://github.com/guardrex)

Bei <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> handelt es sich um einen Erweiterungspunkt für die [Middleware](xref:fundamentals/middleware/index)-Aktivierung.

Die Erweiterungsmethode <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> überprüft, ob der registrierte Typ einer Middleware <xref:Microsoft.AspNetCore.Http.IMiddleware> implementiert. Falls dies der Fall ist, wird die im Container registrierte <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>-Instanz im Container verwendet, um die <xref:Microsoft.AspNetCore.Http.IMiddleware>-Implementierung aufzulösen, anstatt die konventionsbasierte Middlewareaktivierungslogik zu verwenden. Die Middleware wird als [bereichsbezogener oder vorübergehender Dienst](xref:fundamentals/dependency-injection#service-lifetimes) im Dienstcontainer der App registriert.

Vorteile:

* Aktivierung pro Clientanforderung (Injektion von bereichsbezogenen Diensten)
* Starke Typisierung der Middleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> wird pro Clientanforderung (-verbindung) aktiviert, sodass bereichsbezogene Dienste in den Konstruktor der Middleware eingefügt werden können.

[Anzeigen oder Herunterladen von Beispielcode](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility/samples) ([Vorgehensweise zum Herunterladen](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> definiert Middleware für die Anforderungspipeline der App. Die [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*)-Methode verarbeitet Anforderungen und gibt einen <xref:System.Threading.Tasks.Task> zurück, der für die Ausführung der Middleware steht.

Über Konventionen aktivierte Middleware:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Über <xref:Microsoft.AspNetCore.Http.MiddlewareFactory> aktivierte Middleware:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Für Middlewares werden Erweiterungen erstellt:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Es ist nicht möglich, Objekte mit <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> an eine Middleware zu übergeben, die über eine Factory aktiviert wurde:

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Die Middleware, die über eine Factory aktiviert wurde, wird dem integrierten Container in `Startup.ConfigureServices` hinzugefügt:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Beide Middlewares werden in der Anforderungsverarbeitungspipeline in `Startup.Configure` registriert:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> stellt Methoden zur Verfügung, um die Middleware zu erstellen. Die Implementierung der Middlewarefactory wird im Container als bereichsbezogener Dienst registriert.

Die Standardimplementierung von <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, befindet sich im Paket [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
