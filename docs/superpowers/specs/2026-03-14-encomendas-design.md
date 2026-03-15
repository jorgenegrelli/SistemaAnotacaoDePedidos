# Design: Funcionalidade de Encomendas
**Data:** 2026-03-14
**Projeto:** Sistema de Pedidos — Super Negrelli
**Arquivo alvo:** `index.html`

---

## Visão Geral

Adicionar suporte a encomendas no sistema existente, integrando ao fluxo atual sem quebrar funcionalidades existentes. A abordagem escolhida é **expansão inline no card existente (Opção A)**: o bloco "Tipo de Pedido" ganha um segundo toggle "Modo Encomenda", e o formulário de itens ganha suporte a bolos com campos condicionais.

---

## 1. Controles de Tipo de Pedido

O bloco atual "Tipo de Pedido" (que contém o toggle Entrega/Retirada) ganha um segundo toggle independente:

### Toggle 1 — Entrega / Retirada (existente, sem alteração de comportamento)
- Controla visibilidade e obrigatoriedade dos campos de endereço
- Entrega: endereço obrigatório (rua + cidade)
- Retirada: endereço oculto/opcional

### Toggle 2 — Modo Encomenda (novo)
- Quando **ativado**, exibe seção de encomenda com:
  - **Data da encomenda** (date input) — obrigatório
  - **Horário da encomenda** (time input) — obrigatório
  - **Valor do sinal/entrada** (number input, R$) — opcional
- Quando **desativado**, campos ocultos e não validados

---

## 2. Campo de Funcionário

- **Select "Funcionário que tirou o pedido"** adicionado na seção de forma de pagamento
- Sempre visível para **todos os pedidos** (modo encomenda ou não)
- **Obrigatório** para salvar/imprimir
- Opções: Ana, Angelo, Matheus, Jorge Lucas, Jorge Issao, Poleana, Edurvirgem

---

## 3. Item do Tipo Bolo

No formulário de adicionar item, um **checkbox "É bolo"** é adicionado. Quando marcado, campos extras expandem inline:

| Campo | Tipo | Obrigatório |
|---|---|---|
| Formato | select: Redondo / Quadrado | Sim |
| Peso (kg) | number | Sim |
| Sabor | text | Sim |
| Decoração específica | textarea | Não |
| Papel arroz (+R$ 20) | checkbox | — |

**Regra de negócio:** marcar "Papel arroz" soma automaticamente R$ 20,00 ao item.

**Exibição na lista de itens:**
```
🎂 1 un - Bolo Redondo 2kg - Chocolate | Papel arroz +R$20
```

---

## 4. Validação

| Condição | Regra |
|---|---|
| Modo entrega | Rua e cidade obrigatórios |
| Modo retirada | Endereço não obrigatório |
| Modo encomenda ativo | Data e horário obrigatórios |
| Item é bolo | Formato, peso e sabor obrigatórios |
| Sempre | Funcionário obrigatório |

A função `verificarValidadePedido()` existente é expandida para contemplar todas as novas regras.

---

## 5. Modelo de Dados (IndexedDB)

Sem necessidade de migração (IndexedDB é schema-less). Novos campos opcionais adicionados:

### Pedido
```js
{
  codigo, data, cliente, itens, formaPagamento, valorTroco, observacoes,
  funcionario: "string",                          // NOVO — sempre presente
  encomenda: {                                    // NOVO — apenas se modo encomenda
    data: "YYYY-MM-DD",
    horario: "HH:MM",
    sinal: number | null
  }
}
```

### Item (quando bolo)
```js
{
  id, quantidade, unidade, descricao,
  bolo: {                                         // NOVO — apenas se é bolo
    formato: "redondo" | "quadrado",
    kg: number,
    sabor: "string",
    decoracao: "string" | null,
    papelArroz: boolean
  },
  valorBolo: number | null                        // 20 se papelArroz, 0 caso contrário
}
```

---

## 6. Impacto nas Saídas

### Mensagem WhatsApp
- Inclui funcionário sempre: `👤 *Atendente:* {nome}`
- Se encomenda: bloco com data, horário e sinal
- Se item bolo: linha com detalhes do bolo e papel arroz

### Cupom Térmico (impressão)
- Idem — funcionário, dados da encomenda e detalhes do bolo quando presentes

### Histórico de Pedidos
- Badge "ENCOMENDA" exibido nos itens do histórico que tiverem campo `encomenda` preenchido

### Novo Pedido (reset)
- Limpa todos os novos campos (toggle encomenda off, funcionário vazio, checkbox bolo desmarcado)

---

## 7. Arquivos Modificados

- `index.html` — único arquivo do projeto

### Mudanças no HTML
1. Bloco "Tipo de Pedido": adicionar toggle Modo Encomenda + seção `#camposEncomenda` (oculta por padrão)
2. Seção de pagamento: adicionar select `#funcionario`
3. Formulário de itens: adicionar checkbox `#ehBolo` + seção `#camposBolo` (oculta por padrão)

### Mudanças no JavaScript
1. Estado global: `modoEncomenda = false`
2. Toggle Modo Encomenda: event listener + mostrar/ocultar `#camposEncomenda`
3. Checkbox "É bolo": event listener + mostrar/ocultar `#camposBolo`
4. `verificarValidadePedido()`: adicionar validações de encomenda, funcionário e bolo
5. `adicionarItem()`: capturar e incluir campos de bolo no objeto item
6. `atualizarListaItens()`: renderizar detalhes de bolo com ícone 🎂
7. `copiarWhatsApp()`: incluir funcionário, encomenda e bolo na mensagem
8. `imprimirPedido()`: incluir funcionário, encomenda e bolo no cupom + salvar no IndexedDB
9. `novoPedido()`: resetar todos os novos campos
10. `recuperarPedido()`: restaurar novos campos ao recuperar do histórico
11. `mostrarModalHistorico()`: exibir badge ENCOMENDA quando aplicável

---

## 8. Possíveis Melhorias Futuras

- Listagem dedicada de encomendas com filtro por data de encomenda (diferente da data do pedido)
- Notificação/alerta quando uma encomenda está próxima da data de retirada/entrega
- Campo de valor total do bolo (além do sinal)
- Exportar CSV separado só para encomendas
