---
lab:
  title: Konfigurieren und Bereitstellen von CI/CD-Pipelines für Azure SQL-Datenbankprojekte
  module: Develop for an Azure SQL Database
---

# Konfigurieren und Bereitstellen von CI/CD-Pipelines für Azure SQL-Datenbankprojekte

In dieser Übung erstellen, konfigurieren und bereitstellen Sie mit Visual Studio Code und GitHub Actions CI/CD-Pipelines für Azure SQL-Datenbankprojekte. Auf diese Weise können Sie sich mit der Einrichtung von CI/CD-Pipelines für Azure SQL-Datenbankprojekte vertraut machen.

Diese Übung dauert ca. **30** Minuten.

## Vor der Installation

Bevor Sie mit dieser Übung starten können, müssen Sie:

- Ein Azure-Abonnement mit entsprechenden Berechtigungen zum Erstellen und Verwalten von Ressourcen
- Installation von [Visual Studio Code](https://code.visualstudio.com/download) auf Ihrem Computer mit folgenden Erweiterungen:
  - [SQL-Datenbankprojekte](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql)
  - [GitHub-Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github)
- GitHub-Konten.
- Grundlegende Kenntnisse zu *GitHub Actions*-Pipelines.

## Erstellen einer Azure-SQL-Datenbank

Zuerst müssen Sie eine neue Azure SQL-Datenbank erstellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com?azure-portal=true) an. 
1. Navigieren Sie zur Seite **Azure SQL**, und wählen Sie dann **+ Erstellen** aus.
1. Wählen Sie **SQL-Datenbank** > *Einzeldatenbank* und die Schaltfläche **Erstellen** aus.
1. Füllen Sie die erforderlichen Informationen im Dialogfeld **SQL-Datenbank erstellen** aus, und wählen Sie **OK** aus, wobei alle anderen Optionen in den Standardeinstellungen verbleiben.

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
1. Wählen Sie die Option **Azure-Diensten und -Ressourcen den Zugriff auf diesen Server gestatten** aus. Mit dieser Option können GitHub-Aktionen auf die Datenbank zugreifen.

    > **Hinweis:** Beschränken Sie in einer Produktionsumgebung den Zugriff nur auf die erforderlichen IP-Adressen. Außerdem sollten Sie in Betracht ziehen, für Ihre GitHub Action verwaltete Identitäten für den Zugriff auf die Datenbank anstelle der SQL-Authentifizierung zu verwenden. Weitere Informationen finden Sie unter [Verwaltete Identitäten in Microsoft Entra für Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Wählen Sie **Speichern**.

## Einrichten eines GitHub-Repositorys

Als Nächstes müssen Sie ein neues GitHub-Repository einrichten.

1. Öffnen Sie die [GitHub](https://github.com)-Website.
1. Melden Sie sich bei Ihrem GitHub-Konto an.
1. Wechseln Sie zu **Repositorys** unter Ihrem Konto, und wählen Sie **Neu** aus.
1. Wählen Sie unter **Besitzer** Ihr GitHub-Konto aus. Geben Sie den Namen **my-sql-db-repo** ein.
1. Legen Sie das Repository auf **Privat** fest.
1. Klicken Sie auf **Create repository** (Repository erstellen).

### Installieren der Visual Studio Code-Erweiterungen und Klonen des Repositorys

Stellen Sie vor dem Klonen des Repositorys sicher, dass Sie die erforderlichen **Visual Studio Code-Erweiterungen** installiert haben. Weitere Informationen finden Sie im Abschnitt **Bevor Sie beginnen**.

1. Klicken Sie in Visual Studio Code auf **Ansicht** > **Befehlspalette**.
1. Geben Sie in der Befehlspalette `Git: Clone` ein, und wählen Sie ihn aus.
1. Geben Sie die URL ein, die Sie im vorherigen Schritt erstellt haben und wählen Sie **Klonen** aus. Ihre URL sollte dem folgenden Format entsprechen: *https://github.com/<your_account>/<your_repository>.git*.
1. Wählen Sie einen Ordner zum Speichern Ihrer Repositorydateien aus, oder erstellen Sie diesen.

## Erstellen und Konfigurieren eines Azure SQL-Datenbankprojekts

Mit einem Azure SQL-Datenbankprojekt in Visual Studio können Sie Ihr Datenbankschema und Ihre Daten entwickeln, erstellen, testen und veröffentlichen. In diesem Abschnitt erstellen Sie ein Projekt und konfigurieren es so, dass eine Verbindung mit der zuvor eingerichteten Azure SQL-Datenbank hergestellt wird.

1. Klicken Sie in Visual Studio Code auf **Ansicht** > **Befehlspalette**.
1. Geben Sie in der Befehlspalette `Database projects: New` ein, und wählen Sie ihn aus.
    > **Hinweis:** Es kann einige Minuten dauern, bis der SQL-Tools-Dienst für die mssql-Erweiterung installiert wird.
1. Wählen Sie **Azure SQL-Datenbank** aus.
1. Geben Sie den Namen **MyDBProj** ein, und drücken **Sie die EINGABETASTE** , um dies zu bestätigen.
1. Wählen Sie den geklonten GitHub-Repositoryordner aus, um das Projekt zu speichern.
1. Wählen Sie für **Projekte im SDK-Stil** die Option **Ja (empfohlen)** aus.
    > **Hinweis:** Beachten Sie, dass ein neues Projekt mit dem Namen **MyDBProj** erstellt wird.

### Erstellen einer neue SQL-Datei im Projekt

Nachdem das Azure SQL-Datenbankprojekt erstellt wurde, fügen wir dem Projekt eine neue SQL-Datei hinzu, um eine neue Tabelle zu erstellen.

1. Wählen Sie in Visual Studio Code das Symbol **Datenbankprojekte** in der Aktivitätsleiste auf der linken Seite aus.
1. Klicken Sie mit der rechten Maustaste auf Ihren Projektnamen, und wählen Sie **Tabelle hinzufügen** aus.
1. Benennen Sie die Tabelle **Mitarbeitende**, und drücken Sie die **EINGABETASTE**.
1. Ersetzen Sie das vorhandene Skript durch den folgenden Code.

    ```sql
    CREATE TABLE [dbo].[Employees]
    (
        EmployeeID INT PRIMARY KEY,
        FirstName NVARCHAR(50),
        LastName NVARCHAR(50),
        Department NVARCHAR(50)
    );
    ```

1. Schließen Sie den Editor. Beachten Sie, dass die `Employees.sql`-Datei im Projekt gespeichert wird.

## Committen Sie die Änderung im Repository

Nachdem das Azure SQL-Datenbankprojekt erstellt und das Tabellenskript dem Projekt hinzugefügt wurde, committen Sie die Änderungen im Repository.

1. Wählen Sie in Visual Studio Code das Symbol **Quellcodeverwaltung** in der Aktivitätsleiste auf der linken Seite aus.
1. Geben Sie die Nachricht *Projekt erstellt und ein Skript zum Erstellen einer Tabelle hinzugefügt* ein.
1. Klicken Sie auf **Commit**, um die Änderung zu committen.
1. Wählen Sie unter den Auslassungspunkten **Push** aus, um die Änderungen mithilfe von Push an das Repository zu übertragen.

## Überprüfen Sie die Änderungen im Repository

Nachdem Sie nun die Änderungen mithilfe von Push übertragen haben, überprüfen wir sie im GitHub Repository.

1. Öffnen Sie die [GitHub](https://github.com)-Website.
1. Navigieren Sie zum Repository **my-sql-db-repo**.
1. Öffnen Sie auf der Registerkarte **<> Code** den Ordner **MyDBProj**.
1. Überprüfen Sie, ob die Änderungen in der Datei **Employees.sql** auf dem neuesten Stand sind.

## Einrichten der Continuous Integration (CI) mit GitHub Actions

Mit GitHub Actions können Sie Ihre Software Development Workflows direkt in Ihrem GitHub Repository automatisieren, anpassen und ausführen. In diesem Abschnitt konfigurieren Sie einen GitHub Actions-Workflow, um Ihr Azure SQL-Datenbankprojekt zu erstellen und zu testen, indem Sie eine neue Tabelle in der Datenbank erstellen.

### Erstellen eines Dienstprinzipals

1. Wählen Sie oben rechts im Azure-Portal das Symbol **Cloud Shell** aus. Es sieht wie ein `>_`-Symbol aus. Wenn Sie dazu aufgefordert werden, wählen Sie **Bash** als Shelltyp aus.

1. Führen Sie den nachstehenden Befehl im Cloud Shell-Terminal aus. Ersetzen Sie die Werte `<your_subscription_id>`, und `<your_resource_group_name>` durch Ihre tatsächlichen Werte: Sie können diese Werte auf den Seiten **Abonnements** und **Ressourcengruppe** im Azure-Portal abrufen.

    ```azurecli
    az ad sp create-for-rbac --name "MyDBProj" --role contributor --scopes /subscriptions/<your_subscription_id>/resourceGroups/<your_resource_group_name>
    ```

    Öffnen Sie einen Text-Editor, und verwenden Sie die Ausgabe aus dem vorherigen Befehl, um einen Anmeldeinformationsausschnitt wie den Folgenden zu erstellen:
    
    ```
    {
    "clientId": <your_service_principal_appId>,
    "clientSecret": <your_service_principal_password>,
    "tenantId": <your_service_principal_tenant>,
    "subscriptionId": <your_subscription_id>
    }
    ```

1. Lassen Sie den Text-Editor geöffnet. Wir verweisen im nächsten Abschnitt darauf.

### Hinzufügen der Geheimnisse zum Repository

1. Wählen Sie im GitHub-Repository die Option **Einstellungen** aus.
1. Wählen Sie **Geheimnisse und Variablen** und dann **Aktionen** aus.
1. Wählen Sie auf der Registerkarte **Geheimnisse** die Option **Neues Repositorygeheimnis** aus, und geben Sie die folgenden Informationen ein:

    | Name | Wert |
    | --- | --- |
    | AZURE_CREDENTIALS | Die im vorherigen Abschnitt kopierte Dienstprinzipalausgabe.|
    | AZURE_CONN_STRING | Ihre Verbindungszeichenfolge |
   
    Ihre Verbindungszeichenfolge sollte beispielsweise wie folgt aussehen:

    ```Server=<your_sqldb_server>.database.windows.net;Initial Catalog=MyDB;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;Encrypt=True;Connection Timeout=30;```

### Erstellen eines GitHub Actions-Workflows

1. Wählen Sie im GitHub-Repository die Registerkarte **Aktionen** aus.
1. Wählen Sie den Link **Workflow selbst einrichten** aus.
1. Kopieren Sie den folgenden Code in Ihre **main.yml**-Datei. Der Code enthält die Schritte zum Erstellen und Bereitstellen Ihres Datenbankprojekts.

    {% raw %}
    ```yaml
    name: Build and Deploy SQL Database Project
    on:
      push:
        branches:
          - main
    jobs:
      build:
        permissions:
          contents: 'read'
          id-token: 'write'
          
        runs-on: ubuntu-latest  # Can also use windows-latest depending on your environment
        steps:
          - name: Checkout repository
            uses: actions/checkout@v3

        # Install the SQLpackage tool
          - name: sqlpack install
            run: dotnet tool install -g microsoft.sqlpackage
    
          - name: Login to Azure
            uses: azure/login@v1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
    
          # Build and Deploy SQL Project
          - name: Build and Deploy SQL Project
            uses: azure/sql-action@v2.3
            with:
              connection-string: ${{ secrets.AZURE_CONN_STRING }}
              path: './MyDBProj/MyDBProj.sqlproj'
              action: 'publish'
              build-arguments: '-c Release'
              arguments: '/p:DropObjectsNotInSource=true'  # Optional: Customize as needed
    ```
    {% endraw %}
   
      Der Schritt **SQL-Project erstellen und bereitstellen** in Ihrer YAML-Datei stellt mithilfe der im `AZURE_CONN_STRING`-Geheimnis gespeicherten Verbindungszeichenfolge eine Verbindung zu Ihrer Azure SQL-Datenbank her. Die Aktion gibt den Pfad zu Ihrer SQL-Projektdatei an, legt die Aktion zum Veröffentlichen fest, um das Projekt bereitzustellen, und enthält Build-Argumente zum Kompilieren im Release-Modus. Darüber hinaus wird das `/p:DropObjectsNotInSource=true`-Argument verwendet, um sicherzustellen, dass alle Objekte, die nicht in der Quelle vorhanden sind, während der Bereitstellung aus der Zieldatenbank gelöscht werden.

1. Führen Sie für die Änderungen einen Commit aus.

### Testen des GitHub Actions-Workflows

1. Wählen Sie im GitHub-Repository die Registerkarte **Aktionen** aus.
1. Wählen Sie den Workflow **Erstellen und Bereitstellen eines SQL-Datenbankprojekts** aus.
    > **Hinweis:** Sie sehen, wie der Workflow ausgeführt wird. Warten Sie, bis die Überprüfung abgeschlossen ist. Wenn sie bereits abgeschlossen ist, wählen Sie die neueste Ausführung aus, um die Details anzuzeigen.

### Überprüfen Sie die Änderungen in der Azure SQL-Datenbank

Wenn der GitHub Actions-Workflow zum Erstellen und Bereitstellen Ihres Azure SQL-Datenbankprojekts eingerichtet ist, ist es an der Zeit, die Änderungen in Ihrer Azure SQL-Datenbank zu überprüfen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com?azure-portal=true) an. 
1. Navigieren Sie zur **MyDB** SQL-Datenbank.
1. Wählen Sie **Abfrage-Editor** aus.
1. Stellen Sie mithilfe der **sqladmin**-Anmeldeinformationen eine Verbindung mit der Datenbank her.
1. Überprüfen Sie im Abschnitt **Tabellen**, ob die Tabelle **Mitarbeitende** erstellt wurde. Bei Bedarf aktualisieren.

Sie haben erfolgreich einen GitHub Actions-Workflow eingerichtet, um Ihr Azure SQL-Datenbankprojekt zu erstellen und bereitzustellen.

## Bereinigen

Wenn Sie in Ihrem eigenen Abonnement arbeiten, sollten Sie sich am Ende eines Projekts überlegen, ob Sie die erstellten Ressourcen noch benötigen. 

Wenn Ressourcen unnötig ausgeführt werden, kann dies zu zusätzlichen Kosten führen. Sie können Ressourcen einzeln oder die gesamte Gruppe von Ressourcen im [Azure-Portal](https://portal.azure.com?azure-portal=true) löschen.

## Weitere Informationen

Weitere Informationen zur SQL-Datenbank-Projekterweiterung für Azure SQL-Datenbank finden Sie unter [Erste Schritte mit der SQL-Datenbank-Projekterweiterung](https://learn.microsoft.com/azure-data-studio/extensions/sql-database-project-extension-getting-started?azure-portal=true).
