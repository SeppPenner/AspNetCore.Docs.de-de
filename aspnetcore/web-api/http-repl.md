---
title: Testen von Web-APIs mit HTTP REPL
author: scottaddie
description: Hier erfahren Sie, wie Sie das globale .NET Core-Tool HTTP REPL verwenden können, um ASP.NET Core-Web-APIs zu durchsuchen und zu testen.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/12/2019
uid: web-api/http-repl
ms.openlocfilehash: 1774382305cc3d479291700390807d277a24bfa7
ms.sourcegitcommit: b40613c603d6f0cc71f3232c16df61550907f550
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308345"
---
# <a name="test-web-apis-with-the-http-repl"></a>Testen von Web-APIs mit HTTP REPL

Von [Scott Addie](https://twitter.com/Scott_Addie)

HTTP REPL (read–eval–print-Loop):

* Ein schlankes, plattformübergreifendes Befehlszeilentool, das überall unterstützt wird, wo .NET Core unterstützt wird
* Wird zum Senden von HTTP-Anforderungen zum Testen von ASP.NET Core-Web-APIs und zum Überprüfen von deren Ergebnissen verwendet

Folgende [HTTP-Verben](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods) werden unterstützt:

* [DELETE](#test-http-delete-requests)
* [GET](#test-http-get-requests)
* [HEAD](#test-http-head-requests)
* [OPTIONS](#test-http-options-requests)
* [PATCH](#test-http-patch-requests)
* [POST](#test-http-post-requests)
* [PUT](#test-http-put-requests)

[Sehen Sie sich die beispielhafte ASP.NET Core-Web-API an, oder laden Sie sie herunter ([Downloadanleitung](xref:index#how-to-download-a-sample))](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples), um dem Artikel zu folgen.

## <a name="prerequisites"></a>Erforderliche Komponenten

* [!INCLUDE [2.1-SDK](~/includes/2.1-SDK.md)]

## <a name="installation"></a>Installation

Führen Sie den folgenden Befehl aus, um HTTP REPL zu installieren:

```console
dotnet tool install -g dotnet-httprepl
    --version 2.2.0-*
    --add-source https://dotnet.myget.org/F/dotnet-core/api/v3/index.json
```

Ein [globales .NET Core-Tool](/dotnet/core/tools/global-tools#install-a-global-tool) wird über das NuGet-Paket [dotnet-httprepl](https://dotnet.myget.org/feed/dotnet-core/package/nuget/dotnet-httprepl
) installiert, das auf MyGet gehostet wird.

## <a name="usage"></a>Verwendung

Nach der erfolgreichen Installation des Tools können Sie folgenden Befehl ausführen, um HTTP REPL zu starten:

```console
dotnet httprepl
```

Führen Sie einen der folgenden Befehle aus, um eine Liste der verfügbaren HTTP REPL-Befehle anzuzeigen:

```console
dotnet httprepl -h
```

```console
dotnet httprepl --help
```

Die folgende Ausgabe wird angezeigt:

```console
Usage: dotnet httprepl [<BASE_ADDRESS>] [options]

Arguments:
  <BASE_ADDRESS> - The initial base address for the REPL.

Options:
  --help - Show help information.

Once the REPL starts, these commands are valid:

HTTP Commands:
Use these commands to execute requests against your application.

GET            Issues a GET request.
POST           Issues a POST request.
PUT            Issues a PUT request.
DELETE         Issues a DELETE request.
PATCH          Issues a PATCH request.
HEAD           Issues a HEAD request.
OPTIONS        Issues an OPTIONS request.

set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`


Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

set base       Set the base URI. e.g. `set base http://locahost:5000`
set swagger    Set the URI, relative to your base if set, of the Swagger document for this API. e.g. `set swagger /swagger/v1/swagger.json`
ls             Show all endpoints for the current path.
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`.

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell.
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands.
exit           Exit the shell.

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\Program Files\Microsoft VS Code\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line.
ui             Displays the Swagger UI page, if available, in the default browser.

Use help <COMMAND> to learn more details about individual commands. e.g. `help get`
```

In HTTP REPL können Befehle vervollständigt werden. Wenn Sie die <kbd>TAB-TASTE</kbd> drücken, können Sie die Liste der Befehle durchlaufen, die die eingegebenen Zeichen oder API-Endpunkte vervollständigen. In den folgenden Abschnitten werden die verfügbaren CLI-Befehle erläutert.

## <a name="connect-to-the-web-api"></a>Herstellen einer Verbindung mit der Web-API

Stellen Sie mithilfe des folgenden Befehls eine Verbindung mit einer Web-API her:

```console
dotnet httprepl <BASE URI>
```

`<BASE URI>` ist der Basis-URI für die Web-API. Beispiel:

```console
dotnet httprepl https://localhost:5001
```

Alternativ können Sie den folgenden Befehl jederzeit ausführen, während HTTP REPL ausgeführt wird:

```console
set base <BASE URI>
```

Beispiel:

```console
(Disconnected)~ set base https://localhost:5001
```

## <a name="point-to-the-swagger-document-for-the-web-api"></a>Verweis auf ein Swagger-Dokument für die Web-API

Legen Sie den relativen URI auf das Swagger-Dokument für die Web-API fest, um diese ordnungsgemäß zu überprüfen. Führen Sie den folgenden Befehl aus:

```console
set swagger <RELATIVE URI>
```

Beispiel:

```console
https://localhost:5001/~ set swagger /swagger/v1/swagger.json
```

## <a name="navigate-the-web-api"></a>Navigieren in der Web-API

### <a name="view-available-endpoints"></a>Anzeigen verfügbarer Endpunkte

Führen Sie den Befehl `ls` oder `dir` aus, um die unterschiedlichen Endpunkte (Controller) im aktuellen Pfad der Web-API-Adresse aufzulisten:

```console
https://localhot:5001/~ ls
```

Folgendes Ausgabeformat wird angezeigt:

```console
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Die vorherige Ausgabe gibt an, dass die zwei Controller `Fruits` und `People` verfügbar sind. Beide Controller unterstützen parameterlose HTTP GET- und HTTP POST-Vorgänge.

Wenn Sie zu einem bestimmten Controller navigieren, werden weitere Details angezeigt. Aus der Ausgabe des folgenden Befehls geht beispielsweise hervor, dass der Controller `Fruits` auch HTTP GET-, HTTP PUT- und HTTP DELETE-Vorgänge unterstützt. Jeder dieser Vorgänge erwartet einen `id`-Parameter in der Route:

```console
https://localhost:5001/fruits~ ls
.      [get|post]
..     []
{id}   [get|put|delete]

https://localhost:5001/fruits~
```

Führen Sie alternativ den Befehl `ui` aus, um die Swagger-Benutzeroberflächenseite der Web-API in einem Browser zu öffnen. Beispiel:

```console
https://localhost:5001/~ ui
```

### <a name="navigate-to-an-endpoint"></a>Navigieren zu einem Endpunkt

Führen Sie den Befehl `cd` aus, um zu einem anderen Endpunkt in der Web-API zu navigieren:

```console
https://localhost:5001/~ cd people
```

Für den Pfad im Befehl `cd` wird die Groß-/Kleinschreibung nicht beachtet. Folgendes Ausgabeformat wird angezeigt:

```console
/people    [get|post]

https://localhost:5001/people~
```

## <a name="customize-the-http-repl"></a>Anpassen von HTTP REPL

Die [Standardfarben](#set-color-preferences) von HTTP REPL können angepasst werden. Außerdem kann definiert werden, welcher [Text-Editor](#set-the-default-text-editor) standardmäßig verwendet wird. Die HTTP REPL-Einstellungen werden in der aktuellen Sitzung beibehalten und in zukünftigen Sitzungen berücksichtigt. Nach der Änderung werden die Einstellungen in der folgenden Datei gespeichert:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

*%HOME%/.httpreplprefs*

# <a name="macostabmacos"></a>[macOS](#tab/macos)

*%HOME%/.httpreplprefs*

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

*%USERPROFILE%\\.httpreplprefs*

---

Die *HTTPREPLPREFS-Datei* wird beim Start geladen und bei der Laufzeit nicht auf Änderungen überwacht. Manuelle Änderungen an der Datei werden erst nach dem Neustart des Tools wirksam.

### <a name="view-the-settings"></a>Anzeigen der Einstellungen

Führen Sie den Befehl `pref get` aus, um die verfügbaren Einstellungen anzuzeigen. Beispiel:

```console
https://localhost:5001/~ pref get
```

Mit dem vorangehenden Befehl werden die verfügbaren Schlüssel-Wert-Paare angezeigt:

```console
colors.json=Green
colors.json.arrayBrace=BoldCyan
colors.json.comma=BoldYellow
colors.json.name=BoldMagenta
colors.json.nameSeparator=BoldWhite
colors.json.objectBrace=Cyan
colors.protocol=BoldGreen
colors.status=BoldYellow
```

### <a name="set-color-preferences"></a>Festlegen von Farbeinstellungen

Die farbliche Kennzeichnung von Antworten wird derzeit nur für JSON unterstützt. Sie können die Standardfarben des HTTP REPL-Tools anpassen, indem Sie den Schlüssel für die entsprechende Farbe suchen und ändern. Weitere Informationen zum Suchen von Schlüsseln finden Sie im Abschnitt [Anzeigen der Einstellungen](#view-the-settings). Ändern Sie beispielsweise den Wert des Schlüssels `colors.json` folgendermaßen von `Green` in `White`:

```console
https://localhost:5001/people~ pref set colors.json White
```

Es können nur die [zulässigen Farben](https://github.com/aspnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs) verwendet werden. Nachfolgende HTTP-Anforderungen zeigen die Ausgabe mit der neuen Farbgebung an.

Wenn bestimmte Farbtasten nicht festgelegt sind, werden mehr generische Schlüssel berücksichtigt. Anhand des folgenden Beispiels wird das Fallbackverhalten veranschaulicht:

* Wenn `colors.json.name` keinen Wert aufweist, wird `colors.json.string` verwendet.
* Wenn `colors.json.string` keinen Wert aufweist, wird `colors.json.literal` verwendet.
* Wenn `colors.json.literal` keinen Wert aufweist, wird `colors.json` verwendet. 
* Wenn `colors.json` keinen Wert aufweist, wird die Standardtextfarbe (`AllowedColors.None`) der Befehlsshell verwendet.

### <a name="set-indentation-size"></a>Festlegen der Einzugsgröße

Die Anpassung der Einzugsgröße für Antworten wird derzeit nur für JSON unterstützt. Der Standardgröße beträgt zwei Leerzeichen. Beispiel:

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Legen Sie den Schlüssel `formatting.json.indentSize` fest, um die Standardgröße zu ändern. Folgendermaßen werden beispielsweise immer vier Leerzeichen verwendet:

```console
pref set formatting.json.indentSize 4
```

Nachfolgende Antworten berücksichtigen die Einstellung auf vier Leerzeichen:

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-indentation-size"></a>Festlegen der Einzugsgröße

Die Anpassung der Einzugsgröße für Antworten wird derzeit nur für JSON unterstützt. Der Standardgröße beträgt zwei Leerzeichen. Beispiel:

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Legen Sie den Schlüssel `formatting.json.indentSize` fest, um die Standardgröße zu ändern. Folgendermaßen werden beispielsweise immer vier Leerzeichen verwendet:

```console
pref set formatting.json.indentSize 4
```

Nachfolgende Antworten berücksichtigen die Einstellung auf vier Leerzeichen:

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-the-default-text-editor"></a>Festlegen des Standard-Text-Editors

Standardmäßig ist für HTTP REPL kein Text-Editor konfiguriert. Wenn Sie Web-API-Methoden testen möchten, für die ein HTTP-Anforderungstext erforderlich ist, müssen Sie einen Standard-Text-Editor festlegen. Das HTTP REPL-Tool startet den konfigurierten Text-Editor nur zum Verfassen des Anforderungstexts. Führen Sie den folgenden Befehl aus, um Ihren bevorzugten Text-Editor festzulegen:

```console
pref set editor.command.default "<EXECUTABLE>"
```

Im vorherigen Befehl ist `<EXECUTABLE>` der vollständige Pfad zur ausführbaren Datei des Text-Editors. Führen Sie beispielsweise folgenden Befehl aus, um Visual Studio Code als Standard-Text-Editor festzulegen:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```console
pref set editor.command.default "/usr/bin/code"
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

```console
pref set editor.command.default "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code"
```

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```console
pref set editor.command.default "C:\Program Files\Microsoft VS Code\Code.exe"
```

---

Legen Sie den Schlüssel `editor.command.default.arguments` fest, um den Standard-Text-Editor mit bestimmten CLI-Argumenten zu starten. Nehmen Sie beispielsweise an, dass Visual Studio Code der Standard-Text-Editor ist und Sie möchten, dass Visual Studio Code durch HTTP REPL immer in einer neuen Sitzung mit deaktivierten Erweiterungen öffnet. Führen Sie den folgenden Befehl aus:

```console
pref set editor.command.default.arguments "--disable-extensions --new-window"
```

## <a name="test-http-get-requests"></a>Testen von HTTP GET-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
get <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

Für den Befehl `get` sind die folgenden Optionen verfügbar:

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Beispiel

So führen Sie eine HTTP GET-Anforderung aus:

1. Führen Sie den Befehl `get` für einen Endpunkt aus, der ihn unterstützt:

    ```console
    https://localhost:5001/people~ get
    ```

    Über den vorherigen Befehl wird das folgende Ausgabeformat angezeigt:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 03:38:45 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "name": "Scott Hunter"
      },
      {
        "id": 2,
        "name": "Scott Hanselman"
      },
      {
        "id": 3,
        "name": "Scott Guthrie"
      }
    ]


    https://localhost:5001/people~
    ```

1. Rufen Sie einen bestimmten Datensatz ab, indem Sie einen Parameter an den Befehl `get` übergeben:

    ```console
    https://localhost:5001/people~ get 2
    ```

    Über den vorherigen Befehl wird das folgende Ausgabeformat angezeigt:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 06:17:57 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 2,
        "name": "Scott Hanselman"
      }
    ]


    https://localhost:5001/people~
    ```

## <a name="test-http-post-requests"></a>Testen von HTTP POST-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
post <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Beispiel

So führen Sie eine HTTP POST-Anforderung aus:

1. Führen Sie den Befehl `post` für einen Endpunkt aus, der ihn unterstützt:

    ```console
    https://localhost:5001/people~ post -h Content-Type=application/json
    ```

    Im vorherigen Befehl wird der HTTP-Anforderungsheader `Content-Type` so festgelegt, dass er den Medientyp des Anforderungstexts (JSON) angibt. Der Standard-Text-Editor öffnet eine *TMP-Datei* mit einer JSON-Vorlage, die den HTTP-Anforderungstext darstellt. Beispiel:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Weitere Informationen zum Festlegen des Standard-Text-Editors Abschnitt [Festlegen des Standard-Text-Editors](#set-the-default-text-editor).

1. Ändern Sie die JSON-Vorlage so, dass die Anforderungen für die Modellvalidierung erfüllt werden:

  ```json
  {
    "id": 0,
    "name": "Scott Addie"
  }
  ```

1. Speichern Sie die *TMP-Datei*, und schließen Sie den Text-Editor. In der Befehlsshell wird die folgende Ausgabe angezeigt:

    ```console
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Thu, 27 Jun 2019 21:24:18 GMT
    Location: https://localhost:5001/people/4
    Server: Kestrel
    Transfer-Encoding: chunked

    {
      "id": 4,
      "name": "Scott Addie"
    }


    https://localhost:5001/people~
    ```

## <a name="test-http-put-requests"></a>Testen von HTTP PUT-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
put <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Beispiel

So führen Sie eine HTTP PUT-Anforderung aus:

1. *Optional:* Führen Sie den Befehl `get` aus, um die Daten vor dem Ändern anzuzeigen:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `put` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ put 2 -h Content-Type=application/json
    ```

    Im vorherigen Befehl wird der HTTP-Anforderungsheader `Content-Type` so festgelegt, dass er den Medientyp des Anforderungstexts (JSON) angibt. Der Standard-Text-Editor öffnet eine *TMP-Datei* mit einer JSON-Vorlage, die den HTTP-Anforderungstext darstellt. Beispiel:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Weitere Informationen zum Festlegen des Standard-Text-Editors Abschnitt [Festlegen des Standard-Text-Editors](#set-the-default-text-editor).

1. Ändern Sie die JSON-Vorlage so, dass die Anforderungen für die Modellvalidierung erfüllt werden:

    ```json
    {
      "id": 2,
      "name": "Cherry"
    }
    ```

1. Speichern Sie die *TMP-Datei*, und schließen Sie den Text-Editor. In der Befehlsshell wird die folgende Ausgabe angezeigt:

    ```console
    [main 2019-06-28T17:27:01.805Z] update#setState idle
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:28:21 GMT
    Server: Kestrel
    ```

1. *Optional:* Führen Sie den Befehl `get` aus, um die Änderungen anzuzeigen. Wenn Sie beispielsweise „Cherry“ in den Text-Editor eingeben, gibt `get` Folgendes zurück:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:08:20 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Cherry"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-delete-requests"></a>Testen von HTTP DELETE-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
delete <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Beispiel

So führen Sie eine HTTP DELETE-Anforderung aus:

1. *Optional:* Führen Sie den Befehl `get` aus, um die Daten vor dem Ändern anzuzeigen:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `delete` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ delete 2
    ```

    Über den vorherigen Befehl wird das folgende Ausgabeformat angezeigt:

    ```console
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:36:42 GMT
    Server: Kestrel
    ```

1. *Optional:* Führen Sie den Befehl `get` aus, um die Änderungen anzuzeigen. In diesem Beispiel gibt `get` Folgendes zurück:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:16:30 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-patch-requests"></a>Testen von HTTP PATCH-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
patch <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

## <a name="test-http-head-requests"></a>Testen von HTTP HEAD-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
head <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="test-http-options-requests"></a>Testen von HTTP OPTIONS-Anforderungen

### <a name="synopsis"></a>Übersicht

```console
options <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumente

`PARAMETER`

Der Routenparameter (sofern vorhanden), der von der zugeordneten Aktionsmethode des Controllers erwartet wird.

### <a name="options"></a>Optionen

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="set-http-request-headers"></a>Festlegen von HTTP-Anforderungsheadern

Verwenden Sie einen der folgenden Ansätze, um einen HTTP-Anforderungsheader festzulegen:

1. Legen Sie den Header inline mit der HTTP-Anforderung fest. Beispiel:

  ```console
  https://localhost:5001/people~ post -h Content-Type=application/json
  ```

  Beim vorherigen Ansatz benötigt jeder eindeutige HTTP-Anforderungsheader eine eigene `-h`-Option.

1. Legen Sie den Header fest, bevor Sie die HTTP-Anforderung senden. Beispiel:

  ```console
  https://localhost:5001/people~ set header Content-Type application/json
  ```

  Wenn Sie den Header vor dem Senden einer Anforderung festlegen, bleibt er für die Dauer der Befehlsshellsitzung festgelegt. Geben Sie einen leeren Wert an, um den Header zu löschen. Beispiel:

  ```console
  https://localhost:5001/people~ set header Content-Type
  ```

## <a name="toggle-http-request-display"></a>Umschalten der Anzeige von HTTP-Anforderungen

Standardmäßig wird die Anzeige der gesendeten HTTP-Anforderung unterdrückt. Sie können die entsprechende Einstellung für die Dauer der Befehlsshellsitzung ändern.

### <a name="enable-request-display"></a>Aktivieren der Anzeige von Anforderungen

Zeigen Sie die gesendete HTTP-Anforderung an, indem Sie den Befehl `echo on` ausführen. Beispiel:

```console
https://localhost:5001/people~ echo on
Request echoing is on
```

Nachfolgende HTTP-Anforderungen in der aktuellen Sitzung zeigen die Anforderungsheader an. Beispiel:

```console
https://localhost:5001/people~ post

[main 2019-06-28T18:50:11.930Z] update#setState idle
Request to https://localhost:5001...

POST /people HTTP/1.1
Content-Length: 41
Content-Type: application/json
User-Agent: HTTP-REPL

{
  "id": 0,
  "name": "Scott Addie"
}

Response from https://localhost:5001...

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Jun 2019 18:50:21 GMT
Location: https://localhost:5001/people/4
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Scott Addie"
}


https://localhost:5001/people~
```

### <a name="disable-request-display"></a>Deaktivieren der Anzeige von Anforderungen

Unterdrücken Sie die Anzeige der gesendeten HTTP-Anforderung, indem Sie den Befehl `echo off` ausführen. Beispiel:

```console
https://localhost:5001/people~ echo off
Request echoing is off
```

## <a name="run-a-script"></a>Ausführen eines Skripts

Wenn Sie häufig die gleichen HTTP REPL-Befehle ausführen, können Sie diese in einer Textdatei speichern. Die Befehle in der Datei haben das gleiche Format wie die manuell in der Befehlszeile ausgeführten Befehle. Die Befehle können mithilfe des Befehls `run` auf einmal ausgeführt werden. Beispiel:

1. Erstellen Sie eine Textdatei, die einige Befehle enthält, die jeweils in einer neuen Zeile stehen. Sie können die Datei *people-script.txt*, die folgende Befehle enthält, als Beispiel verwenden:

    ```text
    set base https://localhost:5001
    ls
    cd People
    ls
    get 1
    ```

1. Führen Sie den Befehl `run` aus, und übergeben Sie den Pfad der Textdatei. Beispiel:

    ```console
    https://localhost:5001/~ run C:\http-repl-scripts\people-script.txt
    ```

    Die folgende Ausgabe wird angezeigt:

    ```console
    https://localhost:5001/~ set base https://localhost:5001
    Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json
    
    https://localhost:5001/~ ls
    .        []
    Fruits   [get|post]
    People   [get|post]
    
    https://localhost:5001/~ cd People
    /People    [get|post]
    
    https://localhost:5001/People~ ls
    .      [get|post]
    ..     []
    {id}   [get|put|delete]
    
    https://localhost:5001/People~ get 1
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 12 Jul 2019 19:20:10 GMT
    Server: Kestrel
    Transfer-Encoding: chunked
    
    {
      "id": 1,
      "name": "Scott Hunter"
    }
    
    
    https://localhost:5001/People~
    ```

## <a name="clear-the-output"></a>Löschen der Ausgabe

Sie können alle Ausgaben des HTTP REPL-Tools aus der Befehlsshell entfernen, indem Sie den Befehl `clear` oder `cls` ausführen. Nehmen Sie beispielsweise an, dass die Befehlsshell folgende Ausgabe enthält:

```console
dotnet httprepl https://localhost:5001
(Disconnected)~ set base "https://localhost:5001"
Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json

https://localhost:5001/~ ls
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Führen Sie den folgenden Befehl aus, um die Ausgabe zu löschen:

```console
https://localhost:5001/~ clear
```

Nachdem der vorherige Befehl ausgeführt wurde, enthält die Befehlsshell nur die folgende Ausgabe:

```console
https://localhost:5001/~
```

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [REST-API-Anforderungen](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods)
* [GitHub-Repository für HTTP REPL](https://github.com/aspnet/HttpRepl)
