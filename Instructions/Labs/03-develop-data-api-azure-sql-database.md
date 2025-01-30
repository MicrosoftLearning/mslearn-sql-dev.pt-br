---
lab:
  title: Desenvolver uma API de Dados para o Banco de Dados SQL do Azure
  module: Develop a Data API for Azure SQL Database
---

# Desenvolver uma API de Dados para o Banco de Dados SQL do Azure

Neste exercício, você desenvolverá e implantará uma API de dados para um banco de dados Azure SQL usando o Aplicativos Web Estáticos do Azure. Isso fornecerá experiência prática na configuração de uma configuração do construtor de API de dados e implantá-la em um ambiente de Aplicativos Web Estáticos do Azure.

## Pré-requisitos

Antes de iniciar este exercício, certifique-se de ter o seguinte:

- Uma assinatura do Azure ativa.
- Conhecimento básico do Banco de Dados SQL do Azure, Aplicativos Web Estáticos do Azure e GitHub.
- Visual Studio Code instalado com as extensões necessárias.
- Uma conta do GitHub para gerenciar o repositório.

## Configurar o ambiente

Há algumas etapas que você precisa seguir para configurar o ambiente para este exercício.

### Instalar as extensões do Visual Studio Code

Antes de começar este exercício, você precisa instalar as extensões do Visual Studio Code.

1. Abra o Visual Studio Code.
1. Abra uma janela de terminal no Visual Studio Code.
1. Instale o a CLI de Aplicativos Web Estáticos executando o seguinte comando:

    ```bash
    npm install -g @azure/static-web-apps-cli
    ```

1. Instale a CLI construtor de API de dados executando o seguinte comando:

    ```bash
    dotnet tool install --global Microsoft.DataApiBuilder
    ```

O Visual Studio Code agora está configurado com as extensões necessárias.

### Criar um Banco de Dados SQL do Azure

Se você ainda não tiver feito isso, precisará criar um Banco de Dados SQL do Azure.

1. Entre no [portal do Azure](https://portal.azure.com?azure-portal=true). 
1. Navegue até a página do **SQL do Azure** e selecione **+ Criar**.
1. Selecione **Banco de Dados SQL**, *Banco de dados único* e o **botão Criar**.
1. Preencha as informações necessárias na caixa de diálogo **Criar Banco de Dados SQL** e selecione **OK** (deixe todas as outras opções como padrão).

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
1. Selecione **Salvar**.

### Adicionar Dados de Amostra ao Banco de Dados

Agora que você tem um banco de dados SQL do Azure, você precisa adicionar alguns dados de amostra. Isso ajudará você a testar a API assim que estiver em funcionamento.

1. Navegue até o seu banco de dados SQL do Azure recém-criado.
1. Use o **Editor de Consultas** no portal Azure para executar o seguinte script SQL:

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

### Crie um aplicativo web básico no GitHub

Antes de podermos criar um aplicativo Web estático do Azure, precisamos criar um aplicativo web básico no GitHub.

1. Para criar um aplicativo web básico no GitHub, vá para [gerar um site vanilla](https://github.com/staticwebdev/vanilla-basic/generate).
1. Certifique-se de que o modelo de Repositório esteja definido como **staticwebdev/vanilla-basic**.
1. Em ***Proprietário***, selecione sua conta.
1. Em ***Nome do repositório***, insira o nome **my-sql-repo**.
1. Torne o repositório **Privado**.
1. Selecione o botão **Criar repositório**.

## Criar um Aplicativo Web Estático do Azure

Primeiro, criaremos nosso aplicativo da web estático e, em seguida, adicionaremos a configuração do construtor de API de dados a ele.

1. No portal Azure, navegue até a página **Aplicativos Web Estáticos**.
1. Selecione **+ Criar**.
1. Preencha as seguintes informações na caixa de diálogo **Criar Aplicativo Web Estático** (deixe todas as outras opções como padrão):

    | Configuração | Valor |
    | --- | --- |
    | Subscription | Sua assinatura |
    | Grupo de recursos | *Selecione ou crie um novo grupo de recursos* |
    | Nome | *um nome único* |
    | Fonte do plano de hospedagem | *GitHub* |
    | Conta do GitHub | *Selecione sua conta* |
    | Organização | *Provavelmente seu nome de usuário do GitHub* |
    | Repositório | *Selecione o repositório que você criou na etapa anterior* |
    | Branch | *main* |

1. Selecione **Examinar + criar** e depois **Criar**.
1. Uma vez implantado, vá para o recurso.
1. Selecione o botão **Visualizar aplicativo no navegador**, você deve ver uma página da web simples com uma mensagem de boas-vindas. Você pode fechar a guia.

## Adicionar um arquivo de configuração do construtor de API de dados

É hora de adicionar a configuração do Construtor de API de Dados ao Aplicativo Web Estático do Azure. Precisamos criar um novo arquivo no repositório GitHub para adicionar a configuração do Construtor de API de Dados.

1. No Visual Studio Code, clone o repositório GitHub que você criou anteriormente.
1. Abra uma janela de terminal no Visual Studio Code.
1. Execute o seguinte comando para criar um novo arquivo de configuração do Construtor de API de Dados:

    ```bash
    swa db init --database-type "mssql"
    ```

    Isso criará uma nova pasta chamada *swa-db-connections* e um arquivo chamado *staticwebapp.database.config.json* dentro dessa pasta.

1. Execute o seguinte comando para adicionar as entidades do banco de dados ao arquivo de configuração:

    ```bash
    dab add "Employees" --source "dbo.Employees" --permissions "anonymous:*" --config "swa-db-connections/staticwebapp.database.config.json"
    ```

1. Revise o conteúdo do arquivo *staticwebapp.database.config.json*. 
1. Confirme e envie as alterações para o repositório do GitHub.

## Configurar a conexão de banco de dados

1. No portal do Azure, navegue até o aplicativo Web estático do Azure que você criou.
1. Em **Configurações**, selecione **Conexão de banco de dados**.
1. Selecione **Vincular banco de dados existente**.
1. Na caixa de diálogo **Vincular banco de dados*, selecione o Banco de Dados SQL do Azure que você criou anteriormente com as seguintes configurações adicionais.

    | Configuração | Valor |
    | --- | --- |
    | Tipo de Banco de Dados | *Banco de Dados SQL do Azure* |
    | Tipo de autenticação | *Cadeia de Conexão* |
    | Nome de Usuário | *seu nome de usuário administrador* |
    | Senha | *a senha que você deu ao seu usuário administrador* |
    | Caixa de seleção de reconhecimento | *Verificado* |

   > **Observação:** em um ambiente de produção, restrinja o acesso apenas aos endereços IP necessários. Além disso, considere usar Identidades Gerenciadas para seu aplicativo Web estático para acessar o banco de dados em vez da autenticação SQL. Para obter mais informações, consulte [Identidades gerenciadas no Microsoft Entra para SQL do Azure](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).
1. Selecione **Vincular**.

## Testar o ponto de extremidade da API de dados

Agora só precisamos testar o ponto de extremidade da API de dados.

1. No portal do Azure, navegue até o aplicativo Web estático do Azure que você criou.
1. Na página Visão geral, copie a URL do aplicativo web.
1. Abra uma nova guia do navegador e cole a URL. Você ainda deverá ver a página da web simples com a mensagem **Vanilla JavaScript App**.
1. Adicione **/data-api** ao final da URL e pressione **Enter**. Ele deve exibir **Íntegro** para indicar que a API de dados está funcionando.
1. Adicione **/data-api/rest/Employees** ao final da URL e pressione **Enter**. Você deve ver os dados de exemplo adicionados ao Banco de Dados SQL do Azure anteriormente.

Você desenvolveu e implantou com sucesso uma API de dados para um Banco de Dados SQL do Azure usando Aplicativos Web Estáticos do Azure.

## Limpar

Quando você está trabalhando em sua própria assinatura, é uma boa ideia identificar, no final de um projeto, se você ainda precisa dos recursos criados. 

Deixar os recursos funcionando desnecessariamente pode resultar em custos extras. É possível excluir os recursos individualmente ou excluir todo o conjunto de recursos no [portal do Azure](https://portal.azure.com?azure-portal=true).

## Mais informações

Para obter mais informações sobre o construtor de API de dados para Bancos de Dados SQL do Azure, consulte [O que é o construtor de API de dados para Bancos de Dados do Azure?](https://learn.microsoft.com/azure/data-api-builder/overview?azure-portal=true).
