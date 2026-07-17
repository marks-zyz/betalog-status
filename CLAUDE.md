# ⛔️⛔️⛔️ ESTE REPOSITÓRIO É PÚBLICO. LEIA ANTES DE TOCAR EM QUALQUER COISA. ⛔️⛔️⛔️

> **`github.com/marks-zyz/betalog-status` é um repositório PÚBLICO**, servido em
> **`https://status.betalog.com.br`** por GitHub Pages. TUDO aqui é **world-readable**: o código, os
> commits, o histórico inteiro, cada arquivo de dado. Qualquer pessoa na internet lê. Trate como um
> outdoor na rua.

## 🚨 REGRA DURA Nº 1 — ZERO SEGREDO. NADA QUE NÃO POSSA SER PÚBLICO.
É **PROIBIDO** commitar/versionar aqui, em qualquer arquivo (código, JSON de dado, workflow, comentário):
- **Secrets/chaves/tokens** de QUALQUER tipo (Cloudflare, SES, Gemini, Wordfence, GitHub PAT, push-key,
  agent-key, session-secret, senha de beta). Nunca. Nem "só pra testar".
- **URLs/hosts internos** não-públicos, IDs de recurso (D1 database id, account id, KV/R2 ids, worker
  names internos), caminhos de infra que revelem a topologia privada.
- **Dado de cliente/tenant**: nome de site/servidor de cliente, e-mail, IP, domínio de tenant, métrica
  por-conta, qualquer PII. Os arquivos de dado (`status.json`, `uptime-history.json`,
  `panel-history.json`) carregam SÓ agregado anônimo da plataforma — a MESMA garantia do endpoint
  público `/status-feed.json` (ver o worker). Se algo aqui identifica um cliente, é bug de segurança.
- **Detalhe de arquitetura interna** que ajude um atacante (nomes de tabela, rotas privadas, lógica de
  auth). O que aparece aqui é a superfície pública, não o mapa da mina.

Se você precisa de um segredo pra alguma automação deste repo, use **GitHub Actions Secrets** (cofre do
repo, nunca no código) — e ainda assim, só se for estritamente necessário e não vazar em log.

## 🚨 REGRA DURA Nº 2 — HONESTIDADE DO REGISTRO PÚBLICO.
Esta é uma **status page**: uptime e incidentes são um **registro de confiabilidade** que clientes e
prospects leem como verdade. É **PROIBIDO** publicar dado fabricado como genuíno: não pré-popular a
barra de 90 dias com uptime inventado, não criar incidentes falsos que fiquem no ar como reais. Preview
pra ver o visual = só no navegador (não commitado) ou claramente rotulado como demo. Dúvida = não publica.

## 🚨 REGRA DURA Nº 3 — SEMPRE LEIA ESTE ARQUIVO ANTES DE TRABALHAR AQUI.
Este repo é SEPARADO do monorepo do produto (`_Code_2/betalog/`, privado). Quem chega aqui vindo de lá
tem que trocar a cabeça: lá é privado, aqui é público. Reler as 3 regras acima a cada sessão que mexer aqui.

---

## O que este repositório é
A **status page pública** do BetaLog, hospedada **FORA da Cloudflare** de propósito (GitHub Pages +
GitHub Actions) — assim uma queda da própria Cloudflare não derruba junto o medidor de status.

- **`index.html`** — página estática única (HTML/CSS/JS inline, sem build, sem deps). Tema claro/escuro,
  DS "blueprint" do marketing (cantos com ticks em L, `--bp-radius` 3px, fundo de pontos/tracejado
  ESTREITO ~760px pra casar com o conteúdo de 720px). Busca o status assim:
  1. **Fetch DIRETO** de `https://app.betalog.com.br/status-feed.json` (tempo real, teto de 5s).
  2. Fallback: `./status.json` (snapshot commitado pelo Action).
  3. Fallback: último-bom em `localStorage`.
- **`.github/workflows/monitor.yml`** — o Action (a cada ~15-45min real; o GitHub estrangula o cron do
  free tier): baixa o feed → `status.json`; sonda `/health` → `uptime-history.json`; sonda o painel
  (`app.betalog.com.br/`) → `panel-history.json`; **dead-man** (2 falhas consecutivas de `/health` OU do
  painel → `exit 1` → e-mail nativo do GitHub, sem secret).
- **`status.json` / `uptime-history.json` / `panel-history.json`** — dados públicos agregados, escritos
  pelo Action. `deadman-state.json` = contador anti-flap.
- **`CNAME`** — `status.betalog.com.br` (GitHub Pages custom domain, DNS-only na Cloudflare).

## Como trabalhar aqui
- **Página estática**: editar `index.html` e dar push; o Pages reconstrói sozinho (~30-60s). Sem build.
- Validar visualmente em navegador (servir local: `python3 -m http.server` na pasta). Cuidado com CLS,
  tema claro/escuro, e a largura do fundo blueprint (conteúdo é estreito).
- **A lógica de negócio (o que cada componente significa, thresholds, feed) vive no WORKER** (repo
  privado `_Code_2/betalog/worker/src/index.ts`, endpoint `/status-feed.json`). Aqui é só a apresentação
  + o Action de fallback/dead-man. Mudança de contrato do feed → alinhar com o worker.
- Commit/push livres. Deploy = automático pelo Pages. Nunca há `wrangler`/secret aqui.
