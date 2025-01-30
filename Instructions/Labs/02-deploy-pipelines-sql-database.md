---
lab:
  title: Configurar e implantar pipelines de CI/CD para projetos do Banco de Dados SQL do Azure
  module: Develop for an Azure SQL Database
---

# Configurar e implantar pipelines de CI/CD para projetos do Banco de Dados SQL do Azure

Neste exercício, você criará, configurará e implantará pipelines de CI/CD para projetos do Banco de Dados SQL do Azure usando o Visual Studio Code e o GitHub Actions. Isso permite que você se familiarize com o processo de configuração de pipelines de CI/CD para projetos do Banco de Dados SQL do Azure.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

Antes de começar este exercício, você precisa:

- Uma assinatura do Azure com permissões apropriadas para criar e gerenciar recursos.
- [Visual Studio Code](https://code.visualstudio.com/download) instalado no seu computador com as seguintes extensões:
  - [Projetos do Banco de Dados SQL](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql).
  - [Solicitações de pull do GitHub](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github).
- Contas do GitHub.
- Conhecimento básico de pipelines do *GitHub Actions*.

## Criar um Banco de Dados SQL do Azure

Primeiro, você precisa criar um novo Banco de Dados SQL do Azure.

1. Entre no [portal do Azure](https://portal.azure.com?azure-portal=true). 
1. Navegue até a página do **SQL do Azure** e selecione **+ Criar**.
1. Selecione **Banco de Dados SQL**, *Banco de dados único* e o **botão Criar**.
1. Preencha as informações necessárias na caixa de diálogo **Criar banco de dados SQL** e selecione **OK**, deixando todas as outras opções em suas configurações padrão.

    | Configuração | Valor |
    | --- | --- |
    | Oferta gratuita sem servidor | *Aplicar oferta* |
    | Assinatura | Sua assinatura |
    | Grupo de recursos | *Selecione ou crie um novo grupo de recursos* |
    | Nome do banco de dados | *MyDB* |
    | Servidor | *Selecione o link **Criar novo*** |
    | Nome do servidor | *Escolha um nome exclusivo* |
    | Localidade | *Selecionar um local* |
    | Método de autenticação | *Use Autenticação SQL* |
    | Logon de administrador do servidor | *sqladmin* |
    | Senha | *Insira uma senha* |
    | Confirmar senha | *Confirme a senha* |

1. Selecione **Examinar + Criar** e depois **Criar**.
1. Após a conclusão da implantação, navegue até o banco de dados SQL do *servidor* do Azure que você criou.
1. Selecione **Rede** em **Segurança** no painel esquerdo. Adicione seu endereço IP às regras de firewall.
1. Selecione a opção **Permitir que serviços e recursos do Azure acessem este servidor**. Esta opção permite que o GitHub Actions acesse o banco de dados.

    > **Observação:** em um ambiente de produção, restrinja o acesso apenas aos endereços IP necessários. Além disso, considere usar Identidades Gerenciadas para seu GitHub Actions acessar o banco de dados em vez da autenticação SQL. Para obter mais informações, consulte [Identidades gerenciadas no Microsoft Entra para SQL do Azure](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Selecione **Salvar**.

## Configurar um repositório do GitHub

Em seguida, você precisa configurar um novo repositório GitHub.

1. Abra o site do [GitHub](https://github.com).
1. Entre em sua conta do GitHub.
1. Vá para **Repositórios** em sua conta e selecione **Novo**.
1. Selecione sua conta no **Proprietário**. Insira o nome **my-sql-db-repo**.
1. Defina o repositório como **Privado**.
1. Selecione **Criar repositório**.

### Instale as extensões do Visual Studio Code e clone o repositório

Antes de clonar o repositório, certifique-se de ter instalado as extensões necessárias do **Visual Studio Code**. Consulte a seção **Antes de começar** para obter orientação.

1. No Visual Studio Code, selecione **Exibir** > **Paleta de Comandos**.
1. Na paleta de comandos, digite `Git: Clone` e selecione-o.
1. Insira a URL do repositório que você criou na etapa anterior e selecione **Clonar**. Sua URL deve seguir este formato: *https://github.com/<your_account>/<your_repository>.git*
1. Selecione ou crie uma pasta para armazenar os arquivos do repositório.

## Criar e configurar um projeto do Banco de Dados SQL do Azure

Um projeto do Banco de Dados SQL do Azure no Visual Studio permite que você desenvolva, crie, teste e publique seu esquema e dados de banco de dados. Nesta seção, você criará um projeto e o configurará para se conectar ao Banco de Dados SQL do Azure que você configurou anteriormente.

1. No Visual Studio Code, selecione **Exibir** > **Paleta de Comandos**.
1. Na paleta de comandos, digite `Database projects: New` e selecione-o.
    > **Observação:** pode levar alguns minutos para instalar o serviço de ferramentas SQL para a extensão mssql.
1. Selecione **Banco de Dados SQL do Azure**.
1. Digite o nome **MyDBProj** e pressione **Enter** para confirmar.
1. Selecione a pasta do repositório GitHub clonado para salvar o projeto.
1. Para **Projetos no estilo SDK**, selecione **Sim (recomendado)**.
    > **Observação:** observe que um novo projeto é criado com o nome **MyDBProj**.

### Crie um novo arquivo SQL no projeto

Com o projeto do Banco de Dados SQL do Azure criado, vamos adicionar um novo arquivo SQL ao projeto para criar uma nova tabela.

1. No Visual Studio Code, selecione o ícone **Projetos de banco de dados** localizado na barra de atividades à esquerda.
1. Clique com o botão direito do mouse no nome do seu projeto e selecione **Adicionar tabela**.
1. Nomeie a tabela como **Funcionários** e pressione **Enter**.
1. Substitua o script existente pelo código a seguir.

    ```sql
    CREATE TABLE [dbo].[Employees]
    (
        EmployeeID INT PRIMARY KEY,
        FirstName NVARCHAR(50),
        LastName NVARCHAR(50),
        Department NVARCHAR(50)
    );
    ```

1. Feche o editor. Observe que o arquivo `Employees.sql` será salvo no projeto.

## Confirme a alteração no repositório

Com o projeto do Banco de Dados SQL do Azure criado e o script de tabela adicionado ao projeto, vamos confirmar as alterações no repositório.

1. No Visual Studio Code, selecione o ícone **Controle do código-fonte** localizado na barra de atividades à esquerda.
1. Digite a mensagem *Projeto criado e script de criação de tabela adicionado*.
1. Selecione **Confirmar** para confirmar a alteração.
1. Abaixo das reticências, selecione **Push** para enviar as alterações para o repositório.

## Verificar as alterações no repositório

Agora que você enviou as alterações, vamos verificá-las no repositório GitHub.

1. Abra o site do [GitHub](https://github.com).
1. Navegue até o repositório **my-sql-db-repo**.
1. Na aba **<> Code**, abra a pasta **MyDBProj**.
1. Verifique se as alterações no arquivo **Employees.sql** estão atualizadas.

## Configurar a CI (integração contínua) com o GitHub Actions

O GitHub Actions permite que você automatize, personalize e execute seus fluxos de trabalho de desenvolvimento de software diretamente no seu repositório GitHub. Nesta seção, você configurará um fluxo de trabalho do GitHub Actions para criar e testar seu projeto do Banco de Dados SQL do Azure criando uma nova tabela no banco de dados.

### Criar uma entidade de serviço

1. Selecione o ícone **Cloud Shell** no canto superior direito do portal do Azure. Parece um símbolo `>_`. Se solicitado, escolha **Bash** como o tipo de shell.

1. Execute o seguinte comando no terminal do Cloud Shell. Substitua os valores `<your_subscription_id>` e `<your_resource_group_name>` pelos seus valores reais. Você pode obter esses valores nas páginas **Assinaturas** e **Grupos de recursos** no portal do Azure.

    ```azurecli
    az ad sp create-for-rbac --name "MyDBProj" --role contributor --scopes /subscriptions/<your_subscription_id>/resourceGroups/<your_resource_group_name>
    ```

    Abra um editor de texto e use a saída do comando anterior para criar um snippet de credenciais semelhante a este:
    
    ```
    {
    "clientId": <your_service_principal_appId>,
    "clientSecret": <your_service_principal_password>,
    "tenantId": <your_service_principal_tenant>,
    "subscriptionId": <your_subscription_id>
    }
    ```

1. Deixe o editor de texto aberto. Faremos referência a isso na próxima seção.

### Adicionar segredos ao repositório

1. No repositório GitHub, selecione **Settings**.
1. Selecione **Secrets and variables** e depois **Actions**.
1. Na guia **Secrets**, selecione **New repository secret** e forneça as seguintes informações.

    | Nome | Valor |
    | --- | --- |
    | AZURE_CREDENTIALS | A saída da entidade de serviço copiada na seção anterior.|
    | AZURE_CONN_STRING | Sua cadeia de conexão. |
   
    Sua cadeia de conexão deve ser semelhante a esta:

    ```Server=<your_sqldb_server>.database.windows.net;Initial Catalog=MyDB;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;Encrypt=True;Connection Timeout=30;```

### Criar um fluxo de trabalho do GitHub Actions

1. No repositório GitHub, selecione a guia **Ações**.
1. Selecione o link **configurar um fluxo de trabalho você mesmo**.
1. Copie o código abaixo em seu **arquivo main.yml** . O código inclui as etapas para criar e implantar seu projeto de banco de dados.

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

      A etapa **Criar e implantar projeto SQL** no seu arquivo YAML se conecta ao seu Banco de Dados SQL do Azure usando a sequência de conexão armazenada no `AZURE_CONN_STRING` segredo. A ação especifica o caminho para o arquivo do projeto SQL, define a ação para publicar para implantar o projeto e inclui argumentos de compilação para compilar no modo Versão. Além disso, ele usa o argumento `/p:DropObjectsNotInSource=true` para garantir que quaisquer objetos não presentes na origem sejam removidos do banco de dados de destino durante a implantação.

1. Confirme as alterações.

### Testar o fluxo de trabalho do GitHub Actions

1. No repositório GitHub, selecione a guia **Ações**.
1. Selecione o fluxo de trabalho **Criar e Implantar o Projeto do Banco de Dados SQL**.
    > **Observação:** você verá o fluxo de trabalho em andamento. Aguarde a conclusão. Se já tiver terminado, selecione a última execução para ver os detalhes.

### Verificar as alterações no Banco de Dados SQL do Azure

Com o fluxo de trabalho do GitHub Actions configurado para criar e implantar seu projeto do Banco de Dados SQL do Azure, é hora de verificar as alterações no seu Banco de Dados SQL do Azure.

1. Entre no [portal do Azure](https://portal.azure.com?azure-portal=true). 
1. Navegue até o banco de dados SQL **MyDB**.
1. Selecione **Editor de consultas**.
1. Conecte-se ao banco de dados usando as credenciais **sqladmin**.
1. Na seção **Tabelas**, verifique se a tabela **Funcionários** foi criada. Atualize se necessário.

Você configurou com sucesso um fluxo de trabalho do GitHub Actions para criar e implantar seu projeto do Banco de Dados SQL do Azure.

## Limpar

Quando você está trabalhando em sua própria assinatura, é uma boa ideia identificar, no final de um projeto, se você ainda precisa dos recursos criados. 

Deixar os recursos funcionando desnecessariamente pode resultar em custos extras. É possível excluir os recursos individualmente ou excluir todo o conjunto de recursos no [portal do Azure](https://portal.azure.com?azure-portal=true).

## Mais informações

Para obter mais informações sobre a extensão Projetos de Banco de Dados SQL para o Banco de Dados SQL do Azure, consulte [Introdução à extensão Projetos de Banco de Dados SQL](https://learn.microsoft.com/azure-data-studio/extensions/sql-database-project-extension-getting-started?azure-portal=true).
