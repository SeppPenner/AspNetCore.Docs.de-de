---
title: Problembehandlung ASP.net Core auf Azure App Service und IIS
author: guardrex
description: Erfahren Sie, wie Sie Probleme mit Azure App Service und Internetinformationsdienste (IIS) bereit Stellungen von ASP.net Core-apps diagnostizieren können.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/18/2019
uid: test/troubleshoot-azure-iis
ms.openlocfilehash: deae568a6ba88c9a8365b9d7f2df629899bc64a1
ms.sourcegitcommit: 16502797ea749e2690feaa5e652a65b89c007c89
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/25/2019
ms.locfileid: "68483321"
---
# <a name="troubleshoot-aspnet-core-on-azure-app-service-and-iis"></a>Problembehandlung ASP.net Core auf Azure App Service und IIS

Von [Luke Latham](https://github.com/guardrex) und [Justin Kotalik](https://github.com/jkotalik)

Dieser Artikel enthält Informationen zu häufigen App-Start Fehlern und Anweisungen zur Diagnose von Fehlern, wenn eine APP für Azure App Service oder IIS bereitgestellt wird:

[App-Start Fehler](#app-startup-errors)  
Erläutert gängige HTTP-Statuscode Szenarien für den Start.

[Problembehandlung bei Azure App Service](#troubleshoot-on-azure-app-service)  
Bietet Tipps zur Problembehandlung für apps, die in Azure App Service bereitgestellt werden

[Problembehandlung bei den Internetinformationsdiensten (IIS)](#troubleshoot-on-iis)  
Bietet Tipps zur Problembehandlung für apps, die in IIS bereitgestellt oder lokal auf IIS Express ausgeführt werden. Die Anleitung gilt sowohl für Windows Server-als auch für Windows-Desktop Bereitstellungen.

[Löschen von Paket Caches](#clear-package-caches)  
Erläutert, was zu tun ist, wenn inkohärente Pakete eine APP beim Ausführen wichtiger Upgrades oder beim Ändern von Paketversionen unterbrechen.

[Zusätzliche Ressourcen](#additional-resources)  
Listet zusätzliche Problem Behandlungs Themen auf.

## <a name="app-startup-errors"></a>App-Startfehler

::: moniker range=">= aspnetcore-2.2"

In Visual Studio entspricht ein ASP.NET Core-Projekt standardmäßig dem [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)-Hosting während des Debuggens. Ein *502,5-Prozessfehler* oder ein *500,30-Start-Fehler* , der auftritt, wenn das lokale Debuggen mithilfe der in diesem Thema genannten Ratschläge diagnostiziert werden kann.

::: moniker-end

::: moniker range="< aspnetcore-2.2"

In Visual Studio entspricht ein ASP.NET Core-Projekt standardmäßig dem [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)-Hosting während des Debuggens. Ein *502,5-Prozessfehler* , der auftritt, wenn das lokale Debuggen mithilfe der Ratschläge in diesem Thema diagnostiziert werden kann.

::: moniker-end

### <a name="40314-forbidden"></a>403,14 unzulässig

Die APP kann nicht gestartet werden. Der folgende Fehler wird protokolliert:

```
The Web server is configured to not list the contents of this directory.
```

Der Fehler wird normalerweise durch eine unterbrochene Bereitstellung auf dem Host System verursacht, die eines der folgenden Szenarien umfasst:

* Die APP wird im falschen Ordner auf dem Hostingsystem bereitgestellt.
* Beim Bereitstellungs Prozess konnten nicht alle Dateien und Ordner der app in den Bereitstellungs Ordner auf dem Hostingsystem verschoben werden.
* Die Datei " *Web. config* " fehlt in der Bereitstellung, oder der Inhalt der Datei " *Web. config* " ist falsch formatiert.

Führen Sie die folgenden Schritte aus:

1. Löschen Sie alle Dateien und Ordner aus dem Bereitstellungs Ordner auf dem Host System.
1. Stellen Sie den Inhalt des Ordners " *Publish* " der APP mit ihrer normalen Bereitstellungs Methode wie Visual Studio, PowerShell oder manueller Bereitstellung auf dem Hostingsystem erneut bereit:
   * Vergewissern Sie sich, dass die Datei " *Web. config* " in der Bereitstellung vorhanden ist und dass Ihr Inhalt korrekt ist.
   * Vergewissern Sie `D:\home\site\wwwroot` sich beim Hosting auf Azure App Service, dass die APP im Ordner bereitgestellt wird.
   * Wenn die APP von IIS gehostet wird, vergewissern Sie sich, dass die APP im physischen IIS- **Pfad** bereitgestellt wird, der in den **Grundeinstellungen**des **IIS-Managers**angezeigt wird.
1. Vergewissern Sie sich, dass alle Dateien und Ordner der APP bereitgestellt werden, indem Sie die Bereitstellung auf dem Hostingsystem mit dem Inhalt des *Veröffentlichungs* Ordners des Projekts vergleichen.

Weitere Informationen zum Layout einer veröffentlichten ASP.net Core-App finden <xref:host-and-deploy/directory-structure>Sie unter. Weitere Informationen zur Datei " *Web. config* " finden <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>Sie unter.

### <a name="500-internal-server-error"></a>500: Interner Serverfehler

Die App wird gestartet, aber ein Fehler verhindert, dass der Server auf die Anforderung eingeht.

Dieser Fehler tritt im Code der App während des Starts oder bei der Erstellung einer Antwort auf. Die Antwort enthält möglicherweise keinen Inhalt oder die Antwort wird als *500: Interner Serverfehler* im Browser angezeigt. Das Anwendungsereignisprotokoll gibt normalerweise an, dass die Anwendung normal gestartet wurde. Aus Sicht des Servers ist dies richtig. Die App wurde gestartet, aber sie kann keine gültige Antwort generieren. Führen Sie die APP an einer Eingabeaufforderung auf dem Server aus, oder aktivieren Sie das ASP.net Core Modul stdout Log, um das Problem zu beheben.

::: moniker range="= aspnetcore-2.2"

### <a name="5000-in-process-handler-load-failure"></a>500.0: Fehler beim Laden des prozessinternen Handlers

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das [ASP.net Core-Modul](xref:host-and-deploy/aspnet-core-module) kann die .net Core-CLR nicht finden und den in-Process-Anforderungs Handler (*aspnetcorev2_inprocess. dll*) finden. Überprüfen Sie Folgendes:

* Diese App legt entweder das NuGet-Paket [Microsoft.AspNetCore.Server.IIS](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IIS) oder das Metapaket [Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) als Ziel fest.
* Die Version des freigegebenen ASP.NET Core-Frameworks, die von der App als Ziel festgelegt ist, ist auf dem Zielcomputer installiert.

### <a name="5000-out-of-process-handler-load-failure"></a>500.0: Fehler beim Laden des prozessexternen Handlers

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das [ASP.net Core Modul](xref:host-and-deploy/aspnet-core-module) kann den Out-of-Process-hostinganforderungshandler nicht finden. Stellen Sie sicher, dass sich *aspnetcorev2_outofprocess.dll* in einem Unterordner mit *aspnetcorev2.dll* befindet.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="5000-in-process-handler-load-failure"></a>500.0: Fehler beim Laden des prozessinternen Handlers

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Beim Laden [ASP.net Core Modul](xref:host-and-deploy/aspnet-core-module) Komponenten ist ein unbekannter Fehler aufgetreten. Führen Sie eine der folgenden Aktionen aus:

* Wenden Sie sich an den [Microsoft-Support](https://support.microsoft.com/oas/default.aspx?prid=15832) (über **Entwicklertools** > **ASP.NET Core**).
* Stellen Sie Ihre Frage auf Stack Overflow.
* Melden Sie das Problem im [GitHub-Repository](https://github.com/aspnet/AspNetCore).

### <a name="50030-in-process-startup-failure"></a>500.30: Prozessinterner Startupfehler

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das [Modul ASP.net Core](xref:host-and-deploy/aspnet-core-module) versucht, die .net Core-CLR in einem Prozess zu starten, aber es kann nicht gestartet werden. Die Ursache für den Fehler eines Prozess Starts kann in der Regel von Einträgen im Anwendungs Ereignisprotokoll und vom ASP.net Core-Modul "stdout"-Protokoll bestimmt werden.

Eine allgemeine Fehlerbedingung ist, dass die App aufgrund einer Version des freigegebenen ASP.NET Core-Frameworks falsch konfiguriert ist, die nicht vorhanden ist. Überprüfen Sie, welche Versionen des freigegebenen ASP.NET Core-Frameworks auf dem Zielcomputer installiert sind.

### <a name="50031-ancm-failed-to-find-native-dependencies"></a>500.31: Fehler bei der Suche nach nativen Abhängigkeiten in ANCM

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das [Modul ASP.net Core](xref:host-and-deploy/aspnet-core-module) versucht, die .net Core-Laufzeit in-Process zu starten, aber es kann nicht gestartet werden. Die häufigste Ursache dieses Startfehlers ist die fehlende Installation der Laufzeiten `Microsoft.NETCore.App` oder `Microsoft.AspNetCore.App`. Dieser Fehler tritt auf, wenn die App für die Zielversion 3.0 von ASP.NET Core bereitgestellt wurde, die aber auf dem Computer nicht vorhanden ist. Es wird beispielsweise folgende Fehlermeldung angezeigt:

```
The specified framework 'Microsoft.NETCore.App', version '3.0.0' was not found.
  - The following frameworks were found:
      2.2.1 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview5-27626-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27713-13 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27714-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27723-08 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
```

In der Fehlermeldung werden alle installierten .NET Core-Versionen aufgelistet sowie die für die App erforderliche Version. Sie haben folgende Möglichkeiten, um diesen Fehler zu beheben:

* Installieren Sie die entsprechende .NET Core-Version auf Ihrem Computer.
* Ändern Sie die Zielversion der App in eine .NET Core-Version, die auf Ihrem Computer vorhanden ist.
* Veröffentlichen Sie die App als eine [eigenständige Bereitstellung](/dotnet/core/deploying/#self-contained-deployments-scd).

Beim Ausführen im Entwicklungsmodus (wenn die Umgebungsvariable `ASPNETCORE_ENVIRONMENT` auf `Development` festgelegt ist) wird der jeweilige Fehler in die HTTP-Antwort geschrieben. Die Ursache für den Fehler eines Prozess Starts finden Sie auch im Anwendungs Ereignisprotokoll.

### <a name="50032-ancm-failed-to-load-dll"></a>500.32: Fehler beim Laden der DLL in ANCM

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Die häufigste Ursache für diesen Fehler ist die Veröffentlichung der App für eine nicht kompatible Prozessorarchitektur. Der Fehler tritt auf, wenn der Arbeitsprozess als 32-Bit-App ausgeführt wird, die App aber für eine 64-Bit-Ausführung veröffentlicht wurde.

Sie haben folgende Möglichkeiten, um diesen Fehler zu beheben:

* Veröffentlichen Sie die App erneut, sodass Prozessorarchitektur und Arbeitsprozess übereinstimmen.
* Veröffentlichen Sie die App als eine [Framework-abhängige Bereitstellung](/dotnet/core/deploying/#framework-dependent-executables-fde).

### <a name="50033-ancm-request-handler-load-failure"></a>500.33: Fehler beim Laden des Anforderungshandlers in ANCM

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Die App verweist nicht auf das Framework `Microsoft.AspNetCore.App`. Nur apps, die `Microsoft.AspNetCore.App` das Framework als Ziel haben, können vom [ASP.net Core Modul](xref:host-and-deploy/aspnet-core-module)gehostet werden.

Vergewissern Sie sich, dass das Zielframework der App `Microsoft.AspNetCore.App` ist, um den Fehler zu beheben. Überprüfen Sie `.runtimeconfig.json`, um das Zielframework der App anzuzeigen.

### <a name="50034-ancm-mixed-hosting-models-not-supported"></a>500.34: Gemischte Hostingmodelle in ANCM nicht unterstützt

Im selben Arbeitsprozess können nicht gleichzeitig eine In-Process- und eine Out-of-Process-App ausgeführt werden.

Führen Sie die Apps in separaten IIS-Anwendungspools aus, um diesen Fehler zu beheben.

### <a name="50035-ancm-multiple-in-process-applications-in-same-process"></a>500.35: Mehrere In-Process-Anwendungen im selben Prozess in ANCM

Im selben Arbeitsprozess können nicht gleichzeitig eine In-Process- und eine Out-of-Process-App ausgeführt werden.

Führen Sie die Apps in separaten IIS-Anwendungspools aus, um diesen Fehler zu beheben.

### <a name="50036-ancm-out-of-process-handler-load-failure"></a>500.36: Fehler beim Laden des Out-of-Process-Handlers in ANCM

Der Out-of-Process-Anforderungshandler *aspnetcorev2_outofprocess.dll* befindet sich nicht neben der Datei *aspnetcorev2.dll*. Dies deutet auf eine beschädigte Installation des [ASP.net Core Moduls](xref:host-and-deploy/aspnet-core-module)hin.

Reparieren Sie die Installation des [.NET Core-Hostingpakets](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle) (für IIS) oder von Visual Studio (für IIS Express), um diesen Fehler zu beheben.

### <a name="50037-ancm-failed-to-start-within-startup-time-limit"></a>500.37: Fehler beim Starten von ANCM innerhalb des Zeitlimits für den Start

ANCM konnte nicht innerhalb des angegebenen Zeitlimits für den Start gestartet werden. Das Standardzeitlimit beträgt 120 Sekunden.

Dieser Fehler kann beim Starten einer großen Anzahl von Apps auf dem gleichen Computer auftreten. Überprüfen Sie, ob beim Starten CPU-/Speicherauslastungsspitzen auf dem Server vorhanden sind. In diesem Fall sollten Sie möglicherweise den Startvorgang einiger Apps staffeln.

::: moniker-end

### <a name="5025-process-failure"></a>502.5: Prozessfehler

Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das [ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module) kann den Workerprozess nicht starten. Die Ursache für den Fehler eines Prozess Starts kann in der Regel von Einträgen im Anwendungs Ereignisprotokoll und vom ASP.net Core-Modul "stdout"-Protokoll bestimmt werden.

Eine allgemeine Fehlerbedingung ist, dass die App aufgrund einer Version des freigegebenen ASP.NET Core-Frameworks falsch konfiguriert ist, die nicht vorhanden ist. Überprüfen Sie, welche Versionen des freigegebenen ASP.NET Core-Frameworks auf dem Zielcomputer installiert sind. Das *freigegebene Framework* ist der Satz der Assemblys (*DLL*-Dateien), die auf dem Computer installiert werden und auf die ein Metapaket wie `Microsoft.AspNetCore.App` verweist. Der Metapaketverweis kann eine mindestens erforderliche Version angeben. Weitere Informationen finden Sie unter [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Das freigegebene Framework).

Die Fehlerseite *502.5: Prozessfehler* wird zurückgegeben, wenn ein falsch konfiguriertes Hosting oder eine falsch konfigurierte App bewirkt, dass der Workerprozess fehlschlägt:

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>Fehler beim Starten der Anwendung (ErrorCode 0x800700c1)

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

Die App konnte nicht gestartet werden, weil die Assembly der App ( *.dll*) nicht geladen werden konnte.

Dieser Fehler tritt bei einem Bitanzahlkonflikt zwischen der veröffentlichten App und dem w3wp/iisexpress-Prozess auf.

Überprüfen Sie, ob die 32-Bit-Einstellung des Anwendungspools korrekt ist:

1. Wählen Sie im IIS-Manager unter **Anwendungspools** den Anwendungspool aus.
1. Wählen Sie im Bereich **Aktionen** unter **Anwendungspool bearbeiten** **Erweiterte Einstellungen** aus.
1. Wählen Sie **32-Bit-Anwendungen aktivieren** aus:
   * Wenn Sie eine 32-Bit-(x86)-App bereitstellen, legen Sie den Wert auf `True` fest.
   * Wenn Sie eine 64-Bit-(x64)-App bereitstellen, legen Sie den Wert auf `False` fest.

Vergewissern Sie sich, dass kein Konflikt zwischen `<Platform>` einer MSBuild-Eigenschaft in der Projektdatei und der veröffentlichten Bitanzahl der APP besteht.

### <a name="connection-reset"></a>Verbindungszurücksetzung

Falls ein Fehler auftritt, nachdem die Header gesendet wurden, ist es zu spät für den Server, einen **500: Interner Serverfehler** zu senden, wenn ein Fehler auftritt. Dies ist häufig der Fall, wenn während der Serialisierung komplexer Objekte für eine Antwort ein Fehler auftritt. Diese Art von Fehler wird angezeigt, wenn ein *Verbindungszurücksetzungsfehler* auf dem Client auftritt. Mithilfe der [Anwendungsprotokollierung](xref:fundamentals/logging/index) können diese Fehlertypen behoben werden.

### <a name="default-startup-limits"></a>Standardstarteinschränkungen

Das [ASP.net Core-Modul](xref:host-and-deploy/aspnet-core-module) ist mit einem Standardwert von " *StartupTimeLimit* " von 120 Sekunden konfiguriert. Wenn der Standardwert beibehalten wird, dauert der Start einer App möglicherweise bis zu zwei Minuten. Erst danach kann das Modul einen Prozessfehler protokollieren. Weitere Informationen zum Konfigurieren des Moduls finden Sie unter [Attribute des aspNetCore-Elements](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

## <a name="troubleshoot-on-azure-app-service"></a>Problembehandlung bei Azure App Service

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>Anwendungs Ereignisprotokoll (Azure App Service)

Verwenden Sie das Blatt **Diagnose und Problembehandlung** im Azure-Portal, um auf das Anwendungsereignisprotokoll zuzugreifen:

1. Öffnen Sie die App im Azure-Portal unter **App Services**.
1. Klicken Sie auf **Diagnose und Problembehandlung**.
1. Wählen Sie die Überschrift **Diagnosetools** aus.
1. Klicken Sie unter **Supporttools** auf die Schaltfläche **Anwendungsereignisse**.
1. Überprüfen Sie den letzten vom Eintrag *IIS AspNetCoreModule* oder *IIS AspNetCoreModule V2* in der Spalte **Quelle** angegebenen Fehler.

Anstatt das Blatt **Diagnose und Problembehandlung** zu verwenden, können Sie die Anwendungsereignisprotokoll-Datei auch direkt mit [Kudu](https://github.com/projectkudu/kudu/wiki) untersuchen:

1. Öffnen Sie **Erweiterte Tools** im Bereich **Entwicklungstools**. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.
1. Öffnen Sie den Ordner **LogFiles**.
1. Wählen Sie den Bleistift neben der Datei *eventlog.xml* aus.
1. Untersuchen Sie das Protokoll. Scrollen Sie zum Ende des Protokolls, um die aktuellsten Ereignisse anzuzeigen.

### <a name="run-the-app-in-the-kudu-console"></a>Führen Sie die App in der Kudu-Konsole aus

Viele Startfehler erzeugen keine nützlichen Informationen im Anwendungsereignisprotokoll. Sie können die App in der [Kudu](https://github.com/projectkudu/kudu/wiki)-Remote-Ausführungskonsole ausführen, um den Fehler zu ermitteln:

1. Öffnen Sie **Erweiterte Tools** im Bereich **Entwicklungstools**. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.

#### <a name="test-a-32-bit-x86-app"></a>Testen einer 32-Bit-App (x86)

**Aktuelle Version**

1. `cd d:\home\site\wwwroot`
1. Führen Sie die App aus:
   * Wenn es sich bei der App um eine [Framework-abhängige Bereitstellung handelt](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

     ```console
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * Wenn es sich bei der App um eine [eigenständige Bereitstellung](/dotnet/core/deploying/#self-contained-deployments-scd) handelt:

     ```console
     {ASSEMBLY NAME}.exe
     ```

Die Konsolenausgabe der App, die Fehler anzeigt, wird an die Kudu-Konsole weitergeleitet.

**Framework-abhängige Bereitstellung, die in einem Vorschau Release ausgeführt wird**

*Erfordert die Installation der Runtimeerweiterung für ASP.NET Core {VERSION} (x86).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` ist die Runtimeversion.)
1. Führen Sie die App aus: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

Die Konsolenausgabe der App, die Fehler anzeigt, wird an die Kudu-Konsole weitergeleitet.

#### <a name="test-a-64-bit-x64-app"></a>Testen einer 64-Bit-App (x64)

**Aktuelle Version**

* Wenn es sich bei der App um eine [frameworkabhängige Bereitstellung](/dotnet/core/deploying/#framework-dependent-deployments-fdd) (64-Bit, x64) handelt:
  1. `cd D:\Program Files\dotnet`
  1. Führen Sie die App aus: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* Wenn es sich bei der App um eine [eigenständige Bereitstellung](/dotnet/core/deploying/#self-contained-deployments-scd) handelt:
  1. `cd D:\home\site\wwwroot`
  1. Führen Sie die App aus: `{ASSEMBLY NAME}.exe`

Die Konsolenausgabe der App, die Fehler anzeigt, wird an die Kudu-Konsole weitergeleitet.

**Framework-abhängige Bereitstellung, die in einem Vorschau Release ausgeführt wird**

*Erfordert die Installation der Runtimeerweiterung für ASP.NET Core {VERSION} (x64).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` ist die Runtimeversion.)
1. Führen Sie die App aus: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

Die Konsolenausgabe der App, die Fehler anzeigt, wird an die Kudu-Konsole weitergeleitet.

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.net Core Modul "stdout"-Protokoll (Azure App Service)

Das stdout-Protokoll des ASP.NET Core-Moduls zeichnet häufig nützliche Fehlermeldungen auf, die nicht im Anwendungsereignisprotokoll enthalten sind. So aktivieren Sie stdout-Protokolle und zeigen diese an:

1. Navigieren Sie zum Blatt **Diagnose und Problembehandlung** im Azure-Portal.
1. Wählen Sie unter **SELECT PROBLEM CATEGORY** (PROBLEMKATEGORIE WÄHLEN) die Schaltfläche **Web App Down** (Web-App ausgefallen) aus.
1. Klicken Sie unter **Suggested Solutions** (Vorgeschlagene Lösungen)  > **Enable Stdout Log Redirection** (stdout-Protokollumleitung aktivieren) auf **Open Kudu Console to edit Web.Config** (Kudu-Konsole öffnen, um web.config zu bearbeiten).
1. Öffnen Sie in der **Diagnosekonsole** die Ordner unter dem Pfad **site** > **wwwroot**. Scrollen Sie nach unten, um die Datei *web.config* am Ende der Liste einzublenden.
1. Klicken Sie auf den Bleistift neben der Datei *web.config*.
1. Setzen Sie **stdoutLogEnabled** auf `true`, und ändern Sie den Pfad **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Wählen Sie **Speichern** aus, um die aktualisierte Datei *web.config* zu speichern.
1. Führen Sie eine Anforderung an die App aus.
1. Kehren Sie zum Azure-Portal zurück. Wählen Sie das Blatt **Erweiterte Tools** im Bereich **ENTWICKLUNGSTOOLS** aus. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.
1. Wählen Sie den Ordner **LogFiles** aus.
1. Überprüfen Sie die Spalte **Geändert**, und wählen Sie den Bleistift aus, um das stdout-Protokoll mit dem letzten Änderungsdatum zu bearbeiten.
1. Wenn die Protokolldatei geöffnet wird, wird der Fehler angezeigt.

Deaktivieren Sie die stdout-Protokollierung, wenn die Problembehandlung abgeschlossen ist:

1. Kehren Sie in der Kudu-**Diagnosekonsole** zum Pfad **site** > **wwwroot** zurück, um die Datei *web.config* einzublenden. Öffnen Sie die Datei **web.config** erneut, indem Sie den Bleistift auswählen.
1. Legen Sie **stdoutLogEnabled** auf `false` fest.
1. Wählen Sie **Speichern** aus, um die Datei zu speichern.

Weitere Informationen finden Sie unter <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> Wenn das stdout-Protokoll nicht deaktiviert wird, können App- oder Serverfehler auftreten. Für die Protokollgröße oder die Anzahl von erstellten Protokolldateien ist kein Grenzwert festgelegt. Verwenden Sie die stdout-Protokollierung nur für die Behandlung von App-Startproblemen.
>
> Verwenden Sie für die allgemeine Protokollierung in einer ASP.NET Core-App nach dem Start eine Protokollierungsbibliothek, die die Protokolldateigröße beschränkt und Protokolle rotiert. Weitere Informationen finden Sie im Artikel zur [Protokollierung von Drittanbietern](xref:fundamentals/logging/index#third-party-logging-providers).

::: moniker range=">= aspnetcore-2.2"

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>ASP.net Core moduldebugprotokoll (Azure App Service)

Das Debugprotokoll des ASP.NET Core-Moduls ermöglicht die zusätzliche und ausführlichere Protokollierung über das ASP.NET Core-Modul. So aktivieren Sie stdout-Protokolle und zeigen diese an:

1. Führen Sie einen der folgenden Schritte aus, um das erweiterte Diagnoseprotokoll zu aktivieren:
   * Befolgen Sie den Anweisungen in den [erweiterten Diagnoseprotokollen](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs), um die App für die erweiterte Diagnoseprotokollierung zu konfigurieren. Stellen Sie die App erneut bereit.
   * Fügen Sie die in unter [Erweiterte Diagnoseprotokollen](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) enthaltene `<handlerSettings>`-Klasse für die *web.config*-Datei der Live-App über die Kudu-Konsole hinzu:
     1. Öffnen Sie **Erweiterte Tools** im Bereich **Entwicklungstools**. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
     1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.
     1. Öffnen Sie die Ordner unter dem Pfad **site** > **wwwroot**. Bearbeiten Sie die Datei *web.config*, indem Sie auf die Bleistiftschaltfläche klicken. Fügen Sie den Abschnitt `<handlerSettings>` wie in den [erweiterten Diagnoseprotokollen](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) dargestellt hinzu. Klicken Sie auf die Schaltfläche **Speichern**.
1. Öffnen Sie **Erweiterte Tools** im Bereich **Entwicklungstools**. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.
1. Öffnen Sie die Ordner unter dem Pfad **site** > **wwwroot**. Wenn Sie keinen Pfad für die *aspnetcore-debug.log*-Datei angegeben haben, wird die Datei in der Liste aufgeführt. Wenn Sie jedoch einen Pfad angegeben haben, navigieren Sie zum Speicherort der Protokolldatei.
1. Öffnen Sie die Protokolldatei über die Bleistiftschaltfläche neben dem Dateinamen.

Deaktivieren Sie die Debugprotokollierung, wenn die Problembehandlung abgeschlossen ist.

Führen Sie einen der folgenden Schritte aus, um das erweiterte Debugprotokoll zu deaktivieren:

* Entfernen Sie `<handlerSettings>` aus der lokalen *web.config*-Datei, und stellen Sie die App erneut bereit.
* Verwenden Sie die Kudu-Konsole, um die *web.config*-Datei zu bearbeiten und den Abschnitt `<handlerSettings>` zu entfernen. Speichern Sie die Datei.

Weitere Informationen finden Sie unter <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

> [!WARNING]
> Wenn das Debugprotokoll nicht deaktiviert werden kann, können App- oder Serverfehler auftreten. Es gibt keine Begrenzung für die Größe der Protokolldatei. Nutzen Sie die Debugprotokollierung nur für die Behandlung von App-Startproblemen.
>
> Verwenden Sie für die allgemeine Protokollierung in einer ASP.NET Core-App nach dem Start eine Protokollierungsbibliothek, die die Protokolldateigröße beschränkt und Protokolle rotiert. Weitere Informationen finden Sie im Artikel zur [Protokollierung von Drittanbietern](xref:fundamentals/logging/index#third-party-logging-providers).

::: moniker-end

### <a name="slow-or-hanging-app-azure-app-service"></a>Langsame oder hängende app (Azure App Service)

Wenn eine App bei einer Anforderung langsam reagiert oder hängt, lesen Sie folgende Artikel:

* [Problembehandlung von Leistungsproblemen in Azure App Service](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App (Verwenden der Absturzdiagnose-Websiteerweiterung zum Erfassen des Speicherabbilds bei vorübergehenden Ausnahmeproblemen oder Leistungsproblemen der Azure-Web-App)](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>Überwachungsblätter

Überwachungsblätter bieten eine Alternative zur Problembehandlung mit den weiter oben in diesem Thema beschriebenen Methoden. Die Blätter können zur Diagnose von Serie 500-Fehlern verwendet werden.

Vergewissern Sie sich, dass ASP.NET Core-Erweiterungen installiert sind. Wenn die Erweiterungen nicht installiert sind, installieren Sie diese manuell:

1. Wählen Sie im Blattabschnitt **ENTWICKLUNGSTOOLS** das Blatt **Erweiterungen** aus.
1. Die **ASP.NET Core-Erweiterungen** werden in der Liste angezeigt.
1. Wenn die Erweiterungen nicht installiert sind, klicken Sie auf **Hinzufügen**.
1. Wählen Sie die **ASP.NET Core-Erweiterungen** aus der Liste.
1. Klicken Sie auf **OK**, um die rechtlichen Bedingungen zu akzeptieren.
1. Klicken Sie auf **OK** auf dem Blatt **Erweiterung hinzufügen**.
1. Mit einer Popup-Informationsmeldung wird die erfolgreiche Installation der Erweiterungen angezeigt.

Wenn die stdout-Protokollierung nicht aktiviert ist, gehen Sie folgendermaßen vor:

1. Wählen Sie im Azure-Portal das Blatt **Erweiterte Tools** im Bereich **ENTWICKLUNGSTOOLS** aus. Klicken Sie auf **Los&rarr;** . Die Kudu-Konsole wird in einer neuen Browserregisterkarte oder in einem neuen Fenster geöffnet.
1. Öffnen Sie mithilfe der Navigationsleiste am oberen Rand der Seite die **Debugging-Konsole**, und wählen Sie **CMD** aus.
1. Öffnen Sie die Ordner unter dem Pfad **site** > **wwwroot**, und scrollen Sie nach unten, um die Datei *web.config* am Ende der Liste einzublenden.
1. Klicken Sie auf den Bleistift neben der Datei *web.config*.
1. Setzen Sie **stdoutLogEnabled** auf `true`, und ändern Sie den Pfad **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Wählen Sie **Speichern** aus, um die aktualisierte Datei *web.config* zu speichern.

Fahren Sie mit der Aktivierung der Diagnoseprotokollierung fort:

1. Wählen Sie im Azure-Portal das Blatt **Diagnoseprotokolle** aus.
1. Wählen Sie den Parameter **On** für **Anwendungsprotokollierung (Dateisystem)** und **Detaillierte Fehlermeldungen** aus. Klicken Sie auf **Speichern** am oberen Rand des Blatts.
1. Um die Ablaufverfolgung für Anforderungsfehler, auch bekannt als Protokollierung der Anforderungsfehler-Ereignispufferung (FREB), einzuschließen, wählen Sie den Parameter **On** für **Ablaufverfolgung für Anforderungsfehler** aus.
1. Wählen Sie das Blatt **Protokollstream** aus, das direkt unter dem Blatt **Diagnoseprotokolle** im Portal angezeigt wird.
1. Führen Sie eine Anforderung an die App aus.
1. In den Protokollstreamdaten wird die Ursache des Fehlers angegeben.

Achten Sie darauf, die stdout-Protokollierung zu deaktivieren, wenn die Problembehandlung abgeschlossen ist.

So zeigen Sie die Ablaufverfolgungsprotokolle für Anforderungsfehler (FREB-Protokolle) an:

1. Navigieren Sie zum Blatt **Diagnose und Problembehandlung** im Azure-Portal.
1. Wählen Sie **Failed Request Tracing Logs** (Ablaufverfolgungsprotokolle für Anforderungsfehler) im Bereich **SUPPORTTOOLS** der Seitenleiste aus.

Weitere Informationen finden Sie im [Abschnitt „Failed request traces“ im Thema „Enable diagnostics logging for apps in Azure App Service“](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces) und unter [Häufig gestellte Fragen zur Anwendungsleistung von Web-Apps in Azure: Wie schalte ich die Ablaufverfolgung für Anforderungsfehler ein?](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)

[Aktivieren der Diagnoseprotokollierung für Apps in Azure App Service](/azure/app-service/web-sites-enable-diagnostic-log).

> [!WARNING]
> Wenn das stdout-Protokoll nicht deaktiviert wird, können App- oder Serverfehler auftreten. Für die Protokollgröße oder die Anzahl von erstellten Protokolldateien ist kein Grenzwert festgelegt.
>
> Verwenden Sie für die routinemäßige Protokollierung in einer ASP.NET Core-App eine Protokollierungsbibliothek, die die Protokolldateigröße beschränkt und die Protokolle rotiert. Weitere Informationen finden Sie im Artikel zur [Protokollierung von Drittanbietern](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="troubleshoot-on-iis"></a>Problembehandlung bei IIS

### <a name="application-event-log-iis"></a>Anwendungs Ereignisprotokoll (IIS)

Greifen Sie auf das Anwendungsereignisprotokoll zu:

1. Öffnen Sie das Menü „Start“, suchen Sie nach der **Ereignisanzeige**, und wählen Sie anschließend die App **Ereignisanzeige** aus.
1. Öffnen Sie unter **Ereignisanzeige** den Knoten **Windows-Protokolle**.
1. Wählen Sie **Anwendung** aus, um das Anwendungsereignisprotokoll zu öffnen.
1. Suchen Sie nach Fehlern, die mit der fehlerhaften App im Zusammenhang stehen. Fehler weisen in der Spalte *Quelle* den Wert *IIS AspNetCore-Modul* oder *IIS Express AspNetCore-Modul* auf.

### <a name="run-the-app-at-a-command-prompt"></a>Ausführen der App in einer Eingabeaufforderung

Viele Startfehler erzeugen keine nützlichen Informationen im Anwendungsereignisprotokoll. Sie können die Ursache für einige Fehler ermitteln, indem Sie die App in einer Eingabeaufforderung auf dem Hostsystem ausführen.

#### <a name="framework-dependent-deployment"></a>Framework-abhängige Bereitstellung

Wenn es sich bei der App um eine [Framework-abhängige Bereitstellung handelt](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

1. Navigieren Sie in einer Eingabeaufforderung zum Bereitstellungsordner und führen Sie die App aus, indem Sie die Assembly der App mit *dotnet.exe* ausführen. Ersetzen Sie in folgendem Befehl den Namen der Assembly der App durch \<assembly_name>: `dotnet .\<assembly_name>.dll`.
1. Die Konsolenausgabe der App, die Fehler anzeigt, wird in das Konsolenfenster geschrieben.
1. Wenn der Fehler während einer Anforderung an die App auftritt, führen Sie eine Anforderung an den Host und Port aus, auf dem Kestrel empfangsbereit ist. Führen Sie unter Verwendung der Standardhosts und -ports eine Anforderung für `http://localhost:5000/` aus. Wenn die App normalerweise auf die Kestrel-Endpunktadresse reagiert, ist die Problemursache wahrscheinlich die Hostingkonfiguration (anstelle der App).

#### <a name="self-contained-deployment"></a>Eigenständige Bereitstellung

Wenn es sich bei der App um eine [eigenständige Bereitstellung](/dotnet/core/deploying/#self-contained-deployments-scd) handelt:

1. Navigieren Sie in einer Eingabeaufforderung zum Bereitstellungsordner und führen Sie die ausführbaren Dateien der App aus. Ersetzen Sie in folgendem Befehl den Namen der Assembly der App durch \<assembly_name>: `<assembly_name>.exe`.
1. Die Konsolenausgabe der App, die Fehler anzeigt, wird in das Konsolenfenster geschrieben.
1. Wenn der Fehler während einer Anforderung an die App auftritt, führen Sie eine Anforderung an den Host und Port aus, auf dem Kestrel empfangsbereit ist. Führen Sie unter Verwendung der Standardhosts und -ports eine Anforderung für `http://localhost:5000/` aus. Wenn die App normalerweise auf die Kestrel-Endpunktadresse reagiert, ist die Problemursache wahrscheinlich die Hostingkonfiguration (anstelle der App).

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.net Core Modul stdout-Protokoll (IIS)

So aktivieren Sie stdout-Protokolle und zeigen diese an:

1. Navigieren Sie zum Bereitstellungsordner der Website auf dem Hostsystem.
1. Erstellen Sie den Ordner *logs*, wenn dieser nicht vorhanden ist. Anweisungen zum Aktivieren von MSBuild für die automatische Erstellung des Ordners *logs* in der Bereitstellung finden Sie im Thema [Verzeichnisstruktur](xref:host-and-deploy/directory-structure).
1. Bearbeiten Sie die Datei *web.config*. Legen Sie **stdoutLogEnabled** auf `true` fest, und ändern Sie den Pfad von **stdoutLogFile** so, dass auf den Ordner *logs* verwiesen wird (Beispiel: `.\logs\stdout`). `stdout` im Pfad ist das Präfix des Protokolldateinamens. Ein Zeitstempel, eine Prozess-ID und eine Dateierweiterung werden bei der Protokollerstellung automatisch hinzugefügt. Mit `stdout` als Präfix des Dateinamens wird eine typische Protokolldatei als *stdout_20180205184032_5412.log* benannt.
1. Stellen Sie sicher, dass die Identität des Anwendungspools über Schreibberechtigungen für den Ordner *Protokolle* verfügt.
1. Speichern Sie die aktualisierte Datei *web.config*.
1. Führen Sie eine Anforderung an die App aus.
1. Navigieren Sie zu dem Ordner *logs*. Suchen und öffnen Sie das aktuelle stdout-Protokoll.
1. Untersuchen Sie das Protokoll auf Fehler.

Deaktivieren Sie die stdout-Protokollierung, wenn die Problembehandlung abgeschlossen ist:

1. Bearbeiten Sie die Datei *web.config*.
1. Legen Sie **stdoutLogEnabled** auf `false` fest.
1. Speichern Sie die Datei.

Weitere Informationen finden Sie unter <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> Wenn das stdout-Protokoll nicht deaktiviert wird, können App- oder Serverfehler auftreten. Für die Protokollgröße oder die Anzahl von erstellten Protokolldateien ist kein Grenzwert festgelegt.
>
> Verwenden Sie für die routinemäßige Protokollierung in einer ASP.NET Core-App eine Protokollierungsbibliothek, die die Protokolldateigröße beschränkt und die Protokolle rotiert. Weitere Informationen finden Sie im Artikel zur [Protokollierung von Drittanbietern](xref:fundamentals/logging/index#third-party-logging-providers).

::: moniker range=">= aspnetcore-2.2"

### <a name="aspnet-core-module-debug-log-iis"></a>Debugprotokoll des ASP.net Core Moduls (IIS)

Fügen Sie der Datei " *Web. config* " der APP die folgenden Handlereinstellungen hinzu, um ASP.net Core moduldebugprotokoll zu aktivieren:

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

Überprüfen Sie, ob der für das Protokoll angegebene Pfad vorhanden ist, und ob die Identität des Anwendungspools Schreibberechtigungen für den Speicherort hat.

Weitere Informationen finden Sie unter <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

::: moniker-end

### <a name="enable-the-developer-exception-page"></a>Aktivieren der Seite mit Ausnahmen für Entwickler

Die `ASPNETCORE_ENVIRONMENT` [Umgebungsvariable kann zur Datei web.config hinzugefügt werden](xref:host-and-deploy/aspnet-core-module#setting-environment-variables), um die App in der Entwicklungsumgebung auszuführen. Solange die Umgebung nicht beim Starten der App von `UseEnvironment` im Host-Builder außer Kraft gesetzt wird, kann die [Seite mit Ausnahmen für Entwickler](xref:fundamentals/error-handling) durch Festlegen der Umgebungsvariable angezeigt werden, wenn die App ausgeführt wird.

::: moniker range=">= aspnetcore-2.2"

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

::: moniker-end

Das Festlegen der Umgebungsvariablen für `ASPNETCORE_ENVIRONMENT` wird nur bei der Verwendung von Staging- und Testservern empfohlen, die nicht für das Internet verfügbar gemacht werden. Entfernen Sie nach der Fehlerbehebung die Umgebungsvariable aus der Datei *web.config*. Informationen zum Festlegen von Umgebungsvariablen in der Datei *web.config* finden Sie unter [Untergeordnetes environmentVariables-Element von aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

### <a name="obtain-data-from-an-app"></a>Abrufen von Daten aus einer App

Wenn eine App in der Lage sind, auf Anforderungen zu reagieren, erhalten Sie Anforderungen, Verbindungen und zusätzliche Daten von den Apps über die Inlinemiddleware des Terminals. Weitere Informationen und Beispielcode finden Sie unter <xref:test/troubleshoot#obtain-data-from-an-app>.

### <a name="slow-or-hanging-app-iis"></a>Langsame oder hängende app (IIS)

Ein *Absturz* Abbild ist eine Momentaufnahme des System Speichers und kann helfen, die Ursache für einen Absturz der APP, einen Start Fehler oder eine langsame APP zu ermitteln.

#### <a name="app-crashes-or-encounters-an-exception"></a>App stürzt ab, oder es tritt eine Ausnahme auf

Abrufen und Analysieren eines Speicherabbilds aus der [Windows-Fehlerberichterstattung (WER)](/windows/desktop/wer/windows-error-reporting):

1. Erstellen Sie einen Ordner zum Speichern von Absturzabbilddateien unter `c:\dumps`. Der App-Pool muss über Schreibzugriff auf den Ordner verfügen.
1. Führen Sie das [PowerShell-Skript „EnableDumps“](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1) aus:
   * Wenn die App das [In-Process-Hostingmodell](xref:host-and-deploy/iis/index#in-process-hosting-model) verwendet, führen Sie das Skript für *w3wp.exe* aus:

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * Wenn die App das [Out-of-Process-Hostingmodell](xref:host-and-deploy/iis/index#out-of-process-hosting-model) verwendet, führen Sie das Skript für *dotnet.exe* aus:

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. Führen Sie die App unter den Bedingungen aus, die dazu führen, dass der Absturz auftritt.
1. Nachdem der Absturz stattgefunden hat, führen Sie das [PowerShell-Skript „DisableDumps“](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1) aus:
   * Wenn die App das [In-Process-Hostingmodell](xref:host-and-deploy/iis/index#in-process-hosting-model) verwendet, führen Sie das Skript für *w3wp.exe* aus:

     ```console
     .\DisableDumps w3wp.exe
     ```

   * Wenn die App das [Out-of-Process-Hostingmodell](xref:host-and-deploy/iis/index#out-of-process-hosting-model) verwendet, führen Sie das Skript für *dotnet.exe* aus:

     ```console
     .\DisableDumps dotnet.exe
     ```

Nachdem eine Anwendung abgestürzt ist und die Absturzabbildsammlung abgeschlossen ist, kann die App ordnungsgemäß beendet werden. Das PowerShell-Skript konfiguriert WER für die Erfassung von bis zu fünf Absturzabbildern pro App.

> [!WARNING]
> Absturzabbilder können eine große Menge Speicherplatz in Anspruch nehmen (jeweils bis zu mehreren GB).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>App hat sich aufgehängt, es tritt ein Fehler während des Starts auf, oder sie wird normal ausgeführt

Wenn sich eine App *aufgehängt* hat (nicht mehr reagiert, aber nicht abstürzt), ein Fehler während des Starts auftritt bzw. die App normal ausgeführt wird, finden Sie weitere Informationen unter [Benutzermodus-Absturzabbilddateien: Auswählen des besten Tools](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool), um ein geeignetes Tool zum Generieren des Speicherabbilds auszuwählen.

#### <a name="analyze-the-dump"></a>Analysieren des Speicherabbilds

Ein Speicherabbild kann mithilfe mehrerer Ansätze analysiert werden. Weitere Informationen finden Sie unter [Analysieren einer Benutzermodus-Speicherabbilddatei](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file).

## <a name="clear-package-caches"></a>Löschen von Paket Caches

In manchen Fällen tritt ein funktionierender App direkt nach dem Upgrade der .net Core SDK auf dem Entwicklungs Computer oder dem Ändern der Paketversionen in der APP auf. In einigen Fällen können inkohärente Pakete eine App beschädigen, wenn größere Upgrades durchgeführt werden. Die meisten dieser Probleme können durch Befolgung der folgenden Anweisungen behoben werden:

1. Löschen Sie die Ordner *bin* und *obj*.
1. Löschen Sie die Paket Caches, `dotnet nuget locals all --clear` indem Sie von einer Befehlsshell ausführen.

   Das Löschen von Paket Caches kann auch mit dem [nuget. exe](https://www.nuget.org/downloads) -Tool und dem Ausführen `nuget locals all -clear`des Befehls erreicht werden. *nuget.exe* ist wird unter dem Windows Desktop-Betriebssystem nicht gebündelt installiert und muss separat von der [NuGet-Website](https://www.nuget.org/downloads) abgerufen werden.

1. Stellen Sie das Projekt wieder her und erstellen Sie es neu.
1. Löschen Sie alle Dateien im Bereitstellungs Ordner auf dem Server, bevor Sie die APP erneut bereitstellen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure-Dokumentation

* [Application Insights für ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
* [Abschnitt zum Remote Debuggen von Web-Apps unter Problembehandlung für eine Web-App in Azure App Service mit Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Übersicht: Azure App Service-Diagnose](/azure/app-service/app-service-diagnostics)
* [Vorgehensweise: Überwachen von Apps in Azure App Service](/azure/app-service/web-sites-monitor)
* [Problembehandlung in einer Web-App in Azure App Service mithilfe von Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Problembehandlung bei HTTP-Fehler „502 Ungültiges Gateway“ und „503 Dienst nicht verfügbar“ in Ihren Azure-Web-Apps](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Problembehandlung von Leistungsproblemen in Azure App Service](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [FAQ zur Anwendungsleistung von Web-Apps in Azure](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Einschränkungen der App Service-Laufzeiteinschränkung](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday: Azure App Service Diagnostic and Troubleshooting Experience (Azure Friday: Azure App Service-Diagnose und -Fehlerbehebung; zwölfminütiges Video)](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio-Dokumentation

* [Remote debugASP.net Core auf IIS in Azure in Visual Studio 2017](/visualstudio/debugger/remote-debugging-azure)
* [Remote Debuggen ASP.net Core auf einem IIS-Remote Computer in Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Lernen Sie das Debuggen mit Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Dokumentation zu Visual Studio Code

* [Debuggen mit Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging)
