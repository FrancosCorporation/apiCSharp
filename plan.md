# Plan — apiCSharp

## 1. Visão geral
Esqueleto ASP.NET Core **10 LTS** propositalmente sem rotas de negócio: base segura para a primeira rota (CORS desativado por decisão, HSTS fora de Development, AllowedHosts restrito, autorização removida até existir rota protegida).

## 2. Arquitetura e tecnologias
.NET 10 LTS, Swashbuckle 10.3.0 (Swagger em dev), `api/api.csproj`. Sem BD/auth de propósito.

## 3. Funcionalidades existentes
- [x] Pipeline ASP.NET endurecido (SEC-01..03, IMP-01/02) — 2026-10-11.
- [x] CI restore+build+audit — **modernizada nesta sessão** (Ubuntu 24.04, actions pinadas por SHA: checkout v7, setup-dotnet v6).

## 4. Objetivos futuros
- OBJ-01 (P3): primeira rota real acompanhada de projeto de testes + smoke test (TEST-01 segue vedado nesta esteira).
- OBJ-02 (P3): DECISÃO — branch padrão `master` (mantida por compatibilidade remota; rever se o workspace padronizar `main`).

## 5–9. Melhorias / Roadmap / Riscos / Ideias
Sem superfície de negócio ainda; roadmap desbloqueia quando rotas entrarem. Risco principal: não acumular rotas sem testes (ver TEST-01).

*(Gerado em 2026-10-14 pelo agente orquestrador)*
