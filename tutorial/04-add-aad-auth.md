<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph. Nesta etapa, você integrará a [biblioteca de autenticação da Microsoft (MSAL) para .net](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) no aplicativo.

1. Inicialize o [repositório de segredo de desenvolvimento do .net](/aspnet/core/security/app-secrets) abrindo sua CLI no diretório que contém **GraphTutorial. csproj** e executando o comando a seguir.

    ```Shell
    dotnet user-secrets init
    ```

1. Adicione a ID do aplicativo e uma lista de escopos necessários para o repositório secreto usando os seguintes comandos. Substitua `YOUR_APP_ID_HERE` pela ID do aplicativo que você criou no portal do Azure.

    ```Shell
    dotnet user-secrets set appId "YOUR_APP_ID_HERE"
    dotnet user-secrets set scopes "User.Read;MailboxSettings.Read;Calendars.ReadWrite"
    ```

    Vamos examinar os escopos de permissão que você acabou de definir.

    - **User. Read** permitirá que o aplicativo Leia o perfil do usuário conectado para obter informações como nome para exibição e endereço de email.
    - **MailboxSettings. Read** permitirá que o aplicativo Leia o fuso horário, o formato de data e o formato de hora preferencial do usuário.
    - **Calendars. ReadWrite** permitirá que o aplicativo Leia os eventos existentes no calendário do usuário e adicione novos eventos.

## <a name="implement-sign-in"></a>Implementar logon

Nesta seção, você criará um provedor de autenticação que pode ser usado com o SDK do Graph e também pode ser usado para solicitar explicitamente um token de acesso usando o [fluxo do código de dispositivo](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-device-code).

### <a name="create-an-authentication-provider"></a>Criar um provedor de autenticação

1. Crie um novo diretório no diretório **GraphTutorial** chamado **autenticação**.
1. Crie um novo arquivo no diretório de **autenticação** chamado **DeviceCodeAuthProvider.cs** e adicione o código a seguir ao arquivo.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/DeviceCodeAuthProvider.cs" id="AuthProviderSnippet":::

Considere o que esse código faz.

- Ele usa a `IPublicClientApplication` implementação MSAL para solicitar e gerenciar tokens.
- A `GetAccessToken` função:
  - Entra no usuário se ainda não estiverem conectados usando o fluxo de código de dispositivo.
  - Garante que o token retornado seja sempre atualizado usando a `AcquireTokenSilent` função, que retorna o token em cache se ele não tiver expirado e atualiza o token se ele tiver expirado.
- Ele implementa a `IAuthenticationProvider` interface para que o SDK do Graph possa usar a classe para autenticar chamadas de gráfico.

## <a name="sign-in-and-display-the-access-token"></a>Entrar e exibir o token de acesso

Nesta seção, você atualizará o aplicativo para chamar a `GetAccessToken` função, que entrará no usuário. Você também adicionará código para exibir o token.

1. Adicione a função a seguir à classe `Program`.

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="LoadAppSettingsSnippet":::

1. Adicione o seguinte código à `Main` função imediatamente após a `Console.WriteLine(".NET Core Graph Tutorial\n");` linha.

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="InitializationSnippet":::

1. Adicione o seguinte código à `Main` função imediatamente após a `// Display access token` linha.

    ```csharp
    Console.WriteLine($"Access token: {accessToken}\n");
    ```

1. Criar e executar o aplicativo. O aplicativo exibe uma URL e um código de dispositivo.

    ```Shell
    PS C:\Source\GraphTutorial> dotnet run
    .NET Core Graph Tutorial

    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code F7CG945YZ to authenticate.
    ```

    > [!TIP]
    > Se você encontrar erros, Compare seu **Program.cs** com o [exemplo no GitHub](https://github.com/microsoftgraph/msgraph-training-dotnet-core/blob/master/demo/GraphTutorial/Program.cs).

1. Abra um navegador e navegue até a URL exibida. Insira o código fornecido e entre. Depois de concluído, volte para o aplicativo e escolha o **1. Exibir** opção de token de acesso para exibir o token de acesso.
