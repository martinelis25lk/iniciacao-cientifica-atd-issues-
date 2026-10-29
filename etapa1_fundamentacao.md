# Pesquisa IC: Identificação de Problemas Arquiteturais a partir de Issues com LLMs

Repositório de acompanhamento e artefatos da Iniciação Científica (UFPI).
**Orientadores:** Prof. Dr. Armando Soares Sousa e Prof. Dr. Pedro de Alcântara dos Santos Neto.

---

## 1. Matriz de Síntese Bibliográfica (Etapa 1)

Abaixo estão sintetizadas as referências base que fundamentam o método de identificação de Dívida Técnica Arquitetural (ATD) e a evolução para a inspeção automatizada de *issues* via LLMs.

| Autor / Ano | Obra / Título | Conceito Principal | Método Utilizado | Principais Contribuições | Limitações Identificadas |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Kruchten et al. (2012)** / **Li et al. (2015)** | *Technical Debt Landscape* / *SMS on Technical Debt* | Definição e taxonomia de Dívida Técnica (TD), separando elementos visíveis (novas features, bugs) de invisíveis (dívida arquitetural e estrutural). | Revisão e Mapeamento Sistemático da Literatura sobre Gestão de Dívida Técnica (TDM). | Classificação da TD em 10 tipos (incluindo Arquitetural) e definição das 8 atividades essenciais de gerenciamento (identificação, medição, monitoramento, etc.). | Foco amplo em TD de código; carece de métodos automatizados específicos para identificar e mensurar o impacto da dívida estritamente no nível arquitetural. |
| **Besker et al. (2018)** | *Managing Architectural Technical Debt: A Unified Model and SLR* | Dívida Técnica Arquitetural (ATD) e seu impacto severo na manutenibilidade e evolução de sistemas de larga escala. | Revisão Sistemática da Literatura (SLR) para criar um modelo descritivo unificado sobre ATD. | Categorização dos desafios de ATD (dependências arquiteturais, quebras de padrão, conflitos de qualidade) e definição de 5 atividades centrais para gerir ATD. | Aponta a falta de guias práticos na indústria, ausência de consenso na identificação de ATD e escassez de ferramentas automatizadas de monitoramento. |
| **Sousa, Rocha & Britto (2023)** | *Architectural Technical Debt - A Systematic Mapping Study* (Cap. 4 da Tese) | Mapeamento do estado da arte (2012–2022) sobre identificação, medição, monitoramento e cálculo de custo de itens de ATD. | *Systematic Mapping Study* (SMS) analisando 70 estudos primários selecionados na literatura. | Proposta de três novas categorias de ATD (*Model-driven*, *Social* e *Self-Admitted*), além de um *Roadmap* de 8 passos para guiar a gestão de ATD na indústria. | Constata que a maioria das técnicas ainda depende de avaliação manual de especialistas em arquitetura e que faltam ferramentas que integrem histórico de mudanças. |
| **Sousa, A. (2024)** | *A Systematic and Efficient Approach for Identifying ATD* (Tese - Cap. 6, 7 e 8) | Método **ATDCodeAnalyzer** e ferramenta **SysRepoAnalysis** para achar arquivos críticos afetados por ATD usando apenas o histórico do Git. | *Design Science Research*, Mineração de Repositórios (MSR), análise de *Architectural Smells* (Arcan) e entrevistas na Ericsson (projeto Apache Cassandra). | Identificação 100% automatizada de classes críticas combinando *Smells* (Dependência Cíclica e *Hub-like*) com métricas históricas (LOC modificadas, frequência de commits e complexidade) e *co-change*. | O método base foca apenas em artefatos de código-fonte (`.java`) e exige repositórios com histórico longo e volumoso de *commits* para ter alta precisão estatística. |
| **Sousa, Rocha & Britto (2024 / 2026)** | *Mining ATD through LLM-Assisted Issue Inspection* (Cap. 9 da Tese / SBES 2026) | Validação qualitativa e identificação de problemas arquiteturais através da análise textual de *issues* (Jira) e *Self-Admitted Technical Debt* (SATD) com LLMs. | Mineração de 4 projetos Apache (Cassandra, Kafka, Hadoop, ActiveMQ), filtragem por *keywords* de SATD e classificação assistida por LLM (*Few-shot* e *Chain-of-Thought*). | Criação de um classificador semiautomático baseado em LLM (com concordância Kappa de 0.721 frente a humanos) que prova que classes críticas concentram mais *issues* de impacto arquitetural e demoram mais para serem resolvidas. | A análise depende da qualidade do registro textual das *issues* pelos desenvolvedores; necessidade de evoluir os critérios de prompt (`skill.md`) para capturar nuances estruturais mais amplas. |

---

## 2. Técnicas e Ferramentas Candidatas (Etapa 1)

Para o desenvolvimento do protótipo na Etapa 2, foram mapeadas as seguintes tecnologias:

*   **Extração e Mineração de Repositórios (MSR):**
    *   *GitHub REST API / PyGithub:* Para coleta automatizada de *issues* (descrição, comentários, labels) em repositórios hospedados no GitHub.
    *   *Jira REST API:* Caso o escopo utilize projetos do ecossistema Apache (como feito no `ATDissueinspector`).
    *   *PyDriller / GitPython:* Para eventual cruzamento de *issues* com *commits* e arquivos modificados no histórico de versão.
*   **Processamento e Análise com LLMs:**
    *   *OpenAI API (Modelos da família GPT-4o / GPT-5.1):* Motor de inferência para leitura e interpretação semântica dos tickets.
    *   *Engenharia de Prompt Estruturada (`skill.md`):* Uso de *Few-Shot Prompting* (fornecimento de exemplos prévios rotulados) e *Chain-of-Thought* (raciocínio passo a passo) para exigir que a LLM classifique a *issue* e gere uma justificativa técnica.
*   **Manipulação de Dados e Visualização:**
    *   *Python (Pandas / NumPy):* Limpeza textual, estruturação de *datasets* (`.csv` / `.json`) e cálculo de métricas estatísticas.
    *   *Matplotlib / Seaborn / Jupyter Notebooks:* Geração de gráficos de distribuição (ex.: *Boxplots* de tempo de resolução, evolução temporal de problemas arquiteturais).
*   **Ferramenta Base do Grupo de Pesquisa:**
    *   *ATDissueinspector:* Aplicação web desenvolvida pelo Prof. Armando Sousa que automatiza a submissão de lotes de *issues* (`.txt`/`.md`) contra critérios definidos em arquivo de configuração.

---

## 3. Fundamentação Teórica Preliminar (Rascunho para o Capítulo Teórico)

### 3.1. Arquitetura de Software e Degradação Estrutural
Segundo Perry e Wolf (1992) e Bass et al. (2003), a arquitetura de software compreende a estrutura ou estruturas de um sistema, compostas por elementos de software (processamento, dados e conexão), as propriedades externamente visíveis desses elementos e os relacionamentos entre eles[cite: 31, 32]. Taylor et al. (2009) complementam que a arquitetura é o conjunto das principais decisões de design tomadas durante o desenvolvimento e evolução do sistema, impactando diretamente atributos não-funcionais como desempenho, escalabilidade, segurança e manutenibilidade[cite: 32]. Quando os princípios e diretrizes arquiteturais estabelecidos inicialmente são violados ao longo da evolução do software — frequentemente devido a pressões de prazos —, ocorre o fenômeno de degradação arquitetural[cite: 20, 32].

### 3.2. Dívida Técnica (TD) e Dívida Técnica Arquitetural (ATD)
A metáfora de Dívida Técnica (TD) reflete compromissos técnicos assumidos para obter ganhos de curto prazo (como entregas rápidas), mas que sacrificam a saúde de longo prazo do produto de software, gerando "juros" na forma de esforço extra de manutenção futura (Li et al., 2015; Ampatzoglou et al., 2015)[cite: 30]. Kruchten et al. (2012) organizam a paisagem da dívida técnica dividindo-a entre elementos visíveis (novas funcionalidades e defeitos) e elementos majoritariamente invisíveis, focados na evolutibilidade e manutenibilidade interna[cite: 31].

No topo dos elementos invisíveis de maior impacto encontra-se a **Dívida Técnica Arquitetural (ATD)**. A ATD refere-se a violações de boas práticas, quebras de consistência e integridade arquitetural, escolhas tecnológicas subótimas ou acoplamento excessivo entre módulos (Martini & Bosch, 2015; Besker et al., 2018)[cite: 21, 33]. Por ser um problema transversal no ciclo de vida do software, a ATD é difícil de ser identificada apenas olhando para trechos isolados de código, tornando-se visível quando o custo de corrigir *bugs* ou adicionar novas *features* cresce exponencialmente[cite: 20, 33].

### 3.3. Indicadores de Problemas Estruturais: Architectural Smells e Histórico
Uma das formas mais consolidadas na literatura para detectar sintomas de ATD é por meio de *Architectural Smells* (AS), que representam violações de princípios de design em nível de componentes e pacotes (Azadi et al., 2019)[cite: 39]. Dois dos *smells* mais críticos e custosos para refatoração são:
1.  **Dependência Cíclica (*Cyclic Dependency - CD*):** Ocorre quando dois ou mais componentes arquiteturais dependem um do outro direta ou indiretamente, destruindo a modularidade e causando efeito cascata nas alterações (*ripple effect*)[cite: 40, 108].
2.  **Dependência *Hub-like* (*Hub-like Dependency - HLD*):** Ocorre quando uma classe ou abstração central concentra um número excessivo de dependências de entrada (aferentes) e de saída (eferentes), tornando-se um gargalo de manutenção[cite: 40].

Sousa (2024), através do método *ATDCodeAnalyzer*, demonstrou que arquivos de código-fonte que possuem esses *Architectural Smells*, aliados a uma alta frequência de *commits*, alto volume de linhas modificadas (AMLOC) e crescente Complexidade Ciclomática ao longo do tempo, são fortes indicadores da presença real de ATD[cite: 11, 86, 87].

### 3.4. Mineração de Issues e Self-Admitted Technical Debt (SATD)
Além do código-fonte, repositórios de software armazenam artefatos ricos em linguagem natural, como sistemas rastreadores de *issues* (Jira, GitHub Issues) e mensagens de *commit* (Hassan, 2008; Robles, 2010)[cite: 35, 36]. Nesses artefatos, frequentemente ocorre a **Dívida Técnica Autoadmitida (*Self-Admitted Technical Debt - SATD*)**, termo cunhado por Potdar e Shihab (2014) para descrever situações em que os próprios desenvolvedores admitem deliberadamente (via comentários `TODO`, `FIXME`, `hack` ou discussões em tickets) que uma solução implementada é paliativa, frágil ou arquiteturalmente inadequada[cite: 34].

A análise textual da descrição de *issues*, dos comentários debatidos pela equipe ao longo da resolução do ticket e dos logs anexados permite capturar problemas estruturais que ferramentas de análise estática de código não conseguem enxergar — tais como gargalos de concorrência, falhas de consistência em sistemas distribuídos, problemas no modelo de dados e limitações de escalabilidade.

### 3.5. Inspeção Automatizada de Issues Apoiada por LLMs
Tradicionalmente, inspecionar *issues* para verificar se elas possuem impacto arquitetural exigia avaliação manual de especialistas, um processo lento, caro e sujeito a fadiga humana[cite: 23, 124]. Com o avanço dos Grandes Modelos de Linguagem (LLMs), tornou-se viável automatizar essa triagem qualitativa[cite: 124, 135].

Utilizando técnicas de Engenharia de Prompt como *Few-Shot Learning* (apresentação de exemplos de *issues* arquiteturais e não-arquiteturais) e *Chain-of-Thought* (instrução para o modelo justificar passo a passo sua decisão), estudos recentes (Sousa et al., 2024; 2026) comprovaram que LLMs atingem concordância substancial (Kappa > 0.72) e alta precisão (> 92%) em relação a avaliadores humanos na identificação de *issues* com impacto arquitetural[cite: 135, 145]. Ferramentas como o *ATDissueinspector* sistematizam esse fluxo, permitindo parametrizar características estruturais (em arquivos de critério como `skill.md`) para minerar grandes volumes de tickets e revelar padrões de degradação arquitetural ao longo da vida de um projeto.
