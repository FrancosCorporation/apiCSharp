# Política de segurança

## Reportar vulnerabilidades

Envie relatos privados por [GitHub Security Advisories](https://github.com/FrancosCorporation/apiCSharp/security/advisories/new).
Não publique detalhes exploráveis em issues públicas.

## Escopo atual

Este repositório é um esqueleto sem endpoints de negócio, autenticação, dados ou segredos. A política
de `AllowedHosts` é restrita a desenvolvimento local; configure domínios explícitos antes de publicar.
CORS permanece desligado até existir uma origem de frontend aprovada. Rotas protegidas precisam
registrar autenticação e autorização antes de serem adicionadas.
