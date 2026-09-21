# apiCSharp

## 🐳 Instalação e Execução (Docker) — recomendado

### Pré-requisitos
- [Docker](https://docs.docker.com/get-docker/) + Docker Compose

### Rodar com Docker
```bash
docker compose up --build
```
```bash
docker run --rm -v $(pwd):/src -w /src mcr.microsoft.com/dotnet/sdk:8.0 dotnet run --project api
```

### Sem Docker (local)
```bash
# Requer .NET SDK
dotnet build api
dotnet run --project api
```

Projeto de estudo: esqueleto de Web API em ASP.NET Core 5 com Swagger configurado para documentar os endpoints durante o desenvolvimento.

![C#](https://img.shields.io/badge/C%23-239120?style=flat-square&logo=csharp&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-5.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/status-estudo-lightgrey?style=flat-square)

## Sobre

**Projeto de estudo** criado para praticar a estrutura mínima de uma API REST em ASP.NET Core: bootstrap da aplicação (`Program.cs`), pipeline HTTP e injeção de dependências (`Startup.cs`) e documentação interativa com Swagger UI. O repositório é o ponto de partida de outros projetos C# do mesmo autor — não há regra de negócio nem endpoints de domínio implementados.

## Funcionalidades

Comprovadas pelo código:

- Bootstrap de um host ASP.NET Core (`Program.cs`) com `Startup` customizado.
- Pipeline com `UseHttpsRedirection`, `UseRouting` e `UseAuthorization`.
- `AddControllers()` e mapeamento de controllers (`MapControllers`) prontos para receber rotas.
- Swagger/OpenAPI habilitado apenas em ambiente de desenvolvimento, com UI acessível em `/swagger`.

> Não existem controllers, models ou serviços neste repositório; a API sobe sem rotas de negócio.

## Stack

- **Linguagem/framework**: C# com ASP.NET Core 5.0 (Web API)
- **Documentação**: Swashbuckle.AspNetCore 5.6.3 (Swagger/OpenAPI)
- **Configuração**: `appsettings.json` + `Properties/launchSettings.json`

## Como rodar

1. Instale o [.NET 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0).
2. Na pasta `api/`, restaure e execute:

   ```bash
   dotnet restore
   dotnet run
   ```

3. Com o perfil `api` (padrão do `launchSettings.json`), a aplicação sobe em `https://localhost:5001` e `http://localhost:5000`, abrindo o Swagger em `/swagger`.

Não há banco de dados, variáveis de ambiente ou segredos necessários.

## Estrutura do projeto

```
apiCSharp/
└── api/
    ├── Properties/          # Perfis de execução (launchSettings.json)
    ├── Program.cs           # Bootstrap do host
    ├── Startup.cs           # Pipeline e serviços (controllers + Swagger)
    ├── appsettings.json
    ├── appsettings.Development.json
    └── api.csproj           # net5.0 + Swashbuckle.AspNetCore
```

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE).
