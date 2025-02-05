---
lab:
  title: Konfigurieren einer verwalteten Identität für Azure SQL-Datenbank
  module: Explore Azure SQL Database safety practices for development
---

# Konfigurieren einer verwalteten Identität für Azure SQL-Datenbank

In dieser Übung fügen Sie eine verwaltete Identität zu einer Beispiel-Web-App hinzu, ohne die Anmeldeinformationen im Code zu speichern.

Azure App Service bietet eine hoch skalierbare und selbstverwaltende Webhosting-Lösung. Eines der wichtigsten Features ist die Bereitstellung einer verwalteten Identität für Ihre App, die den Schutz des Zugriffs auf Azure SQL-Datenbank und andere Azure-Dienste vereinfacht. Durch die Verwendung verwalteter Identitäten können Sie die Sicherheit Ihrer App verbessern, indem Sie es überflüssig machen, vertrauliche Informationen wie Anmeldeinformationen in Verbindungszeichenfolgen zu speichern. 

Diese Übung dauert ca. **30** Minuten.

## Vor der Installation

Bevor Sie mit dieser Übung starten können, müssen Sie:

- Ein Azure-Abonnement mit entsprechenden Berechtigungen zum Erstellen und Verwalten von Ressourcen
- [**Visual Studio Code**](https://code.visualstudio.com/download?azure-portal=true) mit den folgenden drei Erweiterungen auf Ihrem Computer installiert:
    - [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice?azure-portal=true).

## Erstellen einer Webanwendung und einer Azure SQL-Datenbank

Zuerst erstellen wir eine Webanwendung und eine Azure SQL-Datenbank.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com?azure-portal=true) an.
1. Suchen Sie nach **Abonnements**, und wählen Sie diese Option aus.
1. Navigieren Sie unter **Einstellungen** zu **Ressourcenanbieter**, suchen Sie nach dem Anbieter **Microsoft.Sql** und wählen Sie **Registrieren** aus.
1. Wechseln Sie im Azure-Portal zurück zur Seite „Home“, und klicken Sie auf **Ressource erstellen**.
1. Suchen Sie nach **Web-App + Datenbank**, und wählen Sie die Option aus.
1. Wählen Sie **Erstellen** aus und geben Sie die erforderlichen Details ein:

    | Gruppe | Einstellung | Wert |
    | --- | --- | --- |
    | **Projektdetails** | **Abonnement** | Wählen Sie Ihr Azure-Abonnement. |
    | **Projektdetails** | **Ressourcengruppe** | Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine neue. |
    | **Projektdetails** | **Region** | Wählen Sie die Region aus, in der Sie Ihre Web-App hosten möchten. |
    | **Web App-Details** | **Name** | Geben Sie einen eindeutigen Namen für Ihre Web-App ein. |
    | **Web App-Details** | **Runtimestapel** | .NET 8 (LTS) |
    | **Datenbank** | **Engine** | SqlAzure |
    | **Datenbank** | **Servername** | Geben Sie einen eindeutigen Namen für Ihren SQL-Server ein |
    | **Datenbank** | **Datenbankname** | Geben Sie einen eindeutigen Namen für Ihre Datenbank ein |
    | **Hosting** | **Hostingplan** | Grundlegend |

    > **Hinweis:** Wählen Sie für Produktions-Workloads **Standard – Universelle Produktions-Apps** aus. Benutzername und Kennwort der neuen Datenbank werden automatisch generiert. Um diese Werte nach dem Bereitstellen abzurufen, gehen Sie zu den **Verbindungszeichenfolgen** auf der Seite **Umgebungsvariablen** Ihrer App. 

1. Wählen Sie **Überprüfen + erstellen** und danach **Erstellen** aus. Es kann einige Minuten dauern, bis die Bereitstellung abgeschlossen ist.
1. Stellen Sie in Azure Data Studio eine Verbindung zu Ihrer Datenbank her und führen Sie den folgenden Code aus.

    ```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        ProductName NVARCHAR(100),
        Category NVARCHAR(50),
        Price DECIMAL(10, 2),
        Stock INT
    );
    
    INSERT INTO Products (ProductID, ProductName, Category, Price, Stock) VALUES
    (1, 'Laptop', 'Electronics', 999.99, 50),
    (2, 'Smartphone', 'Electronics', 699.99, 150),
    (3, 'Desk Chair', 'Furniture', 89.99, 200),
    (4, 'Coffee Maker', 'Appliances', 49.99, 100),
    (5, 'Book', 'Books', 19.99, 300);
    ```

## Konto als SQL-Administrator hinzufügen

Als Nächstes fügen Sie Ihren Kontozugriff zur Datenbank hinzu. Dies ist erforderlich, da nur über Microsoft Entra authentifizierte Konten andere Microsoft Entra ID-Benutzende erstellen können, die eine Voraussetzung für die nachfolgenden Schritte in dieser Übung sind.

1. Navigieren Sie zu dem Azure SQL-Server, den Sie zuvor erstellt haben.
1. Wählen Sie im Menü **Einstellungen** auf der linken Seite **Microsoft Entra ID** aus.
1. Wählen Sie **Administrator festlegen** aus.
1. Suchen Sie nach Ihrem Konto und wählen Sie es aus.
1. Wählen Sie **Speichern**.

## Aktivieren einer verwalteten Identität

Als Nächstes aktivieren Sie die systemseitig zugewiesene verwaltete Identität für Ihre Azure Web-App, eine bewährte Sicherheitsmethode, die eine automatisierte Verwaltung von Anmeldeinformationen ermöglicht.

1. Navigieren Sie im Azure-Portal zu Ihrer Web-App.
1. Wählen Sie im Menü auf der linken Seite unter **Einstellungen** die Option **Identität** aus.
1. Wechseln Sie auf der Registerkarte **System zugewiesen** den **Status** zu **Ein** und wählen Sie **Speichern**. Falls Sie eine Meldung erhalten, in der Sie gefragt werden, ob Sie die systemseitig zugewiesene verwaltete Identität für Ihre Web-App aktivieren möchten, wählen Sie **Ja** aus.

## Gewähren des Zugriffs auf die Azure SQL-Datenbank

1. Stellen Sie mit Azure Data Studio eine Verbindung zur Azure SQL-Datenbank her. Wählen Sie **Microsoft Entra ID – Universal mit MFA-Unterstützung** und geben Sie Ihren Benutzernamen ein.
1. Wählen Sie Ihre Datenbank aus und öffnen Sie dann einen neuen Abfrage-Editor.
1. Führen Sie die folgenden SQL-Befehle aus, um einen Benutzenden für die verwaltete Identität zu erstellen und die erforderlichen Berechtigungen zuzuweisen. Bearbeiten Sie das Skript, indem Sie den Namen Ihrer Web-App angeben.

    ```sql
    CREATE USER [your-web-app-name] FROM EXTERNAL PROVIDER;
    ALTER ROLE db_datareader ADD MEMBER [your-web-app-name];
    ALTER ROLE db_datawriter ADD MEMBER [your-web-app-name];
    ```

## Erstellen einer  -Webanwendung

Als Nächstes erstellen Sie eine ASP.NET-Anwendung, die Entity Framework Core mit Azure SQL-Datenbank verwendet, um eine Liste von Produkten auf der Produkttabelle anzuzeigen.

### Erstellen Ihres Projekts

1. Erstellen Sie in VS Code einen neuen Ordner. Benennen Sie den Ordner Ihres Projekts.
1. Öffnen Sie das Terminal und führen Sie den folgenden Befehl aus, um Ihr neues MVC-Projekt zu erstellen.
    
    ```dos
        dotnet new mvc
    ```
    Dadurch wird ein neues ASP.NET MVC-Projekt in dem von Ihnen ausgewählten Ordner erstellt und in Visual Studio Code geladen.

1. Führen Sie den folgenden Befehl aus, um Ihre Anwendung auszuführen. 

    ```dos
    dotnet run
    ```
1. Die Terminalausgaben *Jetzt zuhören: http://localhost:<port>*. Rufen Sie die URL in Ihrem Webbrowser auf, um auf die Anwendung zuzugreifen. 

1. Schließen Sie den Webbrowser und beenden Sie die Anwendung. Alternativ können Sie die Anwendung durch Drücken von `Ctrl+C` im VS Code-Terminal stoppen.

### Aktualisieren Sie Ihr Projekt, um eine Verbindung zur Azure SQL-Datenbank herzustellen

Als Nächstes aktualisieren Sie einige Konfigurationen, die es Ihnen ermöglichen, sich mithilfe einer verwalteten Identität erfolgreich mit der Azure SQL-Datenbank zu verbinden.

1. Fügen Sie in Ihrem Projekt die erforderlichen NuGet-Pakete für SQL Server hinzu.
    ```dos
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    ```
1. Öffnen Sie im Stammordner Ihres Projekts die Datei **appsettings.json** und fügen Sie den `ConnectionStrings`-Abschnitt ein. Hier ersetzen Sie `<server-name>` und `<db-name>` durch die tatsächlichen Namen Ihres Servers und Ihrer Datenbank. Diese Verbindungszeichenfolge wird vom Standardkonstruktor in der `Models/MyDbContext.cs` Datei verwendet, um eine Verbindung zu Ihrer Datenbank herzustellen.

    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "AllowedHosts": "*",
      "ConnectionStrings": {
        "DefaultConnection": "Server=<server-name>.database.windows.net,1433;Initial Catalog=<db-name>;Authentication=Active Directory Default;"
      }
    }
    ```
1. Speichern und schließen Sie die Datei.

### Hinzufügen ihres Codes

1. Erstellen Sie im Ordner **Models** Ihres Projekts eine **Product.cs**-Datei für Ihre Produktentität mit dem folgenden Code. Ersetzen Sie `<app name>` durch den tatsächlichen Namen Ihrer Anwendung.

    ```csharp
    namespace <app name>.Models;
    
    public class Product
    {
        public int ProductId { get; set; }
        public string ProductName { get; set; }
        public string Category { get; set; }
        public decimal Price { get; set; }
        public int Stock { get; set; }
    }
    ```
1. Erstellen Sie den Ordner **Datenbank** im Stammordner Ihres Projekts.
1. Erstellen Sie im Ordner **Database** Ihres Projekts eine Datei **MyDbContext.cs** für Ihre Produktentität mit dem folgenden Code. Ersetzen Sie `<app name>` durch den tatsächlichen Namen Ihrer Anwendung.

    ```csharp
    using <app name>.Models;
    
    namespace <app name>.Database;
    
    using Microsoft.EntityFrameworkCore;
    
    public class MyDbContext : DbContext
    {
        public MyDbContext(DbContextOptions<MyDbContext> options) : base(options)
        {
        }
    
        public DbSet<Product> Products { get; set; }
    }    
    ```
1. Bearbeiten Sie im Ordner **Controller** Ihres Projekts die Klassen `HomeController` und `IActionResult` für die Datei **HomeController.cs** und fügen Sie die `_context` Variable mit dem folgenden Code hinzu.

    ```csharp
    private MyDbContext _context;

    public HomeController(ILogger<HomeController> logger, MyDbContext context)
    {
        _logger = logger;
        _context = context;
    }

    public IActionResult Index()
    {
        var data = _context.Products.ToList();
        return View(data);
    }
    ```
1. Aktualisieren Sie im Ordner **Ansichten -> Startseite** Ihres Projekts die Datei **Index.cshtml** und fügen Sie den folgenden Code hinzu.

    ```html
    <table class="table">
        <thead>
            <tr>
                <th>Product Id</th>
                <th>Product Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Stock</th>
            </tr>
        </thead>
        <tbody>
            @foreach(var item in Model)
            {
                <tr>
                    <td>@item.ProductId</td>
                    <td>@item.ProductName</td>
                    <td>@item.Category</td>
                    <td>@item.Price</td>
                    <td>@item.Stock</td>
                </tr>
            }
        </tbody>
    </table>
    ```

1. Bearbeiten Sie die Datei **Program.cs** und fügen Sie den bereitgestellten Codeausschnitt direkt über der `var app = builder.Build();` Zeile ein. Diese Änderung stellt sicher, dass der Code während der Startsequenz der Anwendung ausgeführt wird. Ersetzen Sie `<app name>` durch den tatsächlichen Namen Ihrer Anwendung.

    ```csharp
    using Microsoft.EntityFrameworkCore;
    using <app name>.Database;

    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddControllersWithViews();
    builder.Services.AddDbContext<MyDbContext>(options =>
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    ```

    > **Hinweis:** Wenn Sie Ihre Anwendung vor dem Bereitstellen ausführen möchten, aktualisieren Sie die Verbindungszeichenfolge mit den SQL-Benutzeranmeldeinformationen. Benutzername und Passwort der neuen Datenbank wurden automatisch generiert. Um diese Werte nach dem Bereitstellen abzurufen, gehen Sie zu den **Verbindungszeichenfolgen** auf der Seite **Umgebungsvariablen** Ihrer App. Sobald Sie bestätigt haben, dass die Anwendung wie erwartet ausgeführt wird, wechseln Sie zurück zur Verwendung der verwalteten Identität für einen sicheren Bereitstellungsprozess.

### Bereitstellen Ihres Codes

1. Öffnen Sie die **Befehlspalette**, indem Sie `Ctrl+Shift+P` drücken.
1. Geben Sie **Azure App Service ein: In Web-App bereitstellen …** und wählen Sie es aus.
1. Wählen Sie den Ordner aus, der Ihren Webanwendungscode enthält.
1. Wählen Sie die Web-App aus, die Sie im vorherigen Schritt erstellt haben.
    > Hinweis: Möglicherweise erhalten Sie die Meldung: „Die für die Bereitstellung erforderliche Konfiguration fehlt in Ihrer App“. Wählen Sie **Konfiguration hinzufügen** aus. Folgen Sie dann den Anweisungen und wählen Sie Ihr Abonnement und die App-Service-Ressource aus.
1. Bestätigen Sie die Bereitstellung, wenn Sie dazu aufgefordert werden.

## Testen Ihrer Anwendung

Führen Sie Ihre Webanwendung aus und überprüfen Sie, ob sie ohne gespeicherte Anmeldedaten eine Verbindung zur Azure SQL-Datenbank herstellen kann.

1. Öffnen Sie einen Browser und navigieren Sie zur URL Ihrer Azure-Webanwendung (z. B. https://your-web-app-name.azurewebsites.net)).
1. Überprüfen Sie, ob Ihre Webanwendung ausgeführt wird und zugänglich ist.
1. Sie sollten eine Webseite wie die folgende sehen.

    ![Ein Screenshot der Webanwendung nach der Bereitstellung.](./Media/01-app-page.png)

## Richten Sie eine kontinuierliche Bereitstellung ein (optional)

1. Öffnen Sie die **Befehlspalette**, indem Sie `Ctrl+Shift+P` drücken.
1. Geben Sie **Azure App Service: Continuous Delivery konfigurieren …** ein und wählen Sie es aus.
1. Folgen Sie den Anweisungen, um die kontinuierliche Bereitstellung von Ihrem GitHub-Repository oder Azure DevOps einzurichten.

Überlegen Sie, in welchen Szenarien es vorteilhaft wäre, **benutzerseitig zugewiesene verwaltete Identität** anstelle von **systemseitig zugewiesene verwalteter Identität** zu verwenden.

## Bereinigen

Wenn Sie in Ihrem eigenen Abonnement arbeiten, sollten Sie sich am Ende eines Projekts überlegen, ob Sie die erstellten Ressourcen noch benötigen. 

Wenn Ressourcen unnötig ausgeführt werden, kann dies zu zusätzlichen Kosten führen. Sie können Ressourcen einzeln oder die gesamte Gruppe von Ressourcen im [Azure-Portal](https://portal.azure.com?azure-portal=true) löschen.

## Weitere Informationen

Weitere Informationen zu Auto-Failover-Gruppen für Azure SQL-Datenbank finden Sie unter [Verwaltete Identitäten in Microsoft Entra für Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).
