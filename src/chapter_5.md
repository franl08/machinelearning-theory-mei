# Teórica 05

## Árvores de Decisão

- Altamente instáveis;
  - A troca de um valor na árvore poderá implicar uma mudança em toda ela.
- Podem ser:
  - Árvores de Classificação;
  - Árvores de Regressão.
- Por convenção, para a esquerda representa-se o verdadeiro e para a esquerda representa-se o falso.

### Árvores de Classificação

#### Medidas de Impureza

- Impuridade de Gini;
- Entropia;
- Ganho de Informação.

##### Impuridade de Gini

$$G_{folha} = 1 - (P_{ac._{1}})^2 - (P_{ac._{2}})^2 - \cdots - (P_{ac._{N}})^2$$

$$Total = medias\ pesadas\ da\ impureza\ de\ Gini\ nas\ folhas$$

**Com atributos contínuos:**
1. Pega-se no atributo contínuo e ordena-se de forma crescente;
2. Calcula-se a média entre os valores adjacentes;
3. Calcula-se a impureza para cada um dos valores médios calculados;
4. O corte na construção da árvore deve ser feito no valor que apresentar o menor resultado de Gini.

#### Construção da Árvore:

1. Calcular o resultado da impureza de Gini;
2. Se o nodo tiver um resultado mais baixo, não se separa mais os nodos e esse torna-se folha da árvore;
3. Caso contrário, escolhe-se a separação com o menor valor de impureza.

### Árvores de Regressão

**Método:**
1. Para cada possível *threshold* calcula-se a média dos *samples* à direita e à esquerda, calculando-se, em seguida, a soma dos erros quadrados para cada *sample*;
2. Seleciona-se o *threshold* com a menor soma dos erros quadrados para cada ramo;
3. Quando o número de *samples* for menor que o valor pré-definido, então será uma folha com o valor igual à média das *samples*.

**Com atributos múltiplos:**
1. Calcula-se o mínimo da soma dos erros quadrados para cada atributo;
2. Seleciona-se o atributo e o *threshold* com a menor soma dos erros quadrados para cada ramo;
3. Quando o número de *samples* for menor que o valor pré-definido, então será uma folha com o valor igual à média das *samples*.

### *Pruning*

- Uma árvore de decisão irá caír sempre em *overfitting* se a deixarmos crescer até à sua máxima profundidade;
- Para evitar isto, podemos efetuar *pre-prunning* (parar o crescimento cedo) ou *post-prunning* (após o treino completo da árvore).

#### *Pre-Prunning*

- `min_sample_split` $\rightarrow$ é o número mínimo de *samples* para cada *split*.

```py
min_samples_split_grid_search = GridSearchCV(
    estimator=DecisionTreeClassifier(random_state=2020),
    scoring=make_scorer(accuracy_scorer),
    param_grid=ParameterGrid(
        {"min_samples_split":[[min_samples_split] for min_samples_split in np.arange(EPS, 1, 0.025)]}
    ),
)
```

- `min_sample_leaf` $\rightarrow$ número mínimo de *samples* para ser uma folha.

```py
min_samples_split_grid_search = GridSearchCV(
    estimator=DecisionTreeClassifier(random_state=2020),
    scoring=make_scorer(accuracy_scorer),
    param_grid=ParameterGrid(
        {"min_samples_leaf":[[min_samples_leaf] for min_samples_leaf in np.arange(0.000001, 1, 0.025)]}
    ),
)
```

#### *Post-Prunning*

- Define uma profundidade máxima para a árvore.

```py
full_tree = DecisionTreeClassifier(random_state=2020)
full_tree.fit(x_train, y_train)

print(full_tree.get_depth())
print(full_tree.get_n_leaves())

max_depth_grid_search = GridSearchCV(
    estimator=DecisionTreeClassifier(random_state=2020),
    scoring=make_scorer(accuracy_scorer),
    param_grid=ParameterGrid(
        {"max_depth":[[max_depth] for max_depth in range(1, max_depth + 1)]}
    ),
)
```

- O *prunning* inicia-se com uma árvore que não tenha sido *prunned*, utilizando uma sequência de sub-árvores (*prunned*), das quais escolhe a melhor através de um processo de *cross-validation*;
- O custo da complexidade deste processo é dado por:

$$R_{\alpha}(T_{t}) = R(T_{t}) + \alpha |T_t|$$

ou

$$TreeScore_t = SSR + \alpha |T_t|$$

onde $R(T)$ representa o total do erro dos nós folhas, $|T|$ representa o número de nós folhas e $\alpha$ o parâmetro da complexidade.

```py
ccp_alphas = full_tree.cost_complexity_pruning_path(x_train, y_train)["ccp_alphas"]
ccp_alpha_grid_search = GridSearchCV(
    estimator=DecisionTreeClassifier(random_state=42),
    scoring=make_scorer(accuracy_scorer),
    param_grid=ParameterGrid(
        {"ccp_alpha":[[alpha] for alpha in ccp_alphas]}
    ),
)
```

### Conclusões acerca das Árvores de Decisão

**Forças:**

- Configuração simples (tem poucos parâmetros de configuração);
- Comparado a outros algoritmos, precisa de menos esforço da preparação dos dados durante o pré-processamento;
- Não precisa de normalização de dados;
- Não precisa de escalonamento dos dados;
- *Missing values* não afeta o processo de construção de forma considerável;
- Muito intuitivo e fácil de explicar às equipas de técnicos e *stakeholders*.

**Fraquezas:**

- Inadequada para problemas com várias interações entre atributos;
- Não evita réplicas de sub-árvores;
- Uma pequena mudança nos dados poderá provocar uma grande mudança na estrutura da árvore causando instabilidade;
- O cálculo poder ser muito complexo comparado a outros algoritmos;
- Pode precisar de muito tempo para treinar o modelo.

