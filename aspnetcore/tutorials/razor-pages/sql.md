---
title: Arbeiten mit einer Datenbank und ASP.NET Core
author: rick-anderson
description: Dieser Artikel erläutert das Arbeiten mit einer Datenbank und ASP.NET Core.
ms.author: riande
ms.date: 12/07/2017
uid: tutorials/razor-pages/sql
ms.openlocfilehash: 6cef55382d8c77e95280ea4eea2dbc2af1c81987
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/27/2019
ms.locfileid: "64885205"
---
# <a name="work-with-a-database-and-aspnet-core"></a>Arbeiten mit einer Datenbank und ASP.NET Core

Von [Rick Anderson](https://twitter.com/RickAndMSFT) und [Joe Audette](https://twitter.com/joeaudette)

[!INCLUDE[](~/includes/rp/download.md)]

Das `RazorPagesMovieContext`-Objekt übernimmt die Aufgabe der Herstellung der Verbindung mit der Datenbank und Zuordnung von `Movie`-Objekten zu Datensätzen in der Datenbank. Der Datenbankkontext wird beim Container [Abhängigkeitsinjektion](xref:fundamentals/dependency-injection) in der `ConfigureServices`-Methode in *Startup.cs* registriert:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio für Mac](#tab/visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

Weitere Informationen über die in `ConfigureServices` verwendeten Methoden finden Sie unter:

* [Unterstützung für die Datenschutz-Grundverordnung (DSGVO) in ASP.NET Core](xref:security/gdpr) für `CookiePolicyOptions`.
* [SetCompatibilityVersion](xref:mvc/compatibility-version)

Das ASP.NET Core-[Konfigurationssystem](xref:fundamentals/configuration/index) liest die `ConnectionString`. Für die lokale Entwicklung wird die Verbindungszeichenfolge aus der Datei *appsettings.json* abgerufen.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Der Name-Wert für die Datenbank (`Database={Database name}`) ist für den generierten Code anders. Beim Name-Wert handelt es sich um einen beliebigen Wert.

[!code-json[](razor-pages-start/sample/RazorPagesMovie22/appsettings.json)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio für Mac](#tab/visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

---

Wenn die App auf einem Test- oder Produktionsserver bereitgestellt wird, kann eine Umgebungsvariable verwendet werden, um die Verbindungszeichenfolge auf einen echten Datenbankserver festzulegen. Weitere Informationen finden Sie unter [Konfiguration](xref:fundamentals/configuration/index).

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

LocalDB ist eine Basisversion der SQL Server Express-Datenbank-Engine, die für die Programmentwicklung bestimmt ist. LocalDB wird bedarfsgesteuert gestartet und im Benutzermodus ausgeführt, sodass keine komplexe Konfiguration anfällt. Standardmäßig erstellt LocalDB `*.mdf`-Dateien im `C:/Users/<user/>`-Verzeichnis.

<a name="ssox"></a>
* Öffnen Sie im Menü **Ansicht** den **SQL Server-Objekt-Explorer** (SSOX).

  ![Menü Ansicht](sql/_static/ssox.png)

* Klicken Sie mit der rechten Maustaste auf die Tabelle `Movie`, und wählen Sie **Designer anzeigen** aus:

  ![Für die Tabelle „Movie“ geöffnetes Kontextmenü](sql/_static/design.png)

  ![Im Designer geöffnete Tabelle „Movie“](sql/_static/dv.png)

Beachten Sie das Schlüsselsymbol neben `ID`. EF erstellt standardmäßig eine Eigenschaft namens `ID` für den Primärschlüssel.

* Klicken Sie mit der rechten Maustaste auf die Tabelle `Movie`, und wählen Sie **Daten anzeigen** aus:

  ![Geöffnete Tabelle „Movie“ mit Tabellendaten](sql/_static/vd22.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio für Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a>Ausführen eines Seedings für die Datenbank

Erstellen Sie im Ordner *Models* mit dem folgenden Code die neue Klasse `SeedData`:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedData.cs?name=snippet_1)]

Wenn in der Datenbank Filme vorhanden sind, wird der Initialisierer des Seedings zurückgegeben, und es werden keine Filme hinzugefügt.

```csharp
if (context.Movie.Any())
{
    return;   // DB has been seeded.
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Hinzufügen des Initialisierers des Seedings

Ändern Sie in der *Program.cs*-Datei die `Main`-Methode, um die folgenden Vorgänge auszuführen:

* Rufen Sie eine Datenbankkontextinstanz aus dem Dependency Injection-Container ab.
* Rufen Sie die Seedmethode auf, indem Sie den Kontext an diese übergeben.
* Löschen Sie den Kontext, wenn die Seedmethode abgeschlossen ist.

Der folgende Code zeigt die aktualisierte *Program.cs*-Datei.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Program.cs)]

Eine Produktions-App würde `Database.Migrate` nicht aufrufen. Es wird zum vorangehenden Code hinzugefügt, um die folgende Ausnahme zu verhindern, wenn `Update-Database` nicht ausgeführt wurde:

SqlException: Die bei der Anmeldung angeforderte Datenbank „RazorPagesMovieContext-21“ kann nicht geöffnet werden. Die Anmeldung ist fehlgeschlagen.
Die Anmeldung des Benutzers „Benutzername“ ist fehlgeschlagen.

### <a name="test-the-app"></a>Testen der App

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Löschen Sie alle Datensätze in der Datenbank. Dies ist über die Links „Löschen“ im Browser oder [SSOX](xref:tutorials/razor-pages/new-field#ssox) möglich.
* Zwingen Sie die App zur Initialisierung (rufen Sie die Methoden in der `Startup`-Klasse auf), damit die Seedmethode ausgeführt wird. Um die Initialisierung zu erzwingen, muss IIS Express beendet und neu gestartet werden. Hierzu können Sie einen der folgenden Ansätze verwenden:

  * Klicken Sie auf der Taskleiste im Infobereich mit der rechten Maustaste auf das Symbol von IIS Express, und wählen Sie **Beenden** oder **Website beenden** aus:

    ![IIS Express-Symbol auf der Taskleiste](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Kontextmenü](sql/_static/stopIIS.png)

    * Wenn Sie Visual Studio im Nicht-Debugmodus ausgeführt haben, drücken Sie F5, um den Debugmodus auszuführen.
    * Wenn Sie Visual Studio im Debugmodus ausgeführt haben, beenden Sie den Debugger, und drücken Sie F5.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Löschen Sie alle Datensätze in der Datenbank (damit die Seed-Methode ausgeführt wird). Beenden und starten Sie die App, um das Seeding der Datenbank auszuführen.

Die App zeigt die per Seeding hinzugefügten Daten.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio für Mac](#tab/visual-studio-mac)

Löschen Sie alle Datensätze in der Datenbank (damit die Seed-Methode ausgeführt wird). Beenden und starten Sie die App, um das Seeding der Datenbank auszuführen.

Die App zeigt die per Seeding hinzugefügten Daten.

---

Die App zeigt die per Seeding hinzugefügten Daten:

![In Chrome geöffnete Movie-Anwendung mit Filmdaten](sql/_static/m55.png)

Im nächsten Tutorial wird die Präsentation der Daten bereinigt.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Dieses Tutorial auf YouTube](https://youtu.be/A_5ff11sDHY)

> [!div class="step-by-step"]
> [Zurück: Scaffolded Razor Pages (Gerüstbau mit Razor Pages)](xref:tutorials/razor-pages/page)
> [Weiter: Updating the pages (Aktualisieren der Seiten)](xref:tutorials/razor-pages/da1)
