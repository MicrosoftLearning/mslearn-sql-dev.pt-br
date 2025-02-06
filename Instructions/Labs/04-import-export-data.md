---
lab:
  title: Importar e exportar dados para desenvolvimento no Banco de Dados SQL do Azure
  module: Import and export data for development in Azure SQL Database
---

# Importar e exportar dados para desenvolvimento no Banco de Dados SQL do Azure

Neste exercício, você importará dados de um ponto de extremidade REST externo (simulado usando o Azure Static Web App) e exportará dados usando uma função do Azure. O laboratório fornecerá experiência prática no trabalho com o Banco de Dados SQL do Azure para fins de desenvolvimento, com foco na integração de APIs REST e Azure Functions para lidar com operações de importação/exportação de dados.

## Pré-requisitos

Antes de iniciar este laboratório, certifique-se de ter o seguinte:

- Uma assinatura do Azure ativa com permissão para criar e gerenciar recursos.
- Conhecimento básico do Banco de Dados SQL do Azure, APIs REST e Azure Functions.
- Visual Studio Code instalado com as seguintes extensões:
      - Extensão Azure Functions.
- Git instalado para clonar o repositório.
- SSMS (SQL Server Management Studio) ou Azure Data Studio para gerenciar o banco de dados.

## Configurar o ambiente

Vamos começar configurando os recursos necessários para este laboratório, incluindo um Banco de Dados SQL do Azure e as ferramentas necessárias para importar e exportar dados.

### Criar um Banco de Dados SQL do Azure

Esta etapa requer que você crie um banco de dados no Azure:

1. No portal do Azure, acesse a página **Bancos de dados SQL**.
1. Selecione **Criar**.
1. Preencha os campos requiridos.

    | Configuração | Valor |
    |---|---|
    | Oferta gratuita sem servidor | Aplicar oferta |
    | Assinatura | Sua assinatura |
    | Grupo de recursos | Selecionar ou criar um grupo de recursos |
    | Nome do banco de dados | **MyDB** |
    | Servidor | Selecione ou crie um novo servidor |
    | Método de autenticação | Autenticação do SQL |
    | Logon de administrador do servidor | **sqladmin** |
    | Senha | Digite uma senha segura |
    | Confirmar senha | Confirme a senha |

1. Selecione **Examinar + Criar** e **Criar**.
1. Após a conclusão da implantação, navegue até a seção **Rede** do seu ***Servidor SQL do Azure*** (não o Banco de Dados SQL do Azure) e:
    1. Adicione seu endereço IP às regras de firewall. Isso permitirá que você use o SSMS (SQL Server Management Studio) ou o Azure Data Studio para gerenciar o banco de dados.
    1. Marque a caixa de seleção **Permitir que os serviços e os recursos do Azure acessem este servidor**. Isso permitirá que o aplicativo do Azure Functions acesse o servidor de banco de dados.
    1. Salve suas alterações.
1. Navegue até a seção **Microsoft Entra ID** do seu **Servidor SQL do Azure** e certifique-se de *desmarcar* **Suportar somente autenticação Microsoft Entra para este servidor** e **Salvar** suas alterações, se selecionado. Este exemplo usa a autenticação SQL, portanto, precisamos desabilitar o suporte somente ao Entra.

> [!NOTE]
> Em um ambiente de produção, você precisará determinar que tipo de acesso e de onde deseja conceder acesso. Embora a função tenha uma pequena alteração se você escolher somente a autenticação Entra, observe que você ainda precisará habilitar *Permitir que serviços e recursos do Azure acessem este servidor* para permitir que o aplicativo de funções do Azure acesse o servidor.

### Clonar o repositório GitHub

1. Abra o **Visual Studio Code**.

1. Clone o repositório GitHub e prepare seu projeto:

    1. No **Visual Studio Code**, abra a **Paleta de Comandos** pressionando **Ctrl+Shift+P** (Windows) ou **Cmd+Shift+P** (Mac).
    1. Digite **Git: Clone** e selecione **Git: Clone**.
    1. No prompt, insira a seguinte URL para clonar o repositório:
        ```bash
        https://github.com/MicrosoftLearning/mslearn-sql-dev.git
        ```

    1. Escolha a pasta de destino onde você gostaria de clonar o repositório.

### Configurar o Armazenamento de Blobs do Azure para dados JSON

Agora configuraremos o **Armazenamento de Blobs do Azure** para hospedar o arquivo **employees.json**. Siga estas etapas no portal do Azure e **Visual Studio Code**.

Vamos começar criando uma conta de armazenamento do Azure.

1. No **portal do Azure**, acesse a página **Contas de armazenamento**.
1. Selecione **Criar**.
1. Preencha os campos requiridos.

    | Configuração | Valor |
    |---|---|
    | Subscription | Sua assinatura |
    | Grupo de recursos | Selecionar ou criar um grupo de recursos |
    | Nome da conta de armazenamento | Escolha um nome globalmente exclusivo |
    | Region | Selecione a região mais próxima de você |
    | Serviço principal | **Armazenamento de Blobs do Azure ou Azure Data Lake Storage Gen2** |
    | Desempenho | Standard |
    | Redundância | Armazenamento com redundância local (LRS) |

1. Selecione **Examinar + Criar** e **Criar**.
1. Aguarde até a conta de armazenamento ser criado.

Agora que temos uma conta, vamos carregar **employees.json** para o Armazenamento de Blobs.

1. Acesse a página **Contas de armazenamento** no portal do Azure.
1. Selecione sua conta de armazenamento.
1. Navegue até a seção **Contêineres**.
1. Crie um novo contêiner chamado **jsonfiles**.
1. Dentro do contêiner, clique em **Upload** e carregue o arquivo **employees.json** localizado em **/Allfiles/Labs/04/blob-storage** no diretório clonado.

Embora possamos permitir o acesso anônimo ao arquivo, em nosso caso, vamos gerar uma *SAS (Assinatura de Acesso Compartilhado)* para esse arquivo para garantir o acesso seguro.

1. No contêiner **jsonfiles**, selecione o arquivo **employees.json**.
1. Selecione **Gerar SAS** no menu de contexto do arquivo.
1. Examine as configurações e selecione **Gerar SAS e URL**.
1. Um token SAS de Blob e uma URL SAS de Blob serão gerados. Copie o **token SAS de Blob** e a **URL SAS de Blob** para uso nas próximas etapas. Você não poderá acessar o valor do token novamente depois de fechar essa janela.

Agora devemos ter uma URL segura para acessar o arquivo **employees.json**, vamos testá-lo.

1. Abra uma nova aba do navegador e cole a **URL SAS do Blob**.
1. Você deverá ver o conteúdo do arquivo **employees.json** exibido no navegador, que deve ficar assim:

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

### Importar dados do armazenamento de blobs para o Banco de Dados SQL do Azure

Agora estamos prontos para importar os dados do arquivo **employees.json** hospedado no Armazenamento de Blobs do Azure para nosso Banco de Dados SQL do Azure.

Precisamos começar criando uma **Chave Mestra** e uma **Credencial com Escopo de Banco de Dados** no Banco de Dados SQL do Azure.

1. Conecte-se ao seu Banco de Dados SQL do Azure usando o SSMS **(SQL Server Management Studio)** ou o **Azure Data Studio**.
1. Execute o seguinte comando SQL para criar uma chave mestra *se você ainda não tiver uma*:

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourStrongPassword!';
    ```

1. Em seguida, crie uma **Credencial com Escopo de Banco de Dados** para acessar o Armazenamento de Blobs do Azure executando o seguinte comando SQL:

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL MyBlobCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', 
    SECRET = '<your-sas-token>';
    ```

    Substitua ***<your-sas-token>*** pelo **token SAS de Blob** gerado anteriormente.

1. Por fim, você precisa de uma **Fonte de Dados** para acessar o Armazenamento de Blobs do Azure. Execute o seguinte comando SQL para criar uma **Fonte de Dados**:

    ```sql
    CREATE EXTERNAL DATA SOURCE MyBlobDataSource
    WITH (
        TYPE = BLOB_STORAGE,
        LOCATION = 'https://<your-storage-account-name>.blob.core.windows.net',
        CREDENTIAL = MyBlobCredential
    );
    ```

    Substitua ***<your-storage-account-name>*** pelo nome da sua Conta de Armazenamento do Azure.

Agora tudo está configurado para importar os dados do arquivo **employees.json** para o *Banco de Dados SQL do Azure*.

Use o seguinte comando SQL para importar dados do arquivo **employees.json** hospedado no *Armazenamento de Blobs do Azure*:

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

Este comando lê o arquivo **employees.json** do contêiner **jsonfiles** no *Armazenamento de Blobs do Azure* e importa os dados para a tabela **employee_data** no Banco de Dados SQL do Azure.

Agora você pode executar o seguinte comando SQL para verificar a importação de dados: 

```sql
SELECT * FROM dbo.employee_data;
```

Você deverá ver os dados do arquivo **employees.json** importados para a tabela **employee_data**.

---

## Exportar dados usando um aplicativo do Azure Functions

Nesta parte do laboratório, você criará um aplicativo do Azure Functions em C# para exportar dados do seu banco de dados SQL do Azure. Essa função recuperará os dados e os retornará como uma resposta JSON.

### Criar um aplicativo do Azure Functions no Visual Studio Code

Vamos começar criando um aplicativo do Azure Functions no Visual Studio Code:

1. Abra o **Visual Studio Code**.
1. No painel do explorador, navegue até a pasta **/Allfiles/Labs/04/azure-functions**.
1. Clique com o botão direito do mouse na pasta **azure-functions** e selecione **Abrir no Terminal Integrado**.
1. No terminal do VS Code, faça login no Azure usando o seguinte comando:

    ```bash
    az login
    ```

1. (Opcional) Se você tiver várias assinaturas, defina a assinatura ativa:

    ```bash
    az account set --subscription <your-subscription-id>
    ```

1. Execute o comando a seguir para criar um aplicativo do Azure Functions:

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

    ***Substitua os espaços reservados pelos seus próprios valores. Não use o nome da conta de armazenamento usada para o arquivo json, este script precisa criar uma nova conta de armazenamento para armazenar o aplicativo do Azure Functions***.


### Criar um novo aplicativo de função no Visual Studio Code

Vamos criar uma nova função no Visual Studio Code para exportar dados do Banco de Dados SQL do Azure:

Talvez seja necessário adicionar a extensão do Azure Functions ao Visual Studio Code, caso ainda não o tenha feito. Você pode fazer isso pesquisando por **Azure Functions** no painel de extensões e instalando-o.

1. No Visual Studio Code, pressione **Ctrl+Shift+P** (Windows) ou **Cmd+Shift+P** (Mac) para abrir a paleta de comandos.
1. Digite e selecione **Azure Functions: Criar Projeto**.
1. Escolha seu diretório **Aplicativo de funções**. Selecione a pasta **/Allfiles/Labs/04/azure-functions** do repositório clone do GitHub.
1. Escolha **C#** como a linguagem.
1. Escolha **.Net 8.0 LTS** como runtime.
1. Escolha **Gatilho HTTP** como modelo.
1. Chame a função **ExportDataFunction**.
1. Crie o namespace **Contoso.ExportFunction**.
1. Dê à função o nível de acesso **anônimo**.

### Escreva o código C# para exportar dados

1. O aplicativo do Azure Functions pode precisar que alguns pacotes sejam instalados primeiro. Você pode instalá-los executando os seguintes comandos:

    ```bash
    dotnet add package Microsoft.Data.SqlClient
    dotnet add package Newtonsoft.Json
    dotnet restore
    
    npm install -g azure-functions-core-tools@4 --unsafe-perm true

    ```

1. Substitua o código da função de espaço reservado pelo seguinte código C# para consultar seu Banco de Dados SQL do Azure e retornar os resultados como JSON:

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

    *Lembre-se de substituir **connectionString** pela cadeia de conexão com seu Banco de Dados SQL do Azure e insira sua senha sqladmin na cadeia de conexão também.*

    > **Observação:** em um ambiente de produção, restrinja o acesso apenas aos endereços IP necessários. Além disso, considere usar Identidades Gerenciadas para seu Aplicativo do Azure Functions para acessar o banco de dados em vez da autenticação SQL. Para obter mais informações, consulte [Identidades gerenciadas no Microsoft Entra para SQL do Azure](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Salve o código da função e certifique-se de que seu arquivo **.csproj** inclua o pacote **Newtonsoft.Json** para serializar objetos para JSON. Se não estiver incluído, adicione-o:

    ```xml
    <PackageReference Include="Newtonsoft.Json" Version="13.X.X" />
    ```

É hora de implantar o aplicativo do Azure Function no Azure.

### Implantar o aplicativo do Azure Functions no Azure

1. No terminal integrado do **Visual Studio Code**, execute o seguinte comando para implantar o Azure Functions no Azure:

    ```bash
    func azure functionapp publish <your-function-app-name>
    ```

    Substitua ***<your-function-app-name>*** pelo nome do seu aplicativo do Azure Function.

1. Aguarde até que a implantação seja concluída.

### Obtenha a URL do aplicativo do Azure Function

1. Abra o portal do Azure e navegue até seu aplicativo do Azure Functions.
1. Na seção *Visão geral*, na guia *Função*, você verá sua nova função listada, selecione-a.
1. Na guia **Código + Teste**, selecione **Obter URL da função**.
1. Copie o **padrão (chave de função)**, precisaremos dele em breve. A URL deve ser mais ou menos assim:
   
   ```url
   https://YourFunctionAppName.azurewebsites.net/api/ExportDataFunction?code=2pjO0HqRyz_13DHQg8ga-ysdDWbDU_eHdtlixbAHLVEGAzFuplomUg%3D%3D
   ```

### Testar o aplicativo do Azire Functions

1. Assim que a implantação for concluída, você poderá testar a função enviando uma solicitação HTTP para a URL da chave de função que você copiou anteriormente do terminal Visual Studio Code:

    ```bash
    curl https://<your-function-app-name>.azurewebsites.net/api/ExportDataFunction?code=<the function key embedded to your function URL>
    ```

1. A resposta deve conter os dados exportados da sua tabela ***employee_data*** no formato JSON.

Embora essa função seja um exemplo simples, você pode estendê-la para incluir lógica mais complexa e processamento de dados, como filtragem, classificação e agregação de dados, e muito mais.  Seu código também pode ser estendido para incluir recursos de tratamento de erros, registro e segurança.

### Limpar os recursos

Depois de concluir o laboratório, você pode excluir os recursos criados neste exercício para evitar incorrer em custos adicionais:

- Exclua o Banco de Dados SQL do Azure.
- Exclua a conta do Armazenamento do Azure.
- Exclua o aplicativo do Azure Functions
- Exclua o grupo de recursos que contém os recursos.
