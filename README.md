# Painel Geoplas · Meta Ads

Dashboard estático de performance com **filtro de data livre**, que se atualiza sozinho de hora em hora.

```
index.html   →  lê  →  data.json        (GitHub Pages serve os dois)
                        ▲
                        │ commit automático a cada hora
              .github/workflows/update-data.yml
                        │
              scripts/fetch-meta.mjs  →  Meta Marketing API
```

O HTML é 100% estático — nenhum servidor, nenhuma chave exposta no navegador. Quem chama a Meta é o **GitHub Actions**, usando um token guardado como *secret*.

---

## O que o painel mostra

| Campanha | Objetivo principal no painel | Métricas de apoio |
|---|---|---|
| **C1** · `CBO-Tráfego-A-VisitasAoPerfil-Filiais` | **Seguidores ganhos** e custo por seguidor | visitas ao perfil e custo por visita, cliques no link, % de visitas que viram seguidor |
| **C2** · `CBO-Reconhecimento-A-Visualização-Vídeo-Filiais` | **Pessoas que viram 50%+ do vídeo** e custo por visualização | ThruPlay e custo por ThruPlay, reproduções, % que chegou a 50 / 75 / 100% |
| **C3** · `CBO-Vendas-Quente-A-Depoimentos` | **Conversas iniciadas** e custo por conversa | conexões de mensagem, saudação vista, primeira resposta, responderam em 7 dias, trocaram 3+ e 5+ mensagens |

Em todas as abas, **CPM, CTR, CPC e frequência** ficam num painel discreto chamado *Diagnóstico de entrega* — servem para explicar o custo, não são a meta.

Cada campanha traz o **ranking de criativos**, ordenado do mais barato ao mais caro **no objetivo daquela campanha** (não no CTR), com selo `★ melhor` / `pior` e um comentário automático de realocação de verba. Criativos com menos de R$20 investidos no período recebem o selo `verba baixa` e ficam fora da comparação — pouco dado ainda não é sinal.

## Navegação e visual

Mesmo sistema de design do [painel Ricardo Mello](https://corvoassessoriatm.github.io/painel-ricardo-mello/): papel quente e ouro no claro, tinta no escuro, com **botão de tema** no topo (a escolha fica salva no navegador). Tipografia Fraunces nos títulos, Hanken Grotesk no texto e IBM Plex Mono nos números.

O painel tem **quatro abas**: *Visão geral* e uma por campanha. Cada aba de campanha traz o hero com o objetivo, oito KPIs, o **funil** daquela campanha (crescimento em C1, retenção de vídeo em C2, conversa em C3), a tendência diária e o ranking de criativos em tabela ou galeria. Os `?` ao lado dos rótulos explicam cada métrica em linguagem de cliente.

## Filtro de período

- **Atalhos:** último dia, 7, 30, 90 dias, este mês, tudo.
- **Data livre:** os dois campos de data no canto direito aceitam qualquer intervalo dentro do histórico disponível. Tudo na página (KPIs, gráfico, cards e ranking de criativos) é recalculado para o intervalo escolhido.

O recorte é feito no navegador a partir das linhas diárias por anúncio guardadas em `data.json` — por isso qualquer intervalo funciona, sem ida à API.

---

## Passo a passo para ligar a atualização automática (~10 min)

### 1) Gerar o token da Meta (System User — não expira)

1. Acesse **[business.facebook.com](https://business.facebook.com)** → **Configurações do Negócio**.
2. **Usuários → Usuários do sistema** → *Adicionar* → crie um System User (função **Admin**).
3. Em **Ativos atribuídos**, adicione a conta de anúncios **`CA - Geoplas`** (ID `409508558303244`) com permissão total.
4. **Gerar novo token** → escolha o App → marque os escopos **`ads_read`** e **`read_insights`**.
5. Copie o token. *(System User token não expira — ideal para automação.)*

### 2) Guardar o token como secret do repositório

**Settings → Secrets and variables → Actions → New repository secret**

- **Name:** `META_TOKEN` · **Value:** _(cole o token)_
- *(Opcional)* `AD_ACCOUNT_ID` = `409508558303244` — já é o padrão no script.
- *(Opcional, em **Variables**)* `SINCE` = `2026-01-01` — data inicial do histórico puxado.

### 3) Ligar o GitHub Pages

**Settings → Pages → Source: _Deploy from a branch_ → Branch: `main` / `/ (root)` → Save.**
Em ~1 min o painel fica no ar em `https://<usuario>.github.io/<repo>/`.

> **Visibilidade:** o Pages só publica de repositório **público** no plano grátis (ou **privado** com GitHub Pro). Como o painel expõe métricas e verba do cliente, decida: manter privado (Pro) ou público com URL discreta.

### Rodar agora, sem esperar a hora cheia

**Actions → Atualizar dados Meta Ads → Run workflow.**

---

## Dados que já vêm no repositório

`data.json` foi semeado com o histórico real da conta, puxado da Meta e **conferido contra os agregados oficiais**: na janela de 29/05 a 26/08/2026 o investimento bate centavo a centavo nas três campanhas (R$2.703,87 · R$1.345,89 · R$2.232,76), assim como CPM, CTR, CPC, seguidores, ThruPlay e views de 50/75/100%.

- **1.632 linhas** diárias por anúncio · **16 anúncios** · **23/01/2026 → 27/08/2026**
- No seed, as métricas do funil de mensagens (C3) foram derivadas de `custo por ação` (`contagem = gasto ÷ custo`, que é exatamente como a Meta calcula). Na primeira execução do Actions elas passam a vir como contagem direta da API.
- As **capas dos criativos** só aparecem depois da primeira execução do Actions (o ranking usa o código do anúncio como marcador até lá).

## Ajustes rápidos

- **Frequência de atualização:** `cron` em `.github/workflows/update-data.yml` (`0 * * * *` = de hora em hora; `0 */6 * * *` = a cada 6h).
- **Trocar / adicionar campanha monitorada:** bloco `PLAN` no topo de `scripts/fetch-meta.mjs` — é o único lugar que conhece a estratégia (qual campanha é C1/C2/C3 e qual é a métrica principal de cada uma).
- **Visual e textos:** tudo em `index.html`.

## Rodar local

`index.html` busca `data.json` via `fetch`, e o navegador bloqueia isso em `file://`. Use um servidor:

```bash
npx serve .
```

---

**Corvo Assessoria de Tráfego e Marketing** · conta `409508558303244`
