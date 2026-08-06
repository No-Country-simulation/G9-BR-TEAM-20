# Insights sobre Variáveis Textuais e Numéricas

### Hackathon ONE · Alura \+ Oracle (OCI) — G9 Team 20

Documento de apoio à apresentação final, consolidando as descobertas sobre o comportamento das variáveis textuais (descrição de transação) e numéricas (renda, endividamento, gastos) usadas nos dois modelos do projeto.

---

## 1\. Variáveis Textuais — Descrição da Transação

### O que é essa variável

A coluna `descricao` (ex: "Uber", "Netflix", "Supermercado Extra") é a única entrada do modelo de classificação de categoria (Fase 5). Todo o aprendizado do modelo depende exclusivamente do texto — não há nenhuma outra informação estrutural (não sabemos, por exemplo, o CNPJ ou o segmento oficial do estabelecimento).

### Insight 1 — Nomes de marca puros não geram sinal generalizável

**Descoberta:** um modelo treinado apenas com nomes próprios ("Extra", "Netflix", "Uber") atinge 100% de acurácia em teste com split ingênuo, mas cai para **15,7%** quando testado com estabelecimentos genuinamente nunca vistos. Nomes de marca são strings arbitrárias — não carregam nenhuma pista textual sobre a categoria a que pertencem.

**Implicação prática:** o mesmo problema existe no mundo real — é por isso que sistemas de pagamento reais usam o **MCC (Merchant Category Code)**, um código atribuído pela adquirente no momento da venda, em vez de depender só do nome fantasia.

### Insight 2 — Palavras genéricas de categoria são o sinal que realmente generaliza

**Descoberta:** ao reformular as descrições para incluir uma palavra genérica junto ao nome de marca (ex: "Supermercado Extra" em vez de só "Extra"; "Farmácia Pague Menos" em vez de só "Pague Menos"), a acurácia em estabelecimentos nunca vistos subiu de 15,7% para **46,3%**, e depois para **87,7%** após reforço adicional de vocabulário.

**Implicação prática:** em qualquer sistema real de classificação de texto curto, o vocabulário de treino precisa cobrir os **termos genéricos e funcionais** que o usuário realmente digita (ex: "Combustível", "Conta de Luz", "Remédio"), não apenas nomes de marca.

### Insight 3 — Ambiguidade de palavra-chave entre categorias prejudica a precisão

**Descoberta:** palavras que aparecem em mais de uma categoria (ex: "Restaurante" em Alimentação e em Lazer; "Seguro" em Moradia e em Transporte) confundem o modelo, gerando previsões enviesadas para uma categoria "padrão" quando o texto de teste é ambíguo.

**Implicação prática:** curadoria de vocabulário precisa checar sobreposição de palavras entre classes, não só riqueza de vocabulário dentro de cada classe isoladamente.

### Insight 4 — Redundância de palavra-chave é mais importante que volume total de vocabulário

**Descoberta:** aumentar o número total de estabelecimentos (de 98 para 148\) não resolveu, sozinho, a instabilidade de resultado entre execuções. O fator decisivo foi garantir que **cada palavra-chave genérica aparecesse em pelo menos 3-4 estabelecimentos diferentes** por categoria — reduzindo a chance de uma palavra-chave "sumir" do treino por azar do sorteio de teste.

### Insight 5 — N-gramas de 2 palavras (bigramas) ajudam a desambiguar

**Descoberta:** o vetorizador TF-IDF foi configurado com `ngram_range=(1, 2)` — considerando não só palavras isoladas, mas pares de palavras. Isso ajudou a diferenciar, por exemplo, "Seguro Residencial" (Moradia) de "Proteção Veicular Porto Seguro" (Transporte), mesmo com a palavra "Seguro" presente nos dois casos.

### Insight 6 — Avaliação de texto com vocabulário pequeno exige múltiplos testes, não um só

**Descoberta:** com um vocabulário de 150-230 palavras únicas, um único sorteio de split treino/teste produziu resultados de acurácia variando entre **76% e 93%** em execuções sucessivas, sem nenhuma mudança real no modelo ou nos dados. A causa: a "dificuldade" de cada sorteio de teste varia conforme quais estabelecimentos específicos calham de ficar de fora do treino.

**Solução metodológica:** validação cruzada com 10 rodadas (sementes diferentes), reportando média e desvio padrão — **86,79% ± 5,33%** foi a métrica final considerada confiável, substituindo qualquer leitura isolada.

### Resumo estatístico do vocabulário textual final

| Métrica | Valor |
| :---- | :---- |
| Total de transações no dataset | 10.000 |
| Estabelecimentos únicos | 161 |
| Palavras únicas no vocabulário (após tokenização) | 233 |
| Categorias | 7 |
| Acurácia média (validação cruzada, 10 rodadas) | 86,79% (± 5,33%) |
| F1-macro médio | 86,26% (± 6,56%) |

---

## 2\. Variáveis Numéricas — Perfil Financeiro e Transações

### O que são essas variáveis

Usadas em dois contextos diferentes:

- **No dataset de transações (Fase 4):** `valor` da transação, gerado a partir de médias reais do IBGE  
- **No dataset de perfil (Fase 6):** `renda_mensal`, `nivel_endividamento`, `frequencia_poupanca`, `comprometimento_gastos`, e o valor gasto em cada uma das 7 categorias

### Insight 1 — Gasto mensal agregado ≠ valor de uma transação individual

**Descoberta:** as médias do IBGE (ex: R$ 658,23/mês em Alimentação) representam a soma de várias compras no mês, não uma compra isolada. Ao gerar transações sintéticas, dividir a média mensal por uma frequência simulada de compras (8 a 20 vezes/mês, dependendo da categoria) produziu valores de transação individual muito mais realistas — caindo de uma média de R$ 658 para uma média de **R$ 54-56** por transação de Alimentação, por exemplo.

| Categoria | Valor médio por transação (R$) | Faixa observada (R$) |
| :---- | :---- | :---- |
| Moradia | 354,79 | 57 – 1.772 |
| Saúde | 169,01 | 27 – 1.452 |
| Educação | 113,69 | 18 – 564 |
| Transporte | 45,28 | 8 – 196 |
| Alimentação | 54,57 | 10 – 230 |
| Serviços | 27,66 | 3 – 160 |
| Lazer | 25,72 | 4 – 197 |

### Insight 2 — Distribuição log-normal reproduz melhor o comportamento real de gasto

**Descoberta:** valores financeiros reais (renda, gasto por transação) tendem a ter uma distribuição assimétrica — muitos valores baixos, poucos valores muito altos, nunca valores negativos. Usar `np.random.lognormal` (em vez de distribuição normal comum ou uniforme) para gerar tanto os valores de transação quanto a renda mensal sintética reproduziu esse padrão de forma mais fiel à realidade econômica.

### Insight 3 — Endividamento segue padrão de cauda longa, não distribuição uniforme

**Descoberta:** ao gerar os 5.000 perfis financeiros sintéticos, o `nivel_endividamento` foi sorteado de uma distribuição **Beta(a=2, b=5)**, deliberadamente enviesada para valores baixos — refletindo que a maioria da população não está fortemente endividada, mas com uma cauda que ainda permite casos de endividamento alto. O resultado da rotulagem confirma essa expectativa:

| Perfil financeiro | Quantidade | Percentual |
| :---- | :---- | :---- |
| Saudável | 2.205 | 44,1% |
| Em observação | 2.137 | 42,7% |
| Em risco | 658 | 13,2% |

### Insight 4 — `nivel_endividamento` é, disparadamente, a variável mais decisiva do modelo de perfil

**Descoberta:** a importância de variável extraída do Random Forest (Fase 6\) mostra uma concentração extrema de poder preditivo em apenas duas variáveis:

| Variável | Importância |
| :---- | :---- |
| `nivel_endividamento` | 69,8% |
| `comprometimento_gastos` | 15,6% |
| `renda_mensal` | 2,9% |
| `frequencia_poupanca` (codificada) | 1,9% |
| Cada categoria de gasto individual (Moradia, Transporte, etc.) | 1,2% – 1,8% cada |

**Implicação prática:** isso confirma que o modelo não está "decorando" nada — ele reconstruiu corretamente a lógica da regra de negócio que gerou os próprios rótulos, já que essas são exatamente as duas variáveis usadas nos limiares de decisão (30%/50%).

### Insight 5 — Codificação ordinal supera one-hot para variáveis com ordem natural

**Descoberta:** `frequencia_poupanca` ("Baixa", "Média", "Alta") foi codificada como valores ordinais (0, 1, 2), não como one-hot encoding. Isso preserva a relação de ordem entre as categorias, o que ajuda modelos baseados em árvore (como o Random Forest) a aprender limiares de decisão de forma mais direta do que trataria variáveis categóricas sem relação de ordem.

### Insight 6 — Variável derivada (`comprometimento_gastos`) tem mais poder preditivo que variáveis brutas de gasto por categoria

**Descoberta:** a variável calculada `comprometimento_gastos` (soma dos gastos ÷ renda mensal, em percentual) teve importância de 15,6% — muito maior que qualquer categoria de gasto isolada (todas abaixo de 2%). Isso é um exemplo concreto de **engenharia de atributos** bem sucedida: uma variável derivada capturou um padrão relevante (proporção do orçamento comprometida) que as variáveis brutas, sozinhas, não expunham tão claramente ao modelo.

### Insight 7 — Alta acurácia é confiável quando o rótulo é determinístico, suspeita quando não é

**Contraste entre as duas fases de modelagem:** no modelo de perfil (variáveis numéricas, rótulo gerado por regra determinística), uma acurácia de 99,6% é esperada e correta. No modelo de categoria (variável textual, rótulo por vocabulário), o mesmo patamar de acurácia era sinal de overfitting. A distinção depende de **como o rótulo foi gerado**, não do valor da métrica isoladamente — um ponto metodológico importante para qualquer avaliação de modelo.

---

## 3\. Quadro-Resumo para a Apresentação

| Tipo de variável | Desafio principal encontrado | Solução aplicada |
| :---- | :---- | :---- |
| Textual (descrição) | Vocabulário fechado gera falsa generalização | Palavras genéricas \+ redundância \+ validação cruzada |
| Textual (descrição) | Ambiguidade entre categorias | Curadoria manual de sobreposição \+ bigramas |
| Numérica (valor de transação) | Média agregada ≠ valor individual | Divisão por frequência simulada \+ distribuição log-normal |
| Numérica (perfil financeiro) | Ausência de rótulo pronto | Regra de negócio fundamentada em dado real (BCB) |
| Numérica (perfil financeiro) | Variáveis brutas com baixo poder preditivo isolado | Engenharia de atributo derivado (`comprometimento_gastos`) |

