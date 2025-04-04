---
lab:
  title: Importieren und Exportieren von Daten für die Entwicklung in Azure SQL-Datenbank
  module: Import and export data for development in Azure SQL Database
---

# Importieren und Exportieren von Daten für die Entwicklung in Azure SQL-Datenbank

In dieser Übung importieren Sie Daten von einem externen REST-Endpunkt (simuliert mit Azure Static Web-App) und exportieren Daten mit einer Azure-Funktion. Das Lab bietet praktische Erfahrung in der Arbeit mit Azure SQL-Datenbank für Entwicklungszwecke, wobei der Schwerpunkt auf der Integration von REST-APIs und Azure-Funktionen zur Abwicklung von Datenimport-/Datenexportvorgängen liegt.

## Voraussetzungen

Bevor Sie mit diesem Lab beginnen, stellen Sie sicher, dass Sie Folgendes haben:

- Ein aktives Azure-Abonnement mit der Berechtigung, Ressourcen zu erstellen und zu verwalten.
- Grundkenntnisse in Azure SQL-Datenbank, REST-APIs und Azure-Funktionen.
- Visual Studio Code wurde mit den folgenden Erweiterungen installiert:
      - Azure Functions-Erweiterung.
- Git wurde zum Klonen des Repositorys installiert.
- SQL Server Management Studio (SSMS) oder Azure Data Studio zur Verwaltung der Datenbank.

## Einrichten der Umgebung

Beginnen wir mit der Einrichtung der erforderlichen Ressourcen für dieses Lab, einschließlich einer Azure SQL-Datenbank und der Tools, die zum Importieren und Exportieren von Daten benötigt werden.

### Erstellen einer Azure-SQL-Datenbank

In diesem Schritt müssen Sie eine Datenbank in Azure erstellen:

1. Gehen Sie im Azure-Portal zur Seite **SQL-Datenbanken**.
1. Klicken Sie auf **Erstellen**.
1. Füllen Sie die erforderlichen Felder aus:

    | Einstellung | Wert |
    |---|---|
    | Kostenloses serverloses Angebot | Angebot anwenden |
    | Abonnement | Ihr Abonnement |
    | Ressourcengruppe | Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine neue |
    | Datenbankname | **MyDB** |
    | Server | Wählen Sie einen Server aus oder erstellen Sie einen neuen. |
    | Authentifizierungsmethode | SQL-Authentifizierung |
    | Serveradministratoranmeldung | **sqladmin** |
    | Kennwort | Ein sicheres Kennwort eingeben |
    | Kennwort bestätigen | Bestätigen Sie das Kennwort |

1. Wählen Sie **Überprüfen + erstellen** und danach **Erstellen** aus.
1. Navigieren Sie nach Abschluss des Bereitstellens zum Abschnitt **Networking** Ihres ***Azure SQL Server*** (nicht der Azure SQL-Datenbank) und:
    1. Fügen Sie Ihre IP-Adresse zu den Firewallregeln hinzu. Dadurch können Sie SQL Server Management Studio (SSMS) oder Azure Data Studio für die Verwaltung der Datenbank verwenden.
    1. Aktivieren Sie das Kontrollkästchen **Azure-Diensten und -Ressourcen den Zugriff auf diesen Server gestatten**. Dadurch kann die Azure Funktions-App auf den Datenbankserver zugreifen.
    1. Speichern Sie die Änderungen.
1. Navigieren Sie zum Abschnitt **Microsoft Entra ID** Ihres **Azure SQL Server** und stellen Sie sicher, dass Sie die Option **Nur Microsoft Entra-Authentifizierung für diesen Server unterstützen** *deaktivieren* und Ihre Änderungen **speichern**, falls diese Option ausgewählt ist. In diesem Beispiel wird die SQL-Authentifizierung verwendet, daher müssen wir den Entra-Support deaktivieren.

> [!NOTE]
> In einer Produktionsumgebung müssen Sie festlegen, welche Art von Zugriff und von wo aus Sie Zugriff gewähren möchten. Die Funktion ändert sich zwar geringfügig, wenn Sie nur die Entra-Authentifizierung wählen, aber beachten Sie, dass Sie immer noch die Option *Zugriff von Azure-Diensten und -Ressourcen auf diesen Server erlauben* aktivieren müssen, damit die Azure Funktions-App auf den Server zugreifen kann.

### Klonen des GitHub-Repositorys

1. Öffnen Sie **Visual Studio Code**.

1. Klonen Sie das GitHub-Repo und bereiten Sie Ihr Projekt vor:

    1. Öffnen Sie in **Visual Studio Code** die **Befehlspalette**, indem Sie **Strg+Umschalt+P** (Windows) oder **Cmd+Umschalt+P** (Mac) drücken.
    1. Geben Sie **Git: Clone** ein und wählen Sie **Git: Clone** aus.
    1. Geben Sie in der Eingabeaufforderung die folgende URL ein, um das Repository zu klonen:
        ```bash
        https://github.com/MicrosoftLearning/mslearn-sql-dev.git
        ```

    1. Wählen Sie den Zielordner aus, in den Sie das Repository klonen möchten.

### Einrichten von Azure Blob Storage für JSON-Daten

Wir richten nun **Azure Blob Storage** ein, um die Datei **employees.json** zu hosten. Führen Sie diese Schritte im Azure-Portal und in **Visual Studio Code** aus.

Beginnen wir mit der Erstellung eines Azure Storage-Kontos.

1. Gehen Sie im **Azure-Portal** zur Seite **Speicherkonten**.
1. Klicken Sie auf **Erstellen**.
1. Füllen Sie die erforderlichen Felder aus:

    | Einstellung | Wert |
    |---|---|
    | Abonnement | Ihr Abonnement |
    | Ressourcengruppe | Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine neue |
    | Speicherkontoname | Wählen Sie einen global eindeutigen Namen |
    | Region | Wählen Sie die Ihnen am nächsten gelegene Region aus. |
    | Primärer Dienst | **Azure Blob Storage oder Azure Data Lake Storage Gen2** |
    | Leistung | Standard |
    | Redundanz | Lokal redundanter Speicher (LRS) |

1. Wählen Sie **Überprüfen + erstellen** und danach **Erstellen** aus.
1. Warten Sie, bis das Speicherkonto erstellt wurde.

Da wir nun über ein Konto verfügen, können wir **employees.json** in Blob Storage hochladen.

1. Gehen Sie im Azure-Portal zur Seite **Speicherkonten**.
1. Wählen Sie dann Ihr Speicherkonto aus.
1. Navigieren Sie zum Abschnitt **Container**.
1. Erstellen Sie einen neuen Container mit dem Namen **jsonfiles**.
1. Klicken Sie im Container auf **Upload** und laden Sie die Datei **employees.json** hoch, die sich unter **/Allfiles/Labs/04/blob-storage** im geklonten Verzeichnis befindet.

Wir könnten zwar einen anonymen Zugriff auf die Datei erlauben, aber in unserem Fall sollten wir eine *Shared Access Signature (SAS)* für diese Datei generieren, um einen sicheren Zugriff zu gewährleisten.

1. Wählen Sie auf dem Container **jsonfiles** die Datei **employees.json** aus.
1. Wählen Sie im Kontextmenü der Datei **Generate SAS** aus.
1. Überprüfen Sie die Einstellungen und wählen Sie **SAS und URL generieren**.
1. Ein Blob-SAS-Token und eine Blob-SAS-URL werden generiert. Kopieren Sie das **Blob-SAS-Token** und die **Blob-SAS-URL** für die nächsten Schritte. Sobald Sie dieses Fenster schließen, können Sie nicht mehr auf den Token-Wert zugreifen.

Wir sollten jetzt eine sichere URL haben, um auf die Datei **employees.json** zuzugreifen. Lassen Sie uns fortfahren und sie testen.

1. Öffnen Sie einen neuen Browser-Tab und fügen Sie die **Blob SAS URL** ein.
1. Sie sollten den Inhalt der Datei **employees.json** im Browser sehen, der wie folgt aussehen sollte:

    ```json
    {
        "employees": [
            {
                "EmployeeID": 1,
                "FirstName": "John",
                "LastName": "Doe",
                "Department": "HR"
            },
            {
                "EmployeeID": 2,
                "FirstName": "Jane",
                "LastName": "Smith",
                "Department": "Engineering"
            }
        ]
    }
    ```

### Importieren Sie Daten aus dem Blob-Speicher in die Azure SQL-Datenbank

Wir sind nun bereit, die Daten aus der Datei **employees.json**, die auf Azure Blob Storage gehostet wird, in unsere Azure SQL-Datenbank zu importieren.

Wir müssen zunächst einen **Hauptschlüssel** und eine **Datenbankweit gültige Anmeldeinformation** in der Azure SQL-Datenbank erstellen.

1. Stellen Sie über **SQL Server Management Studio** (SSMS) oder **Azure Data Studio** eine Verbindung zu Ihrer Azure SQL-Datenbank her.
1. Führen Sie den folgenden SQL-Befehl aus, um einen Hauptschlüssel zu erstellen *, falls Sie noch keinen haben*:

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourStrongPassword!';
    ```

1. Erstellen Sie als Nächstes ein **Datenbankweit gültige Anmeldeinformation**, um auf den Azure Blob Storage zuzugreifen, indem Sie den folgenden SQL-Befehl ausführen:

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL MyBlobCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', 
    SECRET = '<your-sas-token>';
    ```

    Ersetzen Sie <Ihr-SAS-Token> durch das zuvor generierte Blob-SAS-Token.

1. Schließlich benötigen Sie eine **Datenquelle**, um auf den Azure Blob Storage zugreifen zu können. Führen Sie den folgenden SQL-Befehl aus, um eine **Datenquelle** zu erstellen:

    ```sql
    CREATE EXTERNAL DATA SOURCE MyBlobDataSource
    WITH (
        TYPE = BLOB_STORAGE,
        LOCATION = 'https://<your-storage-account-name>.blob.core.windows.net',
        CREDENTIAL = MyBlobCredential
    );
    ```

    Ersetzen Sie <your-storage-account-name> durch den Namen Ihres Azure-Speicherkontos.

Alles ist nun so eingerichtet, dass die Daten aus der Datei **employees.json** in die *Azure SQL-Datenbank* importiert werden können.

Verwenden Sie den folgenden SQL-Befehl, um Daten aus der Datei **employees.json** zu importieren, die auf *Azure Blob Storage* gehostet wird:

```sql
SELECT EmployeeID
    , FirstName
    , LastName
    , Department
INTO dbo.employee_data
FROM OPENROWSET(
    BULK 'jsonfiles/employees.json',
    DATA_SOURCE = 'MyBlobDataSource',
    SINGLE_CLOB
) AS JSONData
CROSS APPLY OPENJSON(JSONData.BulkColumn, '$.employees')
WITH (
    EmployeeID INT '$.EmployeeID',
    FirstName NVARCHAR(50) '$.FirstName',
    LastName NVARCHAR(50) '$.LastName',
    Department NVARCHAR(50) '$.Department'
) AS EmployeeData;
```

Dieser Befehl liest die Datei **employees.json** aus dem Container **jsonfiles** im *Azure Blob Storage* und importiert die Daten in die Tabelle **employee_data** in der Azure SQL-Datenbank.

Sie können nun den folgenden SQL-Befehl ausführen, um den Datenimport zu überprüfen: 

```sql
SELECT * FROM dbo.employee_data;
```

Sie sollten die Daten aus der Datei **employees.json** sehen, die in die Tabelle **employee_data** importiert wurde.

---

## Datenexport mithilfe einer Azure Funktions-App

In diesem Teil des Labs erstellen Sie eine Azure-Funktions-App in C#, um Daten aus Ihrer Azure SQL-Datenbank zu exportieren. Diese Funktion ruft die Daten ab und gibt sie als JSON-Antwort zurück.

### Erstellen einer Azure-Funktions-App in Visual Studio Code

Beginnen wir mit der Erstellung einer Azure Funktions-App in Visual Studio Code:

1. Öffnen Sie **Visual Studio Code**.
1. Navigieren Sie im Bereich „Erkunden“ zum Ordner **/Allfiles/Labs/04/azure-functions**.
1. Klicken Sie mit der rechten Maustaste auf den Ordner **azure-functions** und wählen Sie **In integriertem Terminal öffnen** aus.
1. Melden Sie sich im VS Code-Terminal mit dem folgenden Befehl bei Azure an:

    ```bash
    az login
    ```

1. (Optional) Wenn Sie mehrere Abonnements haben, legen Sie das aktive Abonnement fest:

    ```bash
    az account set --subscription <your-subscription-id>
    ```

1. Führen Sie den folgenden Befehl aus, um eine Azure Funktions-App zu erstellen:

    ```bash
    $functionappname = "YourUniqueFunctionAppName"
    $resourcegroup = "YourResourceGroupName"
    $location = "YourLocation"
    # NOTE - The following should be a new storage account name where your Azure function will resided.
    # It should not be the same Storage Account name used to store the JSON file
    $storageaccount = "YourStorageAccountName"

    az storage account create --name $storageaccount --location $location --resource-group $resourcegroup --sku Standard_LRS
    
    az functionapp create --resource-group $resourcegroup --consumption-plan-location $location --runtime dotnet --name  $functionappname --os-type Linux --storage-account $storageaccount --functions-version 4
    
    ```

    ***Ersetzen Sie die Platzhalter durch Ihre eigenen Werte. Verwenden Sie nicht den Speicherkontonamen, der für die JSON-Datei verwendet wurde. Dieses Skript muss ein neues Speicherkonto erstellen, um die Azure Funktions-App zu speichern***.


### Erstellen einer neuen Funktions-App in Visual Studio Code

Lassen Sie uns eine neue Funktion in Visual Studio Code erstellen, um Daten aus der Azure SQL-Datenbank zu exportieren:

Möglicherweise müssen Sie die Azure Functions-Erweiterung zu Visual Studio Code hinzufügen, falls Sie dies noch nicht getan haben. Sie können dies tun, indem Sie im Bereich „Erweiterungen“ nach **Azure Functions** suchen und es installieren.

1. Drücken Sie in Visual Studio Code **Strg+Umschalt+P** (Windows) oder **Cmd+Umschalt+P** (Mac), um die Befehlspalette zu öffnen.
1. Geben Sie **Azure Functions: Neues Projekt erstellen** ein, und wählen Sie die Option aus.
1. Wählen Sie Ihr Verzeichnis für **Funktions-Apps** aus. Wählen Sie den Ordner **/Allfiles/Labs/04/azure-functions** des GitHub-Klon-Repos aus.
1. Wählen Sie **C#** als Sprache aus.
1. Wählen Sie **.NET 8.0 LTS** als Runtime aus.
1. Wählen Sie **HTTP-Trigger** als Vorlage aus.
1. Rufen Sie die Funktion **ExportDataFunction** auf.
1. Erstellen Sie den Namespace **Contoso.ExportFunction**.
1. Geben Sie der Funktion die Zugriffsebene **anonymous**.

### Schreiben des C#-Codes für den Export von Daten

1. Für die Azure Funktions-App müssen möglicherweise zuerst einige Pakete installiert werden. Sie können sie installieren, indem Sie die folgenden Befehle ausführen:

    ```bash
    dotnet add package Microsoft.Data.SqlClient
    dotnet add package Newtonsoft.Json
    dotnet restore
    
    npm install -g azure-functions-core-tools@4 --unsafe-perm true

    ```

1. Ersetzen Sie den Platzhalter-Funktionscode durch den folgenden C#-Code, um Ihre Azure SQL-Datenbank abzufragen und die Ergebnisse als JSON zurückzugeben:

    ```csharp
    using System;
    using System.IO;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Azure.WebJobs;
    using Microsoft.Azure.WebJobs.Extensions.Http;
    using Microsoft.AspNetCore.Http;
    using Microsoft.Extensions.Logging;
    using Microsoft.Data.SqlClient;
    using Newtonsoft.Json;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    public static class ExportDataFunction
    {
        [FunctionName("ExportDataFunction")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
            ILogger log)
        {
            // Connection string to the database
            // NOTE: REPLACE THIS CONNECTION STRING WITH THE CONNECTION STRING OF YOUR AZURE SQL DATABASE
            string connectionString = "Server=tcp:yourserver.database.windows.net;Database=DataLabDatabase;User ID=youruserid;Password=yourpassword;Encrypt=True;";
            
            // List to hold employee data
            List<Employee> employees = new List<Employee>();
            
            try
            {
                // Establishing connection to the database
                using (SqlConnection conn = new SqlConnection(connectionString))
                {
                    await conn.OpenAsync();
                    var query = "SELECT EmployeeID, FirstName, LastName, Department FROM employee_data";
                    
                    // Executing the query
                    using (SqlCommand cmd = new SqlCommand(query, conn))
                    {
                        // Adding parameters to the query (if needed)
                        // cmd.Parameters.AddWithValue("@ParameterName", parameterValue);

                        using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
                        {
                            // Reading data from the database
                            while (await reader.ReadAsync())
                            {
                                employees.Add(new Employee
                                {
                                    EmployeeID = (int)reader["EmployeeID"],
                                    FirstName = reader["FirstName"].ToString(),
                                    LastName = reader["LastName"].ToString(),
                                    Department = reader["Department"].ToString()
                                });
                            }
                        }
                    }
                }
            }
            catch (SqlException ex)
            {
                // Logging SQL errors
                log.LogError($"SQL Error: {ex.Message}");
                return new StatusCodeResult(500);
            }
            catch (System.Exception ex)
            {
                // Logging unexpected errors
                log.LogError($"Unexpected Error: {ex.Message}");
                return new StatusCodeResult(500);
            }
    
            // Returning the list of employees as a JSON response
            return new OkObjectResult(JsonConvert.SerializeObject(employees, Formatting.Indented));
        }
    
        // Employee class to hold employee data
        public class Employee
        {
            public int EmployeeID { get; set; }
            public string FirstName { get; set; }
            public string LastName { get; set; }
            public string Department { get; set; }
        }
    }
    ```

    *Denken Sie daran, den **connectionString** durch den Verbindungsstring zu Ihrer Azure SQL-Datenbank zu ersetzen und Ihr sqladmin-Kennwort ebenfalls in den Verbindungsstring einzugeben.*

    > **Hinweis:** Beschränken Sie in einer Produktionsumgebung den Zugriff nur auf die erforderlichen IP-Adressen. Ziehen Sie außerdem in Betracht, für Ihre Azure Funktions-App anstelle der SQL-Authentifizierung verwaltete Identitäten für den Zugriff auf die Datenbank zu verwenden. Weitere Informationen finden Sie unter [Verwaltete Identitäten in Microsoft Entra für Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Speichern Sie den Funktionscode und stellen Sie sicher, dass Ihre **.csproj**-Datei das **Newtonsoft.Json**-Paket für die Serialisierung von Objekten in JSON enthält. Wenn es nicht enthalten ist, fügen Sie es hinzu:

    ```xml
    <PackageReference Include="Newtonsoft.Json" Version="13.X.X" />
    ```

Zeit, die Azure Funktions-App in Azure bereitzustellen.

### Bereitstellen der Azure Funktions-App in Azure

1. Führen Sie im integrierten Terminal von **Visual Studio Code** den folgenden Befehl aus, um die Azure-Funktion in Azure bereitzustellen:

    ```bash
    func azure functionapp publish <your-function-app-name>
    ```

    Ersetzen Sie ***<your-function-app-name>*** durch den Namen Ihrer Azure Funktions-App.

1. Warten Sie, bis die Bereitstellung abgeschlossen ist.

### Rufen Sie die URL der Azure Funktions-App ab

1. Öffnen Sie das Azure-Portal und navigieren Sie zu Ihrer Azure Funktions-App.
1. Im Abschnitt *Übersicht* unter der Registerkarte *Funktion* wird Ihre neue Funktion aufgelistet. Wählen Sie sie aus.
1. Wählen Sie auf der Registerkarte **Code + Test** die Option **Funktions-URL abrufen** aus.
1. Kopieren Sie die **Standardeinstellung (Funktionstaste)**, wir werden sie in Kürze benötigen. Die URL sollte in etwa so aussehen:
   
   ```url
   https://YourFunctionAppName.azurewebsites.net/api/ExportDataFunction?code=2pjO0HqRyz_13DHQg8ga-ysdDWbDU_eHdtlixbAHLVEGAzFuplomUg%3D%3D
   ```

### Testen der Azure Funktions-App

1. Sobald die Bereitstellung abgeschlossen ist, können Sie die Funktion testen, indem Sie eine HTTP-Anfrage an die URL der Funktionstaste senden, die Sie zuvor aus dem Visual Studio Code-Terminal kopiert haben:

    ```bash
    curl https://<your-function-app-name>.azurewebsites.net/api/ExportDataFunction?code=<the function key embedded to your function URL>
    ```

1. Die Antwort sollte die exportierten Daten aus Ihrer Tabelle ***employee_data*** im JSON-Format enthalten.

Diese Funktion ist zwar ein einfaches Beispiel, aber Sie können sie erweitern, um komplexere Logik und Datenverarbeitung wie Filtern, Sortieren und Aggregieren von Daten und vieles mehr einzubeziehen.  Ihr Code könnte auch um Fehlerbehandlung, Protokollierung und Sicherheitsfunktionen erweitert werden.

### Bereinigen der Ressourcen

Nach Abschluss des Labs können Sie die in dieser Übung erstellten Ressourcen löschen, um zusätzliche Kosten zu vermeiden:

- Löschen Sie die Azure SQL-Datenbank.
- Löschen Sie das Azure-Speicherkonto.
- Löschen Sie die Azure Funktions-App.
- Löschen Sie die Ressourcengruppe, die die Ressourcen enthält.
