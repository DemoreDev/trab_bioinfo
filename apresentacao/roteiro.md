# Roteiro da apresentação

| Parte | Quem | Slides | Tema | Tempo |
|---|---|---|---|---|
| 1 | Leonardo | 1–5 | Problema, dados e pré-processamento | ~1:40 |
| 2 | Thiago | 6–10 | k-means, cotovelo e agrupamento hierárquico | ~1:40 |
| 3 | Julio | 11–15 | Validação, PCA, discussão e conclusões | ~1:40 |

Os slides têm só palavras-chave. Em cada slide, a frase marcada com '➜' guia a apresentação

---

## Parte 1 — Leonardo (slides 1–5)

### O que ler no notebook
- **§1.1**: constantes `N_GENES = 2000` e `SEED = 42`.
- **§1.2–1.3 `download_data` e `load_data`**: origem dos dados (UCI) e o `assert` que confere se amostras e rótulos estão na mesma ordem. Os rótulos ficam guardados em `labels` até o §4.
- **§1.4 `quality_summary`**: 0 valores ausentes, 267 genes constantes, valores de 0 a 20,8 (escala log, então sem outra transformação).
- **§1.5 `select_top_variance_genes`**: variância por gene → ordena → mantém os 2.000 maiores.
- **§1.6 `standardize`**: `StandardScaler` e a conferência (média ≈ 0, desvio = 1).
- Relatório, seções 1 e 2.

### Fala por slide
| # | Slide | Tempo | O que dizer |
|---|---|---|---|
| 1 | Capa | 10 s | Apresentar o tema e o grupo e dizer quem fala cada parte. |
| 2 | A pergunta | 20 s | No projeto anterior os tipos de tumor eram conhecidos e só coloriam a PCA. ➜ Agora escondemos os rótulos: o algoritmo forma grupos só com a expressão gênica, e só no fim conferimos se esses grupos são os 5 tipos de tumor. |
| 3 | Os dados | 20 s | ➜ 801 amostras de RNA-Seq do TCGA, cada uma com 20.531 genes, ou seja, cada amostra é um ponto num espaço de 20 mil dimensões. São 5 tipos de tumor, e o BRCA sozinho tem 300 amostras. |
| 4 | Pipeline | 20 s | ➜ Seguimos as 5 etapas do enunciado, que são as 5 seções do notebook. Nas três primeiras os rótulos ficam escondidos; nas duas últimas eles são usados só para avaliar. |
| 5 | Pré-processamento | 30 s | Duas etapas. ➜ **Filtrar:** genes que quase não variam só acrescentam ruído à distância, então ficamos com os 2.000 mais variáveis. A variância é medida antes de padronizar; depois todos teriam variância 1. ➜ **Padronizar:** os métodos usam distância euclidiana, e sem o z-score um gene pesaria até 10× mais que outro. Resultado: matriz 801 × 2.000. "Agora o Thiago mostra como agrupamos essas amostras." |

**Saiba responder:** por que filtrar genes? Por que medir a variância antes do z-score? A filtragem usa os rótulos? (Não.)

---

## Parte 2 — Thiago (slides 6–10)

### O que ler no notebook
- **§2.2 `run_kmeans`**: `KMeans(n_clusters=k, n_init=10, random_state=SEED)` para k de 2 a 8; o atributo `inertia_`; por que `n_init=10` (evita mínimos locais ruins).
- **§2.3 `find_elbow`**: normaliza os eixos em [0, 1], usa a reta `x + y − 1 = 0` entre os extremos e escolhe o ponto de maior distância `|x + y − 1| / √2`.
- **§2.4–2.5**: gráfico do cotovelo e tabela de quedas da inércia (14,1% → 15,4% → 10,8% → 5,2% → 2,6% → 2,0%).
- **§3.2 `build_linkage`**: `linkage(..., method="ward")` e a matriz `Z` (800 uniões).
- **§3.3 `cut_tree`**: `fcluster(Z, k, "maxclust")`, altura de corte 372,3 e renumeração dos grupos da esquerda para a direita.
- **§3.4–3.5**: dendrograma e tamanhos dos grupos (146, 136, 302, 78, 139).
- Relatório, seção 3 e subseções 4.1–4.2.

### Fala por slide
| # | Slide | Tempo | O que dizer |
|---|---|---|---|
| 6 | k-means | 20 s | ➜ O k-means sorteia k centros, coloca cada amostra no centro mais próximo, move cada centro para a média do seu grupo e repete até estabilizar. Ele minimiza a inércia, a soma das distâncias² até o centro. Rodamos k de 2 a 8, cada um com 10 inicializações. |
| 7 | Método do cotovelo | 20 s | ➜ A inércia sempre cai quando k aumenta, então não dá para escolher só o menor valor. Procuramos onde a queda deixa de compensar: até k = 5 ela cai 10–15% por passo, e depois só 5% ou menos. |
| 8 | Escolhendo k | 20 s | ➜ Para não depender do olho, normalizamos os eixos e pegamos o ponto mais distante da reta entre os extremos: k = 5. Mas foi por pouco, 0,248 contra 0,232 do k = 4. O cotovelo é uma heurística, então conferimos com um segundo método. |
| 9 | Hierárquico (Ward) | 20 s | ➜ No hierárquico, cada amostra começa sozinha e os dois grupos mais próximos vão se unindo até formar uma árvore. Com Ward, une-se o par que menos aumenta a variância interna, o mesmo critério do k-means. Cortamos a árvore no mesmo k = 5. |
| 10 | O dendrograma | 20 s | ➜ Abaixo do corte aparecem 5 ramos bem definidos, com tamanhos quase iguais aos do k-means. Repare que o ramo G3 se divide e que G4 e G5 se unem bem perto do corte. "O Julio volta nisso e mostra se esses grupos são mesmo os tipos de tumor." |

**Saiba responder:** por que não escolher o k de menor inércia? Por que normalizar os eixos? Por que Ward e não outra ligação? Por que os números dos grupos diferem entre os métodos?

---

## Parte 3 — Julio (slides 11–15)

### O que ler no notebook
- **§4.1–4.2 `contingency_table`**: `pd.crosstab` de grupos × tipos reais.
- **§4.3 `majority_summary`**: tipo majoritário e pureza (≥ 99%) de cada grupo.
- **§5.2 `run_pca`**: PCA com 2 componentes sobre a mesma matriz `X` (PC1 = 20,3%, PC2 = 11,8%, total 32%).
- **§5.3 `plot_pca_clusters`**: cor = grupo encontrado; círculo = amostra cujo tipo real difere do majoritário do grupo.
- **§6 Respostas às perguntas**: o texto-base dos slides 13 e 14.
- Relatório, subseções 4.3–4.4 e seções 5 e 6.

### Fala por slide
| # | Slide | Tempo | O que dizer |
|---|---|---|---|
| 11 | Só agora: os rótulos | 25 s | ➜ Só agora usamos os rótulos: cada linha é um grupo encontrado, cada coluna um tipo real. Quase tudo cai num único tipo: o k-means acertou 797 de 801 amostras e o Ward 799. A numeração dos grupos muda entre os métodos, mas os tipos são os mesmos. |
| 12 | Os grupos na PCA | 20 s | ➜ No plano da PCA, LUAD e COAD se sobrepõem, mas o agrupamento separou os dois porque usa os 2.000 genes, e o gráfico mostra só 32% da variância. Os poucos erros (círculos) ficam nas fronteiras entre as nuvens. |
| 13 | Diferente não é errado | 25 s | ➜ Se cortamos a mesma árvore em outra altura, o resultado muda: com k = 4, COAD e LUAD se juntam; com k = 6, BRCA se divide em dois. Isso não é erro: o método minimiza variância e não procura "tipos de tumor". Cada corte mostra uma estrutura real em outra escala. |
| 14 | Respostas às perguntas | 15 s | As três respostas, em uma frase cada: sim, k = 5, mas por pouco; nenhum tipo foi dividido ou misturado; e o método otimiza geometria, não rótulos. |
| 15 | Conclusões | 15 s | ➜ O sinal da expressão gênica é forte, e os dois métodos concordam. A escolha de k é o ponto crítico: ela decide qual escala da estrutura enxergamos. Obrigado! |

**Saiba responder:** o que é uma tabela de contingência? Por que a PCA não foi usada para agrupar? O que pode significar a divisão de BRCA em k = 6? (Uma possível subestrutura, talvez subtipos moleculares; não foi testada.)

---

## Dicas para a gravação
- Combinem as frases de passagem (fim do slide 5 → Thiago; fim do slide 10 → Julio).
- Números que todos devem dizer igual: **801 amostras · 20.531 → 2.000 genes · k = 5 · 797/801 (k-means) · 799/801 (Ward) · 32% da variância na PCA**.
- Façam um ensaio cronometrado. Se passar de 5 min, encurtem primeiro os slides 4, 7 e 14.
- Para editar os slides, alterem [apresentacao/apresentacao.tex](apresentacao/apresentacao.tex) e compilem com `xelatex apresentacao.tex` (duas vezes) dentro de `apresentacao/`. As figuras vêm direto de `results/`.
