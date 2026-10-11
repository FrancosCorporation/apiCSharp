# Melhorias — apiCSharp

## Escopo

Este repositório é um esqueleto ASP.NET Core: ainda não tem rotas de negócio, modelos,
autenticação ou banco de dados. As mudanças abaixo mantêm esse escopo e preparam uma base segura
para a primeira rota.

## Estado da execução — 2026-10-11

| Item | Estado | Resultado |
|---|---|---|
| SEC-01 / BUG-01 — artefatos de build | Já resolvido ao iniciar | `.gitignore` cobre `obj/` e `bin/`; nenhum artefato está rastreado. |
| SEC-02 — `AllowedHosts` aberto | Concluído | Configuração local aceita `localhost;127.0.0.1`; em produção, definir os hosts reais. |
| SEC-03 — CORS indefinido | Concluído | CORS permanece desativado; a decisão está documentada no código e no README. |
| IMP-01 — autorização sem configuração | Concluído | Removido `UseAuthorization`; registrar autenticação/autorização junto com futuras rotas protegidas. |
| IMP-02 — HSTS | Concluído | `UseHsts()` é aplicado fora de `Development`. |
| TEST-01 — CI e testes | Parcial | CI compila e audita dependências; não há projeto de testes nem smoke test. Nenhum teste foi adicionado ou executado. |
| DEVOPS-01 — runtime sem suporte | Concluído | Migração para .NET 10 LTS e Swashbuckle 10.3.0. |
| DEVOPS-02 — branch `master` | Adiado | Branch mantida para respeitar a configuração remota existente. |
| DEVOPS-03 — imagem Docker | Adiado | Sem requisito de implantação em container para este esqueleto. |
| DOC-01 / DOC-02 — documentação | Concluído | README descreve os limites atuais; `SECURITY.md` criado. |

## Validação realizada

- `dotnet build api/api.csproj --configuration Release --warnaserror` — passou, zero avisos e erros.
- `dotnet list api/api.csproj package --vulnerable --include-transitive` — nenhuma dependência vulnerável encontrada.
- `git diff --check`, parse de `appsettings.json` e do projeto XML — passaram.
- Nenhuma suíte de testes foi executada e nenhum servidor ou container foi iniciado.

## Decisões para a primeira rota

- Não habilitar CORS global. Se houver frontend em outra origem, criar allowlist explícita.
- Registrar `AddAuthentication`, `AddAuthorization`, `UseAuthentication` e `UseAuthorization` com a primeira rota protegida.
- Ajustar `AllowedHosts` para os nomes de produção antes de publicar.
- Não considerar a existência deste CI como substituta de testes de comportamento.

## Itens adiados

- Renomear `master` para `main`: exige alinhar a configuração remota.
- Criar Dockerfile: depende de requisito real de implantação.
- Adicionar smoke test: passa a fazer sentido quando existir uma rota com contrato definido.
