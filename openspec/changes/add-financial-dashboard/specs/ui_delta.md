# Delta: Interface

**Change ID:** `add-financial-dashboard`
**Affects:** `src/pages/`, `src/components/`
**Fonte de verdade visual:** `design-system/dashboard-financeira/MASTER.md`

---

## Direção visual

**Estilo:** Minimalism & Swiss Style — base em grid, alto contraste, tipografia como estrutura, zero ornamento. Escolhido porque o produto é leitura de números: legibilidade vence decoração. A direção é única e não deve ser misturada com outras.

**Tipografia:** IBM Plex Sans (headings e corpo), pesos 300–700. Números monetários em variante tabular (`font-variant-numeric: tabular-nums`) para que as colunas alinhem verticalmente.

**Modo:** dark-first. Light mode é suportado pelo estilo, mas o dark é o padrão de entrega.

### Tokens de cor

Base vinda do `MASTER.md`:

| Papel | Hex |
|---|---|
| Background | `#0F172A` |
| Foreground | `#F8FAFC` |
| Card | `#222735` |
| Muted | `#272F42` |
| Muted Foreground | `#94A3B8` |
| Border | `#334155` |
| Primary | `#F59E0B` |
| Ring (foco) | `#F59E0B` |

**Tokens semânticos financeiros** — adicionados por este delta, ajustados para contraste sobre fundo escuro:

| Estado | Token | Hex |
|---|---|---|
| Pago | `--status-paid` | `#34D399` |
| Pendente | `--status-pending` | `#94A3B8` |
| Vence em breve (≤3 dias) | `--status-due-soon` | `#FBBF24` |
| Atrasado | `--status-overdue` | `#F87171` |
| Sem fatura (valor zero) | `--status-empty` | `#64748B` |

Os tons `400` foram escolhidos no lugar dos `500` originais porque `#EF4444` sobre `#0F172A` fica próximo do limite de 4.5:1.

### Anti-padrões (proibidos)

- Gradientes roxo/rosa "de IA" — o accent `#8B5CF6` sugerido pela base **não é adotado**, por conflitar com o próprio anti-padrão do design system e com a sobriedade exigida por produto financeiro.
- Emoji como ícone. Usar Lucide (já incluso no shadcn/ui), viewBox 24×24.
- Pilha de cards sem hierarquia — ver requisito abaixo.
- Hover com `scale` que desloca layout; usar transição de cor/borda em 200ms.

---

## ADDED

### Requirement: Hierarquia da tela principal

A tela do mês não deve ser uma grade de cards de peso igual. Uma métrica é a protagonista; as demais são suporte.

**Falta Pagar** é a métrica-herói: é a única acionável — responde "quanto ainda preciso pagar este mês".

#### Scenario: Leitura em três segundos
- GIVEN a tela do mês atual carregada
- WHEN o usuário olha a tela
- THEN `Falta Pagar` aparece em tipografia significativamente maior que as demais métricas
- AND as métricas Total Gasto, Pago, Renda Total, Valor Restante e Caixa Atual aparecem como suporte secundário

#### Scenario: Distinção entre projeção e caixa
- GIVEN `Valor Restante` (projetado) e `Caixa Atual` exibidos juntos
- WHEN o usuário passa o mouse ou foca em `Valor Restante`
- THEN um tooltip explica que assume todas as contas pagas
- AND os dois valores nunca aparecem com o mesmo rótulo genérico de "saldo"

---

### Requirement: Comunicação de status

Nenhum status é comunicado apenas por cor. Cada status combina **cor + ícone + rótulo textual**.

| Status | Cor | Ícone Lucide | Rótulo |
|---|---|---|---|
| Pago | `--status-paid` | `check-circle` | Pago |
| Pendente | `--status-pending` | `clock` | Pendente |
| Vence em breve | `--status-due-soon` | `alert-circle` | Vence em X dias |
| Atrasado | `--status-overdue` | `alert-triangle` | Atrasado há X dias |
| Sem fatura | `--status-empty` | `minus-circle` | Sem fatura este mês |

#### Scenario: Usuário com daltonismo
- GIVEN a tabela de gastos com status variados
- WHEN as cores são removidas
- THEN cada status continua identificável pelo ícone e pelo texto

---

### Requirement: Gráfico de categorias em barras horizontais

A distribuição por categoria usa **barra horizontal ordenada de forma decrescente**, não gráfico de pizza.

Justificativa: a base de conhecimento classifica pizza como risco de acessibilidade **alto**, recomendando-a apenas para até 5 categorias e desaconselhando-a em contexto acessibility-first. O projeto define 6 categorias. Barra é risco baixo, suporta até 15 categorias e torna o ranking — que é a informação desejada — imediato.

#### Scenario: Distribuição do mês
- GIVEN gastos categorizados com valor maior que zero
- WHEN o gráfico é renderizado
- THEN as barras aparecem ordenadas da maior para a menor
- AND cada barra tem rótulo de categoria e valor visíveis, sem depender de hover
- AND gastos com valor zero estão ausentes do gráfico (RN-08)

#### Scenario: Alternativa acessível
- GIVEN o gráfico de categorias
- WHEN o usuário navega por teclado
- THEN existe uma tabela equivalente com os mesmos valores

---

### Requirement: Gráfico de evolução entre meses

Linha multi-série com Total Gasto, Renda Total e Valor Restante.

#### Scenario: Séries distinguíveis sem cor
- GIVEN as três séries renderizadas
- WHEN o gráfico é exibido
- THEN cada série usa um estilo de traço distinto (sólido, tracejado, pontilhado)
- AND cada série tem rótulo direto, sem depender exclusivamente de legenda por cor

#### Scenario: Poucos dados
- GIVEN menos de quatro meses disponíveis
- WHEN a tela é renderizada
- THEN os valores aparecem como cards de estatística em vez de gráfico de linha (RN-12)

#### Scenario: Mês ausente
- GIVEN uma lacuna entre meses existentes
- WHEN a série é renderizada
- THEN a linha apresenta uma descontinuidade, nunca um ponto em zero

---

### Requirement: Tela de próximos vencimentos

Lista das pendências do mês ordenada por data, atrasados primeiro.

#### Scenario: Ação possível apesar do read-only
- GIVEN um gasto pendente na lista
- WHEN o usuário deseja marcá-lo como pago
- THEN um link abre a aba do mês no Google Sheets
- AND a interface deixa claro que a dashboard é somente leitura

#### Scenario: Nada pendente
- GIVEN todos os gastos do mês pagos
- WHEN a tela é aberta
- THEN um estado vazio afirmativo é exibido ("Tudo pago este mês"), nunca uma área em branco

---

### Requirement: Estados obrigatórios

Toda tela implementa loading, erro, vazio e sucesso.

#### Scenario: Carregamento
- GIVEN dados sendo buscados
- WHEN a tela renderiza
- THEN é exibido skeleton preservando o layout final, com `aria-busy`
- AND não há spinner piscando para respostas quase instantâneas

#### Scenario: Erro específico
- GIVEN falha na API
- WHEN a tela renderiza
- THEN a mensagem distingue key inválida, planilha inacessível e ausência de rede
- AND oferece ação de repetir

---

### Requirement: Responsividade

#### Scenario: Tabela em tela estreita
- GIVEN a tabela de gastos em viewport de 375px
- WHEN a tela é exibida
- THEN cada gasto vira um card empilhado, ou a tabela recebe wrapper com rolagem horizontal
- AND não existe rolagem horizontal na página

#### Scenario: Alvos de toque
- GIVEN elementos interativos em mobile
- WHEN medidos
- THEN têm no mínimo 44×44px

---

### Requirement: Acessibilidade e movimento

#### Scenario: Navegação por teclado
- GIVEN um usuário navegando por Tab
- WHEN percorre a interface
- THEN a ordem de foco acompanha a ordem visual
- AND o anel de foco usa `--color-ring` e é sempre visível

#### Scenario: Movimento reduzido
- GIVEN `prefers-reduced-motion: reduce`
- WHEN a interface renderiza
- THEN transições e animações de entrada de gráfico são suprimidas

---

## Checklist de pré-entrega

- [ ] Nenhum emoji como ícone; conjunto único (Lucide)
- [ ] `cursor-pointer` em todo elemento clicável
- [ ] Transições entre 150–300ms, sem deslocamento de layout
- [ ] Contraste mínimo 4.5:1 em ambos os modos
- [ ] Foco visível em toda navegação por teclado
- [ ] `prefers-reduced-motion` respeitado
- [ ] Verificado em 375, 768, 1024 e 1440px
- [ ] Nenhum status comunicado apenas por cor

---

## MODIFIED

(Nenhum — projeto greenfield.)

## REMOVED

(Nenhum.)
