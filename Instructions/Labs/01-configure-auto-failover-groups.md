---
lab:
  title: Habilitar resiliência de aplicativos com grupos de failover automático para o Banco de Dados SQL do Azure
  module: Get started with Azure SQL Database for cloud-native application development
---

# Habilitar resiliência de aplicativos com grupos de failover automático para o Banco de Dados SQL do Azure

Neste exercício, você criará dois bancos de dados SQL do Azure que atuarão como primário e secundário. Você configurará grupos de failover automático para garantir alta disponibilidade e recuperação de desastres dos bancos de dados do seu aplicativo e validará o status de replicação no seu aplicativo.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

Antes de começar este exercício, você precisará de:

- Uma assinatura do Azure com permissões apropriadas para criar e gerenciar recursos.
- [**Visual Studio Code**](https://code.visualstudio.com/download?azure-portal=true) instalado no seu computador com a seguinte extensão instalada:
    - [Kit de Desenvolvimento do C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit?azure-portal=true).

## Crie servidores SQL do Azure primários e secundários

Primeiro, configuraremos os servidores primário e secundário e usaremos o banco de dados de exemplo **AdventureWorksLT**.

1. Entre no [portal do Azure](https://portal.azure.com?azure-portal=true).

1. Selecione o ícone do Cloud Shell no canto superior direito do portal do Azure. Parece um símbolo `>_`. Se solicitado, escolha **Bash** como o tipo de shell.

1. Execute os seguintes comandos no terminal do Cloud Shell. Substitua os valores `<your_resource_group>`, `<your_primary_server>`, `<your_location>`, `<your_secondary_server>` e `<your_admin_password>` pelos seus valores reais:

    * Criar um grupo de recursos
    ```azurecli
    az group create --name <your_resource_group> --location <your_location>
    ```

    * Criar o servidor SQL primário
    ```azurecli        
    az sql server create --name <your_primary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>
    ```

    * Criar o servidor SQL secundário. Mesmo script, apenas altere o nome e a localização do servidor
    ```azurecli
    az sql server create --name <your_secondary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>    
    ```

    * Criar um banco de dados de amostra no servidor primário com o tipo de preço especificado
    ```azurecli
    az sql db create --resource-group <your_resource_group> --server <your_primary_server> --name AdventureWorksLT --sample-name AdventureWorksLT --service-objective "S0"    
    ```
    
1. Após a conclusão das implantações, navegue até o servidor SQL do Azure principal que você criou.
1. Selecione **Rede** em **Segurança** no painel esquerdo. Adicione seu endereço IP às regras de firewall.
1. Selecione a opção **Permitir que serviços e recursos do Azure acessem este servidor**.
1. Selecione **Salvar**.
1. Repita os passos acima para o servidor secundário.

    Essas etapas garantem que você tenha um ambiente de Banco de Dados SQL do Azure estruturado e redundante pronto para uso.

## Configurar grupos de failover automático

Em seguida, você criará um grupo de failover automático para o Banco de Dados SQL do Azure configurado anteriormente. Isso envolve estabelecer um grupo de failover entre dois servidores e verificar a configuração para garantir que esteja funcionando corretamente.

1. Execute os seguintes comandos no terminal do Cloud Shell. Substitua os valores `<your_failover_group>`, `<your_resource_group>`, `<your_primary_server>` e `<your_secondary_server>` pelos seus valores reais:

    * Criar o grupo de failover
    ```azurecli
    az sql failover-group create -n <your_failover_group> -g <your_resource_group> -s <your_primary_server> --partner-server <your_secondary_server> --failover-policy Automatic --grace-period 1 --add-db AdventureWorksLT
    ```

    * Verifique o grupo de failover
    ```azurecli    
    az sql failover-group show -n <your_failover_group> -g <your_resource_group> -s <your_primary_server>
    ```

    > Reserve um momento para revisar os resultados e os valores `partnerServers`. Por que isso é importante?

    > Ao verificar o atributo `role` em cada servidor parceiro, você pode determinar se um servidor está atualmente atuando como primário ou secundário. Essas informações são cruciais para entender a configuração atual e a prontidão do grupo de failover. Ela ajuda você a avaliar o impacto potencial em seu aplicativo durante cenários de failover e garante que sua configuração esteja corretamente configurada para alta disponibilidade e recuperação de desastres.
    
## Integrar com o código do aplicativo

Para conectar seu aplicativo .NET ao ponto de extremidade do Banco de Dados SQL do Azure, você precisará seguir estas etapas.

1. No Visual Studio Code, abra o terminal e execute os seguintes comandos para instalar o pacote `Microsoft.Data.SqlClient` e criar um novo aplicativo de console .NET.

    ```bash
    dotnet new console -n AdventureWorksLTApp
    cd AdventureWorksLTApp 
    dotnet add package Microsoft.Data.SqlClient --version 5.2.1
    ```

1. Abra a pasta `AdventureWorksLTApp` que foi criada na etapa anterior no **Visual Studio Code**.

1. Crie um arquivo `appsettings.json` na raiz do diretório do seu projeto. Este arquivo de configuração armazenará sua cadeia de conexão com o banco de dados. Certifique-se de substituir os valores `<your_failover_group>` e `<your_password>` na cadeia de conexão pelos seus detalhes reais.

    ```json
    {
      "ConnectionStrings": {
        "FailoverGroupConnection": "Server=tcp:<your_failover_group>.database.windows.net,1433;Initial Catalog=AdventureWorksLT;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
      }
    }
    ```

1. Abra o arquivo `.csproj` no **Visual Studio Code** e adicione o seguinte conteúdo logo abaixo do rótulo `</PropertyGroup>`.

    ```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.Data.SqlClient" Version="5.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="6.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
    </ItemGroup>
    ```

    Seu arquivo `.csproj` completo deve ser semelhante a este.

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

1. Abra o arquivo `Program.cs` em **Visual Studio Code**. No editor, substitua todo o código existente pelo código fornecido abaixo.

    > **Observação:** reserve um momento para revisar o código e observar como ele imprime informações sobre os servidores primário e secundário no grupo de failover automático.

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

1. Execute o código selecionando **Executar** > **Iniciar Depuração** no menu ou simplesmente pressione **F5**. Você também pode selecionar o botão de reprodução na barra de ferramentas superior para iniciar o aplicativo.

    > **Importante:** se você receber uma mensagem *"Você não tem uma extensão para depurar C#. Devemos encontrar uma extensão C# no Marketplace?" *, certifique-se de que a extensão **Kir de Desenvolvimento C#** esteja instalada.

1. Depois de executar o código, você deve ver a saída na guia **Console de Depuração** no Visual Studio Code.

    ```
    Primary Server: <your_server_name>
    Secondary Server: <your_server_name>
    Database: AdventureWorksLT
    Replication State: CATCH_UP
    ```
    
    O estado de replicação `CATCH_UP` significa que o banco de dados está totalmente sincronizado com seu parceiro e está pronto para failover. O monitoramento do estado de replicação pode ajudar a identificar gargalos de desempenho e garantir que a replicação de dados esteja ocorrendo com eficiência.

## Faz o failover na região secundária

Imagine um cenário em que o Banco de Dados SQL do Azure primário esteja enfrentando problemas devido a uma interrupção regional. Para manter a continuidade do serviço e minimizar o tempo de inatividade, você precisa mudar seu aplicativo para a réplica secundária realizando um failover forçado.

Durante um failover forçado, todas as novas sessões TDS são roteadas automaticamente para o servidor secundário, que se torna o servidor primário. A melhor parte é que você não precisa alterar a cadeia de conexão do aplicativo, pois o ponto final permanece o mesmo.

Vamos iniciar um failover e executar nosso aplicativo para verificar o status de nossos servidores primários e secundários.

1. Volte para o portal do Azure e abra uma nova instância do terminal Cloud Shell. Execute o código a seguir. Substitua os valores `<your_failover_group>`, `<your_resource_group>` e `<your_primary_server>` pelos seus valores reais. O valor do parâmetro `--server` deve ser o secundário atual.

    ```azurecli
    az sql failover-group set-primary --name <your_failover_group> --resource-group <your_resource_group> --server <your_server_name>
    ```

    > **Observação**: essa operação poderá levar alguns minutos.

1. Quando o failover estiver concluído, execute o aplicativo novamente para verificar o status da replicação. Você deve ver que o servidor secundário agora assumiu como o primário, e o servidor primário original se tornou o secundário.

Considere por que você pode querer colocar seus bancos de dados de aplicativos primários e secundários na mesma região e quando pode ser benéfico escolher regiões diferentes.

## Limpar

Quando você está trabalhando em sua própria assinatura, é uma boa ideia identificar, no final de um projeto, se você ainda precisa dos recursos criados. 

Deixar os recursos funcionando desnecessariamente pode resultar em custos extras. É possível excluir os recursos individualmente ou excluir todo o conjunto de recursos no [portal do Azure](https://portal.azure.com?azure-portal=true).

## Mais informações

Para obter mais informações sobre grupos de failover automático para o Banco de dados SQL do Azure, consulte [Visão geral dos grupos de failover e práticas recomendadas (Banco de dados SQL do Azure)](https://learn.microsoft.com/azure/azure-sql/database/failover-group-sql-db?azure-portal=true).
