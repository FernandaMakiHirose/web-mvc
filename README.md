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
No projeto/terminal digite:
`docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=App@223020" --name sqldica -p 1433:1433 -d mcr.microsoft.com/mssql/server:2019-latest`

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
