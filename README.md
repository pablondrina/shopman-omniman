# shopman-omniman

Kernel de pedidos omnichannel para Django. Gerencia todo o ciclo de vida de um pedido — desde a sessão de compra até o fulfillment — com suporte a múltiplos canais (balcão, delivery, WhatsApp, marketplace).

Part of the [Django Shopman](https://github.com/pablondrina/django-shopman) commerce framework.

## Domínio

O Omniman modela o fluxo completo de pedidos:

- **Session** — carrinho ativo do cliente. Items, dados de entrega, pagamento. Expira automaticamente.
- **SessionItem** — item no carrinho com SKU, quantidade, preço.
- **Order** — pedido commitado a partir da Session. Status machine com transitions configuráveis.
- **OrderItem** — item do pedido (snapshot imutável do momento do commit).
- **OrderEvent** — timeline de eventos (status changes, ações do operador, sistema).
- **Channel** — canal de venda (web, pos, whatsapp, ifood). Config JSON com cascata.
- **Directive** — fila de tarefas pós-commit (stock.hold, notification.send, payment.capture). Retry com backoff exponencial.
- **Fulfillment / FulfillmentItem** — entrega ou retirada vinculada ao pedido.
- **IdempotencyKey** — proteção contra requests duplicados.

## Services

| Service | Responsabilidade |
|---------|-----------------|
| `CommitService` | Session → Order. Copia dados, cria directives pós-commit. |
| `ModifyService` | Atualiza sessão (add/remove items, update data). |
| `WriteService` | Escrita direta em Order.data (notas internas, ajustes). |
| `ResolveService` | Resolução de referências e lookups. |

## Contribs

- `omniman.contrib.refs` — Geração de referências sequenciais (ORD-001, ORD-002...) com escopo por canal.
- `omniman.contrib.stock` — Bridge omniman↔stockman para validação de estoque no commit.

## Instalação

```bash
pip install shopman-omniman
```

```python
INSTALLED_APPS = [
    "shopman.omniman",
    "shopman.omniman.contrib.refs",   # opcional: refs sequenciais
    "shopman.omniman.contrib.stock",  # opcional: validação de estoque
]
```

## Signals

- `order_changed(sender, order, event_type, actor)` — disparado em toda mudança de status.
- `session_committed(sender, session_id, order_id)` — disparado após commit.

## Development

Desenvolvido no monorepo [django-shopman](https://github.com/pablondrina/django-shopman) em `packages/omniman/`.

```bash
git clone https://github.com/pablondrina/django-shopman.git
cd django-shopman && pip install -e packages/omniman
make test-omniman  # ~205 testes
```

## License

MIT — Pablo Valentini
