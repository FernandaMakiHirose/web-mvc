# Configuração da Arquitetura do front-end e back-end

## Requisitos
- Conhecimento em JavaScript, HTML, CSS, C#, .NET Core, API restfull
- Visual Studio
- Docker

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
