# Design: Funcionalidade de Encomendas
**Data:** 2026-03-14
**Projeto:** Sistema de Pedidos — Super Negrelli
**Arquivo alvo:** `index.html`

---

## Visão Geral

Adicionar suporte a encomendas no sistema existente, integrando ao fluxo atual sem quebrar funcionalidades existentes. A abordagem escolhida é **expansão inline no card existente (Opção A)**: o bloco "Tipo de Pedido" ganha um segundo toggle "Modo Encomenda", e o formulário de itens ganha suporte a bolos com campos condicionais.

---

## 1. Controles de Tipo de Pedido

### Estado atual do código
O código já possui um toggle Entrega/Retirada (`toggleTipoPedido`), a variável `modoRetirada`, e um bloco `#camposRetirada` com `dataRetirada`, `horarioRetirada` e `observacoesRetirada`. Esse bloco parcialmente implementava encomendas.

### Mudança
O `#camposRetirada` existente (data, horário, observações) é **removido** e substituído pelo novo painel `#camposEncomenda`. A variável `modoRetirada` é mantida para controlar apenas o endereço. Uma nova variável `modoEncomenda` controla os campos de encomenda.

### Toggle 1 — Entrega / Retirada (existente — sem mudança de comportamento)
- `modoRetirada`: controla visibilidade e obrigatoriedade dos campos de endereço
- Entrega (`modoRetirada = false`): endereço obrigatório (rua + cidade)
- Retirada (`modoRetirada = true`): endereço oculto/não obrigatório

### Toggle 2 — Modo Encomenda (novo)
- `modoEncomenda`: boolean, default `false`
- Quando **ativado**, exibe `#camposEncomenda` com:
  - **Data da encomenda** (`#dataEncomenda`, date input) — obrigatório
  - **Horário da encomenda** (`#horarioEncomenda`, time input) — obrigatório
  - **Valor do sinal/entrada** (`#valorSinal`, number input, R$) — opcional
- Quando **desativado**, `#camposEncomenda` oculto e campos não validados
- Os dois toggles são independentes: um pedido pode ser "Entrega + Encomenda", "Retirada + Encomenda", "Entrega normal", etc.

### Mapeamento de IDs removidos → substituídos

O bloco `#camposRetirada` e seus filhos são **removidos** do HTML. Todas as referências no JS devem ser atualizadas:

| ID removido | Substituto | Funções afetadas |
|---|---|---|
| `#camposRetirada` | `#camposEncomenda` | `novoPedido`, listener do toggle |
| `#dataRetirada` | `#dataEncomenda` | `copiarWhatsApp`, `imprimirPedido`, `novoPedido` |
| `#horarioRetirada` | `#horarioEncomenda` | `copiarWhatsApp`, `imprimirPedido`, `novoPedido` |
| `#observacoesRetirada` | **removido sem substituto** | `copiarWhatsApp`, `imprimirPedido`, `novoPedido` |
| listener `toggleTipoPedido` que mostrava `#camposRetirada` | agora controla apenas visibilidade de endereço | toggle listener |

**Decisão sobre `#observacoesRetirada`:** o campo era usado para anotações específicas de retirada/encomenda. Com a nova estrutura, esse uso é absorvido pelo campo `#observacoesPedido` já existente (observações gerais do pedido). Não há substituto direto — as observações de encomenda ficam no campo geral.

---

## 2. Campo de Funcionário

- **Select `#funcionario`** adicionado na seção de forma de pagamento
- Sempre visível para **todos os pedidos** (modo encomenda ou não)
- **Obrigatório** — a opção padrão é `<option value="">Selecione o funcionário</option>` (valor vazio)
- `verificarValidadePedido()` valida que `funcionario.value !== ''`
- Opções: Ana, Angelo, Matheus, Jorge Lucas, Jorge Issao, Poleana, Edurvirgem
- Nota de manutenção: lista hard-coded no HTML; futuras alterações de equipe requerem edição manual do select

---

## 3. Item do Tipo Bolo

No formulário de adicionar item, um **checkbox `#ehBolo`** é adicionado como **elemento único compartilhado**, posicionado **após os dois blocos de layout responsivo** (mobile e desktop) e **antes do botão `#adicionarItem`**. Isso garante que o checkbox seja visível em ambos os layouts sem duplicação. Quando marcado, a seção `#camposBolo` expande inline (também fora dos blocos responsivos):

| Campo | ID | Tipo | Obrigatório |
|---|---|---|---|
| Formato | `#boloFormato` | select: Redondo / Quadrado | Sim |
| Peso (kg) | `#boloKg` | number (step=0.5) | Sim |
| Sabor | `#boloSabor` | text | Sim |
| Decoração específica | `#boloDecoracao` | textarea | Não |
| Papel arroz (+R$ 20) | `#boloPapelArroz` | checkbox | — |

**Layout responsivo:** Os campos de bolo ficam em um bloco único abaixo dos campos de item existentes, visível tanto no layout mobile quanto no desktop. Não replicar em duas versões — usar apenas uma versão dos campos de bolo (responsividade via Tailwind).

**Regra de negócio:** marcar `#boloPapelArroz` soma R$ 20,00 ao item como `valorAdicional = 20`.

**Exibição na lista de itens (`atualizarListaItens`):**
```
🎂 1 un - Bolo Redondo 2kg - Chocolate | Decoração: rosas | +R$20 papel arroz
```
`atualizarListaItens()` deve detectar `item.bolo` e renderizar template diferenciado — o campo `descricao` armazena apenas o texto livre, não a string formatada completa.

**Edição de item (`editarItem`):** deve restaurar todos os campos de bolo além dos campos base (quantidade, unidade, descrição). Se o item tem `bolo`, marca `#ehBolo`, exibe `#camposBolo` e preenche todos os sub-campos.

---

## 4. Validação

`verificarValidadePedido()` é expandida para contemplar todas as regras:

| Condição | Regra |
|---|---|
| Modo entrega (`!modoRetirada`) | `rua` e `cidade` obrigatórios |
| Modo retirada (`modoRetirada`) | Endereço não obrigatório |
| Modo encomenda ativo | `#dataEncomenda` e `#horarioEncomenda` obrigatórios |
| Item sendo adicionado com `#ehBolo` marcado | `#boloFormato`, `#boloKg` e `#boloSabor` obrigatórios — validar em `adicionarItem()` |
| Sempre | `#funcionario` com valor não vazio |

---

## 5. Modelo de Dados (IndexedDB)

**Estratégia de migração:** `DB_VERSION` permanece em `1`. Os novos campos são aditivos — registros antigos sem `funcionario` ou `encomenda` retornarão `undefined` nessas propriedades, tratados com guard `|| ''` / `|| null` no código de exibição. Não há mudança estrutural nos object stores.

### Pedido
```js
{
  codigo, data, cliente, itens, formaPagamento, valorTroco, observacoes,
  funcionario: "string",        // NOVO — sempre presente (string não vazia)
  encomenda: {                  // NOVO — apenas se modoEncomenda = true
    data: "YYYY-MM-DD",
    horario: "HH:MM",
    sinal: number | null        // null se não informado
  } | null
}
```

### Item (quando bolo)
```js
{
  id, quantidade, unidade, descricao,
  bolo: {                       // NOVO — presente apenas se ehBolo = true
    formato: "redondo" | "quadrado",
    kg: number,
    sabor: "string",
    decoracao: "string" | null,
    papelArroz: boolean
  } | null,
  valorAdicional: number | null // 20 se papelArroz, null caso contrário
}
```

Nota: `valorBolo` removido do modelo — o único valor automático é o `valorAdicional` de R$20 do papel arroz. Valor total do bolo fica para melhoria futura.

---

## 6. Impacto nas Saídas

### Mensagem WhatsApp (`copiarWhatsApp`)
```
👤 *Atendente:* {funcionario}

🎂 *ENCOMENDA*
📅 *Data:* {dataEncomenda formatada}
⏰ *Horário:* {horarioEncomenda}
💰 *Sinal:* R$ {valorSinal} (ou omitir se não informado)

🛍️ *Itens:*
• 1 un - Bolo Redondo 2kg - Chocolate (papel arroz +R$20)
• ...
```

### Cupom Térmico (`imprimirPedido`)
- Adicionar bloco `<div id="printEncomenda">` ao `#printArea`
- Exibir funcionário, dados da encomenda e detalhes do bolo quando presentes

### Histórico de Pedidos (`carregarHistoricoPedidos`)
- Badge "ENCOMENDA" exibido com guard: `pedido.encomenda ? '<span>ENCOMENDA</span>' : ''`
- Registros antigos sem `encomenda` não exibem badge

### CSV (`exportarCSV`)
Colunas expandidas:
```
Codigo, Data, Funcionario, Cliente, Telefone, Endereco, Tipo, DataEncomenda, HorarioEncomenda, Sinal, Itens, Pagamento, Observacoes
```
- `Funcionario`: `pedido.funcionario || ''`
- `Tipo`: três valores possíveis:
  - `"Encomenda"` quando `pedido.encomenda != null` (independente de modoRetirada)
  - `"Retirada"` quando `pedido.encomenda == null` e `pedido.modoRetirada == true`
  - `"Entrega"` quando `pedido.encomenda == null` e `pedido.modoRetirada == false` (ou ausente)
  - Nota: salvar `modoRetirada` no objeto `pedido` para permitir esse cálculo no CSV
- `DataEncomenda`, `HorarioEncomenda`, `Sinal`: campos do objeto `pedido.encomenda` ou `''`

---

## 7. Reset — Novo Pedido (`novoPedido`)

Além dos campos já resetados, adicionar:
```js
// Novas variáveis
modoEncomenda = false;

// Toggle encomenda
toggleModoEncomenda.checked = false;
camposEncomenda.classList.add('hidden');

// Campos de encomenda
document.getElementById('dataEncomenda').value = '';
document.getElementById('horarioEncomenda').value = '';
document.getElementById('valorSinal').value = '';

// Funcionário
document.getElementById('funcionario').value = '';

// Campos de bolo (estado do formulário)
document.getElementById('ehBolo').checked = false;
document.getElementById('camposBolo').classList.add('hidden');
document.getElementById('boloFormato').value = 'redondo';
document.getElementById('boloKg').value = '';
document.getElementById('boloSabor').value = '';
document.getElementById('boloDecoracao').value = '';
document.getElementById('boloPapelArroz').checked = false;
```

---

## 8. Recuperar Pedido (`recuperarPedido`)

Além dos campos já restaurados, adicionar:
```js
// Funcionário
document.getElementById('funcionario').value = pedido.funcionario || '';

// Encomenda
if (pedido.encomenda) {
    modoEncomenda = true;
    toggleModoEncomenda.checked = true;
    camposEncomenda.classList.remove('hidden');
    document.getElementById('dataEncomenda').value = pedido.encomenda.data || '';
    document.getElementById('horarioEncomenda').value = pedido.encomenda.horario || '';
    document.getElementById('valorSinal').value = pedido.encomenda.sinal || '';
} else {
    modoEncomenda = false;
    toggleModoEncomenda.checked = false;
    camposEncomenda.classList.add('hidden');
}
// Nota: campos de bolo por item são restaurados via atualizarListaItens() + editarItem()
```

---

## 9. Arquivos Modificados

Apenas `index.html`.

### Mudanças no HTML
1. **Bloco "Tipo de Pedido":** remover `#camposRetirada` atual; adicionar toggle Modo Encomenda + seção `#camposEncomenda`
2. **Seção de pagamento:** adicionar select `#funcionario` com placeholder e 7 opções
3. **Formulário de itens:** adicionar checkbox `#ehBolo` + seção `#camposBolo` (única versão, sem duplicar para mobile/desktop)
4. **`#printArea`:** adicionar `<div id="printEncomenda">` para dados de encomenda no cupom

### Mudanças no JavaScript
| Função/Handler | Alteração |
|---|---|
| Estado global | Adicionar `modoEncomenda = false` |
| `toggleTipoPedido` listener | Manter — controla apenas endereço |
| Novo `toggleModoEncomenda` listener | Mostrar/ocultar `#camposEncomenda`, atualizar `modoEncomenda` |
| Novo `#ehBolo` listener | Mostrar/ocultar `#camposBolo` |
| `verificarValidadePedido()` | Adicionar validações: funcionário, encomenda (data/horário), bolo (via flag no form) |
| `adicionarItem()` | Capturar campos de bolo quando `#ehBolo` marcado; validar campos obrigatórios do bolo |
| `limparCamposItem()` | Resetar `#ehBolo` e `#camposBolo` após adicionar item |
| `atualizarListaItens()` | Detectar `item.bolo` e renderizar template diferenciado com 🎂 |
| `editarItem()` | Restaurar campos de bolo quando item tem `bolo` |
| `copiarWhatsApp()` | Incluir funcionário, encomenda e detalhes de bolo |
| `imprimirPedido()` | Incluir funcionário, encomenda e bolo no cupom; salvar no IndexedDB com novos campos |
| `novoPedido()` | Resetar todos os novos campos (ver Seção 7) |
| `recuperarPedido()` | Restaurar todos os novos campos (ver Seção 8) |
| `carregarHistoricoPedidos()` | Exibir badge ENCOMENDA com guard para registros antigos |
| `exportarCSV()` | Adicionar colunas Funcionario, Tipo, DataEncomenda, HorarioEncomenda, Sinal |

---

## 10. Melhorias Futuras

- Listagem dedicada de encomendas com filtro por data de encomenda
- Alertas quando uma encomenda está próxima da data de retirada/entrega
- Campo de valor total do bolo (além do sinal)
- Interface para manutenção da lista de funcionários (atualmente hard-coded)
