# Configuração da Arquitetura do front-end e back-end

## Requisitos
- Conhecimento em JavaScript, HTML, CSS, C#, .NET Core, API restfull
- Visual Studio
- Docker

## Licença
Distribuido sob a licença MIT License. Veja `LICENSE` para mais informações.

## Criação do projeto
- Visual Studio
- ASP.NET Core Web Application
- Web Application (Model-View-Controller)

## Executar o projeto curso.web.mvc
- Abra o projeto clicando duas vezes no arquivo `curso.sln`, em seguida execute o projeto no Visual Studio clicando no botão `IIS Express`

## Executar o projeto curso.api
Abra o path desse projeto no terminal e execute o comando `dotnet run`. Em seguida abra o endereço no navegador.

Passo 2:
`docker start sqldica`

Passo 3:
`docker logs sqldica`

Passo 4:
Quando o projeto abrir, no Authorize escreva: `Bearer`

## Objetivo front-end
- Configurar o front-end
- Design Pattern MVC
- Integrar com o back-end
- Proteger o front-end

## Entendendo o código:
`web-mvc\curso.web.mvc\Views\Shared\_Layout.cshtml`: É o menu da aplicação.
`web-mvc\Controllers`: Apresenta as controllers, ao clicar nos itens do menu é possível abrir um conteúdo por conta das controllers e a View mostra o layout.
`web-mvc\Views`: Apresenta as views, com as views e as controllers configuradas, o os itens do menu redirecionam uma página.
`web-mvc\Models`: Apresenta o Json convertido para C#. O Json foi pego na documentação (Swagger).

## Como criar a controller?
1. Clique com o botão direito no diretório Controller > Add > Controller > MVC Controller - Empty > Dê o nome da controller

## Como criar a view?
Clique com o botão direito no diretório da View > Razor View > Template: Create > Model class (insira a classe Model criada para determinada view)

## Converter um Json para C#
- [Converte um Json em C#](https://json2csharp.com/)
