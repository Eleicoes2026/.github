# Eleicoes2026

Plataforma de monitoramento, coleta, análise e operação de dados para o ecossistema `Eleicoes2026`.

## Estado atual da operação

O cutover AWS -> Hostinger foi concluído e a stack está operacional na Hostinger.

Ambiente oficial atual:

- VPS de aplicação oficial: `31.97.162.229`
- VPS de banco oficial: `31.97.166.138`
- VPS de backup oficial: `31.97.163.74`
- Domínios públicos resolvendo para Hostinger em DNS público
- HTTPS ativo com certificado Let's Encrypt
- PostgreSQL oficial em produção na VPS de banco
- Solr operacional localmente na VPS de aplicação
- AWS pode permanecer ligada apenas como contingência até o desligamento controlado

Containers ativos na VPS de aplicação:

- `eleicoes-backend`
- `eleicoes-brandwatch`
- `eleicoes-frontend`
- `eleicoes-solr`

Domínios publicados:

- `2026.datasocial.info`
- `dashboard2026.datasocial.info`
- `backendeleicoes2026.datasocial.info`
- `coletorel2026.datasocial.info`

A documentação viva da operação fica em [github.com/Eleicoes2026/docs](https://github.com/Eleicoes2026/docs).

## Visão geral da organização

A organização está separada por responsabilidade para facilitar evolução técnica, deploy e operação:

- `docs`: documentação operacional, relatórios, estado da migração, inventário, runbooks e procedimentos de handover.
- `eleicoes2026.backend`: API principal e camada de negócio do produto.
- `eleicoes2026.brandwatch.collector`: coletor e processador de menções com integração Brandwatch, Solr e Anthropic.
- `eleicoes2026.frontend`: interface web para análise, acompanhamento e comparação de candidatos.
- `eleicoes2026.gitops`: manifests, pipelines e infraestrutura declarativa do ambiente AWS legado/contingência.

## Como os repositórios se conectam

O fluxo principal da plataforma é:

1. O `eleicoes2026.brandwatch.collector` coleta dados de fontes monitoradas, normaliza resultados e envia para Solr/PostgreSQL.
2. O `eleicoes2026.backend` consulta os dados consolidados, expõe APIs e concentra autenticação, filtros e regras de negócio.
3. O `eleicoes2026.frontend` consome a API para mostrar dashboards, menções, perfis e comparativos.
4. O `docs` centraliza memória operacional, relatórios, runbooks, estado da infraestrutura e procedimentos de deploy.
5. O `eleicoes2026.gitops` permanece como referência do ambiente AWS legado/contingência enquanto houver necessidade de rollback curto.

## Repositórios

### `docs`

Repositório de documentação viva do projeto.

Contém:

- estado da operação e da stack
- relatórios operacionais
- inventário das VPS
- plano de deploy para VPS via GitHub Actions
- runbooks de acesso, logs, banco, backup, restore e troubleshooting
- procedimentos de handover e aceite

Link: [github.com/Eleicoes2026/docs](https://github.com/Eleicoes2026/docs)

### `eleicoes2026.backend`

Backend principal em Spring Boot responsável por autenticação, API REST, integração com Solr/PostgreSQL e endpoints analíticos.

Contém:

- autenticação JWT
- endpoints de candidatos, menções, sentimentos, tags, word cloud e perfis
- importação de CSV
- workflow manual de deploy para Hostinger com backup pré-update

Link: [github.com/Eleicoes2026/eleicoes2026.backend](https://github.com/Eleicoes2026/eleicoes2026.backend)

### `eleicoes2026.brandwatch.collector`

Serviço de coleta e processamento assíncrono das menções.

Contém:

- schedulers por campanha/query
- integração com Brandwatch
- persistência em PostgreSQL
- indexação em Solr
- processamento em lote com Anthropic
- workflow manual de deploy para Hostinger com backup pré-update

Link: [github.com/Eleicoes2026/eleicoes2026.brandwatch.collector](https://github.com/Eleicoes2026/eleicoes2026.brandwatch.collector)

### `eleicoes2026.frontend`

Aplicação Angular do painel analítico.

Contém:

- overview executivo
- perfil de candidato
- quadro de menções
- comparativo entre candidatos
- autenticação e navegação protegida
- workflow manual de deploy para Hostinger com backup pré-update

Link: [github.com/Eleicoes2026/eleicoes2026.frontend](https://github.com/Eleicoes2026/eleicoes2026.frontend)

### `eleicoes2026.gitops`

Infraestrutura declarativa e operação do ambiente AWS legado/contingência.

Contém:

- aplicações ArgoCD
- overlays Kustomize
- pipelines Tekton
- manifests de backend, frontend, coletor e monitoring
- automação de renovação de secret do ECR

Link: [github.com/Eleicoes2026/eleicoes2026.gitops](https://github.com/Eleicoes2026/eleicoes2026.gitops)

## Operação em Hostinger

### Acesso às VPS

O acesso operacional é feito por SSH para as VPS oficiais:

```
ssh <usuario>@31.97.162.229
ssh <usuario>@31.97.166.138
ssh <usuario>@31.97.163.74
```

Função de cada VPS:

| VPS | IP | Função |
|---|---:|---|
| VPS 1 | `31.97.162.229` | Aplicação, containers, Apache, TLS e Solr |
| VPS 2 | `31.97.166.138` | Banco PostgreSQL oficial |
| VPS 3 | `31.97.163.74` | Destino de backups |

As credenciais, chaves privadas, senhas e secrets não devem ser publicadas nos repositórios. Use apenas canais seguros, GitHub Actions Secrets ou cofre de senhas autorizado.

### Logs das aplicações

Na VPS de aplicação:

```
ssh <usuario>@31.97.162.229
cd /opt/<repo>
docker compose ps
docker compose logs -f --tail=200 <service>
```

Serviços esperados:

- `eleicoes-backend`
- `eleicoes-brandwatch`
- `eleicoes-frontend`
- `eleicoes-solr`

Exemplos:

```
cd /opt/eleicoes2026.backend
docker compose logs -f --tail=200 eleicoes-backend
```

```
cd /opt/eleicoes2026.brandwatch.collector
docker compose logs -f --tail=200 eleicoes-brandwatch
```

```
cd /opt/eleicoes2026.frontend
docker compose logs -f --tail=200 eleicoes-frontend
```

Para consultar o status geral dos containers:

```
docker ps
docker compose ps
```

Para visualizar logs recentes sem acompanhar em tempo real:

```
docker compose logs --tail=200 <service>
```

### Banco de dados

O PostgreSQL oficial roda nativamente na VPS de banco:

```
ssh <usuario>@31.97.166.138
psql -h localhost -U <usuario_banco> -d <nome_banco>
```

Para acesso local com túnel SSH:

```
ssh -L 5432:localhost:5432 <usuario>@31.97.166.138
```

Depois, em outro terminal:

```
psql -h localhost -p 5432 -U <usuario_banco> -d <nome_banco>
```

Boas práticas:

- não expor a porta do PostgreSQL publicamente sem necessidade
- não versionar strings de conexão
- usar usuário de aplicação com permissões limitadas
- usar usuário administrativo somente para manutenção controlada
- registrar alterações estruturais de banco no repositório correspondente
- realizar backup antes de alterações operacionais relevantes

Comandos úteis no PostgreSQL:

```
\l
\c <nome_banco>
\dt
\du
```

Consulta simples de saúde:

```
SELECT now();
```

Tamanho do banco:

```
SELECT pg_size_pretty(pg_database_size(current_database()));
```

### Solr

O Solr roda localmente na VPS de aplicação.

Uso esperado:

- backend consulta Solr local via configuração do compose
- collector indexa dados no Solr local
- acesso externo direto ao Solr deve permanecer restrito

Para verificar container:

```
ssh <usuario>@31.97.162.229
docker ps | grep solr
```

Para consultar logs:

```
cd /opt/<repo>
docker compose logs -f --tail=200 eleicoes-solr
```

### Reinício de aplicação em crash

Na VPS de aplicação:

```
ssh <usuario>@31.97.162.229
cd /opt/<repo>
docker compose ps
docker compose logs --tail=200 <service>
docker compose restart <service>
```

Se for necessário reconstruir a imagem:

```
docker compose up -d --build <service>
docker compose logs -f --tail=200 <service>
```

Fluxo recomendado para incidente:

1. acessar a VPS de aplicação
2. identificar container com falha
3. coletar logs recentes
4. verificar uso de CPU, memória e disco
5. reiniciar somente o serviço afetado
6. acompanhar logs após o restart
7. validar endpoint ou tela pública relacionada
8. registrar a ocorrência no repositório `docs` se houver impacto operacional

Comandos auxiliares:

```
docker ps
docker ps -a
docker stats
df -h
free -h
uptime
```

Ver logs anteriores de um container reiniciado:

```
docker compose logs --tail=500 <service>
```

### Deploy

Os repositórios `backend`, `frontend` e `brandwatch.collector` possuem workflow manual de deploy para Hostinger via GitHub Actions.

Arquivo de workflow em cada repositório:

```
.github/workflows/deploy-hostinger.yml
```

Modelo operacional:

- execução manual por `workflow_dispatch`
- validação de build
- conexão SSH na VPS de aplicação
- backup automático antes da atualização
- sincronização do repositório para `/opt/<repo>`
- execução de `docker compose up -d --build`
- smoke test local na própria VPS

Passo a passo:

1. Entrar no repositório do serviço.
2. Abrir a aba `Actions`.
3. Selecionar o workflow `Deploy ... to Hostinger`.
4. Clicar em `Run workflow`.
5. Informar `ref`, normalmente `main`.
6. Informar `reason`, descrevendo o motivo do deploy.
7. Acompanhar build, backup, sync, compose e smoke test.
8. Validar logs e saúde do serviço após o deploy.

Ordem recomendada em deploys coordenados:

1. backend
2. collector
3. frontend

Essa ordem pode mudar quando a alteração for exclusiva de frontend ou exclusiva de collector.

Antes do deploy:

- revisar alterações do commit
- confirmar branch/ref correta
- verificar se não há incidente ativo
- garantir que o backup pré-update será executado
- conferir se secrets do GitHub Actions estão configurados

Depois do deploy:

- verificar conclusão com sucesso do workflow
- acessar VPS e validar containers
- conferir logs do serviço alterado
- executar smoke test funcional
- registrar evidências se for deploy relevante

Comandos pós-deploy:

```
ssh <usuario>@31.97.162.229
docker ps
cd /opt/<repo>
docker compose ps
docker compose logs --tail=200 <service>
```

## Backups

A rotina de backup está preparada em três camadas:

- backup pré-update antes de cada deploy
- backup periódico automático da aplicação
- backup periódico automático do banco

Script principal:

```
/usr/local/bin/eleicoes2026-backup-before-update.sh
```

Timers operacionais:

- VPS 1: `eleicoes2026-backup-app.timer`, recorrência a cada 3 dias
- VPS 2: `eleicoes2026-backup-db.timer`, recorrência a cada 3 dias

Retenção:

- origem local nas VPS 1 e 2: 14 dias
- destino na VPS 3: 45 dias

Destino de backups:

```
/srv/backups/eleicoes2026
```

Para consultar timers:

```
systemctl list-timers | grep eleicoes2026
```

Para consultar status do timer de aplicação:

```
systemctl status eleicoes2026-backup-app.timer
```

Para consultar status do timer de banco:

```
systemctl status eleicoes2026-backup-db.timer
```

Para listar backups na VPS de backup:

```
ssh <usuario>@31.97.163.74
ls -lah /srv/backups/eleicoes2026
```

## Segurança operacional

Regras obrigatórias:

- não versionar senhas
- não versionar chaves privadas
- não publicar tokens no README, issues, commits ou workflows
- usar GitHub Actions Secrets para secrets de pipeline
- rotacionar credenciais após handover ou exposição
- limitar acesso SSH ao menor grupo possível
- manter backups antes de alterações relevantes
- evitar acesso direto ao banco fora de janelas controladas
- registrar incidentes e mudanças operacionais relevantes no `docs`

Credenciais sensíveis devem ficar apenas em:

- cofre de senhas autorizado
- GitHub Actions Secrets
- canal seguro de handover
- ferramenta corporativa de gestão de segredos

## Convenções

- cada repositório possui ciclo de vida próprio
- configurações sensíveis devem ser fornecidas por variáveis de ambiente, secrets ou cofre autorizado
- credenciais nunca devem ser versionadas
- deploys de aplicação devem passar por backup pré-update
- alterações operacionais relevantes devem atualizar o repositório `docs`
- AWS deve ser tratada como ambiente legado/contingência até encerramento definitivo
- após aceite final, chaves, senhas e tokens usados na migração devem ser rotacionados

## Checklist operacional rápido

### Acessar VPS

```
ssh <usuario>@31.97.162.229
ssh <usuario>@31.97.166.138
ssh <usuario>@31.97.163.74
```

### Ver containers

```
docker ps
```

### Ver logs

```
cd /opt/<repo>
docker compose logs -f --tail=200 <service>
```

### Reiniciar serviço

```
cd /opt/<repo>
docker compose restart <service>
```

### Rebuild de serviço

```
cd /opt/<repo>
docker compose up -d --build <service>
```

### Acessar banco

```
ssh <usuario>@31.97.166.138
psql -h localhost -U <usuario_banco> -d <nome_banco>
```

### Deploy

```
GitHub -> repositório -> Actions -> Deploy ... to Hostinger -> Run workflow
```

## Pendências pós-cutover

- executar e registrar sucesso final dos pipelines de backend, frontend e brandwatch
- validar fluxo funcional com usuário real
- confirmar monitoramento e alertas mínimos de serviços, disco, banco e SSL
- formalizar aceite final do cliente
- desligar AWS por etapas com plano de rollback curto
- rotacionar credenciais sensíveis após aceite final
