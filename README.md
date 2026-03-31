# README

## 1. Instruções de execução

Este notebook foi desenvolvido para executar uma checagem de compliance de posição de
suportes de bandejas elétricas em instalações industriais. O caso abordado na versão atual do código consiste em unistruts apoiados em (ou pendurados de) vigas perfil I e H. Estas análises requerem o uso de arquivos IFC e enriquecimento do modelo a partir do parâmetro STS_Name, que deve ser preenchido com a classificação do suporte (trapeze, unistrut ou cantilever). 

Essa checagem conta com o auxílio de uma dupla de agentes de AI sob a arquitetura de CrewAI. O agente 01, compliance specialist, executa as tools de análise de compliance e reporting, provendo informação ao agente 02, o compliance assistant, que analisa estas saídas para responder aos questionamentos do usuário.

O código está preparado para integração com API Anthropic, portanto, uma chave API Anthropic é necessária como um dos inputs do usuário.

Dentre os outputs, além das respostas para as perguntas do usuário, a aplicação também gera um relatório de compliance HTML e um resumo dos itens não conformes, em .xls.

### Requisitos mínimos

Instale os pacotes principais antes de rodar o notebook:

```bash
pip install crewai anthropic
```

Além disso, para a parte de leitura e processamento do IFC, o notebook também depende das bibliotecas utilizadas ao longo das células, especialmente `ifcopenshell`, além de bibliotecas auxiliares do ecossistema Python, todas mencionadas na primeira célula do notebook.

### Passo a passo sugerido

1. Abra o notebook no Jupyter Notebook, JupyterLab ou VS Code.
2. Garanta que o ambiente Python tenha as dependências instaladas.
3. Defina o caminho do arquivo IFC nas variáveis de entrada do notebook (segunda célula)
4. Configure a chave da Anthropic:
   - preferencialmente via variável de ambiente `ANTHROPIC_API_KEY`; ou
   - diretamente na célula indicada para teste de conexão.


### Ordem de execução recomendada

- Execute o notebook de cima para baixo.
- Primeiro, rode as células de importação, parâmetros e funções de leitura/análise do IFC.
- Depois, execute as células de checagem, geração de dataframe, exportação e relatório.
- Em seguida, teste a conexão com a Anthropic.
- Por fim, execute as células dos agentes e a célula final de teste com a pergunta do usuário.

---

## 2. Objetivo do código

O objetivo deste código é demonstrar, em ambiente de notebook, uma pipeline de verificação automatizada de conformidade aplicada a modelos IFC, com foco na relação entre elementos de suporte e vigas estruturais.

De forma mais específica, o notebook busca:

- abrir e interpretar um arquivo IFC;
- identificar elementos relevantes para a análise, como vigas e proxies de suporte;
- realizar o pareamento geométrico entre unistruts e vigas;
- aplicar regras de checagem de compliance, com classificação em **PASS** ou **FAIL**;
- consolidar os resultados em tabela;
- exportar os resultados para formatos de saída, como Excel e HTML;
- disponibilizar essas rotinas como tools de um fluxo simples em **CrewAI**, permitindo que um agente execute a análise e outro apresente os resultados conforme a pergunta do usuário.

---

## 3. Reflexão crítica com análise das limitações

Este entregável atende ao objetivo de construir um fluxo inicial e funcional para verificação automatizada em modelo IFC, mas ainda apresenta limitações importantes do ponto de vista técnico e metodológico.

A primeira limitação está na própria qualidade e estrutura dos dados exportados para IFC. Em especial, **em famílias aninhadas, o IFC tem dificuldades de exportar corretamente o nome da família e o tipo**. Em muitos casos, a exportação ocorre no nível do componente isolado. Com isso, quando um mesmo componente aparece tanto isoladamente quanto dentro de uma família aninhada, e ambos compartilham a mesma nomenclatura, torna-se difícil — ou até inviável — identificar, apenas pelas propriedades de nome e tipo, que aquele componente pertence à família aninhada. Essa limitação afeta diretamente a confiabilidade da identificação dos elementos no modelo e pode comprometer parte da lógica de classificação. Para contornar este problema, o enriquecimento do modelo com parâmetros auxiliares é necessário. Neste caso, foi usado o parâmetro STS_Name.

Outra limitação é que a checagem depende fortemente de heurísticas geométricas, tolerâncias e regras pré-definidas. Isso significa que pequenas variações de modelagem, nomenclatura ou exportação podem alterar os resultados. Em outras palavras, o desempenho da rotina ainda depende de um contexto relativamente controlado. Esta aplicação surgiu com o intuito de criar várias tools, uma para cada categoria de suporte diferente (trapeze, cantilever e unistrut), porém, o nível de detalhamento das regras geométricas necessárias para atingir o mínimo de qualidade e reduzir o número de falsos negativos a uma quantidade aceitável, excederia o tempo disponível de desenvolvimento proposto para este trabalho. Vale ressaltar que ainda há algumas falhas nas análises, ou seja, o caso 01 (unistruts apoiados/suportados por vigas) ainda precisaria ser melhor reforçado para aumentar a confiabilidade da informação. No entanto, a facilidade de checagem demonstra potencial imenso de otimização do processo de checagem de qualidae de modelagem, especialmente no Level of Development mais alto. Resultados promissores podem ser obtidos se explorada a integração desta ferramente com visualizador 3D.

Também é importante destacar que o uso de CrewAI e Anthropic, neste caso, não substitui a lógica determinística da checagem. Os agentes atuam principalmente como camada de orquestração e comunicação. Assim, a qualidade da resposta final continua dependente da robustez das funções Python implementadas anteriormente no notebook.

Por fim, este trabalho deve ser entendido como um **protótipo acadêmico**: ele é útil para demonstrar viabilidade, testar arquitetura e apoiar futuras evoluções, mas ainda não representa uma solução plenamente robusta para uso generalizado em diferentes modelos, disciplinas e padrões de exportação IFC.