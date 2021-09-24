# Configuração da Arquitetura do front-end e back-end

## Requisitos
- Conhecimento em JavaScript, HTML, CSS, C#, .NET Core, API restfull
- Visual Studio
- Docker

## Executar o projeto
1. Faça o clone do projeto
2. Clique 2 vezes no arquivo `curso.sln` 
3. Execute o projeto no Visual Studio

## Licença
Distribuido sob a licença MIT License. Veja `LICENSE` para mais informações.

## Objetivo front-end
- Configurar o front-end
- Design Pattern MVC
- Integrar com o back-end
- Proteger o front-end

## Criação do projeto
- Visual Studio
- ASP.NET Core Web Application
- Web Application (Model-View-Controller)

## Executar o projeto curso.web.mvc
Abra o projeto clicando duas vezes no arquivo `curso.sln`, em seguida execute o projeto no Visual Studio clicando no botão `IIS Express`

## Executar o projeto curso.api
Abra o path desse projeto no terminal e execute o comando `dotnet run`. Em seguida abra o endereço no navegador.

## Entendendo o código:
`web-mvc\curso.web.mvc\Views\Shared\_Layout.cshtml`: É o menu da aplicação. <br>
`web-mvc\Controllers`: Apresenta as controllers, ao clicar nos itens do menu é possível abrir um conteúdo por conta das controllers e a View mostra o layout. <br>
`web-mvc\Views`: Apresenta as views, com as views e as controllers configuradas, o os itens do menu redirecionam uma página. <br>
`web-mvc\Models`: Apresenta o Json convertido para C#. O Json foi pego na documentação (Swagger). <br>

## Como criar a controller?
Clique com o botão direito no diretório Controller > Add > Controller > MVC Controller - Empty > Dê o nome da controller

## Como criar a view?
Clique com o botão direito no diretório da View > Razor View > Template: Create > 
1. Model class (insira a classe Model criada para determinada view)
2. List (cria mais de um dado)

## Converter um Json para C#
[Converte um Json em C#](https://json2csharp.com/)

## Validações 
Validações foram adicionadas, por exemplo:
1. Validação de e-mail:
>[EmailAddress(ErrorMessage = "O e-mail é inválido")]

2. O campo é obrigatório:
>[Required(ErrorMessage = "A senha é obrigatória")]

## Integrando com o backend
- Remoção dos mocks
- Realizando integração com o HttpClient
- Refatorando com a biblioteca Refit

## Integração com o HttpClient
No arquivo `UsuarioController.cs`:
```
var clientHandler = new HttpClientHandler();
clientHandler.ServerCertificateCustomValidationCallback = (sender, cert, chain, sslPolicyErrors) => { return true; };

var httpClient = new HttpClient(clientHandler);
httpClient.BaseAddress = new Uri("https://localhost:5001/");
var registrarUsuarioViewModelInputJson =  JsonConvert.SerializeObject(registrarUsuarioViewModelInput);
var httpContent = new StringContent(registrarUsuarioViewModelInputJson, Encoding.UTF8, "application/json");

var httpPost = httpClient.PostAsync("/api/v1/usuario/registrar", httpContent).GetAwaiter().GetResult();

if(httpPost.StatusCode == System.Net.HttpStatusCode.Created)
{
    ModelState.AddModelError("", "Os dados foram cadastrado com sucesso");
}
else
{
    ModelState.AddModelError("", "Erro ao cadastrar");
```

## Refit
1. Instalação da biblioteca Refit no NuGet (`Refit` e `Refit.HttpClientFactory`)
2. Com ela adicionamos o endpoint Post no arquivo `IUsuarioService.cs`, exemplo:
>[Post("/api/v1/usuario/registrar")]

3. Deixa o código assíncrono 
4. Por fim adicionamos o seguinte código no arquivo `Startup.cs`:

```
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddHttpContextAccessor();

    var clientHandler = new HttpClientHandler
    {
        ServerCertificateCustomValidationCallback = (sender, cert, chain, sslPolicyErrors) => { return true; }
    };

    services.AddRefitClient<IUsuarioService>()
        .ConfigureHttpClient(c =>
        {
            c.BaseAddress = new Uri(Configuration.GetValue<string>("UrlApiCurso"));
        }).ConfigurePrimaryHttpMessageHandler(c => clientHandler);
```

5. Com o Refit podemos comentar o código da integração com o HttpClient, simplificando assim:
```
[HttpPost]
public async Task<IActionResult> Cadastrar(RegistrarUsuarioViewModelInput registrarUsuarioViewModelInput)
{
    try
    {
        var usuario = await _usuarioService.Registrar(registrarUsuarioViewModelInput);

        ModelState.AddModelError("", $"Os dados foram cadastrado com sucesso para o login {usuario.Login}");
    }
    catch (ApiException ex)
    {
        ModelState.AddModelError("", ex.Message);
    }
    catch (Exception ex)
    {
        ModelState.AddModelError("", ex.Message);
    }
```

## Token
1. No arquivo `Startup.cs` adicionei:
>.AddHttpMessageHandler<BearerTokenMessageHandler>()
    
2. No arquivo `ICursoService.cs` adicionei:
>[Headers("Authorization: Bearer")] 
    
3. No arquivo `BearerTokenMessageHandler.cs` criamos o código que mostra uma mensagem do Bearer Handler
4. No arquivo `UsuarioController.cs` foi adicionado o seguinte código:
```
[HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Logar(LoginViewModelInput loginViewModelInput)
        {
            try
            {
                await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

                var usuario = await _usuarioService.Logar(loginViewModelInput);

                var claims = new List<Claim>
                {
                    new Claim(ClaimTypes.NameIdentifier, usuario.Usuario.Codigo.ToString()),
                    new Claim(ClaimTypes.Name, usuario.Usuario.Login),
                    new Claim(ClaimTypes.Email, usuario.Usuario.Email),
                    new Claim("token", usuario.Token),
                };

                var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

                var authProperties = new AuthenticationProperties
                {
                    ExpiresUtc = new DateTimeOffset(DateTime.UtcNow.AddDays(1))
                };
                await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, new ClaimsPrincipal(claimsIdentity), authProperties);

                ModelState.AddModelError("", $"O usuário está autenticado {usuario.Token}");
            }
            catch (ApiException ex)
            {
                ModelState.AddModelError("", ex.Message);
            }
            catch (Exception ex)
            {
                ModelState.AddModelError("", ex.Message);
            }

            return View();
        }
```
5. Protegemos o projeto no arquivo `CursoController.cs` (Caso abra o projeto na guia anônima, não será possível abrir o projeto):
>[Microsoft.AspNetCore.Authorization.Authorize]

6. Autenticação se o usuário está logado:
```
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
.AddCookie(options =>
{
   options.LoginPath = "/Usuario/Logar";
   options.AccessDeniedPath = "/Usuario/Logar";
});    
```
## Objetivo Back-end
- Configurar as rotas (endpoints)
- Configuração do Swagger (Instalação do pacota NuGet, Configuração do XML, Configuração do Service (IOC), Habilitar o Middleware
- Setup validação de entrada de dados: Criação e Configuração das ViewModels, Configuração do ActionFilter e do Startup
- Configuração 'provider' JWT
- Modelar as entidades do banco de dados, implementar a HasForeingKey e configurar o DbContext
    
### Criação do projeto
- Visual Studio
- ASP.NET Core Web Application
- API
- Configure for HTTPS [X]
- Habilitar o Docker []
- Arquivos apagados: `WeatherForecast.cs` e `WeatherForecastController.cs` <br>
- No arquivo `launchSettings.json` o código `"launchUrl": "weatherforecast"` ficou `"launchUrl": ""`
    
## Postman
- Caso queira usar o Postman para fazer requisição:
- POST = https://localhost44341/api/Usuario
- Body, raw, JSON
```
    "Login": "teste",
    "Senha": "senha"
```
## Swagger
- Clique com o botão direito no projeto > Propriedades > Build > XML documentation file: curso.api.xml [X] > (Deixe apenas o nome do arquivo e marque a caixinha)
- Clique com o botão direito nas dependências > NuGet > (Instale o pacote: `Swashbuckle.AspNetCore` e `Swashbuckle.AspNetCore.Annotations`)
- Adicionou código no Startup.cs
- O arquivo `UsuarioController.cs` apresenta códigos do Swagger para ajudar a mandar mensagens

## Configuração 'provider' JWT
- Clique com o botão direito nas dependências > NuGet > (Instale o pacote: `Microsoft.AspNetCore.Authentication` e `Microsoft.AspNetCore.Authentication.JwtBearer`)
- Adicionou código no Startup.cs de chave de configuração
- O arquivo `appsettings.json` apresenta o código da chave de configuração:
```
  "JwtConfigurations": {
    "Secret": "MzfsT&d9gprP>!9$Es(X!5g@;ef!5sbk:jH\\2.}8ZP'qY#7"
  },    
```
    
## Banco de dados
- Clique com o botão direito nas dependências > NuGet > (Instale o pacote: `Microsoft.Entity.FrameworkCore`, `Microsoft.Entity.FrameworkCore.Relational`, `Microsoft.Entity.FrameworkCore.SqlServer` e `Microsoft.EntityFrameworkCore.Tools`)
- Tudo isso fica dentro do diretório `Infraestruture`
- O arquivo `DbFactoryDbContext.cs`: gera o versionamento da base
    
## Quiz
### Ao habilitar a dataannsations  [Required] nas nossas model a validação é feita em qual lado?
Cliente.
    
### Quando precisamos utilizar HttpClientHandler?
Customizar o objeto HttpClient, como por exemplo, desabilitar certificado digital do lado do cliente em tempo de desenvolvimento. 

### O MVC Pattern é um padrão arquitetural?
Falso.

### Qual a finalidade do Microsoft Identity?
Configuração Autenticação e autorização.

### Em caso de não sucesso do retorno da biblioteca Refit, qual é o tipo de exception a ser recuperada?
ApiException.

### Baseado em um mecanismo de autenticação e autorização JWT, qual o tipo de informação devemos enviar o token headers?
Autorization : Bearer.

### No padrão MVC qual a responsabilidade de uma controller?
Orquestrar entra a view e o model.

### A classe HttpClient é utilizada para qual finalidade?
Realizar integração com webservices.

### Qual atributo  é responsável para configurarmos um ou mais propriedades da Model quando é obrigatório?
[Required].

### É correto afirmar que um handler nos ajuda a interceptação antes de request?
Verdadeiro.

### Documentar API é importar por que?
Ajudar na comunicação com outros times.

### Qual atributo é responsável para configurarmos um ou mais propriedades da Viewmodel quando é obrigatório?
[Required]

### A técnica de refatoração nos ajuda a melhorar a manutenção do código a longo prazo.
Verdadeiro.

### Como podemos desabilitar o model state das controller para validação automática de campos obrigatórios no arquivo statup.cs?
```
services.AddControllers().ConfigureApiBeharviorOptions(options => { options.SuppressModelStateInvalidFilter = true }    
```
    
### Seguindo as boas práticas do estilo arquitetural RESTFUL, ao realizar uma chamada a um endpoint do verbo “Post” para criação de um novo recurso, qual é o código do status indicado?
201.

### O que o Entity Framework é?
ORM.

### Seguindo as boas práticas do estilo arquitetural RESTFUL, ao realizar uma tentativa a um endpoint sem permissão, qual é o código do status indicado?
403.

### Seguindo as boas práticas do estilo arquitetural RESTFUL, ao realizar uma tentativa a um endpoint de autenticação sem sucesso, qual é o código do status indicado?
401.

### Qual a finalidade do provider JWT?
Autenticação e autorização.

### O Repository Pattern é um padrão arquitetural?
Falso.
