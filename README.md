#  Sistema de Controle de Estoque e Vendas para Mercado

Pr

## Funcionalidades

- Cadastro, listagem, edição e desativação de produtos
- Entrada e ajuste de estoque, com histórico de movimentações
- Registro de vendas com vários itens, forma de pagamento (dinheiro, cartão, pix) e troco
- Bloqueio de venda sem estoque suficiente
- Cadastro de clientes (CPF validado) e cartão da loja com desconto e pontos
- Cancelamento e devolução de vendas
- Alerta de produtos abaixo do estoque mínimo
- Site com Flask: PDV, produtos, clientes, painel de estoque baixo e relatórios

*(Funcionalidades marcadas conforme as fases forem concluídas. Veja o roteiro abaixo.)*

## Tecnologias

- Python (funções, dicionários, módulos)
- SQLite, com SQL puro (sem ORM no início)
- Flask + HTML/CSS com templates Jinja2 (Fase 4)
- pytest para testes
- Git/GitHub

## Como rodar

```bash
# clonar o repositório
git clone <url-do-repositorio>
cd <nome-da-pasta>

# criar e ativar o ambiente virtual
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows

# instalar dependências
pip install -r requirements.txt

# criar o banco com dados fictícios
python <script-de-criacao>.py   # nome do script a definir
```

## Modelagem do banco

| Tabela | Campos principais |
|---|---|
| `produtos` | id, nome, codigo_barras (UNIQUE), preco_centavos, quantidade, estoque_minimo, ativo |
| `clientes` | id, nome, cpf (UNIQUE, só dígitos), telefone, data_cadastro, pontos, ativo |
| `vendas` | id, data_hora, cliente_id (opcional), forma_pagamento, desconto_centavos, total_centavos, status (concluída/cancelada) |
| `itens_venda` | id, venda_id, produto_id, quantidade, preco_unitario_centavos (preço no momento da venda) |
| `movimentacoes_estoque` | id, produto_id, tipo (entrada, venda, devolução, ajuste), quantidade, data_hora, venda_id (opcional) |
| `movimentacoes_pontos` | id, cliente_id, pontos, motivo, venda_id (opcional), data_hora |

Todas as tabelas são criadas desde o início, mesmo que algumas só sejam usadas em fases posteriores.

### Decisões de modelagem

- **Dinheiro em centavos, como inteiro** (R$ 4,50 vira `450`). O SQLite não tem tipo decimal e `float` gera erros de arredondamento. A conversão acontece só na tela.
- **Nada é apagado, apenas desativado** (`ativo`), para não quebrar o histórico de vendas.
- **Uma única fonte da verdade para o estoque:** toda mudança (venda, entrada, devolução, ajuste) gera uma linha em `movimentacoes_estoque`, e a quantidade atual só é alterada por esse caminho. O mesmo vale para os pontos do cartão.
- **Cliente opcional:** na compra normal não há cadastro, e a venda fica com `cliente_id` vazio.
- **Preço guardado em `itens_venda`**, porque o preço do produto muda com o tempo.
- **Desconto registrado na venda**, para explicar o total depois.
- **Datas em texto ISO** (`2026-10-02 14:30:00`), sempre no mesmo padrão.
- **CPF só com dígitos**, para que busca e unicidade funcionem.
- **Venda só por unidade** no MVP. Produtos por peso ficam como melhoria futura (peso em gramas, inteiro).

### Regras no próprio banco

O Python pode ter bugs, e o banco é a última barreira:

- `PRAGMA foreign_keys = ON` a cada conexão (o SQLite vem com isso desligado)
- `CHECK (quantidade >= 0)` e `CHECK (preco_centavos > 0)`
- `UNIQUE` em `codigo_barras` e `cpf`

## Roteiro (fases)

- [ ] **Fase 0: preparação**: repositório, ambiente virtual, `.gitignore`, README, modelagem no papel, script de criação do banco com dados fictícios
- [ ] **Fase 1: produtos e estoque (terminal)**: CRUD de produtos com validações, entrada de estoque, ajuste com motivo
- [ ] **Fase 2: vendas (terminal, com testes)**: venda com vários itens e troco, tudo em transação, bloqueio sem estoque, cliente opcional, testes com pytest
- [ ] **Fase 3: clientes e cartão da loja**: clientes com validação de CPF, histórico de compras, desconto e pontos, alerta de estoque mínimo, cancelamento e devolução
- [ ] **Fase 4: site com Flask**: interface web sobre a lógica existente, segurança (CSRF, consultas parametrizadas, chave secreta em variável de ambiente), login com senha em hash e perfis (caixa e gerente), relatórios (mais vendidos, faturamento do dia)

## Limitações

- Projeto didático, **sem emissão de nota fiscal**.
- Usa **apenas dados fictícios**. Não use dados reais de clientes e não hospede online com dados reais.
- Venda apenas por unidade.
- **Sem tratamento de concorrência:** dois caixas vendendo a última unidade ao mesmo tempo podem gerar inconsistência.
- **Sem ferramenta de migração:** um script recria o banco com dados de exemplo. Alembic fica para depois.

## Decisões de projeto

Registro das escolhas feitas ao longo do desenvolvimento e das mudanças de ideia.

| Tema | Decisão | Motivo |
|---|---|---|
| Total da venda | *a definir* (pode ser guardado, se gravado na mesma transação dos itens) | |
| Arredondamento de descontos | *a definir* (arredondar ou truncar; ex.: 5% de 1999 centavos = 99,95) | |
| Código de barras de produto desativado | *a definir* (pode ser reaproveitado? o `UNIQUE` afeta isso) | |

## Fluxo de trabalho com Git

- Commits pequenos, com mensagens claras (ex.: `adiciona cadastro de produto`)
- Uma branch por funcionalidade (`git switch -c cadastro-produto`), depois merge

## Licença

