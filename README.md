# Inteligência Artificial Generativa e o Mercado de Trabalho Brasileiro

**Dissertação de Mestrado** — Análise de Exposição Ocupacional e seus Efeitos Distributivos.  
**Aluno:** Manoel Brasil Orlandi.

Este repositório contém o código e os notebooks da análise empírica, organizados em três etapas: (1) análise descritiva PNADc + índice ILO, (2) DiD com CAGED e (3) Triple-DiD com conectividade municipal.

---

## Estrutura de pastas

```
source code/
├── README.md                    # Este arquivo
├── .gitignore
├── .vscode/
│   └── settings.json
└── Notebooks/
    ├── etapa_1a_preparacao_dados_ilo_pnadc.ipynb    # Preparação PNAD + ILO
    ├── etapa_1b_analise_dados_ilo_pnadc.ipynb      # Análise descritiva PNAD/ILO
    ├── etapa_2a_preparacao_dados_did_caged_ilo.ipynb  # Preparação painel CAGED + ILO
    ├── etapa_2b_analise_did_caged_ilo.ipynb        # Estimação DiD (emprego formal)
    ├── etapa_2c_resultados.ipynb                   # Consolidação e síntese DiD
    ├── etapa_3a_preparacao_dados_did_municipio_conectividade.ipynb  # Painel CAGED × município × Anatel
    └── etapa_3b_analise_did_municipio_conectividade.ipynb  # Triple-DiD (conectividade)
```

Os notebooks esperam uma estrutura de dados (geralmente relativa ao diretório de trabalho onde o notebook é executado):

- **data/input/** — arquivos de entrada (planilha ILO, crosswalks CBO/ISCO, etc.)
- **data/raw/** — microdados brutos (PNAD, CAGED por ano)
- **data/processed/** — arquivos intermediários (PNAD limpa, ILO processado, painéis parquet)
- **data/output/** — bases analíticas finais (merged PNAD-ILO, painéis DiD-ready)
- **outputs/tables/** e **outputs/figures/** — tabelas e figuras geradas pelos notebooks de análise (2b, 2c, 3b)

---

## Resumo do que tem em cada notebook

| Notebook | Conteúdo principal |
|----------|--------------------|
| **etapa_1a** | Preparação dos dados: download PNAD (BigQuery), processamento do índice ILO (ISCO-08), crosswalk COD→ISCO-08, limpeza e variáveis derivadas, merge final. **Saída:** `data/output/pnad_ilo_merged.csv`. |
| **etapa_1b** | Análise descritiva da exposição: perfil por quintil/decil, desigualdade e renda, gênero e raça, idade e escolaridade, formalidade, augmentation vs. automação, setor e ocupação, região (incl. mapas e shift-share), índice de vulnerabilidade, análise multivariada e síntese. **Entrada:** `data/output/pnad_ilo_merged.csv`. |
| **etapa_2a** | Preparação do painel DiD: IPCA, download CAGED (BigQuery), agregação mensal por ocupação, crosswalk CBO→ISCO-08 (2d e 4d), definição de tratamento (alta exposição), enriquecimento (salário real, controles), opcional Anthropic (automação vs. amplificação). **Saída:** `data/output/painel_caged_did_ready.parquet`. |
| **etapa_2b** | Análise DiD: carregar painel, tabela de balanço pré-tratamento, tendências paralelas, estimação DiD principal (vários modelos e outcomes), event study, testes de robustez, heterogeneidade (triple-DiD), mecanismos, tabelas LaTeX, síntese. **Entrada:** painel da 2a; **Saída:** tabelas e figuras em `outputs/tables` e `outputs/figures`. |
| **etapa_2c** | Resultados: listagem e exibição das tabelas e figuras do 2b, gráficos adicionais, resumo narrativo, tabela-síntese de achados com relevância estatística. **Entrada:** arquivos em `outputs/`. |
| **etapa_3a** | Preparação do painel tridimensional: Anatel (banda larga fixa, período pré-tratamento), IBGE (população, PIB, domicílios), índice de conectividade municipal, CAGED agregado por ocupação×município×mês (com faixa etária), merge com exposição e conectividade, decomposição etária. **Saída:** `data/output/painel_caged_municipio_anatel.parquet` (e versão v2 com variáveis etárias). |
| **etapa_3b** | Análise Triple-DiD: balanço por conectividade, DiD por subgrupo, modelo principal (triple_did), event study por conectividade, robustez (tendência diferencial pré, proxies alternativas), decomposição etária e modelo focado em jovens. **Entrada:** painel da 3a; **Saída:** tabelas/figuras em `outputs/`. |

---

## Passo a passo metodológico por notebook

### Etapa 1a — Preparação dos dados ILO e PNADc

**Objetivo:** Construir a base analítica que une PNAD Contínua (3º trim/2025) ao índice de exposição à IA generativa da OIT (ILO), com ocupações em COD e ISCO-08.

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 1 | Configuração do ambiente: caminhos (`data/input`, `data/raw`, `data/processed`, `data/output`), parâmetros PNAD (ano, trimestre, projeto GCP), arquivo ILO, mapeamentos (região, grandes grupos, raça, CNAE setor, etc.) e funções de estatísticas ponderadas. | Células 1–3 |
| 2a | Download dos microdados PNAD: leitura de `data/raw/pnad_*.parquet` se existir; senão, query BigQuery (Base dos Dados), variáveis (sexo, idade, raça, instrução, cod_ocupacao, CNAE, posição, rendimentos, horas, peso). Saída: `data/raw/pnad_YYYYqN.parquet`. | Seção 2a (células 4–5) |
| 2b | Verificação (checkpoint) dos microdados PNAD: shape, colunas, UFs, população, preenchimento das variáveis. | Seção 2b (células 6–7) |
| 3a | Processar índice ILO: leitura de `data/input/Final_Scores_ISCO08_Gmyrek_et_al_2025.xlsx`, renomear colunas, agregar por ocupação ISCO-08 (exposure_score, exposure_gradient). Saída: `data/processed/ilo_exposure_clean.csv`. | Seção 3a (células 8–9) |
| 3b | Verificação do índice ILO: número de ocupações, range de scores, distribuição por gradiente. | Seção 3b (células 10–11) |
| 4a | Limpeza e variáveis derivadas PNAD: filtros (idade 18–65, ocupação válida), flags (tem_renda, formal), faixas (etária, renda em SM), winsorização de renda, região, raça agregada, grande grupo. Saída: `data/processed/pnad_clean.csv`. | Seção 4a (células 13–14) |
| 4b | Verificação da limpeza: perda de observações, missings, distribuições. | Seção 4b (células 15–16) |
| 5a | Crosswalk COD → ISCO-08: match hierárquico 4d → 3d → 2d → 1d; atribuição de exposure_score e exposure_gradient; coluna match_level. | Seção 5a (células 17–18) |
| 5b | Verificação do crosswalk: cobertura, distribuição por nível de match, sanity check por grande grupo. | Seção 5b (células 19–20) |
| 6 | Merge final: quintis e decis de exposição (ponderados), setor agregado CNAE, setor_critico_ia; seleção de colunas; exportação. **Saída principal:** `data/output/pnad_ilo_merged.csv`. | Seção 6 (células 21–22) |

**Arquivos principais:**  
- Entrada: `data/input/Final_Scores_ISCO08_Gmyrek_et_al_2025.xlsx`, microdados PNAD em `data/raw/`.  
- Saídas: `data/processed/ilo_exposure_clean.csv`, `data/processed/pnad_clean.csv`, `data/output/pnad_ilo_merged.csv`.

---

### Etapa 1b — Análise descritiva dos dados ILO e PNADc

**Objetivo:** Caracterizar a exposição do mercado de trabalho brasileiro à IA generativa (perfil, desigualdade, gênero, raça, formalidade, setor, região) usando a base gerada na etapa 1a.

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 1 | Configuração: caminhos, parâmetros, mapeamentos (região, grandes grupos, ordem/cores dos gradientes), carregar `data/output/pnad_ilo_merged.csv`. | Seção 1 |
| 2 | Perfil da exposição: distribuição por quintil/decil, médias ponderadas por grupo, gráficos. | Seção 2 |
| 3 | Desigualdade e renda: rendimento por quintil de exposição, razão Q5/Q1, análises ponderadas. | Seção 3 |
| 4 | Gênero e raça: % mulheres e negros por quintil, exposição média por sexo e raça. | Seção 4 |
| 4b | Idade e escolaridade: exposição por faixa etária e nível de instrução. | Seção 4b |
| 5 | Formalidade: % formal por quintil, exposição no formal vs. informal. | Seção 5 |
| 5b | Augmentation vs. automação: comparação entre gradientes de complementaridade (G1–G2) e transformação (G3–G4). | Seção 5b |
| 6 | Setor e ocupação: exposição por setor agregado e grande grupo, setores críticos IA. | Seção 6 |
| 6b | Fluxo escolaridade → ocupação → gradiente de exposição. | Seção 6b |
| 7 | Região: exposição por região/UF, mapas. | Seção 7 |
| 7b | Mapas regionais. | Seção 7b |
| 7c | Decomposição shift-share regional. | Seção 7c |
| 7d | Índice de vulnerabilidade composta. | Seção 7d |
| 8 | Análise multivariada: modelos (ex.: regressão da exposição em características). | Seção 8 |
| 9 | Síntese e conclusões. | Seção 9 |

**Arquivo de entrada:** `data/output/pnad_ilo_merged.csv` (gerado na etapa 1a).

---

### Etapa 2a — Preparação do painel CAGED + ILO (DiD)

**Objetivo:** Construir o painel mensal de ocupações formais (2021–2025) com CAGED, crosswalk CBO 2002 → ISCO-08 (2d e 4d) e índice ILO, pronto para DiD (evento: ChatGPT, pós a partir de dez/2022).

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 1 | Configuração: caminhos, período (2021–2025), tratamento (dez/2022), arquivos ILO e crosswalks, caches (painel mensal, crosswalk, tratamento, final). | Seção 1 |
| 2 | Índice IPCA: série mensal (BCB SGS), base Dez/2024 = 100. Saída: `data/processed/ipca_mensal.parquet`. | Seção 2 |
| 2a | Download CAGED: por ano, BigQuery ou cache; salvar em `data/raw/caged_YYYY.parquet`. | Seção 2a |
| 2b | Verificação (checkpoint) dos dados CAGED. | Seção 2b |
| 3a | Agregação: microdados → painel mensal por ocupação (CBO 4d, ano, mês): admissões, desligamentos, salário médio, idade, % mulher, % superior, etc. Saída: `data/processed/painel_caged_mensal.parquet`. | Seção 3a |
| 3b | Verificação do painel agregado. | Seção 3b |
| 4a | Crosswalk CBO 2002 → ISCO-08: especificação dual (2d principal, 4d robustez), uso de correspondência ISCO-88/08 e fallback hierárquico. Saída: `data/processed/painel_caged_crosswalk.parquet`. | Seção 4a |
| 4b | Verificação do crosswalk. | Seção 4b |
| 5a | Definição de tratamento: alta exposição (ex.: mediana ou percentil do exposure_score), variáveis post, período. Saída intermediária: `data/processed/painel_caged_tratamento.parquet`. | Seção 5a |
| 5b | Verificação do tratamento. | Seção 5b |
| 6a | Enriquecimento: merge com IPCA (salário real), controles, período, FE. | Seção 6a |
| Anexo 1 | Integração índice Anthropic (automação vs. amplificação) por CBO, se aplicável. | Anexo 1 |
| 7 | Salvar dataset analítico final. **Saída:** `data/output/painel_caged_did_ready.parquet` (e opcional CSV). | Seção 7 |

**Arquivos principais:**  
- Entrada: `data/processed/ilo_exposure_clean.csv`, crosswalks em `data/input/`, microdados CAGED em `data/raw/caged_*.parquet`.  
- Saída: `data/output/painel_caged_did_ready.parquet`.

---

### Etapa 2b — Análise DiD (CAGED e ILO)

**Objetivo:** Estimar o efeito da exposição à IA generativa sobre o emprego formal (admissões, desligamentos, salários) via Difference-in-Differences.

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 0 | Estratégia de identificação: tratamento = alta exposição (ILO), evento = ChatGPT (pós dez/2022); outcomes definidos. | Início (markdown) |
| 1 | Configuração e carga do painel (`data/output/painel_caged_did_ready.parquet`). | Seção 1 |
| 2 | Exploração dos dados: variáveis, períodos, grupos. | Seção 2 |
| 3 | Tabela de balanço pré-tratamento: controle vs. tratamento (médias, diff. normalizada). Saída: `outputs/tables/balance_table_pre.csv`. | Seção 3 |
| 4 | Tendências paralelas: inspeção visual (evolução dos outcomes por grupo). | Seção 4 |
| 5a | Estimação DiD principal: vários modelos (Basic, FE, FE+Controls), outcomes (ln_admissoes, ln_desligamentos, ln_salário, etc.), erros clusterizados por ocupação. | Seção 5a |
| 5b | Checkpoint dos resultados DiD. | Seção 5b |
| 6 | Event study: coeficientes por tempo relativo ao evento. | Seção 6 |
| 6b | Teste formal de tendências paralelas. | Seção 6b |
| 7 | Heterogeneidade (triple-DiD): subgrupos (ex.: setor tecnológico, gênero). | Seção 7 |
| 8 | Testes de robustez: especificações alternativas, contínuo vs. binário, 4d. | Seção 8 |
| 10 | Mecanismos e heterogeneidade adicional. | Seção 10 |
| 11 | Exportação de tabelas LaTeX. | Seção 11 |
| 12 | Síntese e conclusões. | Seção 12 |

**Entrada:** `data/output/painel_caged_did_ready.parquet`.  
**Saídas:** tabelas em `outputs/tables/`, figuras em `outputs/figures/`.

---

### Etapa 2c — Resultados: consolidação e síntese DiD

**Objetivo:** Organizar tabelas e figuras do 2b para leitura e apresentação, com resumo narrativo e tabela-síntese de achados.

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 1 | Configuração: caminhos `outputs/tables` e `outputs/figures`. | Seção 1 |
| 2 | Tabelas do etapa_2b: balanceamento pré (2.1), resultados DiD principais (2.2), outras tabelas salvas em `outputs/tables`. | Seção 2 |
| 3 | Figuras geradas no etapa_2b (listagem e exibição). | Seção 3 |
| 4 | Gráficos adicionais para ilustrar achados. | Seção 4 |
| 5 | Resumo narrativo dos achados. | Seção 5 |
| 6 | Tabela-síntese de achados com relevância estatística (***/**/*) e destaque. | Seção 6 |
| 7 | Nota sobre reprodução. | Seção 7 |

**Entrada:** arquivos em `outputs/tables/` e `outputs/figures/` gerados pelo notebook 2b.

---

### Etapa 3a — Preparação do painel CAGED × município × conectividade

**Objetivo:** Construir painel ocupação (CBO 4d) × município × mês com CAGED, exposição (Etapa 2a) e índice de conectividade municipal (Anatel), para Triple-DiD.

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 1 | Configuração: caminhos (incl. `notebook/data`, `outputs/tables`), parâmetros (período pré conectividade até out/2022), cache CAGED municipal, arquivos de saída. | Seção 1 |
| 2 | Anatel banda larga fixa: BigQuery (Base dos Dados), período 2021 e jan–out/2022, agregado por município (média acessos, % fibra). Saída: `data/processed/anatel_pre_tratamento.parquet`. | Seção 2 |
| 3 | IBGE: população (2022), PIB (2021), domicílios (Censo 2022 ou proxy). Saída: `data/processed/ibge_municipios.parquet`. | Seção 3 |
| 4 | Índice de conectividade: merge Anatel + IBGE, penetração = acessos/domicílios, alta_conectividade (acima da mediana), filtro população ≥ 50k. Saída: `data/processed/conectividade_municipal.parquet`. | Seção 4 |
| 5 | CAGED por ocupação × município × período: agregação a partir de `data/raw/caged_*.parquet` (ou cache), mesmas métricas da Etapa 2a, mais faixas etárias (jovem, intermediário, sênior). Saída: `data/processed/painel_caged_municipio.parquet`. | Seção 5 |
| 6 | Merge: CAGED municipal + exposição (do painel Etapa 2) + conectividade; variáveis post, tempo_relativo_meses, triple_did (post × alta_exp × alta_conectividade), salário real (IPCA); filtro mínimo de movimentações no pré. | Seção 6 |
| 7 | Validação e estatísticas descritivas: balanço pré por conectividade, correlação penetração × PIB per capita. | Seção 7 |
| 8 | Exportação. **Saída:** `data/output/painel_caged_municipio_anatel.parquet`; conectividade em `outputs/tables/conectividade_municipal.csv`. | Seção 8 |
| 9 | Decomposição por faixa etária: share jovem/sênior, ln_adm por faixa, salário real por faixa, razão salarial jovem/sênior. **Saída:** `data/output/painel_caged_municipio_anatel_v2.parquet`. | Seção 9 |

**Arquivos principais:**  
- Entrada: painel da Etapa 2 (`data/output/painel_caged_did_ready.parquet`), CAGED em `data/raw/`, IPCA em `data/processed/`.  
- Saída: `data/output/painel_caged_municipio_anatel.parquet` e `painel_caged_municipio_anatel_v2.parquet`.

---

### Etapa 3b — Análise Triple-DiD (município e conectividade)

**Objetivo:** Estimar o efeito diferencial da IA generativa entre municípios de alta e baixa conectividade (Triple-DiD). Coeficiente de interesse: β₇ (triple_did = post × alta_exp × alta_conectividade).

| Passo | O que é feito | Onde está no notebook |
|-------|----------------|------------------------|
| 0 | Dependências (pyfixest, pandas, etc.). | Seção 0 |
| - | Estratégia de identificação: tabela com elementos do Triple-DiD. | Markdown |
| 1 | Configuração e carga: `data/output/painel_caged_municipio_anatel.parquet` (ou v2). | Seção 1 |
| 2 | Tabela de balanço por conectividade (pré-tratamento). | Seção 2 |
| 3 | DiD por subgrupo de conectividade (motivação do Triple-DiD). | Seção 3 |
| 4 | Triple-DiD — modelo principal: regressão com post, alta_exp, alta_conectividade e triple_did; vários outcomes. | Seção 4 |
| 5 | Event study por grupo de conectividade. | Seção 5 |
| 6 | Testes de robustez e síntese. | Seção 6 |
| 7 | Correção de tendência diferencial pré (Caminho 1). | Seção 7 |
| 8 | Proxies alternativas de conectividade (Caminho 2): fibra, capital, percentis. | Seção 8 |
| 9 | Decomposição etária do Triple-DiD (outcomes por faixa etária). | Seção 9 |
| 10 | Modelo reformulado: impacto sobre jovens. | Seção 10 |
| - | Verificação metodológica (conferência com o plano Etapa 3). | Final |

**Entrada:** `data/output/painel_caged_municipio_anatel.parquet` ou `_v2.parquet` (Notebook 3a).  
**Saídas:** tabelas e figuras em `outputs/tables/` e `outputs/figures/`.

---

## Ordem sugerida de execução

1. **etapa_1a** → gera `pnad_ilo_merged.csv`.  
2. **etapa_1b** → usa a base da 1a; apenas análise, sem alterar dados.  
3. **etapa_2a** → gera `painel_caged_did_ready.parquet` (depende de `ilo_exposure_clean.csv`, que pode ser gerado na 1a ou obtido de `data/processed/`).  
4. **etapa_2b** → usa o painel da 2a; gera tabelas e figuras em `outputs/`.  
5. **etapa_2c** → consolida e sintetiza os resultados do 2b.  
6. **etapa_3a** → usa o painel da 2a e dados Anatel/IBGE; gera `painel_caged_municipio_anatel.parquet` (e v2).  
7. **etapa_3b** → usa o painel da 3a; gera tabelas e figuras em `outputs/`.

---

## Referências principais (citadas nos notebooks)

- Gmyrek, P., Berg, J. & Cappelli, D. (2025). *Generative AI and Jobs: An updated global assessment of potential effects on job quantity and quality*. ILO Working Paper 140.
- IBGE. PNAD Contínua (PNADc), 3º trimestre de 2025.
- Hui, X., Reshef, O. & Zhou, L. (2024). *The Short-Term Effects of Generative AI on Employment*. Organization Science.
- Callaway, B. & Sant'Anna, P. (2021). *Difference-in-differences with multiple time periods*. Journal of Econometrics.
- Muendler, M.-A. & Poole, J.P. (2004). *Job Concordances for Brazil: Mapping CBO to ISCO-88*. UC San Diego.

---

## Requisitos e ambiente

- Python 3 (recomendado 3.10+).
- Pacotes: pandas, numpy, pyarrow, openpyxl, xlrd, matplotlib, seaborn, scipy, statsmodels, pyfixest, python-bcb (IPCA), google-cloud-bigquery ou basedosdados.
- Acesso ao BigQuery (projeto GCP, ex.: `mestrado-pnad-2026`) para PNAD, CAGED e Anatel via Base dos Dados.
- Os caminhos dentro dos notebooks podem assumir execução a partir do diretório que contém `data/` e `Notebooks/` (ou `notebook/`); ajuste os `Path` conforme sua raiz de projeto.
