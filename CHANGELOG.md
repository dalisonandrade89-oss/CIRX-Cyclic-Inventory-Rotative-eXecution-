# Changelog

Todas as mudanças relevantes do **CIRX (Cyclic Inventory & Rotative eXecution)** — antes “Contagens de estoque 2026” — ficam registradas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e o projeto usa [Versionamento Semântico](https://semver.org/lang/pt-BR/) (`MAJOR.MINOR.PATCH`).
A versão exibida no rodapé do app vem da constante `APP` no HTML e **tem de coincidir** com a entrada mais recente deste arquivo.

| Tipo de mudança | Sobe | Exemplo |
|---|---|---|
| Quebra de compatibilidade (layout da planilha muda, migração de dados, troca de arquitetura) | MAJOR | 1.4.2 → 2.0.0 |
| Funcionalidade nova, nova aba, novo indicador | MINOR | 1.1.0 → 1.2.0 |
| Correção de bug, ajuste de texto/estilo, atualização só dos dados embutidos | PATCH | 1.1.0 → 1.1.1 |

---

## [Não lançado]

_Nada por enquanto._

---

## [1.2.1] — 2026-10-08

### Alterado
- **Novo nome do projeto: CIRX (Cyclic Inventory & Rotative eXecution).** Substitui “Contagens de estoque 2026” no cabeçalho (sigla em destaque, nome por extenso em peso normal), no título da aba do navegador e no rodapé.
- Constante `APP` ganha `short:'CIRX'` (sigla) além de `name` (nome completo).
- O nome do arquivo HTML (`Contagens de stock 2026.html`) e as chaves de armazenamento do navegador foram mantidos: nada salvo se perde.

---

## [1.2.0] — 2026-10-08

### Alterado
- **Navegação do cabeçalho**: as 7 abas horizontais foram substituídas por um **menu suspenso** (botão “Seção ▾” que mostra a seção atual).
  - Abre ao **passar o mouse** e fecha ao sair, com tolerância de ~0,2 s para não fechar no caminho até o painel.
  - **Destaque deslizante** acompanha o item sob o mouse (ou com foco do teclado), com barra de cor à esquerda; a seção atual fica marcada com ✓.
  - Em celular/tablet (sem mouse) abre e fecha por **toque**; clicar fora também fecha.
  - **Teclado**: Enter/Espaço/↓ abrem, ↑/↓/Home/End navegam, Enter escolhe, Esc fecha e devolve o foco ao botão.
  - Seções agrupadas: *Visão geral, Planejar, Contagens, Validações, Qualidade dos dados* | *Endereços, Auditorias de endereços*.
  - Os contadores (Planejar, Validações, Qualidade dos dados) continuam nos itens do menu; um **ponto vermelho** no botão avisa quando alguma seção tem pendências.
- Acessibilidade: o menu usa `role="menu"`/`menuitemradio` com `aria-expanded` e `aria-checked`; animações desligadas quando o sistema pede movimento reduzido.

---

## [1.1.1] — 2026-10-08

### Alterado
- **Idioma padronizado em português do Brasil** em toda a interface: *ficheiro → arquivo*, *registo → registro*, *planear/planeada → planejar/planejada* (aba **Planejar**), *guardar → salvar*, *folha → aba* (do Excel), *detetada → detectada*, *ligação → conexão*, *grelha → grade*, *partilham → compartilham*, *A carregar…/A mostrar → Carregando…/Exibindo*, *Seguinte → Próxima*, *por contar → a contar*, *numa/num → em uma/em um*.
- Nome do aplicativo: **Contagens de estoque 2026** (antes “Contagens de stock 2026”). O nome do arquivo HTML foi mantido para não quebrar atalhos e links.
- Arquivo exportado pela aba Planejar: `contagens_planejadas.csv` (antes `contagens_planeadas.csv`).
- Números nos eixos e dicas dos gráficos no formato brasileiro (`1.000` em vez de `1,000`).
- README, CHANGELOG e `tools/atualizar_dados.py` revisados para português do Brasil.

### Sem impacto nos dados
- As chaves internas de armazenamento do navegador (IndexedDB `contagens-dash`: `upload`, `stock`, `plan`) não mudaram: planilhas carregadas e planos já salvos continuam disponíveis.

---

## [1.1.0] — 2026-10-07

### Adicionado
- **Versão do aplicativo no rodapé** (`Contagens de stock 2026 · v1.1.0 · 07/10/2026`), com a origem da base de dados (planilha embutida ou arquivo carregado).
- **Histórico de versões dentro do app**: clicar na versão do rodapé abre o painel com as mudanças de cada versão.
- Título da aba do navegador inclui a versão (`Contagens de stock 2026 · v1.1.0`).
- Constante única `APP` (`version`, `date`, `dataDate`, `changes`) como fonte de verdade da versão.
- Script `tools/atualizar_dados.py` para atualizar os dados embutidos a partir da planilha (resultado idêntico ao do botão “Atualizar contagens”).

### Alterado
- Dados embutidos atualizados com `Contagens 2026.xlsx` de 07/10/2026:
  - C01: 3.750 contagens (sem alteração).
  - C02: 724 contagens com data (antes: 610, até 02/10/2026).
  - Aud 9 e Aud oc 6 com as ruas lançadas desde a última atualização.
- A exportação de endereços do WMS embutida continua a de 02/10/2026 (não foi fornecida uma nova).

### Corrigido
- **Segurança (XSS)**: no painel de detalhe da contagem, qualquer texto vindo da planilha que começasse com `<` (ex.: na coluna “Ação imediata”) era inserido como HTML e podia executar script. Agora apenas os campos gerados pelo próprio app são tratados como HTML; o resto é sempre escapado.
- **Aba Planejar**: a cada atualização da tela era registrado mais um ouvinte de eventos no contêiner da aba (vazamento de memória e processamento duplicado ao marcar SKUs). Passa a ser registrado uma única vez.

---

## [1.0.0] — 2026-10-02

Versão de referência: o estado do webapp antes da introdução do controle de versão. A data corresponde à dos dados embutidos nessa entrega.

### Funcionalidades existentes
- **Visão geral**: % de SKUs contados por ciclo e curva, estimativa de término por ritmo médio, KPIs de resultado (correto, falta, sobra, saldo líquido), pendências e prazos vencidos, gráficos temporais, por curva e por contador, top 10 divergências.
- **Planejar**: geração automática de plano por curva/rank em dias úteis, criação manual, detecção automática de “Feita”, exportação CSV. Planos salvos no navegador (IndexedDB).
- **Contagens**: tabela filtrável/ordenável com detalhe e exportação CSV.
- **Validações**: filas de pendentes, sem ação imediata, com ação a fechar, prazo vencido e resolvidas.
- **Qualidade dos dados**: 10 verificações automáticas (duplicados, fórmula sobrescrita, prazos, espaços extras, valores extremos, etc.).
- **Endereços**: mapa de ruas/prédios/níveis por ocupação ou andamento da contagem, áreas especiais e virtuais, validação da exportação do WMS, planejamento por rua/endereço.
- **Auditorias de endereços**: posições vazias com produto e posições ocupadas com erro, por auditoria e por rua.
- Carregamento da planilha de contagens e da exportação de endereços (salvos no navegador), tema claro/escuro.
