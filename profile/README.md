# Eleicoes2026

Plataforma de monitoramento, coleta, análise e operação de dados para o ecossistema `Eleicoes2026`.

## Visão geral

A organização está separada por responsabilidade para facilitar evolução técnica, deploy e operação:

- `docs`: documentação operacional, relatórios e estado da migração.
- `eleicoes2026.backend`: API principal e camada de negócio do produto.
- `eleicoes2026.brandwatch.collector`: coletor e processador de menções com integração Brandwatch, Solr e Anthropic.
- `eleicoes2026.frontend`: interface web para análise, acompanhamento e comparação de candidatos.
- `eleicoes2026.gitops`: manifests, pipelines e infraestrutura declarativa do ambiente.

## Como os repositórios se conectam

O fluxo principal da plataforma hoje é este:

1. O `eleicoes2026.brandwatch.collector` coleta dados de fontes monitoradas, normaliza resultados e envia para Solr/PostgreSQL.
2. O `eleicoes2026.backend` consulta os dados consolidados, expõe APIs e concentra autenticação, filtros e regras de negócio.
3. O `eleicoes2026.frontend` consome a API para mostrar dashboards, menções, perfis e comparativos.
4. O `eleicoes2026.gitops` publica e opera esse ecossistema em Kubernetes com ArgoCD, Tekton, monitoring e automações auxiliares.
5. O `docs` centraliza relatórios de migração, histórico operacional e materiais de apoio.

## Repositórios

### `docs`
Repositório de documentação viva do projeto.

Contém:
- estado da migração e stack
- relatórios operacionais
- registros de acompanhamento

Link: [github.com/Eleicoes2026/docs](https://github.com/Eleicoes2026/docs)

### `eleicoes2026.backend`
Backend principal em Spring Boot responsável por autenticação, API REST, integração com Solr/PostgreSQL e endpoints analíticos.

Contém:
- autenticação JWT
- endpoints de candidatos, menções, sentimentos, tags, word cloud e perfis
- importação de CSV
- integrações auxiliares com Brandwatch e coletor

Link: [github.com/Eleicoes2026/eleicoes2026.backend](https://github.com/Eleicoes2026/eleicoes2026.backend)

### `eleicoes2026.brandwatch.collector`
Serviço de coleta e processamento assíncrono das menções.

Contém:
- schedulers por campanha/query
- integração com Brandwatch
- persistência em PostgreSQL
- indexação em Solr
- processamento em lote com Anthropic

Link: [github.com/Eleicoes2026/eleicoes2026.brandwatch.collector](https://github.com/Eleicoes2026/eleicoes2026.brandwatch.collector)

### `eleicoes2026.frontend`
Aplicação Angular do painel analítico.

Contém:
- overview executivo
- perfil de candidato
- quadro de menções
- comparativo entre candidatos
- autenticação e navegação protegida

Link: [github.com/Eleicoes2026/eleicoes2026.frontend](https://github.com/Eleicoes2026/eleicoes2026.frontend)

### `eleicoes2026.gitops`
Infraestrutura declarativa e operação do ambiente.

Contém:
- aplicações ArgoCD
- overlays Kustomize
- pipelines Tekton
- manifests de backend, frontend, coletor e monitoring
- automação de renovação de secret do ECR

Link: [github.com/Eleicoes2026/eleicoes2026.gitops](https://github.com/Eleicoes2026/eleicoes2026.gitops)

## Convenções

- Cada repositório possui ciclo de vida próprio.
- Configurações sensíveis devem ser fornecidas por variáveis de ambiente ou secrets do cluster.
- O repositório `gitops` é a fonte de verdade para deploy e operação declarativa.
- O repositório `docs` concentra memória operacional e relatórios.

## Próximo passo recomendado

Padronizar variáveis de ambiente, checklists de deploy e diagramas de arquitetura entre backend, coletor, frontend e gitops para reduzir acoplamento operacional.
