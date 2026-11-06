# Matriz de Auditoria (122 conceitos)

Use esta matriz no passo 3 do processo: selecione **somente** os itens aplicáveis ao material enviado.

**Legenda:** os códigos (`[A01]`, `[G112]`...) seguem a numeração original dos conceitos e por isso não são sequenciais. Itens marcados com 🏗️ são conceitos de **arquitetura/infraestrutura**: só cite quando o usuário enviar descrição de arquitetura, Dockerfile, configuração de deploy, ou pedir uma revisão geral. Em um trecho de código isolado, ignore-os.

**Índice:** [A. Borda e Tráfego](#a-escudo-de-entrada-tráfego-e-borda) · [B. Integrações e Tempo Real](#b-integrações-externas-rede-e-tempo-real) · [C. Assincronismo e Distribuídos](#c-assincronismo-filas-e-sistemas-distribuídos) · [D. Banco, Concorrência e Memória](#d-banco-de-dados-concorrência-e-memória) · [E. Segurança](#e-segurança-identidade-e-criptografia-zero-trust) · [F. Infra, Deploy e Operação](#f-infraestrutura-deploy-observabilidade-e-operação) · [G. Pontos cegos de Vibe Coding](#g-camada-de-aplicação-e-pontos-cegos-de-vibe-coding)

---

## A. Escudo de Entrada, Tráfego e Borda

- **[A01] Rate Limiting:** limite de requisições por IP/usuário em rotas públicas, de autenticação e em APIs pagas (LLMs).
- **[A02] Cache de Aplicação:** cache em memória/Redis com TTL adequado para leituras repetitivas.
- 🏗️ **[A03] Balanceamento de Carga:** aplicações stateless prontas para distribuição horizontal de tráfego.
- **[A04] Proxies Reversos:** proteção do IP/porta de origem via Cloudflare/Nginx e cabeçalhos `X-Forwarded-For` confiáveis.
- 🏗️ **[A05] API Gateways:** autenticação, roteamento e políticas de tráfego centralizados na entrada.
- **[A48] CDN:** ativos estáticos e mídia entregues por rede de distribuição global.
- **[A49] Edge Caching:** `Cache-Control` configurado para cache na borda, perto do usuário.
- **[A50] Invalidação de Cache:** estratégia clara de purge ou `stale-while-revalidate` para evitar dados obsoletos.

## B. Integrações Externas, Rede e Tempo Real

- **[B10] Circuit Breakers:** corte automático de chamadas a APIs externas degradadas.
- **[B11] Timeouts:** tempo limite explícito (ex.: 5s a 8s) em toda chamada de rede/HTTP externa.
- **[B12] Retries:** nova tentativa controlada, apenas para falhas transitórias de rede ou 5xx.
- **[B13] Backoff Exponencial:** espera progressiva com Jitter entre tentativas.
- **[B14] Idempotência:** chave única (`Idempotency-Key`) em pagamentos, webhooks e mutações críticas.
- **[B22] WebSockets:** canal bidirecional persistente apenas quando houver necessidade real de baixa latência mútua.
- **[B23] Long Polling:** alternativa controlada ao polling agressivo quando WebSockets/SSE não são viáveis.
- **[B24] Server-Sent Events (SSE):** streaming unidirecional (respostas de IA, notificações) no lugar de `setInterval` + polling.
- 🏗️ **[B91] Latência:** redução de round-trips desnecessários na rede.
- **[B92] Throughput:** vazão não bloqueante em rotas de alto volume.
- 🏗️ **[B93] Latência P99:** eliminação de gargalos que afetam o 1% de requisições mais pesadas.
- 🏗️ **[B94] Latência de Cauda (Tail Latency):** mitigação de respostas lentas em cascata em chamadas distribuídas.
- 🏗️ **[B95] Partições de Rede:** tolerância a falhas parciais de comunicação entre serviços.
- **[B96] Clock Skew:** timestamps UTC centralizados (banco/servidor) em vez do relógio local do cliente ou de nós isolados, para ordenar eventos críticos.
- 🏗️ **[B97] DNS:** resolução eficiente e cache de DNS em chamadas recorrentes.
- 🏗️ **[B98] TCP vs UDP:** escolha do protocolo de transporte conforme a necessidade de confiabilidade vs velocidade.
- 🏗️ **[B99] HTTP/2 e HTTP/3:** multiplexação e reaproveitamento de conexões Keep-Alive.
- 🏗️ **[B100] gRPC:** comunicação binária tipada de alta performance entre microsserviços internos.
- **[B101] Webhooks:** recepção orientada a eventos (no lugar de polling), com validação obrigatória de assinatura criptográfica (HMAC) e resposta rápida.

## C. Assincronismo, Filas e Sistemas Distribuídos

- **[C15] Filas de Mensagens:** tarefas pesadas (e-mail, PDF, IA, vídeo) fora da thread HTTP principal (BullMQ, SQS, RabbitMQ).
- 🏗️ **[C16] Pub/Sub:** distribuição assíncrona de mensagens para múltiplos consumidores independentes.
- 🏗️ **[C17] Arquitetura Orientada a Eventos:** domínios de negócio desacoplados por emissão de eventos.
- 🏗️ **[C18] Transações Distribuídas:** coordenação segura de operações que cruzam múltiplos bancos ou serviços.
- 🏗️ **[C19] Padrão Saga:** ações de compensação (rollback lógico) quando uma etapa de um fluxo distribuído falha (ex.: pagamento + estoque).
- **[C20] Dead Letter Queues (DLQ):** isolamento e retenção de mensagens que falharam após todas as tentativas, para análise e reprocessamento.
- **[C21] Cron Jobs:** tarefas agendadas idempotentes e com trava de execução concorrente.

## D. Banco de Dados, Concorrência e Memória

- **[D25] Índices no Banco:** índices explícitos em colunas usadas em `WHERE`, `JOIN`, `ORDER BY` e chaves estrangeiras.
- **[D26] Otimização de Queries:** selecionar apenas as colunas necessárias e usar filtros eficientes.
- **[D27] Queries N+1:** eliminar queries dentro de loops (`for`/`map`) com `JOIN`, `include` ou `dataloader`/batching.
- **[D28] Pool de Conexões:** Connection Pooling (PgBouncer/Supavisor) e instância singleton do cliente de banco em Serverless.
- 🏗️ **[D29] Réplicas de Leitura:** separar tráfego intenso de leitura do banco primário de escrita.
- 🏗️ **[D30] Sharding:** distribuição horizontal de dados entre múltiplos servidores de banco quando necessário.
- 🏗️ **[D31] Particionamento:** divisão de tabelas massivas (por data ou região) para manter índices leves.
- 🏗️ **[D32] Replicação:** sincronização de dados entre nós para alta disponibilidade.
- 🏗️ **[D33] Eleição de Líder:** failover automático do banco primário sem split-brain.
- 🏗️ **[D34] Teorema CAP:** decisão consciente entre consistência imediata e disponibilidade em cenários de partição.
- 🏗️ **[D35] Consistência Eventual:** aceitar e tratar atraso de propagação em leituras não críticas.
- **[D36] Lock Otimista:** controle de concorrência por coluna de versão (`version`) ou timestamp antes do update.
- **[D37] Lock Pessimista:** bloqueio de linha (`SELECT FOR UPDATE`) em operações financeiras ou de estoque.
- 🏗️ **[D38] Locks Distribuídos:** exclusão mútua entre múltiplos servidores (ex.: Redlock/Redis).
- **[D39] Condições de Corrida (Race Conditions):** evitar leituras e escritas concorrentes sem atomicidade (ex.: saldo ou estoque negativo).
- **[D40] Deadlocks:** ordem consistente de aquisição de locks e transações curtas.
- **[D41] Vazamento de Memória (Memory Leak):** limpar `EventListeners`, timers (`setInterval`/`setTimeout`), subscriptions, closures e caches globais sem limite, no front-end (retorno do `useEffect`) e no back-end.
- **[D42] Coleta de Lixo (Garbage Collection):** evitar alocação massiva de objetos temporários que travem o Event Loop.
- **[D43] Thread Safety:** proteger estado compartilhado mutável em execuções concorrentes/assíncronas.
- **[D44] Backpressure:** arquivos e payloads grandes via Streams controlados, sem carregar tudo na RAM.
- **[D81] Migrations de Banco:** evolução rastreável e reversível via arquivos de migration versionados (proibido alterar schema manualmente em produção).
- **[D82] Versionamento de Schema:** compatibilidade retroativa em mudanças de colunas/tabelas sem derrubar o app em produção.

## E. Segurança, Identidade e Criptografia (Zero Trust)

- **[E67] Gestão de Segredos:** proibição absoluta de chaves/senhas hardcoded ou expostas no bundle do front-end (`NEXT_PUBLIC_` indevido); usar variáveis de ambiente.
- **[E68] IAM (Controle de Acesso):** menor privilégio para permissões de serviços, buckets e bancos.
- **[E69] OAuth:** autenticação delegada com validação de `state` e PKCE.
- **[E70] Rotação de JWT:** tokens de acesso de curta duração com Refresh Tokens rotativos em cookies `httpOnly`, `Secure` e `SameSite`.
- **[E71] TLS:** comunicação estritamente via HTTPS/TLS atualizado.
- **[E72] Criptografia em Repouso:** hashing forte de senhas (Argon2/bcrypt) e criptografia de dados sensíveis armazenados.
- **[E73] Criptografia em Trânsito:** proteção do tráfego externo e interno entre serviços.
- 🏗️ **[E74] WAF:** filtragem de payloads maliciosos na camada de borda.
- 🏗️ **[E75] Proteção contra DDoS:** mitigação de exaustão de recursos em rotas caras.
- **[E76] CORS:** proibido `Access-Control-Allow-Origin: *` em rotas autenticadas; lista branca estrita de domínios.
- **[E77] CSRF:** tokens anti-CSRF ou políticas estritas de cookies em mutações baseadas em sessão.
- **[E78] SQL Injection:** queries parametrizadas ou Prepared Statements (proibido concatenar strings em SQL bruto).
- **[E79] XSS (Cross-Site Scripting):** sanitizar HTML/inputs; proibido `dangerouslySetInnerHTML`/`innerHTML` sem sanitização.
- **[E80] SSRF (Server-Side Request Forgery):** validar URLs fornecidas pelo usuário e bloquear chamadas a IPs privados e metadados de nuvem (`169.254.169.254`, `127.0.0.1`).

## F. Infraestrutura, Deploy, Observabilidade e Operação

- **[F06] CI/CD:** lint, testes e build automatizados antes do deploy.
- **[F07] Docker:** containers leves, multi-stage builds e execução com usuário não-root.
- 🏗️ **[F08] Kubernetes:** orquestração resiliente de containers quando a escala exigir.
- 🏗️ **[F09] Descoberta de Serviços:** resolução dinâmica de endereços internos, sem IPs hardcoded.
- 🏗️ **[F45] Autoscaling:** escalar instâncias automaticamente conforme CPU/memória/fila.
- **[F46] Escala Horizontal:** código stateless (sem sessão ou arquivos no disco local da instância).
- 🏗️ **[F47] Escala Vertical:** dimensionamento adequado de recursos para cargas intensivas em CPU/RAM.
- **[F51] Feature Flags:** separar deploy de liberação de funcionalidades críticas, com desligamento instantâneo.
- 🏗️ **[F52] Deploy Blue-Green:** troca segura de tráfego entre ambientes idênticos.
- 🏗️ **[F53] Releases Canário:** liberação gradual para uma fração dos usuários antes do rollout total.
- 🏗️ **[F54] Rolling Deploys:** atualização progressiva de instâncias sem downtime.
- **[F55] Rollbacks:** capacidade imediata de reverter código e configuração sem quebrar dados.
- **[F56] Health Checks:** rota `/health` verificando conectividade real com dependências vitais.
- **[F57] Liveness e Readiness Probes:** separar checagem de travamento (reiniciar) de prontidão para receber tráfego.
- 🏗️ **[F58] Monitoramento:** acompanhamento contínuo da saúde técnica e de negócio.
- **[F59] Logs:** logs estruturados em JSON (Pino/Winston) com níveis adequados, no lugar de `console.log` soltos.
- **[F60] Rastreamento Distribuído (Tracing):** propagar `correlationId`/`traceId` entre chamadas e serviços (OpenTelemetry).
- **[F61] Métricas:** coleta de uso, latência e taxa de erro.
- **[F62] Alertas:** notificações acionáveis baseadas em sintomas reais de degradação.
- 🏗️ **[F63] SLOs:** metas claras de nível de serviço (ex.: 99,9% de disponibilidade).
- 🏗️ **[F64] SLIs:** medição real das métricas que compõem a meta (latência, taxa de sucesso).
- 🏗️ **[F65] Error Budgets:** controle de risco de novos deploys conforme a estabilidade recente.
- 🏗️ **[F66] Observabilidade:** união de logs, métricas e traces para depurar falhas desconhecidas.
- 🏗️ **[F83] Recuperação de Desastres:** plano e capacidade de restaurar toda a operação após falha catastrófica.
- 🏗️ **[F84] Backups:** rotinas automatizadas com verificação periódica de restauro (restore test).
- 🏗️ **[F85] Failover:** redirecionamento automático para instâncias/serviços secundários em caso de queda.
- 🏗️ **[F86] Deploy Multirregião:** redundância geográfica para sistemas de missão crítica.
- 🏗️ **[F87] Engenharia do Caos:** resiliência validada contra falhas injetadas ou queda de dependências.
- **[F88] Otimização de Custos (FinOps):** prevenir loops infinitos, chamadas redundantes a API paga e recursos ociosos.
- **[F89] Cold Starts:** reduzir o tamanho do bundle e as dependências pesadas no boot de funções Serverless.
- **[F90] Limites do Serverless:** respeitar tetos de tempo de execução, tamanho de payload e memória (Lambda/Vercel).
- **[F102] Versionamento de API:** prefixo e contrato estável de rotas (`/api/v1`) para não quebrar clientes ativos.
- **[F103] Versionamento Semântico (SemVer):** controle de atualizações de pacotes, evitando breaking changes automáticas (versões fixadas).
- 🏗️ **[F104] Infraestrutura como Código (IaC):** recursos de nuvem declarados e versionados em código.
- 🏗️ **[F105] Terraform:** provisionamento reprodutível e auditável de infraestrutura.
- 🏗️ **[F106] Helm Charts:** empacotamento padronizado de manifestos Kubernetes.
- **[F107] Cache de Build:** ordem otimizada das camadas no Dockerfile e cache de dependências no CI.
- **[F108] Inferno de Dependências:** remover bibliotecas redundantes, pesadas ou conflitantes geradas por IA.
- 🏗️ **[F109] Incidentes em Produção:** contenção rápida de impacto e isolamento de falhas.
- 🏗️ **[F110] Plantão (On-Call):** código e logs claros o bastante para diagnóstico rápido sob pressão.
- 🏗️ **[F111] Postmortems:** documentação de causa raiz sem culpa (Blameless), acompanhada de teste automatizado preventivo.

## G. Camada de Aplicação e Pontos Cegos de Vibe Coding

- **[G112] BOLA / IDOR (Broken Object Level Authorization):** toda leitura, edição ou exclusão de registro deve filtrar pelo `userId`/`tenantId` da **sessão autenticada no servidor**, nunca confiando só no `id` enviado na URL ou no body.
- **[G113] Validação de Schema na Entrada:** validação estrita de tipos, tamanhos e formatos no início da rota/função (Zod, Pydantic, Joi, Valibot). Nunca confiar em `req.body`.
- **[G114] Mass Assignment (Atribuição em Massa):** proibido passar `req.body` direto para criação/atualização do ORM (ex.: `data: req.body` permite `{"role":"admin","saldo":99999}`); selecionar explicitamente os campos permitidos.
- **[G115] Segurança de LLMs (Prompt Injection & Token Caps):** separação estrita entre instruções de sistema e input do usuário, validação/sanitização da saída e `max_tokens` e limites de gasto por usuário obrigatórios.
- **[G116] Paginação Obrigatória:** proibidas buscas abertas (`findMany()`/`SELECT` sem `LIMIT`); exigir paginação por cursor ou offset com teto máximo por página.
- **[G117] Segurança de Upload e Presigned URLs:** validar o MIME type real (magic bytes), limitar tamanho, renomear com UUID e enviar direto ao storage (S3/R2) via Presigned URLs, sem passar o binário pelo servidor.
- **[G118] Graceful Shutdown:** capturar `SIGTERM`/`SIGINT` para encerrar requisições em andamento e fechar conexões de banco/fila antes de finalizar o processo.
- **[G119] Código Alucinado e Erros Engolidos:** remover pacotes/métodos inexistentes inventados pela IA, funções duplicadas, código morto e blocos `catch (e) {}` vazios que ocultam falhas.
- **[G120] Tratamento Global de Erros:** erros HTTP padronizados, sem jamais vazar Stack Traces, estrutura de tabelas ou variáveis internas ao cliente em produção.
- **[G121] Privacidade por Design (LGPD/GDPR e PII Masking):** mascarar dados sensíveis (CPF, cartão, e-mail, senhas, tokens) nos logs e suportar exclusão/anonimização de conta (Soft Delete vs Hard Delete planejado).
- **[G122] Validação de Ambiente no Boot (Fail-Fast `.env`):** validação tipada de todas as variáveis obrigatórias na inicialização; falhar imediatamente se faltar configuração crítica.
