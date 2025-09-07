# CLAUDE.md

Este arquivo fornece orientação ao Claude Code (claude.ai/code) ao trabalhar com código neste repositório.

## Visão Geral do Projeto

Esta é uma aplicação web de página única para gestão de pedidos do supermercado Super Negrelli. A aplicação é construída como um arquivo HTML autônomo com CSS e JavaScript incorporados - não requer processo de build.

## Arquitetura

**Aplicação de Arquivo Único**: A aplicação inteira está contida no `index.html` com:
- Estrutura HTML para interface do formulário de pedidos
- CSS incorporado com Tailwind CDN para estilização e efeitos glass morphism
- JavaScript vanilla para toda a funcionalidade
- IndexedDB para persistência local de dados

**Componentes Principais**:
- Coleta de dados do cliente com busca baseada no telefone
- Gestão de itens do pedido com quantidade, unidade, descrição e preço
- Seleção de método de pagamento (cartões, Pix, dinheiro, etc.)
- Busca de endereço via API ViaCEP
- Geração de mensagem WhatsApp para pedidos
- Funcionalidade de impressão de cupom térmico
- Exportação CSV para pedidos diários

**Armazenamento de Dados**:
- Usa IndexedDB com dois object stores:
  - `clientes` (customers) - chaveado por número de telefone
  - `pedidos` (orders) - chaveado por código de pedido gerado
- Nenhum banco de dados externo ou servidor necessário

## Funcionalidades Principais

**Gestão de Pedidos**:
- Busca de cliente baseada no telefone e preenchimento automático
- Busca de CEP para completar endereço automaticamente
- Lista dinâmica de itens com cálculo de total em tempo real
- Múltiplos métodos de pagamento com campos condicionais (ex: valor do troco para dinheiro)

**Opções de Saída**:
- Cópia de mensagem formatada para WhatsApp para área de transferência
- Impressão de cupom térmico (formato 80mm)
- Exportação CSV diária de todos os pedidos

**Experiência do Usuário**:
- Validação de formulário em tempo real
- Navegação por teclado (Enter para adicionar itens, progressão de foco automática)
- Design responsivo com estilização glass morphism
- Mensagens de status para feedback do usuário

## Desenvolvimento

**Executando a Aplicação**:
```bash
# Simplesmente abra o index.html em um navegador web
# Sem processo de build, sem dependências para instalar
```

**Sem Ferramentas de Build**: Esta é uma aplicação vanilla HTML/CSS/JavaScript. Não há scripts npm, bundlers ou etapas de transpilação.

**Testes**: Apenas testes manuais - abra no navegador e teste a funcionalidade.

**Compatibilidade de Navegador**: Navegadores modernos necessários para IndexedDB, CSS backdrop-filter e recursos ES6+.

## Estrutura de Arquivos

```
├── index.html          # Aplicação completa (HTML + CSS + JS)
├── package-lock.json   # Mínimo, sem dependências reais
└── .claude/           # Configurações do Claude Code
    └── settings.local.json
```

## Padrões de Código Principais

**Operações de Banco de Dados**: Todas as operações IndexedDB usam wrappers Promise em torno da API baseada em callback.

**Manipulação de Formulário**: Arquitetura orientada a eventos com validação em tempo real e formatação automática (telefone, CEP).

**Gerenciamento de Estado**: Variáveis globais simples para estado do pedido (`itensCompra`, `valorTotalPedido`, `formaPagamentoSelecionada`).

**Integração de API**: Chamadas baseadas em Fetch para ViaCEP para busca de endereço.

**Funcionalidade de Impressão**: Usa regras CSS `@media print` e `window.print()` para formatação de cupom térmico.