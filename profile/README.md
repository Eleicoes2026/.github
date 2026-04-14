# Eleicoes2026

Plataforma de monitoramento, coleta, analise e operacao de dados para o ecossistema `Eleicoes2026`.

## Estado atual da operacao

Hoje o ambiente oficial continua na AWS.

A Hostinger foi preparada para o cutover final com esta estrategia:

- VPS principal preparada: `31.97.162.229`
- banco oficial preparado na Hostinger: PostgreSQL nativo do host
- Hostinger em `standby` ate a janela final
- `31.97.166.138` e `31.97.163.74` fora da rota ativa
- Solr ainda dependente da AWS nesta etapa

A documentacao viva dessa migracao fica em [github.com/Eleicoes2026/docs](https://github.com/Eleicoes2026/docs).

## Visao geral da organizacao

A organizacao esta separada por responsabilidade para facilitar evolucao tecnica, deploy e operacao:

- `docs`: documentacao operacional, relatorios, estado da migracao e runbooks.
- `eleicoes2026.backend`: API principal e camada de negocio do produto.
- `eleicoes2026.brandwatch.collector`: coletor e processador de mencoes com integracao Brandwatch, Solr e Anthropic.
- `eleicoes2026.frontend`: interface web para analise, acompanhamento e comparacao de candidatos.
- `eleicoes2026.gitops`: manifests, pipelines e infraestrutura declarativa do ambiente AWS atual.

## Como os repositorios se conectam

O fluxo principal da plataforma hoje e este:

1. o `eleicoes2026.brandwatch.collector` coleta dados de fontes monitoradas, normaliza resultados e envia para Solr/PostgreSQL
2. o `eleicoes2026.backend` consulta os dados consolidados, expõe APIs e concentra autenticacao, filtros e regras de negocio
3. o `eleicoes2026.frontend` consome a API para mostrar dashboards, mencoes, perfis e comparativos
4. o `eleicoes2026.gitops` publica e opera o ambiente Kubernetes na AWS enquanto a AWS seguir oficial
5. o `docs` centraliza memoria operacional, relatorios, estado das VPS e plano de cutover/deploy

## Repositorios

### `docs`
Repositorio de documentacao viva do projeto.

Contem:

- estado da migracao e da stack
- relatorios operacionais
- inventario das VPS
- plano de deploy para VPS via GitHub Actions

Link: [github.com/Eleicoes2026/docs](https://github.com/Eleicoes2026/docs)

### `eleicoes2026.backend`
Backend principal em Spring Boot responsavel por autenticacao, API REST, integracao com Solr/PostgreSQL e endpoints analiticos.

Contem:

- autenticacao JWT
- endpoints de candidatos, mencoes, sentimentos, tags, word cloud e perfis
- importacao de CSV
- workflow de deploy manual para Hostinger

Link: [github.com/Eleicoes2026/eleicoes2026.backend](https://github.com/Eleicoes2026/eleicoes2026.backend)

### `eleicoes2026.brandwatch.collector`
Servico de coleta e processamento assincrono das mencoes.

Contem:

- schedulers por campanha/query
- integracao com Brandwatch
- persistencia em PostgreSQL
- indexacao em Solr
- processamento em lote com Anthropic
- workflow de deploy manual para Hostinger

Link: [github.com/Eleicoes2026/eleicoes2026.brandwatch.collector](https://github.com/Eleicoes2026/eleicoes2026.brandwatch.collector)

### `eleicoes2026.frontend`
Aplicacao Angular do painel analitico.

Contem:

- overview executivo
- perfil de candidato
- quadro de mencoes
- comparativo entre candidatos
- autenticacao e navegacao protegida
- workflow de deploy manual para Hostinger

Link: [github.com/Eleicoes2026/eleicoes2026.frontend](https://github.com/Eleicoes2026/eleicoes2026.frontend)

### `eleicoes2026.gitops`
Infraestrutura declarativa e operacao do ambiente AWS atual.

Contem:

- aplicacoes ArgoCD
- overlays Kustomize
- pipelines Tekton
- manifests de backend, frontend, coletor e monitoring
- automacao de renovacao de secret do ECR

Link: [github.com/Eleicoes2026/eleicoes2026.gitops](https://github.com/Eleicoes2026/eleicoes2026.gitops)

## Deploy para VPS

Os repositorios `backend`, `frontend` e `brandwatch.collector` passaram a ter workflows manuais de deploy para Hostinger via GitHub Actions.

Modelo atual:

- execucao manual por `workflow_dispatch`
- conexao SSH na VPS
- sincronizacao do repositorio para `/opt/<repo>`
- `docker compose up -d --build <service>` no host
- smoke test local na propria VPS

Essa automacao foi preparada para uso apos o cutover final ou em homologacao controlada. Enquanto a AWS seguir como ambiente oficial, ela nao deve ser usada para publicar producao automaticamente.

## Convencoes

- cada repositorio possui ciclo de vida proprio
- configuracoes sensiveis devem ser fornecidas por variaveis de ambiente ou secrets
- o repositorio `gitops` e a fonte de verdade para o ambiente AWS atual
- o repositorio `docs` concentra memoria operacional e o estado da migracao
- a Hostinger deve permanecer em `standby` ate a janela oficial de virada

## Proximo passo recomendado

Executar o cutover final de forma controlada, ativar a stack oficial na Hostinger e, so depois disso, decidir se os workflows de deploy passam a ser automaticos no merge da `main`.
