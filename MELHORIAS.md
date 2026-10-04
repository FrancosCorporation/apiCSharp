# MELHORIAS — apiCSharp

> **Gerado por análise de código em 2026-10-02** · Stack: .NET 5 (ASP.NET Core, apenas Swashbuckle)
> Branch `master` · base `b7797c7` · **127 LOC** · 0 testes · sem CI
>
> **Este arquivo é um plano de execução.** Cada item tem ID, `arquivo:linha`, mudança exata,
> critério de aceite e comando de verificação.

---

## 0. Como usar este documento

1. Execute na ordem **P0 → P1 → P2 → P3**, respeitando as ondas da §8.
2. Ao terminar um item: marque `- [x]`, rode o **Verificação**, comite `fix(<ID>): descrição`.
3. **Este projeto é um esqueleto do template do ASP.NET.** 127 LOC: `Program.cs`, `Startup.cs` e
   artefatos de build. **Não há controller, não há rota, não há modelo.** Não há o que explorar
   hoje — o risco é de **futuro**: alguém adicionar a primeira rota e achar que está seguro.
4. **Idioma:** português.

---

## 1. Diagnóstico executivo

Projeto ASP.NET Core 5 recém-criado, sem uma única rota. `Program.cs` (26) + `Startup.cs` (59) são o
padrão do template; o resto do "código" são `obj/Debug/net5.0/*.cs` (artefatos).

**O que está bom (não reaça):**

| Item | Evidência |
|---|---|
| **Sem segredo** em `appsettings.json` versionado (verificado — só `Logging` e `AllowedHosts`) | `api/appsettings.json` |
| Swagger só em Development | `Startup.cs:40-42` |
| HTTPS redirection ativa | `Startup.cs:47` |
| `appsettings.Development.json` separado (padrão correto) | presente |
| Dependência mínima (só Swashbuckle) | `api/api.csproj:8` |

**O que está quebrado (risco futuro, não explorabilidade hoje):**

1. **`.gitignore` não cobre `obj/`/`bin/`** — e eles **estão versionados** (42 arq no inventário).
2. **.NET 5 EOL** desde 05/2022, sem patch.
3. **Sem CORS explícito** — seguro por acidente (sem CORS = same-origin), mas quem precisar
   cross-origin vai adicionar `AllowAnyOrigin` às cegas.
4. `AllowedHosts: "*"` no appsettings.
5. Zero testes, zero CI — e sem CI, a primeira linha de rota entra sem barreira.

---

## 2. Tabela de prioridades

| ID | Título | Sev | Arquivo | Depende de |
|---|---|---|---|---|
| SEC-01 | `.gitignore` não cobre `obj/`/`bin/` (e estão versionados) | **P1** | `.gitignore` | — |
| SEC-02 | `AllowedHosts: "*"` | **P2** | `api/appsettings.json` | — |
| SEC-03 | Sem CORS explícito (armadilha para a primeira rota cross-origin) | **P2** | `api/Startup.cs` | — |
| BUG-01 | Artefatos de build versionados sujam o histórico | **P2** | `obj/`, `bin/` | SEC-01 |
| IMP-01 | `UseAuthorization()` sem `AddAuthorization`/`UseAuthentication` | **P2** | `api/Startup.cs:50` | — |
| IMP-02 | Sem `app.UseHsts()` em produção | **P2** | `api/Startup.cs` | — |
| TEST-01 | Zero testes / zero CI numa API que vai crescer | **P1** | *(ausente)* | — |
| DEVOPS-01 | .NET 5 EOL → .NET 8 LTS | **P1** | `api/api.csproj:4` | — |
| DEVOPS-02 | Branch `master` sem `main` (divergência de padrão) | **P3** | git | — |
| DEVOPS-03 | Sem Dockerfile | **P3** | *(ausente)* | — |
| DOC-01 | README diz "pronto para usar" mas não há rota | **P2** | `README.md` | — |
| DOC-02 | Falta `SECURITY.md` | **P3** | *(ausente)* | — |

**Placar: 0 P0 · 3 P1 · 7 P2 · 2 P3 = 12 itens.**

---

## 3. Segurança

### SEC-01 · `.gitignore` não cobre `obj/`/`bin/` (e estão versionados) · [P1]

- **Arquivo:** `.gitignore` (51 linhas) · artefatos em `api/obj/Debug/net5.0/` e `api/bin/`
- **Evidência:** `grep -n 'obj\|bin' .gitignore` → **nada** (o inventário lista
  `api/obj/Debug/net5.0/api.AssemblyInfo.cs` entre os fontes, ou seja, está no índice).
- **Impacto:** (a) **armadilha de segredo**: o `.gitignore` é o que protege `.env`; se ele não cobre
  `obj/`, o `appsettings.Development.json` gerado no build (com conexão local) fica versionado, e
  qualquer `secrets.json` de produção colocado ali **vai** pro GitHub; (b) ruído: o próximo
  `dotnet build` mostra dezenas de arquivos modificados; (c) risco de **conflito** entre
  `.csproj` de versões diferentes.
- **Mudança:** (1) adicionar ao `.gitignore`:
  ```
  [Bb]in/
  [Oo]bj/
  [Dd]ebug/
  [Rr]elease/
  *.user
  ```
  (2) `git rm -r --cached api/obj api/bin`; (3) confirmar que `appsettings.Production.json`
  (se criado) está ignorado.
- **Aceite:** `obj/`/`bin/` fora do índice; `dotnet build` não suja o `git status`.
- **Verificação:**
  ```bash
  git rm -r --cached api/obj api/bin 2>/dev/null
  git ls-files | grep -cE '^(api/)?(obj|bin)/'   # 0
  dotnet build && git status --porcelain | wc -l   # 0
  ```

### SEC-02 · `AllowedHosts: "*"` · [P2]

- **Arquivo:** `api/appsettings.json` (`"AllowedHosts": "*"`)
- **Evidência:** literal no arquivo versionado.
- **Impacto:** aceita qualquer header `Host` — modelo de *cache/host poisoning* e
  *password reset poisoning* (quando houver rota de reset). Impacto baixo hoje (não há rota), real
  quando houver.
- **Mudança:** `AllowedHosts` = lista explícita por ambiente (env var em produção).
- **Aceite:** `Host: evil.example` → `400` quando houver rota.
- **Verificação:**
  ```bash
  grep -n 'AllowedHosts' api/appsettings.json   # nao deve ser "*"
  ```

### SEC-03 · Sem CORS explícito (armadilha para a primeira rota cross-origin) · [P2]

- **Arquivo:** `api/Startup.cs` (`ConfigureServices`/`Configure`)
- **Evidência:** **nenhum** `AddCors`/`UseCors` no projeto. O template não traz CORS.
- **Impacto:** hoje é seguro (sem CORS = só same-origin). O problema é a **armadilha**: quando a
  primeira rota precisar ser chamada de um front em outra origem, o caminho idiomático e **errado**
  é `services.AddCors(o => o.AddDefaultPolicy(p => p.AllowAnyOrigin()))` — abrindo global. Como
  este é um esqueleto, é a hora de **decidir** o padrão antes de existir rota.
- **Mudança:** (1) deixar explícito no código que **não há CORS** (comentário), ou (2) já registrar
  uma política com allowlist por env, pronta para uso, nunca `AllowAnyOrigin`.
- **Aceite:** política CORS explícita com allowlist, ou comentário documentando a decisão.
- **Verificação:**
  ```bash
  grep -n 'AddCors\|AllowAnyOrigin' api/Startup.cs   # AllowAnyOrigin nao pode aparecer
  ```

---

## 4. Bugs e defeitos funcionais

### BUG-01 · Artefatos de build versionados sujam o histórico · [P2]

- **Arquivo:** `api/obj/Debug/net5.0/*.cs`, `api/bin/Debug/net5.0/*`
- **Evidência:** presentes no inventário como "fonte", logo estão no índice.
- **Impacto:** cada `dotnet build`/`dotnet restore` gera diff; e o `api.csproj` de `bin/` pode
  referenciar caminho de outra máquina. Ver `SEC-01` para a correção (é o mesmo trabalho).
- **Mudança:** desindexar (ver `SEC-01`).
- **Aceite:** ver `SEC-01`.
- **Verificação:** `git ls-files | grep -cE '^(api/)?(obj|bin)/'` → 0.

### IMP-01 · `UseAuthorization()` sem `AddAuthorization`/`UseAuthentication` · [P2]

- **Arquivo:** `api/Startup.cs:50`
- **Evidência:** o pipeline chama `app.UseAuthorization()` (linha 50), mas `ConfigureServices`
  (linhas 27-34) só faz `AddControllers()` e `AddSwaggerGen` — **não** há
  `services.AddAuthentication()` nem `AddAuthorization()`, nem `UseAuthentication()` antes do
  `UseAuthorization`.
- **Impacto:** hoje funciona (sem `[Authorize]`, o middleware é no-op). **Quando a primeira rota
  `[Authorize]` for criada, ela não vai ser protegida** — o middleware sem `AddAuthorization`
  registrado não aplica a política, e o autor pode achar que está protegendo.
- **Mudança:** (1) antes de criar rotas autenticadas, registrar
  `services.AddAuthentication(...)` + `AddAuthorization()` e chamar `app.UseAuthentication()` **antes**
  de `UseAuthorization()`; (2) enquanto não houver auth, remover o `UseAuthorization()` órfão para não
  sugerir que existe proteção.
- **Aceite:** rota `[Authorize]` sem token → `401`.
- **Verificação:**
  ```bash
  grep -n 'AddAuthentication\|UseAuthentication' api/Startup.cs   # deve existir antes de usar [Authorize]
  ```

### IMP-02 · Sem `UseHsts()` em produção · [P2]

- **Arquivo:** `api/Startup.cs` (`Configure`)
- **Evidência:** `app.UseHttpsRedirection()` (linha 47) existe, mas **não** há `app.UseHsts()`.
- **Impacto:** sem HSTS, o primeiro acesso pode ser em HTTP claro. Como há HTTPS redirection, o
  impacto é menor — mas HSTS é 1 linha e fecha o downgrade no primeiro acesso.
- **Mudança:** `if (!env.IsDevelopment()) app.UseHsts();` antes do `UseHttpsRedirection()`.
- **Aceite:** resposta em produção traz `Strict-Transport-Security`.
- **Verificação:**
  ```bash
  curl -sI https://<host>/ | grep -i strict-transport-security
  ```

---

## 5. Qualidade: testes

### TEST-01 · Zero testes / zero CI numa API que vai crescer · [P1]

- **Arquivo:** *(ausente)* — nenhum `*Tests*.csproj`, nenhum `.github/workflows/`.
- **Evidência:** projeto sem uma rota e sem teste. Vazio hoje.
- **Impacto:** é a janela ideal para montar a base (teste + CI) **antes** da primeira rota de
  negócio. Depois, com lógica, é mais caro.
- **Mudança:** (1) criar `api.Tests` (xUnit) com um smoke test (`WebApplicationFactory` → `GET /`
  responde o que se espera); (2) `ci.yml` com `dotnet build --warnaserror`, `dotnet test` e
  `--vulnerable` (ver `DEVOPS-01`).
- **Aceite:** CI verde no primeiro push, cobrindo build + teste + CVE.
- **Verificação:**
  ```bash
  dotnet test
  ```

---

## 6. DevOps / Infra

### DEVOPS-01 · .NET 5 EOL → .NET 8 LTS · [P1]

- **Arquivo:** `api/api.csproj:4`
- **Evidência:** `<TargetFramework>net5.0</TargetFramework>` — .NET 5 saiu de suporte em
  **10/05/2022**. Dependência `Swashbuckle 5.6.3` (antiga).
- **Impacto:** runtime sem patch de segurança. Como a API ainda não tem rota, o risco é de
  **futuro** — mas começar já em versão suportada evita retrabalho.
- **Mudança:** (1) `net8.0` (LTS); (2) alinhar `Swashbuckle` na versão atual; (3)
  `dotnet list package --vulnerable --include-transitive` vazio.
- **Aceite:** compila em net8.0, sem CVE.
- **Verificação:**
  ```bash
  sed -i 's|net5.0|net8.0|' api/api.csproj
  dotnet restore && dotnet build && dotnet test
  dotnet list package --vulnerable --include-transitive
  ```

### DEVOPS-02 · Branch `master` (sem `main`) · [P3]

- **Arquivo:** git
- **Evidência:** branch atual `master`; os projetos novos desta conta usam `main`.
- **Impacto:** divergência de padrão; automações que assumem `main` não funcionam.
- **Mudança:** renomear para `main` (ou deixar `master` — mas ser consistente com a conta).
- **Aceite:** branch documentada; push sem argumento vai para a branch certa.
- **Verificação:**
  ```bash
  git branch --show-current
  ```

### DEVOPS-03 · Sem Dockerfile · [P3]

- **Arquivo:** *(ausente)* `Dockerfile`
- **Evidência:** não há Dockerfile nem compose (o repo é só o projeto .NET).
- **Impacto:** baixo — rodar com `dotnet run` basta. Mas padronizar em container facilita o mesmo
  CI nos repos vizinhos.
- **Mudança:** (1) `Dockerfile` multi-stage (`mcr.microsoft.com/dotnet/sdk:8.0` build →
  `aspnet:8.0` run, `USER app`, `EXPOSE`); (2) `EXPOSE` só a porta da app.
- **Aceite:** `docker build` + `run` sobe a API.
- **Verificação:**
  ```bash
  docker build -t apicsharp . && docker run --rm -p 5000:5000 apicsharp
  ```

---

## 7. Documentação

### DOC-01 · README diz "pronto" mas não há rota · [P2]

- **Arquivo:** `README.md` (81 linhas)
- **Evidência:** o README é de instalação (o README de instalação padrão do acervo). Mas o projeto
  **não tem nenhuma rota** — é só o esqueleto do template.
- **Impacto:** quem chega lê "pronto para usar" e não encontra endpoint. Confusão.
- **Mudança:** (1) declarar explicitamente "esqueleto — sem rotas ainda"; (2) como a API **precisa**
  documentar o estado real, listar o que **não** está implementado (auth, rotas, models).
- **Aceite:** README deixa claro que é esqueleto.
- **Verificação:** `grep -n 'esqueleto\|sem rotas\|TODO' README.md`.

### DOC-02 · Falta `SECURITY.md` · [P3]

- **Arquivo:** *(ausente)* `SECURITY.md`
- **Evidência:** tem `LICENSE`/README.
- **Impacto:** baixo hoje. Quando a primeira rota autenticada entrar, a política de `iss`/`aud`/chave
  fica só no código.
- **Mudança:** criar com canal + a invariante "segredo só em env/secret store, nunca em
  `appsettings.json` versionado".
- **Aceite:** arquivo existe.
- **Verificação:** `ls SECURITY.md`

---

## 8. Ordem de execução (waves)

### Wave 1 — Base limpa (P1)
1. **`SEC-01`** + **`BUG-01`** — `.gitignore` + desindexar `obj/`/`bin/`.
2. **`DEVOPS-01`** — .NET 8 + CVE.
3. **`TEST-01`** — CI + smoke test.

> Depois da Wave 1, o esqueleto está pronto para a primeira rota **com** barreira.

### Wave 2 — Prevenir a armadilha de auth (P2)
4. **`IMP-01`** — `AddAuthentication`/`AddAuthorization` (ou remover o `UseAuthorization` órfão).
5. **`SEC-03`** — CORS explícito (decidir agora, antes da primeira rota).
6. **`SEC-02`** — `AllowedHosts` explícito.

### Wave 3 — Polimento (P2/P3)
7. **`IMP-02`** — HSTS.
8. **`DEVOPS-02`**, **`DEVOPS-03`**, **`DOC-01`**.

### Wave 4 — Registro (P3)
9. **`DOC-02`**.

**Dependências que não podem ser invertidas:**
`SEC-01` antes de `BUG-01` (desindexar sem regra = volta) · `IMP-01` **antes** da primeira rota
`[Authorize]` (senão ela nasce desprotegida) · `SEC-03` antes da primeira rota cross-origin ·
`DEVOPS-01` antes de `TEST-01` (o CI usa a versão nova).

---

## 9. Fora de escopo / riscos

| Item | Decisão | Motivo |
|---|---|---|
| Adicionar rotas de negócio agora | **Não** | É esqueleto; primeiro a base limpa (`SEC-01`, `DEVOPS-01`, `TEST-01`). |
| Migrar para minimal APIs / .NET 8 style | **Não** | Refactor sem ganho neste tamanho. |
| Implementar auth completa agora | **Não** | Sem rota, é especulativo. `IMP-01` registra o que precisa vir antes. |
| Copiar um controller de outro projeto | **Não** | Traga o **padrão** (allowlist, sem `AllowAnyOrigin`), não o código com os bugs dele. |

**Riscos desta execução:**

- **P0 inexistente hoje é o achado principal:** o projeto não é explorável **agora**. Os itens
  preparem a base para quando a primeira rota entrar — não[![UTF](ignore) para protelê-los.
- **`DEVOPS-01` (net8.0)** pode exigir ajuste em `Swashbuckle` — valide o build local antes.
- **`DEVOPS-02` renomear branch** pode quebrar link de PR aberto. Faça se não houver PR aberto.

---

## 10. Definição de pronto (DoD)

**Segurança**
- [ ] `SEC-01` — `obj/`/`bin/` fora do índice e ignorados; `dotnet build` não suja o status
- [ ] `SEC-02` — `AllowedHosts` não é `*`
- [ ] `SEC-03` — CORS explícito (allowlist) ou decisão documentada; sem `AllowAnyOrigin`

**Funcional**
- [ ] `BUG-01` — nenhum artefato de build versionado
- [ ] `IMP-01` — `AddAuthentication`+`UseAuthentication` registrados (ou `UseAuthorization` removido)
- [ ] `IMP-02` — HSTS ativo em produção

**Testes e infra**
- [ ] `TEST-01` — CI verde (build + teste + CVE) com smoke test
- [ ] `DEVOPS-01` — net8.0, `--vulnerable` vazio
- [ ] `DEVOPS-02` — branch consistente
- [ ] `DEVOPS-03` — Dockerfile (opcional, mas feito)

**Documentação**
- [ ] `DOC-01` — README declara que é esqueleto, sem rotas
- [ ] `DOC-02` — `SECURITY.md`

**Validação final:**
```bash
dotnet build --warnaserror && dotnet test
dotnet list package --vulnerable --include-transitive   # vazio
git ls-files | grep -cE '^(api/)?(obj|bin)/'             # 0
```

---

*Fim do plano. Gerado por leitura direta do código em 2026-10-02. Nenhum item já estava corrigido.*
