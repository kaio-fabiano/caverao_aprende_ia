# Implementation Tasks: Dashboard Financeira

**Change ID:** `add-financial-dashboard`

Referências obrigatórias durante a implementação:
- `openspec/project.md` — padrões do projeto
- `openspec/changes/add-financial-dashboard/specs/domain_delta.md` — regras RN-01..RN-12
- `openspec/changes/add-financial-dashboard/specs/ui_delta.md` — especificação das telas
- `design-system/dashboard-financeira/MASTER.md` — tokens de cor, tipografia e efeitos

---

## Fase 0: Migração da planilha (manual, BLOQUEANTE)

Nenhuma task de código pode começar antes desta fase. Implementar contra o formato atual significa reescrever tudo depois.

- [ ] 0.1 Renomear a aba atual para `2026-01`
- [ ] 0.2 Apagar os 11 blocos duplicados, mantendo apenas um bloco de gastos e um de renda
- [ ] 0.3 Adicionar a coluna E `Categoria` e preencher os 11 gastos conforme o mapeamento da proposal
- [ ] 0.4 Reorganizar o bloco de renda: cabeçalho `Renda | Valor` com `Salário` e `Renda Extra`
- [ ] 0.5 Duplicar a aba para os demais meses que já possuem dados reais, nomeando `YYYY-MM`
- [ ] 0.6 Criar a API key no Google Cloud e restringi-la à Google Sheets API

**Quality Gate:**
- [ ] `GET .../spreadsheets/{id}?includeGridData=false` lista apenas abas no formato `YYYY-MM`
- [ ] Um `values:batchGet` com `UNFORMATTED_VALUE` devolve `Valor` como number e `Estado` como boolean

---

## Fase 1: Fundação

- [ ] 1.1 Scaffold Vite + React + TypeScript (`npm create vite@latest`)
- [ ] 1.2 Tailwind CSS + shadcn/ui inicializados
- [ ] 1.3 Tokens de cor do `MASTER.md` aplicados como CSS variables, incluindo os tokens semânticos financeiros do `ui_delta.md`
- [ ] 1.4 Fonte IBM Plex Sans carregada
- [ ] 1.5 ESLint + Prettier + Vitest configurados
- [ ] 1.6 `.env.example` com `VITE_GOOGLE_API_KEY` e `VITE_SHEET_ID`; `.env.local` no `.gitignore`
- [ ] 1.7 `src/domain/types.ts`: `Expense`, `Income`, `MonthData`, `MonthSummary`, `ExpenseStatus`, `DataIssue`

**Quality Gate:**
- [ ] `npm run lint` limpo
- [ ] `npm run build` sem erro

---

## Fase 2: Domínio (sem UI)

Toda task desta fase é puro TypeScript, testável sem rede e sem React.

- [ ] 2.1 `domain/parse.ts`: resposta da API → `MonthData` (RN-01, RN-02, RN-07, RN-10)
- [ ] 2.2 `domain/calc.ts`: totais e status (RN-03, RN-04, RN-05, RN-06, RN-08)
- [ ] 2.3 `domain/format.ts`: `Intl` para moeda BRL e datas (RN-11)
- [ ] 2.4 `domain/series.ts`: agregação multi-mês para evolução e comparação por gasto (RN-12)
- [ ] 2.5 Fixtures a partir dos dados reais: Total Gasto `3271.59`, Renda Total `9500`, Valor Restante `6228.41`
- [ ] 2.6 Testes de borda: dia 30 em fevereiro, `Valor` não numérico, aba ausente, mês sem bloco de renda, mês sem nenhum gasto

**Quality Gate:**
- [ ] Cada RN de RN-01 a RN-12 possui pelo menos um teste nomeado com seu ID
- [ ] `npm run test` passando

---

## Fase 3: Camada de dados

- [ ] 3.1 `data/sheetsClient.ts`: listagem de abas + `values:batchGet` com `valueRenderOption=UNFORMATTED_VALUE`
- [ ] 3.2 Filtro de abas pelo regex `^\d{4}-(0[1-9]|1[0-2])$`, expondo a lista de abas descartadas
- [ ] 3.3 `data/useSheetData.ts`: loading, erro, dados, `refresh()` manual e cache em memória
- [ ] 3.4 Tratamento de erro distinguindo: key inválida (403), planilha não encontrada (404), sem rede
- [ ] 3.5 Nenhuma regra de negócio neste módulo — apenas I/O e tipagem

**Quality Gate:**
- [ ] Dados reais da planilha chegam tipados ao domínio
- [ ] Cada modo de erro exibe mensagem específica, nunca "algo deu errado"

---

## Fase 4: Telas

Implementar na ordem. Cada tela cobre os quatro estados: loading, erro, vazio e sucesso.

- [ ] 4.1 Shell da aplicação: navegação, seletor de mês, indicador de aba ativa, `refresh`
- [ ] 4.2 Tela **Mês Atual**: métrica-herói + cards secundários + tabela de gastos com status
- [ ] 4.3 Gráfico de **barras horizontais por categoria** (substitui pizza — ver ui_delta.md)
- [ ] 4.4 Tela **Próximos Vencimentos**: pendências ordenadas por data, atrasados no topo, link para a aba no Sheets
- [ ] 4.5 Tela **Evolução entre Meses**: linha multi-série com estilos de traço distintos e rótulos diretos
- [ ] 4.6 Tela **Comparação por Gasto**: seletor de gasto, série temporal, média e variação percentual

**Quality Gate:**
- [ ] As quatro telas funcionam com a planilha real
- [ ] Nenhum status comunicado apenas por cor — sempre cor + ícone + texto
- [ ] Sem scroll horizontal em 375px

---

## Fase 5: Acabamento

- [ ] 5.1 Painel de problemas nos dados (RN-10): linhas descartadas e abas ignoradas
- [ ] 5.2 Responsividade verificada em 375, 768, 1024 e 1440px
- [ ] 5.3 Acessibilidade: foco visível, navegação por teclado, contraste 4.5:1, `prefers-reduced-motion`
- [ ] 5.4 Revisão visual com a skill `playwright-cli` (screenshots desktop e mobile)
- [ ] 5.5 README: como criar e restringir a API key, como nomear as abas, como rodar

**Quality Gate:**
- [ ] `npm run lint`, `npm run test` e `npm run build` limpos
- [ ] Checklist de pré-entrega do `MASTER.md` cumprido

---

## Completion Checklist

- [ ] Todas as fases concluídas
- [ ] Todos os quality gates aprovados
- [ ] Verificação ponta a ponta da proposal executada
- [ ] Pronto para `/openspec-archive`
