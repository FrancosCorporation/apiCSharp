# apiCSharp

Esqueleto de estudo para uma Web API ASP.NET Core. O repositório **não tem controllers, rotas de
negócio, modelos, autenticação ou banco de dados**; o Swagger aparece vazio até alguém adicionar
endpoints.

![C#](https://img.shields.io/badge/C%23-239120?style=flat-square&logo=csharp&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-10-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/status-estudo-lightgrey?style=flat-square)

## Executar localmente

Requisitos: .NET 10 SDK.

```bash
dotnet restore api/api.csproj
dotnet run --project api
```

O perfil local abre `https://localhost:5001` e `http://localhost:5000`. Em `Development`, a UI do
Swagger fica em `/swagger`; não haverá operações enquanto o projeto não tiver controllers.

Para compilar e conferir dependências vulneráveis:

```bash
dotnet build api/api.csproj --configuration Release --warnaserror
dotnet list api/api.csproj package --vulnerable --include-transitive
```

## Segurança do esqueleto

- `AllowedHosts` aceita apenas `localhost` e `127.0.0.1` por padrão. Em produção, defina a variável
  `AllowedHosts` com os nomes reais da API; não use `*`.
- CORS está intencionalmente desativado. Ao adicionar um frontend em outra origem, defina uma
  allowlist específica antes de habilitar `UseCors`.
- HSTS é aplicado fora de `Development`; HTTPS redirection permanece ativo.
- O pipeline ainda não registra autenticação/autorização porque não há rotas protegidas. Configure
  esses serviços e middlewares antes de adicionar uma rota `[Authorize]`.
- Não há segredo ou conexão de banco necessários neste projeto.

Consulte [SECURITY.md](SECURITY.md) para reportar vulnerabilidades. O projeto usa
Swashbuckle.AspNetCore 10.3.0 para OpenAPI/Swagger.

## Estrutura

```text
apiCSharp/
├── api/
│   ├── Program.cs
│   ├── Startup.cs
│   ├── Properties/launchSettings.json
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   └── api.csproj
├── .github/workflows/ci.yml
├── MELHORIAS.md
└── SECURITY.md
```

Licença MIT — veja [LICENSE](LICENSE).
