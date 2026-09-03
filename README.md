# 📦 Dashboard Executivo de Logística & Estoque

Dashboard em Power BI para diagnóstico de capital imobilizado, risco de ruptura e oportunidades de otimização de estoque — projeto de portfólio.

![Dashboard preview](.dashboard-preview.jpg)

> ⚠️ **Sobre o dado**: este projeto usa um dataset simulado do Kaggle para fins de demonstração técnica. As análises e recomendações abaixo são construídas *como se* fosse uma operação real, para exercitar o processo analítico completo — da pergunta de negócio ao dashboard final.

---

## 🎯 O problema de negócio

Um gestor de logística abre esse dashboard com perguntas específicas. É isso que ele encontra:

| Pergunta do gestor | Resposta no dashboard |
|---|---|
| Quanto capital está parado em estoque? | R$ 89,57 Mi imobilizados no total |
| Isso está bem distribuído ou concentrado? | 70,3% do catálogo (R$ 77,9 Mi) está acima de 3x o ponto de reposição — excesso real, não pontual |
| Corro risco de faltar produto? | Sim: 237 itens (7,4% do catálogo) estão abaixo do ponto de reposição |
| O atendimento ao cliente está bom? | 84,97% — abaixo da meta de mercado (≥95%) |
| Onde estão as maiores oportunidades? | R$ 77,9 Mi liberáveis reduzindo o excesso de estoque |
| Por onde eu começo a agir? | Priorizar reposição dos 237 itens em risco de ruptura — sem aumentar capital total investido |

**O insight central do projeto**: excesso de estoque e risco de ruptura acontecem simultaneamente — 70% do catálogo em excesso não impede que uma fatia relevante esteja com risco de faltar. O problema não é volume de estoque, é alocação por produto.

---

## 🗂️ Fonte de dados

- **Dataset**: [Logistics Warehouse Dataset](https://www.kaggle.com/datasets/ziya07/logistics-warehouse-dataset) (Kaggle)
- **Volume**: 3.204 produtos, 23 colunas (estoque, ponto de reposição, giro, lead time, custo de estocagem, atendimento, popularidade, entre outras)
- **Escopo**: 5 categorias (Farmacêutico, Eletrônicos, Automotivo, Mercearia, Vestuário), 4 zonas de armazém

---

## 🧹 Qualidade dos dados

Antes de qualquer análise, validei a integridade da base (Python/pandas):

```python
df.isnull().sum().sum()        # 0 valores nulos em todas as 23 colunas
df.duplicated().sum()          # 0 linhas duplicadas
df['item_id'].duplicated().sum()  # 0 IDs de produto duplicados
(df['order_fulfillment_rate'] < 0).sum() | (df['order_fulfillment_rate'] > 1).sum()  # 0 taxas fora de 0-1
(df['stock_level'] < 0).sum()  # 0 estoques negativos
(df['unit_price'] <= 0).sum()  # 0 preços zerados ou negativos
```

**Resultado**: base limpa, sem nulos, duplicados ou valores fora de faixa. (Checagem equivalente também pode ser feita direto no Power Query, usando a aba **Exibir → Qualidade da coluna / Distribuição da coluna**, que mostra % de erros, vazios e valores distintos por campo sem sair do Power BI.)

---

## ⚙️ O que eu fiz — passo a passo

1. **Definição das perguntas de negócio** antes de tocar no dado — capital imobilizado, risco de ruptura, giro, atendimento.
2. **ETL no Power Query**: carga do CSV, tipagem de colunas, validação de qualidade (nulos/duplicados/faixas).
3. **Modelagem no Power BI**: colunas calculadas para classificação de status de estoque, medidas DAX para os KPIs e cards de oportunidade.
4. **Definição de critérios de negócio auditáveis** para cada categoria (ex: "excesso" = estoque acima de 3x o ponto de reposição — critério documentado e defensável, não um número arbitrário).
5. **QA das próprias medidas**: encontrei e corrigi uma medida (`Capital Baixo Giro`) que usava um threshold (giro < 0,5) que nunca era atingido por nenhum item da base — o card sempre mostrava R$ 0. Corrigido para giro < 2, alinhado ao critério usado no resto do dashboard.
6. **Design do dashboard**: KPIs no topo, visão por categoria, distribuição por status, top 10 produtos com badge de criticidade, cards de oportunidade com impacto financeiro.
7. **Validação cruzada**: conferi que todos os números do dashboard (KPIs, donut, tabela, cards) usam a mesma definição de "excesso" — evitando o erro comum de um dashboard "contar duas histórias diferentes" com o mesmo dado.

---

## 📊 Principais métricas do modelo

| Métrica | Fórmula (DAX) | Resultado |
|---|---|---|
| Capital Imobilizado Total | `SUMX(logistica, Estoque * Preço Unitário)` | R$ 89,57 Mi |
| Itens em Risco de Ruptura | `CALCULATE(COUNTROWS(logistica), Estoque < Ponto de Reabastecimento)` | 237 / 7,4% |
| Status Criticidade | `SWITCH(TRUE(), Estoque < Ponto, "Risco", Estoque > 3*Ponto, "Excesso", "Adequado")` | 70,3% / 22,3% / 7,4% |

---

## 🛠️ Stack

- **Power BI** — modelagem, DAX, visualização
- **Python (pandas)** — validação e checagem de qualidade de dados
- **Power Query** — ETL

---

## 🚀 Como reproduzir

1. Baixe o [dataset original](https://www.kaggle.com/datasets/ziya07/logistics-warehouse-dataset)
2. Abra `logistica_dashboard.pbix` no Power BI Desktop
3. Atualize a origem de dados (`Transformar Dados → Configurações da Fonte de Dados`) para o caminho local do CSV
4. Clique em **Atualizar**

---

## 📬 Contato

**Bruno de Mello Canuto**
[LinkedIn](https://www.linkedin.com/in/bruno-de-mello-canuto/)
