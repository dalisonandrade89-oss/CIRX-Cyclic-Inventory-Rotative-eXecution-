# CIRX — Cyclic Inventory & Rotative eXecution

Dashboard de inventários rotativos.

Webapp para acompanhar, planejar e validar as **contagens rotativas de estoque** (ciclos C01, C02…), as **auditorias de endereços** e o **mapa de endereços do WMS**.

- **Versão atual:** `1.2.1` (08/10/2026). Ver [CHANGELOG.md](CHANGELOG.md).
- **Base de dados hoje:** a planilha `Contagens 2026.xlsx`. O app lê a planilha; o trabalho de lançamento continua a ser feito no Excel.
- **Objetivo:** passar a trabalhar só no web (ver [Roadmap](#roadmap-planilha--web)).

---

## Sumário

1. [O que o app faz](#o-que-o-app-faz)
2. [Como usar no dia a dia](#como-usar-no-dia-a-dia)
3. [Contrato da planilha](#contrato-da-planilha)
4. [Contrato da exportação de endereços (WMS)](#contrato-da-exportação-de-endereços-wms)
5. [Regras de negócio](#regras-de-negócio)
6. [Arquitetura atual](#arquitetura-atual)
7. [Versionamento e como lançar uma versão](#versionamento-e-como-lançar-uma-versão)
8. [Limitações conhecidas](#limitações-conhecidas)
9. [Roadmap: planilha → web](#roadmap-planilha--web)

---

## O que o app faz

| Seção | Para que serve |
|---|---|
| **Visão geral** | % de SKUs contados por ciclo e curva (A/B/C), estimativa de término pelo ritmo médio, KPIs (saldo correto, falta, sobra, saldo líquido, pendentes, prazos vencidos), gráficos por período, curva e contador, top 10 divergências. |
| **Planejar** | Gera plano automático (curva → rank, em dias úteis, repartido entre contadores) ou manual. Marca “Feita” sozinho quando a planilha mostra o SKU contado. Exporta CSV para colar no Excel. |
| **Contagens** | Todas as contagens com filtros, ordenação, detalhe e exportação CSV. |
| **Validações** | Filas de trabalho: pendentes, divergentes sem ação imediata, com ação a fechar, prazo vencido, resolvidas. |
| **Qualidade dos dados** | Verificações automáticas sobre a planilha (duplicados, fórmula sobrescrita, prazos, nomes com espaços extras, valores ≥ 10.000, auditorias com 0 no WMS…), com o nº da linha no Excel. |
| **Endereços** | Mapa rua × prédio × nível colorido por ocupação ou andamento da contagem; áreas especiais/virtuais; validação da exportação do WMS; planejar contagem por rua ou endereço. |
| **Auditorias de endereços** | Posições vazias com produto (% ponderado) e posições ocupadas com erro, por auditoria e por rua. |

**Navegação:** as seções ficam no menu **Seção ▾** do cabeçalho — abre ao passar o mouse (ou por clique/toque/teclado) e mostra os contadores de pendências; um ponto vermelho no botão avisa quando alguma seção tem pendências.

Filtros globais: ciclo, curva, período, contador de estoque/picking, resultado, situação e busca por SKU/referência/descrição.

---

## Como usar no dia a dia

1. **Abrir** o arquivo `Contagens de stock 2026.html` (o nome do arquivo foi mantido; o app se chama CIRX) no navegador (Chrome/Edge). Abre com os dados embutidos (data no rodapé: “Base de dados: planilha embutida de …”).
2. **Atualizar contagens**: botão *Atualizar contagens* → escolher a versão mais recente de `Contagens 2026.xlsx`. O app salva o arquivo **neste navegador** e passa a mostrá-lo nas próximas aberturas. *Voltar aos dados originais* descarta o arquivo carregado.
3. **Atualizar endereços**: botão *Atualizar endereços* → exportação do WMS. A data é lida do nome do arquivo (`…DDMMAA.xlsx`); sem isso, usa a data de modificação.
4. **Planejar**: os planos ficam salvos apenas neste navegador. Para compartilhar, *Exportar CSV*.
5. **Versão**: o rodapé mostra a versão do app; clicar nela abre o histórico de versões.

> Recomendado: navegador atualizado e acesso à internet na primeira abertura (os gráficos e a leitura de Excel vêm do CDN `cdnjs`).

---

## Contrato da planilha

O app só lê as abas com estes nomes exatos (o resto, como `Dashboard 1`, `Dados1 (C01)`, é ignorado):

| Aba | Padrão do nome | Dados a partir da linha |
|---|---|---|
| Contagens por ciclo | `Contagens 2026 (C01)`, `Contagens 2026 (C02)`, … | 2 |
| Auditoria de posições vazias | `Aud 1`, `Aud 2`, … | 2 |
| Auditoria de posições ocupadas | `Aud oc 1`, `Aud oc 2`, … | 3 |

### `Contagens 2026 (Cxx)` — colunas A a U (a ordem importa)

| Col. | Cabeçalho | Uso no app |
|---|---|---|
| A | CURVA | A/B/C |
| B | RANK | ordem de prioridade dentro da curva |
| C | COD | SKU (chave) |
| D | REF | referência |
| E | DESC | descrição |
| F | Data | data da contagem — **linha sem data = SKU ainda não contado** |
| G | N° Invent | nº do inventário |
| H | CONT | nº da contagem |
| I | FÍSICO | quantidade contada |
| J | DISP 266 | saldo do sistema |
| K | RESERV | reservado |
| L | PEND | pendente |
| M | RESULTADO | se vazio, o app calcula |
| N | ENDEREÇOS VISITADOS | produtividade por contador |
| O | CONT EST | contador(es) de estoque, separados por `/` |
| P | CONT PICK | contador(es) de picking, separados por `/` |
| Q | AÇÃO IMEDIATA | |
| R | TRATATIVA | |
| S | RESULTADO (da tratativa) | |
| T | PRAZO PARA RESOLUÇÃO | |
| U | SITUAÇÃO | só o texto `Resolvido` conta como resolvido |

### `Aud N` (posições vazias) — colunas A a F
Rua · Data · Vazios no WMS · Com produto · % (ignorada, o app recalcula) · Executor.

### `Aud oc N` (posições ocupadas) — colunas A a G
Rua · Início · Fim · Executor · Erros · (não usada) · Observações.

**Não renomeie abas nem insira/remova colunas** sem atualizar o app — isso é uma mudança MAJOR.

---

## Contrato da exportação de endereços (WMS)

Primeira aba, cabeçalhos na linha 1 (acentos, maiúsculas e espaços são ignorados na comparação).

- **Obrigatórias:** Código endereço, Depósito, Rua, Prédio, Nível, Cod Winthor, Quantidade, Tipo de endereço.
- **Opcionais:** Apartamento, Bloqueio (`S`), Referência, Descrição, Lastro palet, Camada palet, Capacidade pickg, Ponto de reposição, Quantidade pendente entrada, Quantidade pendente saída, Quantidade disponível.

---

## Regras de negócio

- **Resultado** = (Físico − Sistema) − (Reservado + Pendente). Negativo = falta, positivo = sobra, zero = saldo correto. Se a coluna M diverge da fórmula em mais de 0,5, aparece em *Qualidade dos dados*.
- **SKU contado no ciclo** = existe pelo menos uma linha do ciclo com data. Vale a contagem mais recente.
- **Resolvido** = coluna SITUAÇÃO igual a `Resolvido` (sem diferenciar maiúsculas).
- **Prazo vencido** = não resolvido e prazo anterior a hoje.
- **Estimativa de término** = SKUs em falta ÷ média de SKUs distintos por dia de contagem, em dias úteis (seg–sex, sem feriados).
- **Contadores em dupla** (`Ana/João`) dividem igualmente endereços e contagens.
- **Endereço “contado”** (mapa) = todos os SKUs do endereço que estão na lista do ciclo já foram contados.
- **% de vazios com produto** = soma de “com produto” ÷ soma de “vazios no WMS” (ponderado). A planilha usa a média dos % por rua, que costuma dar valores maiores.

---

## Arquitetura atual

```
Contagens de stock 2026.html   (≈ 3,3 MB, arquivo único; o app se chama “CIRX”)
├── <style>                    tokens de cor (claro/escuro), layout responsivo
├── <script> CDN               Chart.js 4.4.1 · SheetJS (xlsx) 0.18.5
└── <script> app (IIFE)
    ├── APP                    versão, data, dataDate, histórico  ← fonte única da versão
    ├── EMBED                  planilha de contagens/auditorias embutida (JSON)
    ├── EMBED_S                exportação de endereços do WMS embutida (JSON)
    ├── build()/mkRow()        normalização das linhas → modelo em memória (DB)
    ├── render*()              uma função por seção; re-render completo a cada filtro
    ├── NAV / goTab()          menu suspenso de seções (hover, toque, teclado)
    ├── Table()                tabela com ordenação/paginação
    └── IndexedDB 'contagens-dash'
        ├── upload             última planilha carregada
        ├── stock              última exportação de endereços carregada
        └── plan               contagens planejadas
tools/atualizar_dados.py       atualiza EMBED a partir do .xlsx
```

**Por que é assim:** zero infraestrutura, funciona offline depois de aberto, não exige servidor nem login. Ótimo como painel de leitura sobre uma planilha; limitado como sistema de trabalho multiusuário (ver abaixo).

---

## Versionamento e como lançar uma versão

O projeto usa **SemVer** (`MAJOR.MINOR.PATCH`) — ver tabela no topo do [CHANGELOG.md](CHANGELOG.md).

**A cada modificação do webapp:**

1. No HTML, edite a constante `APP` (logo antes de `const EMBED=`):
   ```js
   const APP={name:'CIRX (Cyclic Inventory & Rotative eXecution)',short:'CIRX',version:'1.3.0',date:'2026-10-15',dataDate:'2026-10-15',
     changes:[
       {v:'1.3.0',d:'2026-10-15',items:['Descrição curta da mudança.']},
       // …entradas anteriores
     ]};
   ```
   - `name` / `short`: nome completo (cabeçalho, título e rodapé) e sigla.
   - `version` / `date`: versão e data do app (aparecem no rodapé e no título).
   - `dataDate`: data da planilha embutida (aparece como “Base de dados: planilha embutida de …”).
   - `changes`: o histórico mostrado ao clicar na versão.
2. Acrescente a mesma entrada no topo do `CHANGELOG.md` (Adicionado / Alterado / Corrigido / Removido / Segurança).
3. Teste: abrir o HTML, percorrer as abas, conferir o rodapé e o histórico, console do navegador sem erros.

**Só atualizar os dados embutidos** (PATCH):
```bash
pip install openpyxl
python3 tools/atualizar_dados.py "Contagens de stock 2026.html" "Contagens 2026.xlsx" 2026-10-15
```
Depois suba o PATCH em `APP.version` e registre no CHANGELOG.

---

## Limitações conhecidas

| # | Limitação | Impacto | Onde se resolve |
|---|---|---|---|
| 1 | Planos e arquivos carregados ficam **só no navegador** de quem os criou | Dois supervisores veem planos diferentes; limpar o navegador apaga planos | Fase 1 (backend) |
| 2 | Sem login/perfis; dados operacionais embutidos no HTML | Quem tiver o arquivo vê tudo | Fase 1 |
| 3 | Fonte de verdade é a planilha: sem histórico de quem alterou o quê | Auditoria fraca, risco de fórmula sobrescrita | Fase 2 |
| 4 | Nomes de aba com o ano fixo (`Contagens 2026 (Cxx)`) | Em 2027 é preciso alterar o app | Fase 1 (ciclo como dado) |
| 5 | Erros de gravação no IndexedDB são silenciosos | Plano pode não ficar salvo sem aviso | PATCH futuro |
| 6 | Scripts do CDN sem `integrity` (SRI) e app inteiro em um arquivo de 3,3 MB | Segurança/desempenho | Fase 1 (build com bundler) |
| 7 | Dias úteis não consideram feriados | Estimativas um pouco otimistas | MINOR futuro |

Dado observado na planilha de 07/10/2026: `Contagens 2026 (C02)`, linha 3375 (SKU 2884) está como **Resolvido sem data de contagem** — o app a trata como não contada.

---

## Roadmap: planilha → web

Meta: o web passa a ser o **sistema de registro**; a planilha vira exportação.

### Fase 0 — hoje (v1.x)
HTML único lê a planilha. Excel = fonte de verdade.

### Fase 1 — backend espelho (v2.0)
- **Stack sugerida:** Next.js (App Router, TypeScript) + PostgreSQL gerenciado (ex.: Supabase/Neon) + Drizzle ORM + login corporativo via OIDC (Microsoft Entra ID / Google Workspace).
- Importação da planilha **no servidor** (o mesmo parser de hoje), com as validações de *Qualidade dos dados* bloqueando importações com erro grave.
- Planos de contagem no banco (acaba a limitação 1), perfis `contador`, `validador`, `gestor`.
- O dashboard atual é reaproveitado quase inteiro: as funções `build/render*` passam a ler de uma API em vez de `EMBED`.

### Fase 2 — trabalho direto no web (v3.0)
- **Formulário mobile para contadores** (SKU por leitura de código de barras, físico, endereços visitados) → resultado calculado no servidor.
- **Fluxo de validação** no web: ação imediata, tratativa, prazo, resolução, com registro de quem/quando (*audit log*).
- Auditorias de endereços lançadas no web.
- Planilha passa a **exportação** (botão “Baixar Excel” no mesmo layout de hoje, para transição).

### Fase 3 — integração (v3.x)
- Importação automática da exportação do WMS (Winthor), agendada.
- Alertas (prazo vencido, divergência acima de limite) por e-mail/Teams.
- Desativar a planilha.

### Modelo de dados proposto (Fase 1)

```
ciclo(id, codigo 'C02', ano, inicio, fim)
sku(cod PK, ref, descricao)
ciclo_sku(ciclo_id, cod, curva, rank)                    -- lista do ciclo
contador(id, nome, tipo estoque|picking, ativo)
contagem(id, ciclo_id, cod, data, n_inventario, n_contagem,
         fisico, sistema, reservado, pendente,
         resultado GENERATED ALWAYS AS ((fisico-sistema)-(reservado+pendente)) STORED,
         enderecos_visitados, criado_por, criado_em)
contagem_contador(contagem_id, contador_id, papel)       -- substitui "Ana/João"
tratativa(contagem_id, acao_imediata, tratativa, resultado, prazo,
          situacao ENUM('pendente','resolvido'), atualizado_por, atualizado_em)
plano_contagem(id, ciclo_id, cod, data_planejada, contador_id, lote, criado_por)
auditoria(id, tipo 'vazia'|'ocupada', numero)
auditoria_rua(auditoria_id, rua, inicio, fim, wms_vazios, com_produto, erros, executor, obs)
snapshot_enderecos(id, data, arquivo) + endereco_saldo(snapshot_id, …colunas do WMS)
audit_log(tabela, registro_id, campo, de, para, usuario, em)
```

Princípios: o resultado é sempre calculado pelo banco (acaba “fórmula sobrescrita”), nomes de contadores vêm de cadastro (acaba “espaços extras”), situação é enum (acaba “Resolvido” digitado de formas diferentes), e cada alteração fica no `audit_log`.
