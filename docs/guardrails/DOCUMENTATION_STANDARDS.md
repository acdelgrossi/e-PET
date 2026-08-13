# Padrões de Documentação (Guardrails)

Este documento atua como um **Guardrail Passivo** para o projeto e-PET. Ele estabelece as fronteiras do que deve ser documentado e como mantermos o Princípio da Fonte Única de Verdade (Single Source of Truth - SSOT).

## 1. Single Source of Truth (SSOT)
A pasta `docs/` é o **único local** onde a documentação do projeto vive.
- **Não** adicione guias detalhados no `README.md`. O `README.md` serve exclusivamente como índice e direcionador para a pasta `docs/`.
- **Não** mantenha documentos técnicos em plataformas externas que não sejam controladas via Git, pois eles invariavelmente descolarão da versão atual (Information Drift).

## 2. A Estrutura da Pasta `docs/`
- `docs/architecture/`: Documente aqui as decisões técnicas, ADRs (Architecture Decision Records) e Diagramas de funcionamento da aplicação web.
- `docs/guides/`: Guias voltados para o desenvolvedor (como gerenciar dependências, deployment).
- `docs/guardrails/`: Este diretório (regras e limites estabelecidos).
- `docs/handover/`: O material para onboarding de um novo membro da equipe.

## 3. Information Drift (O que NÃO documentar)
- Evite explicar linha a linha do código na documentação (o código deve ser legível por si só).
- O e-PET é uma aplicação web focada nos gestores, então mudanças de UI são comuns. Evite screenshots estáticas detalhadas na documentação central a menos que essencial.

## 4. O Checklist do Pull Request (Active Guardrail)
É **estritamente proibido** aprovar um Pull Request (PR) que adicione ou altere uma regra de negócio complexa sem a devida atualização do respectivo arquivo `.md` na pasta `docs/`. O `.github/PULL_REQUEST_TEMPLATE.md` fará essa verificação.
