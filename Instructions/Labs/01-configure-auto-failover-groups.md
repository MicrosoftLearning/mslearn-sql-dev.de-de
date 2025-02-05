---
lab:
  title: Ermöglichen von Anwendungsstabilität mit automatischen Failover-Gruppen für Azure SQL-Datenbank
  module: Get started with Azure SQL Database for cloud-native application development
---

# Ermöglichen von Anwendungsstabilität mit automatischen Failover-Gruppen für Azure SQL-Datenbank

In dieser Übung erstellen Sie zwei Azure SQL-Datenbanken, die als primäre und sekundäre Datenbank fungieren. Sie konfigurieren Autofailover-Gruppen, um die Hochverfügbarkeit und Notfallwiederherstellung Ihrer Anwendungsdatenbanken sicherzustellen, und überprüfen den Replikationsstatus Ihrer Anwendung.

Diese Übung dauert ca. **30** Minuten.

## Vor der Installation

Bevor Sie mit dieser Übung beginnen können, benötigen Sie:

- Ein Azure-Abonnement mit entsprechenden Berechtigungen zum Erstellen und Verwalten von Ressourcen
- [**Visual Studio Code**](https://code.visualstudio.com/download?azure-portal=true) mit den folgenden drei Erweiterungen auf Ihrem Computer installiert:
    - [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit?azure-portal=true).

## Erstellen von primären und sekundären Azure SQL-Servern

Zuerst richten wir sowohl den primären als auch den sekundären Server ein und verwenden die **AdventureWorksLT**-Beispieldatenbank.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com?azure-portal=true) an.

1. Wählen Sie das Cloud Shell-Symbol in der oberen rechten Ecke des Azure-Portals aus. Es sieht wie ein `>_`-Symbol aus. Wenn Sie dazu aufgefordert werden, wählen Sie **Bash** als Shelltyp aus.

1. Führen Sie die folgenden Befehle im Cloud Shell-Terminal aus. Ersetzen Sie die Werte `<your_resource_group>`, `<your_primary_server>`, `<your_location>`, `<your_secondary_server>` und `<your_admin_password>` durch Ihre tatsächlichen Werte:

    * Erstellen einer Ressourcengruppe
    ```azurecli
    az group create --name <your_resource_group> --location <your_location>
    ```

    * Erstellen des primären SQL-Servers
    ```azurecli        
    az sql server create --name <your_primary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>
    ```

    * Erstellen des sekundären SQL-Servers Gleiches Skript, nur Servername und Standort ändern
    ```azurecli
    az sql server create --name <your_secondary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>    
    ```

    * Erstellen Sie eine Beispieldatenbank auf dem Primärserver mit der angegebenen Preisstufe.
    ```azurecli
    az sql db create --resource-group <your_resource_group> --server <your_primary_server> --name AdventureWorksLT --sample-name AdventureWorksLT --service-objective "S0"    
    ```
    
1. Nachdem die Bereitstellungen abgeschlossen sind, navigieren Sie zu dem von Ihnen erstellten primären Azure SQL-Server.
1. Wählen Sie im linken Bereich unter **Sicherheit** die Option **Netzwerk** aus. Fügen Sie Ihre IP-Adresse zu den Firewallregeln hinzu.
1. Wählen Sie die Option **Azure-Diensten und -Ressourcen den Zugriff auf diesen Server gestatten** aus.
1. Wählen Sie **Speichern**.
1. Wiederholen Sie die oben genannten Schritte für den sekundären Server.

    Diese Schritte stellen sicher, dass Sie über eine strukturierte und redundante Azure SQL-Datenbankumgebung verfügen, die einsatzbereit ist.

## Konfigurieren von Autofailover-Gruppen

Als Nächstes erstellen Sie eine Autofailover-Gruppe für die zuvor eingerichtete Azure SQL-Datenbank. Dazu gehört die Einrichtung einer Failover-Gruppe zwischen zwei Servern und die Überprüfung der Einrichtung, um sicherzustellen, dass sie ordnungsgemäß funktioniert.

1. Führen Sie die folgenden Befehle im Cloud Shell-Terminal aus. Ersetzen Sie die Werte `<your_failover_group>`, `<your_resource_group>`, `<your_primary_server>` und `<your_secondary_server>` durch Ihre tatsächlichen Werte:

    * Erstellen der Failovergruppe
    ```azurecli
    az sql failover-group create -n <your_failover_group> -g <your_resource_group> -s <your_primary_server> --partner-server <your_secondary_server> --failover-policy Automatic --grace-period 1 --add-db AdventureWorksLT
    ```

    * Überprüfen Sie die Failover-Gruppe
    ```azurecli    
    az sql failover-group show -n <your_failover_group> -g <your_resource_group> -s <your_primary_server>
    ```

    > Nehmen Sie sich einen Moment Zeit, um die Ergebnisse und die `partnerServers`-Werte zu überprüfen. Warum ist dies wichtig?

    > Durch Überprüfung des `role` Attributs auf jedem Partnerserver können Sie feststellen, ob ein Server derzeit als primärer oder sekundärer Server fungiert. Diese Informationen sind für das Verständnis der aktuellen Konfiguration und Bereitschaft der Failover-Gruppe von entscheidender Bedeutung. Es hilft Ihnen, die potenziellen Auswirkungen auf Ihre Anwendung während Failover-Szenarien zu bewerten und stellt sicher, dass Ihre Einrichtung für Hochverfügbarkeit und Notfallwiederherstellung korrekt konfiguriert ist.
    
## Integrieren mit dem Anwendungscode

Um Ihre .NET-Anwendung mit dem Azure SQL-Datenbank-Endpunkt zu verbinden, müssen Sie diese Schritte befolgen.

1. Öffnen Sie in Visual Studio Code das Terminal, und führen Sie die folgenden Befehle aus, um das `Microsoft.Data.SqlClient`-Paket zu installieren und eine neue .NET-Konsolenanwendung zu erstellen.

    ```bash
    dotnet new console -n AdventureWorksLTApp
    cd AdventureWorksLTApp 
    dotnet add package Microsoft.Data.SqlClient --version 5.2.1
    ```

1. Öffnen Sie den `AdventureWorksLTApp`-Ordner, der im vorherigen Schritt in **Visual Studio Code** erstellt wurde.

1. Erstellen Sie eine `appsettings.json`-Datei im Stammverzeichnis Ihres Projektverzeichnisses. In dieser Konfigurationsdatei wird Ihre Datenbank-Verbindungszeichenfolge gespeichert. Ersetzen Sie die Werte `<your_failover_group>` und `<your_password>` in der Verbindungszeichenfolge unbedingt durch Ihre tatsächlichen Angaben.

    ```json
    {
      "ConnectionStrings": {
        "FailoverGroupConnection": "Server=tcp:<your_failover_group>.database.windows.net,1433;Initial Catalog=AdventureWorksLT;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
      }
    }
    ```

1. Öffnen Sie die `.csproj`-Datei in **Visual Studio Code**, und fügen Sie den folgenden Inhalt direkt unter dem `</PropertyGroup>`-Tag hinzu.

    ```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.Data.SqlClient" Version="5.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="6.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
    </ItemGroup>
    ```

    Ihre vollständige `.csproj`-Datei sollte in etwa so aussehen.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net8.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.Data.SqlClient" Version="5.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="6.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
      </ItemGroup>
    
    </Project>
    ```

1. Öffnen Sie die `Program.cs`-Datei in **Visual Studio Code**. Ersetzen Sie im Editor den gesamten vorhandenen Code durch den unten angegebenen Code.

    > **Hinweis:** Nehmen Sie sich einen Moment Zeit, um den Code zu überprüfen und zu beobachten, wie er Informationen über die primären und sekundären Server in der Autofailover-Gruppe ausgibt.

    ```csharp
    using System;
    using Microsoft.Data.SqlClient;
    using Microsoft.Extensions.Configuration;
    using System.IO;
    
    namespace AdventureWorksLTApp
    {
        class Program
        {
            static void Main(string[] args)
            {
                var configuration = new ConfigurationBuilder()
                    .SetBasePath(Directory.GetCurrentDirectory())
                    .AddJsonFile("appsettings.json")
                    .Build();
    
                string connectionString = configuration.GetConnectionString("FailoverGroupConnection");
    
                ExecuteQuery(connectionString);
            }
    
            static void ExecuteQuery(string connectionString)
            {
                using (SqlConnection connection = new SqlConnection(connectionString))
                {
                    try
                    {
                        connection.Open();
                        string query = @"
                            SELECT 
                                @@SERVERNAME AS [Primary_Server],
                                partner_server AS [Secondary_Server],
                                partner_database AS [Database],
                                replication_state_desc
                            FROM 
                                sys.dm_geo_replication_link_status";
    
                        using (SqlCommand command = new SqlCommand(query, connection))
                        {
                            using (SqlDataReader reader = command.ExecuteReader())
                            {
                                while (reader.Read())
                                {
                                    Console.WriteLine($"Primary Server: {reader["Primary_Server"]}");
                                    Console.WriteLine($"Secondary Server: {reader["Secondary_Server"]}");
                                    Console.WriteLine($"Database: {reader["Database"]}");
                                    Console.WriteLine($"Replication State: {reader["replication_state_desc"]}");
                                }
                            }
                        }
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"Error executing query: {ex.Message}");
                    }
                    finally
                    {
                        connection.Close();
                    }
                }
            }
        }
    }
    ```

1. Führen Sie den Code aus, indem Sie im Menü **Ausführen** > **Debugging starten** auswählen oder einfach **F5** drücken. Sie können auch auf die Wiedergabetaste in der oberen Symbolleiste klicken, um die Anwendung zu starten.

    > **Wichtig:** Wenn Sie die Meldung *Sie haben keine Erweiterung für das Debuggen von C#. Sollen wir eine C#-Erweiterung im Marketplace suchen?* erhalten, stellen Sie sicher, dass die Erweiterung **C# Dev Kit** installiert ist.

1. Nachdem Sie den Code ausgeführt haben, sollten Sie die Ausgabe im Register **Debug Console** in Visual Studio Code sehen.

    ```
    Primary Server: <your_server_name>
    Secondary Server: <your_server_name>
    Database: AdventureWorksLT
    Replication State: CATCH_UP
    ```
    
    Der Replikationsstatus `CATCH_UP` bedeutet, dass die Datenbank vollständig mit ihrem Partner synchronisiert und für den Failover bereit ist. Die Überwachung des Replikationsstatus kann dabei helfen, Leistungsengpässe zu erkennen und sicherzustellen, dass die Datenreplikation effizient abläuft.

## Ausführen von Failover auf eine sekundäre Region

Stellen Sie sich ein Szenario vor, in dem die primäre Azure SQL-Datenbank aufgrund eines regionalen Ausfalls Probleme hat. Um die Servicekontinuität aufrechtzuerhalten und Ausfallzeiten zu minimieren, müssen Sie Ihre Anwendung auf die sekundäre Replik umstellen, indem Sie ein erzwungenes Failover durchführen.

Bei einem erzwungenen Failover werden alle neuen TDS-Sitzungen automatisch auf den sekundären Server umgeleitet, der dann zum primären Server wird. Das Beste daran ist, dass Sie die Verbindungszeichenfolge der Anwendung nicht ändern müssen, da der Endpunkt gleich bleibt.

Lassen Sie uns ein Failover einleiten und unsere Anwendung ausführen, um den Status unserer primären und sekundären Server zu überprüfen.

1. Gehen Sie zurück zum Azure-Portal und öffnen Sie eine neue Instanz des Cloud Shell-Terminals. Führen Sie den folgenden Code aus. Ersetzen Sie die Werte `<your_failover_group>`, `<your_resource_group>` und `<your_primary_server>` durch Ihre tatsächlichen Werte. Der Wert des `--server`parameters sollte der aktuelle Sekundärwert sein.

    ```azurecli
    az sql failover-group set-primary --name <your_failover_group> --resource-group <your_resource_group> --server <your_server_name>
    ```

    > **Hinweis**: Dieser Vorgang kann einige Minuten in Anspruch nehmen.

1. Sobald das Failover abgeschlossen ist, führen Sie die Anwendung erneut aus, um den Replikationsstatus zu überprüfen. Sie sollten sehen, dass der sekundäre Server nun die Rolle des primären Servers übernommen hat und der ursprüngliche primäre Server zum sekundären Server geworden ist.

Überlegen Sie, warum Sie Ihre primären und sekundären Anwendungsdatenbanken in derselben Region platzieren sollten und wann es von Vorteil sein könnte, verschiedene Regionen zu wählen.

## Bereinigen

Wenn Sie in Ihrem eigenen Abonnement arbeiten, sollten Sie sich am Ende eines Projekts überlegen, ob Sie die erstellten Ressourcen noch benötigen. 

Wenn Ressourcen unnötig ausgeführt werden, kann dies zu zusätzlichen Kosten führen. Sie können Ressourcen einzeln oder die gesamte Gruppe von Ressourcen im [Azure-Portal](https://portal.azure.com?azure-portal=true) löschen.

## Weitere Informationen

Weitere Informationen zu Autofailover-Gruppen für Azure SQL-Datenbank finden Sie unter [Failovergruppen – Übersicht und bewährte Methoden (Azure SQL-Datenbank)](https://learn.microsoft.com/azure/azure-sql/database/failover-group-sql-db?azure-portal=true).
