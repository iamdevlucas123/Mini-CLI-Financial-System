# CLAUDE.md — Mini-CLI-Financial-System

Sistema de controle financeiro 100% executado pelo terminal, escrito em Python.

**Este é um projeto de estudo.** O produto (controlar receitas e despesas) é o pretexto; o objetivo real é aplicar na prática matemática, Python moderno, estruturas de dados, algoritmos, complexidade (Big-O) e análise numérica. Toda decisão deve favorecer o aprendizado e a clareza, não o atalho.

---

## Como o Claude deve trabalhar neste projeto

- **Explique antes de implementar.** Para cada conceito novo (estrutura, fórmula, recurso da linguagem), explique o que é, por que resolve o problema e qual o custo antes de escrever código.
- **Um tópico por vez.** Siga o roadmap na ordem. Só avance quando o tópico atual estiver implementado, testado e analisado. Não antecipe fases futuras.
- **Não entregue a solução pronta das estruturas de dados** (DynamicArray, HashMap, Heap, Tree) sem que eu peça. Prefira: explicar a ideia → propor a interface/assinaturas → deixar eu implementar → revisar meu código apontando bugs, casos de borda e custo.
- **Sempre justifique a escolha de estrutura/algoritmo** comparando com pelo menos uma alternativa e sua complexidade.
- **Aponte onde cada conceito matemático aparece** no código (ex.: "isto é crescimento exponencial", "aqui usamos radiciação para a taxa equivalente").
- Revisões devem ser honestas: se algo está errado, ineficiente ou numericamente frágil, diga claramente.
- Responda em português.

---

## Stack e decisões

| Item | Decisão |
|---|---|
| Linguagem | Python 3.12+ |
| Ambiente | `venv` (`.venv/` na raiz, fora do git) |
| Empacotamento | `pyproject.toml` com layout `src/`, entry point `finance` |
| CLI | `argparse` (stdlib) — sem frameworks de CLI |
| Persistência | Arquivo JSON local, escrito via context manager com escrita atômica |
| Dinheiro | `decimal.Decimal` — **nunca `float` para valores monetários** |
| Numérico | `numpy` apenas para estatística/análise (relatórios, comparação entre meses, benchmarks) |
| Testes | `pytest` |
| Qualidade | `mypy --strict` e `ruff` |

Dependências externas devem ser mínimas. Antes de adicionar qualquer biblioteca, pergunte.

---

## Comandos

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -e ".[dev]"

finance --help                     # ou: python -m finance --help
pytest                             # todos os testes
pytest tests/structures -v         # só estruturas de dados
mypy src
ruff check . && ruff format .
python -m benchmarks.run           # benchmarks de complexidade
```

Antes de considerar qualquer tarefa concluída: `pytest`, `mypy src` e `ruff check .` devem passar.

---

## Estrutura do projeto

```
mini-cli-financial-system/
├── pyproject.toml
├── README.md
├── CLAUDE.md
├── src/finance/
│   ├── __main__.py          # python -m finance
│   ├── cli.py               # argparse, subcomandos, saída formatada
│   ├── models.py            # dataclasses: Transaction, Category, Budget
│   ├── money.py             # Decimal, arredondamento, conversão de moedas
│   ├── errors.py            # exceções do domínio
│   ├── storage.py           # leitura/escrita JSON (context manager)
│   ├── finmath/
│   │   ├── percent.py       # porcentagem, razão, proporção
│   │   ├── interest.py      # juros simples, compostos, taxa equivalente
│   │   ├── installments.py  # parcelas (Tabela Price, parcelamento simples)
│   │   └── growth.py        # linear, quadrático, exponencial, logaritmos
│   ├── structures/          # IMPLEMENTAÇÕES MANUAIS
│   │   ├── dynamic_array.py
│   │   ├── hash_map.py
│   │   ├── heap.py
│   │   └── category_tree.py
│   └── services/
│       ├── ledger.py        # cadastro, histórico, busca por ID
│       ├── reports.py       # relatório mensal, comparação entre meses
│       └── budget.py        # orçamento por categoria
├── tests/                   # espelha src/finance/
├── benchmarks/              # medições empíricas de complexidade
└── docs/
    └── complexidade.md      # análise Big-O de cada estrutura e algoritmo
```

Regra de dependência: `cli` → `services` → (`structures`, `finmath`, `models`, `money`). `structures` e `finmath` não importam nada de `services` ou `cli`.

---

## Estruturas de dados (implementação manual)

| Problema | Estrutura | Operação-chave | Complexidade esperada |
|---|---|---|---|
| Histórico de transações | DynamicArray | append / acesso por índice | O(1) amortizado / O(1) |
| Buscar transação por ID | HashMap | get / put | O(1) médio, O(n) pior |
| Ordenar despesas / top-k | Heap binário | push / pop | O(log n); heapsort O(n log n) |
| Categorizar transações | Árvore n-ária | inserir / subtotal por categoria | O(profundidade) / O(n) DFS |

Regras obrigatórias:

- **Proibido usar** `dict`, `set`, `heapq`, `sorted`, `list.sort` ou `bisect` **dentro** das implementações de `structures/`. Uma `list` Python de tamanho fixo pode ser usada apenas como "bloco de memória" bruto (ex.: `[None] * capacity`).
- `DynamicArray`: crescimento por duplicação de capacidade; documentar por que o append é O(1) amortizado.
- `HashMap`: função de hash + tratamento de colisões (encadeamento ou endereçamento aberto) + redimensionamento por fator de carga. Documentar por que o pior caso é O(n).
- `Heap`: baseado em array (pai `(i-1)//2`, filhos `2i+1` e `2i+2`); implementar `heapify` O(n).
- `CategoryTree`: categorias hierárquicas (ex.: `Moradia > Aluguel`), subtotal recursivo por subárvore.
- Cada estrutura deve ter **genéricos com type hints** (`TypeVar`/`Generic` ou a sintaxe `class X[T]` do 3.12), implementar `__len__` e, quando fizer sentido, `__iter__` como **generator**.
- Os testes usam as estruturas nativas (`dict`, `heapq`, `sorted`) **como oráculo** para validar os resultados das implementações manuais.

---

## Matemática: onde cada conceito vive

| Conceito | Aplicação no sistema |
|---|---|
| Operações básicas, decimais | Saldo, somas, arredondamento com `Decimal` (`ROUND_HALF_EVEN` vs `ROUND_HALF_UP`) |
| Frações | Divisão de um valor em parcelas; distribuição do centavo residual |
| Razão e proporção | Peso de cada categoria no total; comparação entre meses |
| Porcentagem | % do orçamento consumido, variação percentual mês a mês |
| Potenciação | Juros compostos: `M = C·(1+i)^n` |
| Radiciação | Taxa equivalente: `i_mensal = (1+i_anual)^(1/12) − 1`; taxa média de crescimento |
| Logaritmos | Tempo para atingir uma meta: `n = log(M/C) / log(1+i)`; tempo para dobrar |
| Notação científica | Exibição de valores muito grandes/pequenos; erro de ponto flutuante (`float` epsilon) |
| Crescimento linear | Juros simples; custo O(n) |
| Crescimento quadrático | Soma de aportes crescentes (progressão aritmética); algoritmos O(n²) nos benchmarks |
| Crescimento exponencial | Juros compostos; comparação linear × exponencial ao longo do tempo |

Parcelas com juros usam a Tabela Price: `PMT = P · i / (1 − (1+i)^−n)`.

Toda função em `finmath/` deve ter docstring com a fórmula, o significado de cada variável e um exemplo numérico que também vire teste.

---

## Análise numérica

- Valores monetários: sempre `Decimal` criado a partir de `str` ou `int` (`Decimal("0.10")`, nunca `Decimal(0.1)`).
- Arredondar somente na borda (exibição/persistência), não em cálculos intermediários.
- Parcelamento: a soma das parcelas deve ser **exatamente** igual ao total — o centavo residual vai na última (ou primeira) parcela, e há teste para isso.
- Quando uma fórmula exigir `float` (log, raiz fracionária), converter conscientemente, documentar a perda de precisão e voltar para `Decimal` no resultado.
- Manter ao menos um teste/demonstração que mostre `0.1 + 0.2 != 0.3` com `float` e a correção com `Decimal`.

---

## Complexidade e Big-O

Toda estrutura e todo algoritmo não trivial deve documentar, na docstring e em `docs/complexidade.md`:

- Tempo: melhor caso, caso médio, pior caso
- Uso de memória
- Pelo menos uma alternativa comparada (ex.: busca por ID em lista O(n) × HashMap O(1) médio)

Os benchmarks em `benchmarks/` devem:

- Medir com `time.perf_counter` (encapsulado num context manager `timer`) para tamanhos crescentes de entrada (ex.: 10³ a 10⁶)
- Usar `numpy` para ajustar a inclinação em escala log-log e estimar empiricamente a ordem de crescimento
- Confirmar (ou contradizer) a análise teórica — e explicar a diferença, se houver

---

## Convenções de código

- `@dataclass(frozen=True, slots=True)` para modelos imutáveis (ex.: `Transaction`).
- Validação no `__post_init__` (valor positivo, data válida, categoria existente), levantando exceções de `errors.py`.
- Type hints em todo o código; nada de `Any` sem justificativa.
- Exceções específicas do domínio (`InvalidAmountError`, `TransactionNotFoundError`, etc.); a CLI captura e mostra mensagem amigável, sem stack trace.
- Generators para percorrer transações com filtros (por mês, categoria, tipo) sem criar listas intermediárias.
- Context managers para arquivo de dados (escrita atômica: escreve em temporário e renomeia) e para medição de tempo.
- Funções pequenas e puras em `finmath/`; efeitos colaterais só em `storage.py` e `cli.py`.
- Nomes de código em inglês; mensagens da CLI e documentação em português.

---

## Testes

- Todo módulo tem teste correspondente em `tests/`.
- Estruturas de dados: testar casos de borda (vazio, um elemento, colisões, redimensionamento, remoção do último, duplicatas).
- Matemática: testar contra valores calculados à mão ou de referência conhecida.
- Usar `tmp_path` do pytest para testes de persistência; nunca tocar no arquivo de dados real.

---

## Roadmap (seguir em ordem)

- [ ] **Fase 1 — Fundação:** venv, `pyproject.toml`, `models.py` com dataclasses, `money.py` com `Decimal`, exceções, primeiro teste
- [ ] **Fase 2 — Array dinâmico:** `DynamicArray` + histórico de transações + cadastrar receita/despesa + saldo
- [ ] **Fase 3 — Hash Map:** `HashMap` + busca por ID + benchmark lista × hash map
- [ ] **Fase 4 — Matemática financeira:** porcentagem, juros simples/compostos, parcelas, taxa equivalente, logaritmos, conversão de moedas
- [ ] **Fase 5 — Heap:** `Heap` + ordenar despesas + top-k maiores gastos + comparação com ordenação O(n²)
- [ ] **Fase 6 — Árvore:** `CategoryTree` + criar categorias + categorizar transações + subtotal por categoria
- [ ] **Fase 7 — Relatórios:** relatório mensal, orçamento, comparação entre meses (com `numpy`)
- [ ] **Fase 8 — Entrega:** persistência completa, benchmarks finais, `docs/complexidade.md`, empacotamento e instalação via `pip`

Ao concluir uma fase, marque-a aqui e registre em `docs/complexidade.md` o que foi aprendido.
