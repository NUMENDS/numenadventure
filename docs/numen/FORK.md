# Numen Adventure — fork e sync com WorkAdventure

Este repositório é um fork de [workadventure/workadventure](https://github.com/workadventure/workadventure) mantido pela Numen para uso interno (IAM, branding, maps, integrações).

## Remotes

| Remote | URL | Uso |
|--------|-----|-----|
| `origin` | `https://github.com/NUMENDS/numenadventure.git` | Repositório Numen |
| `upstream` | `https://github.com/workadventure/workadventure.git` | Projeto original |

Configuração inicial (já feita no clone da Numen):

```bash
git remote add upstream https://github.com/workadventure/workadventure.git
git fetch upstream
```

## Branches

| Branch | Papel |
|--------|--------|
| `main` | Produto Numen (default). Trabalho do time, customizações e deploy. |
| `upstream-sync` | Espelho limpo do `master` do WorkAdventure. **Nunca** recebe commits Numen. |
| `feature/*` | Features do time; abrir PR contra `main`. |

A branch `master` legada pode permanecer no remoto por um tempo, mas o fluxo oficial é `main`.

## Fluxo de sync (ports com agente)

O objetivo é **portar seletivamente** melhorias do WorkAdventure, não fazer merge cego de tudo.

### 1. Atualizar o espelho upstream

```bash
git fetch upstream
git checkout upstream-sync
git reset --hard upstream/master
git push origin upstream-sync --force-with-lease
```

`--force-with-lease` é esperado nesta branch: ela só espelha o upstream.

### 2. Ver o que mudou desde a última base Numen

```bash
git checkout main
git log --oneline main..upstream-sync
git diff main...upstream-sync
```

### 3. Portar para `main` via PR

1. Criar branch `sync/wa-YYYY-MM-DD` a partir de `main`.
2. Revisar o diff (humano ou agente) e aplicar só o que faz sentido para a Numen.
3. Abrir PR contra `main`.
4. Resolver conflitos nas áreas Numen (IAM, branding, maps, `contrib/numen/`).

### 4. Registrar o sync

No final do PR de sync, anotar o commit do WorkAdventure sincronizado:

```bash
git rev-parse upstream-sync
```

Atualize a seção **Último sync** abaixo.

## Último sync

| Data | Commit WA (`upstream-sync`) | PR Numen | Notas |
|------|----------------------------|----------|-------|
| 2026-08-19 | _(setup inicial — `main` e `upstream-sync` criados no mesmo ponto do WA)_ | — | Fork organizado |

## Regras para customizar sem travar ports futuros

Ordem de preferência (da mais fácil de manter à mais custosa):

1. **Config / env** — IAM via OpenID nativo do WA ([openid.md](../others/self-hosting/openid.md)): variáveis `OPENID_*`, secrets, URLs.
2. **Artefatos Numen** — maps, assets, overlays Docker Compose, pasta `contrib/numen/`.
3. **Adapters finos** — wrappers ou módulos isolados (`*-numen`), sem espalhar lógica no core.
4. **Core WA** — evitar editar Phaser/Svelte/pusher em massa. Se inevitável, commits pequenos com prefixo `numen:` para facilitar diff e ports.

## Proteção de branches

- **`main`**: PR obrigatório; sem force-push.
- **`upstream-sync`**: force-push permitido apenas para atualizar o espelho do upstream.

## Prompt sugerido para agente (port de sync)

Use após atualizar `upstream-sync`:

```
Analise o diff entre main e upstream-sync neste fork Numen Adventure.
Liste commits/mudanças do WorkAdventure que valem port (bugfix, security, perf).
Ignore o que conflita com customizações Numen ou não se aplica ao nosso deploy.
Proponha um PR sync/wa-YYYY-MM-DD com os patches mínimos.
```

## Referências

- [WorkAdventure — upgrade guide](../../UPGRADE.md)
- [OpenID self-hosting](../others/self-hosting/openid.md)
- [Upstream original](https://github.com/workadventure/workadventure)
