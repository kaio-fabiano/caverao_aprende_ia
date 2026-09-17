# Projeto: Dashboard Financeira

Convenções e padrões deste projeto. Toda implementação deve aderir a este documento.

---

## Stack

| Camada | Escolha |
|---|---|
| Build | Vite |
| UI | React 18 + TypeScript |
| Estilo | Tailwind CSS |
| Componentes | shadcn/ui |
| Gráficos | Recharts |
| Testes | Vitest |
| Lint/Format | ESLint + Prettier |

Sem backend. Sem biblioteca de estado global — `useSheetData` + `useMemo` bastam nesta escala.

---

## Estrutura de pastas

```
src/
  domain/          # regras de negócio puras, zero dependência de React
    types.ts       # Expense, Income, MonthData, MonthSummary
    parse.ts       # resposta da API -> MonthData
    calc.ts        # totais e status
    format.ts      # Intl money/date
  data/
    sheetsClient.ts  # fetch da API do Google, sem regra de negócio
    useSheetData.ts  # hook: cache, loading, erro, refresh manual
  components/
    ui/            # shadcn/ui gerado
  pages/
  App.tsx
```

### Regra estrutural (inegociável)

- `domain/` é **puro**: não importa React, não faz fetch, é 100% testável sem mock de rede.
- `data/` faz I/O e **não contém regra de negócio**.
- `components/` apresentam. Não fazem fetch e **não calculam** — consomem o que `domain/` devolve.

Essa separação existe porque as regras financeiras precisam ser testáveis isoladamente: erro silencioso em cálculo de dinheiro é o pior modo de falha deste projeto.

---

## Padrões de código

- TypeScript `strict: true`. Proibido `any` em `domain/`.
- Cálculos operam sobre `number`. Formatação acontece **apenas na borda de renderização**, nunca no meio do cálculo.
- Dinheiro formatado via `Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' })`.
- Nomes de domínio em português (`gasto`, `vencimento`, `pago`) por espelharem a planilha; nomes técnicos em inglês.
- Toda regra de negócio (RN-XX) tem teste unitário correspondente, nomeado com o ID da regra.

---

## Configuração e segredos

- Variáveis em `.env.local`: `VITE_GOOGLE_API_KEY`, `VITE_SHEET_ID`.
- `.env.local` **sempre** no `.gitignore`. `.env.example` versionado, sem valores reais.
- A API key deve ser restrita no Google Cloud à **Google Sheets API apenas**.
- Qualquer deploy público futuro exige adicionalmente restrição por HTTP referrer — a key vai exposta no bundle.

---

## Acesso à API do Google Sheets

- Listar abas: `GET https://sheets.googleapis.com/v4/spreadsheets/{id}?includeGridData=false`
- Ler valores: `values:batchGet` com **`valueRenderOption=UNFORMATTED_VALUE`**

`UNFORMATTED_VALUE` é obrigatório: devolve `Valor` como número nativo e `Estado` como boolean, eliminando todo o parsing de `"R$ 1.000,00"` (formato pt-BR com `.` de milhar e `,` decimal). Usar CSV ou `FORMATTED_VALUE` reintroduz esse parsing e está proibido.

---

## Gates de qualidade

Nenhuma fase é considerada concluída sem:

- `npm run lint` limpo
- `npm run test` passando
- `npm run build` sem erro

---

## Estados obrigatórios de UI

Todo componente que depende de dados remotos implementa os quatro estados: **loading**, **erro**, **vazio** e **sucesso**. Tela vazia sem explicação é considerada bug.
