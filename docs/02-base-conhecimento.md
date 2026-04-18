# Base de Conhecimento

## Dados Utilizados

Descreva se usou os arquivos da pasta `data`, por exemplo:

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar temas já tratados ex: CDB, Tesouro Selic, metas... e manter continuidade na conversa. |
| `perfil_investidor.json` | JSON | Personalizar recomendações por perfil moderado, baixa aceitação a risco e meta de reserva de emergência |
| `produtos_financeiros.json` | JSON | Filtrar e sugerir produtos aderentes ex: Tesouro Selic e CDB para reserva, evitando fundos de ações. |
| `transacoes.csv` | CSV | Calcular saldo do período (positivo), gastos por categoria (moradia/alimentação) e capacidade de aporte |

> [!TIP]
> **Quer um dataset mais robusto?** Você pode utilizar datasets públicos do [Hugging Face](https://huggingface.co/datasets) relacionados a finanças, desde que sejam adequados ao contexto do desafio.

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

Nenhuma modificação nos dados originais — todos os 4 arquivos foram usados inalterados para garantir fidelidade e auditabilidade. O agente apenas deriva variáveis calculadas em memória durante a execução:

Saldo do período: entradas - saídas (R$ 5k - R$ 2.5k ≈ positivo).

Gastos por categoria: groupby no Pandas (top: moradia R$1.38k, alimentação R$570, transporte R$295).

Progresso da meta: (reserva_atual / meta) = 66.7% (R$10k / R$15k).

Produtos aderentes: filtro por risco baixo/médio + match com "reserva de emergência" e "segurança".

Essas derivações são reprodutíveis e não alteram os arquivos fonte.

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

Os dados são carregados uma única vez no startup via @st.cache_data no Streamlit, usando pd.read_csv() para CSVs e json.load() para JSONs. Isso garante performance e consistência durante a sessão. Caminho relativo: BASE_DIR / "data" / "arquivo.ext".

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Consultados dinamicamente pelo orquestrador Python:

Pré-processamento: Calcula resumo financeiro e lista produtos aderentes.

Matching por intenção: Usa regras if/else para mapear pergunta → dados relevantes (ex: "gastos?" → resumo transações).

Resposta grounded: Monta string com fatos reais + explicação + limites.

Pronto para LLM: O formato atual é rules-based; para LLM, os dados seriam injetados no user prompt após o system prompt de segurança.



---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

```
Dados do Cliente:
- Nome: João Silva (32 anos, analista de sistemas)
- Perfil: Moderado | Aceita risco: Não
- Renda mensal: R$ 5.000 | Reserva atual: R$ 10.000
- Meta reserva: R$ 15.000 (até 2026-06) | Progresso: 66.7%

Resumo Transações (período mockado):
- Entradas: R$ 5.000 (salário)
- Saídas: R$ 2.489 (moradia R$1.380, alimentação R$570, transporte R$295)
- Saldo: +R$ 2.511

Histórico Atendimento:
- Temas: CDB, Tesouro Selic, metas financeiras (todos resolvidos)

Produtos Aderentes (risco baixo/médio, foco reserva):
- Tesouro Selic (100% Selic, R$30 min, reserva/emergência)
- CDB Liquidez Diária (102% CDI, R$100 min, segurança)
- LCI/LCA (95% CDI, R$1k min, isento IR)
...
```
