# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é

SECRE·TINA — PWA de finanças pessoais. Cada página é um arquivo HTML autocontido
(HTML + CSS + JS inline, sem build step, sem framework). Não há `package.json`
nem bundler — os arquivos são servidos como estão.

## Rodar localmente

```
python3 -m http.server 8080
```
Abrir `http://localhost:8080/index.html`.

## Deploy

GitHub Pages (build "legacy") serve a raiz da branch `main` diretamente —
`git push` para `main` já publica em `secretina.barukstore.com.br` (domínio
custom definido em `CNAME`). Não existe pipeline de build: o que está em
`main` é exatamente o que vai ao ar.

## Páginas e fluxo

| Arquivo | Papel |
|---|---|
| `index.html` | Login/cadastro. Ao autenticar, redireciona para `secretina-dashboard.html` |
| `secretina-onboarding.html` | Configuração inicial (grupos, orçamento, cartões) — só na primeira vez |
| `secretina-dashboard.html` | App principal (SPA de uma página só, ~5600 linhas) |
| `secretina-importar.html` | Importação de extrato (usa PDF.js) |
| `secretina-ajuda.html` | Central de ajuda estática |

Redirecionamento entre páginas é feito via `window.location.href` (não há
router). **Atenção**: `secretina-onboarding.html` e `secretina-importar.html`
redirecionam para `secretina-login-firebase.html` quando não há usuário
autenticado — esse arquivo não existe no repo (deveria ser `index.html`); é um
bug pré-existente, não um arquivo que falta criar.

## Arquitetura do dashboard (`secretina-dashboard.html`)

- Cada seção do app é uma `<div class="page" id="page-<nome>">` (`home`,
  `financeiro`, `cartoes`, `agenda`, `investimentos`, `evolucao`, `relatorios`,
  `config`, `mais`). Troca de tela é só CSS (`.page.active`), sem re-render de
  DOM completo.
- `window.goPage(name)` (por volta da linha 2059) faz a troca: remove `.active`
  de todas as `.page`/`.nav-btn`, ativa a página/botão alvo e dispara a função
  `render*` correspondente daquela seção (ex.: `renderFinanceiro()`,
  `renderCartoes()`, `renderAgenda()`, `renderInvestimentos()`,
  `renderEvolucaoFinanceira()`). Ao adicionar uma seção nova, registrar o
  side-effect nesse switch.
- Estado em memória fica em `window.STATE` (objeto global único: `grupos`,
  `orcamento`, `cartoes`, `tickets`, `agendaTab`, `viewMonth`, etc.), carregado
  do Firestore no boot e mantido sincronizado nas ações do usuário.
- Funções chamadas via `onclick` inline são penduradas em `window.<nome>`
  (ex.: `window.doLogin`, `window.toggleTheme`) — é o padrão do projeto, não
  uma escolha a "corrigir".

## Firebase

Projeto `secretina-1d89a`, Firebase JS SDK 10.12.0 via CDN (`gstatic.com`),
sem Firebase Hosting (o hosting é GitHub Pages) — `firebaseConfig` fica inline
em cada HTML. Auth por e-mail/senha (`browserLocalPersistence`).

Estrutura Firestore (tudo sob `users/{uid}`):
- `perfil/dados`
- `config/{grupos, orcamento, cartoes, contasFixas, ief, iefConquistas,
  iefHistorico, perfil, saldoInicial, tickets}` — um doc por chave de
  configuração, não um doc único
- `onboarding/progress`
- `lancamentos/{id}`, `agenda/{id}`, `investimentos/{id}` — subcoleções

Não há `firestore.rules` neste repo — regras de segurança são geridas fora
daqui (console/CLI do Firebase), não versionadas.

## localStorage

`secretina_onboarding` (flag de onboarding concluído), `secretina_theme`
(dark/light), `tut_visto` (tutorial já visto).

## icones-fonte/

PNGs de origem dos ícones do PWA (não usados em runtime — os ícones
efetivamente servidos ficam na raiz: `favicon.png`, `apple-touch-icon.png`,
`icon-192.png`, `icon-512.png`).
