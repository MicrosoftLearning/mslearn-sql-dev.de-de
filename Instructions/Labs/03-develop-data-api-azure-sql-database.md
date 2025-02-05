---
lab:
  title: Entwickeln einer Daten-API für Azure SQL-Datenbank
  module: Develop a Data API for Azure SQL Database
---

# Entwickeln einer Daten-API für Azure SQL-Datenbank

In dieser Übung entwickeln Sie mit Azure Static Web Apps eine Daten-API für eine Azure SQL-Datenbank und stellen diese bereit. Dies bietet praktische Erfahrungen beim Einrichten einer Daten-API-Generator-Konfiguration und deren Bereitstellung in einer Azure Static Web App-Umgebung.

## Voraussetzungen

Bevor Sie mit dieser Übung beginnen, stellen Sie Folgendes sicher:

- Ein aktives Azure-Abonnement.
- Grundlegende Kenntnisse über Azure SQL-Datenbank, Azure Static Web-Apps und GitHub.
- Visual Studio Code ist mit den erforderlichen Erweiterungen installiert.
- Ein GitHub-Konto zum Verwalten des Repositorys.

## Einrichten der Umgebung

Es gibt einige Schritte, die Sie ausführen müssen, um die Umgebung für diese Übung einzurichten.

### Installieren Sie die Visual Studio Code-Erweiterungen 

Bevor Sie mit der Übung beginnen können, müssen Sie die Visual Studio Code-Erweiterungen installieren.

1. Öffnen Sie Visual Studio Code.
1. Öffnen Sie ein Terminal-Fenster in Visual Studio Code.
1. Installieren Sie die Static Web Apps-CLI mit dem folgenden Befehl:

    ```bash
    npm install -g @azure/static-web-apps-cli
    ```

1. Installieren Sie die Data API Builder-CLI mit dem folgenden Befehl:

    ```bash
    dotnet tool install --global Microsoft.DataApiBuilder
    ```

Visual Studio Code ist jetzt mit den erforderlichen Erweiterungen eingerichtet.

### Erstellen einer Azure-SQL-Datenbank

Wenn Sie dies noch nicht getan haben, müssen Sie eine Azure SQL-Datenbank erstellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com?azure-portal=true) an. 
1. Navigieren Sie zur Seite **Azure SQL**, und wählen Sie dann **+ Erstellen** aus.
1. Wählen Sie **SQL-Datenbank** > *Einzeldatenbank* und die Schaltfläche **Erstellen** aus.
1. Füllen Sie die erforderlichen Informationen im Dialogfeld **SQL-Datenbank erstellen** aus, und wählen Sie **OK** aus (lassen Sie alle anderen Optionen auf ihrer Standardeinstellung).

    | Einstellung | Wert |
    | --- | --- |
    | Kostenloses serverloses Angebot | *Angebot anwenden* |
    | Abonnement | Ihr Abonnement |
    | Ressourcengruppe | *Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine neue* |
    | Datenbankname | *MyDB* |
    | Server | *Klicken Sie auf den Link **Neu erstellen*** |
    | Servername | *Einen eindeutigen Namen wählen* |
    | Location | *Wählen Sie einen Standort aus* |
    | Authentifizierungsmethode | *SQL-Authentifizierung verwenden* |
    | Serveradministratoranmeldung | *sqladmin* |
    | Kennwort | *Geben Sie ein Kennwort ein.* |
    | Kennwort bestätigen | *Bestätigen Sie das Kennwort* |

1. Wählen Sie **Überprüfen und erstellen** und danach **Erstellen** aus.
1. Navigieren Sie nach Abschluss der Bereitstellung zum Azure SQL-Datenbank *Server*, den Sie erstellt haben.
1. Wählen Sie im linken Bereich unter **Sicherheit** die Option **Netzwerk** aus. Fügen Sie Ihre IP-Adresse zu den Firewallregeln hinzu.
1. Wählen Sie **Speichern**.

### Fügen Sie Beispieldaten zur Datenbank hinzu

Nachdem Sie nun über eine Azure SQL-Datenbank verfügen, müssen Sie einige Beispieldaten hinzufügen. Auf diese Weise können Sie die API testen, sobald sie ausgeführt wird.

1. Navigieren Sie zu Ihrer neu erstellten Azure SQL-Datenbank.
1. Verwenden Sie den **Abfrage-Editor** im Azure-Portal, um das folgende SQL-Skript auszuführen:

    ```sql
    CREATE TABLE [dbo].[Employees]
    (
        EmployeeID INT PRIMARY KEY,
        FirstName NVARCHAR(50),
        LastName NVARCHAR(50),
        Department NVARCHAR(50)
    );
    
    INSERT INTO [dbo].[Employees] VALUES (1,'John', 'Smith', 'Accounting');
    INSERT INTO [dbo].[Employees] VALUES (2,'Michelle', 'Sanchez', 'IT');
    
    SELECT * FROM [dbo].[Employees];
    ```

### Erstellen einer einfachen Web-App in GitHub

Bevor wir eine Azure Static Web-App erstellen können, müssen wir eine einfache Web-App in GitHub erstellen.

1. Um eine einfache Web-App in GitHub zu erstellen, wechseln Sie zur [Generierung einer Vanilla-Website](https://github.com/staticwebdev/vanilla-basic/generate).
1. Stellen Sie sicher, dass die Repostiory-Vorlage auf **staticwebdev/vanilla-basic** festgelegt ist.
1. Wählen Sie unter ***Besitzer*** Ihr GitHub-Konto aus.
1. Geben Sie unter ***Repositorynamen*** den Namen **my-sql-repo** ein.
1. Legen Sie das Repository als **Privat** fest.
1. Wählen Sie die Schaltfläche **Repository erstellen** aus.

## Erstellen Sie eine Azure Static Web-App

Zuerst erstellen wir unsere statische Web-App und fügen dann die Konfiguration des Daten-API-Generators hinzu.

1. Gehen Sie im Azure-Portal zur Seite **Static Web Apps**.
1. Wählen Sie **+ Erstellen** aus.
1. Füllen Sie die folgenden Informationen im Dialogfeld **Static Web App erstellen** aus (lassen Sie alle anderen Optionen auf ihren Standardeinstellungen):

    | Einstellung | Wert |
    | --- | --- |
    | Abonnement | Ihr Abonnement |
    | Ressourcengruppe | *Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine neue* |
    | Name | *Ein eindeutiger Name* |
    | Hostingplanquelle | *GitHub* |
    | GitHub-Konto | *Wählen Sie Ihr Konto aus* |
    | Organisation | *Höchstwahrscheinlich Ihr GitHub-Benutzername* |
    | Repository | *Wählen Sie das Repository aus, das Sie im vorherigen Schritt erstellt haben* |
    | Bankfiliale | *main* |

1. Wählen Sie **Überprüfen + erstellen** und danach **Erstellen** aus.
1. Gehen Sie nach der Bereitstellung zur Ressource.
1. Wählen Sie die Schaltfläche **App im Browser anzeigen** aus. Es sollte eine einfache Webseite mit einer Willkommensnachricht angezeigt werden. Sie können die Registerkarte schließen.

## Hinzufügen einer Konfigurationsdatei für den Daten-API-Generator

Es ist an der Zeit, die Konfiguration des Daten-API-Generators zur Azure Static Web App hinzuzufügen. Wir müssen eine neue Datei im GitHub-Repository erstellen, um die Konfiguration des Daten-API-Generators hinzuzufügen.

1. Klonen Sie in Visual Studio Code das GitHub-Repository, das Sie zuvor erstellt haben.
1. Öffnen Sie ein Terminal-Fenster in Visual Studio Code.
1. Führen Sie den folgenden Befehl aus, um eine neue Konfigurationsdatei für den Daten-API-Generator zu erstellen:

    ```bash
    swa db init --database-type "mssql"
    ```

    Dadurch wird ein neuer Ordner mit dem Namen *swa-db-connections* und eine Datei namens *staticwebapp.database.config.json* in diesem Ordner erstellt.

1. Führen Sie den folgenden Befehl aus, um die Datenbankentitäten zur Konfigurationsdatei hinzuzufügen:

    ```bash
    dab add "Employees" --source "dbo.Employees" --permissions "anonymous:*" --config "swa-db-connections/staticwebapp.database.config.json"
    ```

1. Überprüfen Sie die Inhalte der Datei *staticwebapp.database.config.json*. 
1. Commiten und pushen Sie die Änderungen in das Git-Repository.

## Konfigurieren der Datenbankverbindung

1. Navigieren Sie im Azure-Portal zu der von Ihnen erstellten Azure Static Web App.
1. Wählen Sie unter **Einstellungen** die Option **Datenbankverbindung** aus.
1. Wählen Sie **Vorhandene Datenbanken verknüpfen** aus.
1. Wählen Sie im Dialogfeld **Datenbank verknüpfen* die Azure SQL-Datenbank aus, die Sie zuvor mit den folgenden zusätzlichen Einstellungen erstellt haben.

    | Einstellung | Wert |
    | --- | --- |
    | Datenbanktyp | *Azure SQL-Datenbank* |
    | Authentifizierungstyp | *Verbindungszeichenfolge* |
    | Benutzername | *Ihr Administratorbenutzername* |
    | Kennwort | *das Kennwort, das Sie dem Administratorbenutzer gegeben haben* |
    | Bestätigungskontrollkästchen | *Überprüft* |

   > **Hinweis:** Beschränken Sie in einer Produktionsumgebung den Zugriff nur auf die erforderlichen IP-Adressen. Darüber hinaus sollten Sie verwaltete Identitäten für Ihre Static Web App verwenden, um auf die Datenbank anstatt auf die SQL-Authentifizierung zuzugreifen. Weitere Informationen finden Sie unter [Verwaltete Identitäten in Microsoft Entra für Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).
1. **Link** wählen.

## Testen des Daten-API-Endpunkts

Jetzt müssen wir nur noch den Daten-API-Endpunkt testen.

1. Navigieren Sie im Azure-Portal zu der von Ihnen erstellten Azure Static Web App.
1. Kopieren Sie auf der Seite „Übersicht“ die URL der Web-App.
1. Öffnen Sie eine neue Browserregisterkarte, und fügen Sie die URL ein. Die einfache Webseite mit der Meldung **Vanilla JavaScript App** sollte weiterhin angezeigt werden.
1. Fügen Sie **/data-api** am Ende der URL hinzu, und drücken Sie dann die **EINGABETASTE**. Es sollte **Fehlerfrei** angezeigt werden, um anzugeben, dass die Daten-API funktioniert.
1. Fügen Sie am Ende der URL **/data-api/rest/Employees** hinzu, und drücken Sie die **EINGABETASTE**. Sie sollten die Beispieldaten sehen, die Sie zuvor der Azure SQL-Datenbank hinzugefügt haben.

Sie haben erfolgreich eine Daten-API für eine Azure SQL-Datenbank mit Azure Static Web Apps entwickelt und bereitgestellt.

## Bereinigen

Wenn Sie in Ihrem eigenen Abonnement arbeiten, sollten Sie sich am Ende eines Projekts überlegen, ob Sie die erstellten Ressourcen noch benötigen. 

Wenn Ressourcen unnötig ausgeführt werden, kann dies zu zusätzlichen Kosten führen. Sie können Ressourcen einzeln oder die gesamte Gruppe von Ressourcen im [Azure-Portal](https://portal.azure.com?azure-portal=true) löschen.

## Weitere Informationen

Weitere Informationen zum Daten-API-Generator für Azure SQL-Datenbanken finden Sie unter [Was ist der Daten-API-Generator für Azure-Datenbanken?](https://learn.microsoft.com/azure/data-api-builder/overview?azure-portal=true).
