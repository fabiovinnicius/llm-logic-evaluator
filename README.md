# llm-logic-evaluator

## Sobre o Projeto
Este repositório contém um pipeline de testes focado em avaliar a capacidade de raciocínio lógico de Large Language Models (LLMs) na resolução de problemas de satisfatibilidade (Constraint Satisfaction Problems).

O estudo de caso central envolve a elaboração de uma prova com restrições específicas (cruzamento de múltiplos tópicos e diferentes níveis de dificuldade). O objetivo é mensurar o desempenho da IA através de duas abordagens distintas.

## Metodologia

O projeto avalia as LLMs testando a seguinte hipótese: *A IA é mais eficiente resolvendo problemas lógicos puramente através de texto, ou atuando como tradutora de restrições para ferramentas matemáticas rigorosas?*

O pipeline é dividido em duas estratégias:
* **Abordagem A (Dedução Direta):** Avaliação da capacidade da LLM de interpretar o problema em linguagem natural e fornecer a solução diretamente.
* **Abordagem B (Geração de Código + Solver):** A LLM recebe o problema e tem a tarefa de modelá-lo utilizando a API do `Z3 Theorem Prover` em Python. O código gerado é então validado.

Em ambas as abordagens, os resultados são comparados com um gabarito dinâmico gerado previamente, garantindo a corretude das validações.

## Status Atual e Resultados Parciais
* **Abordagem A (NLP):** Concluída. O modelo atingiu **80% de precisão** inferindo a lógica sem execução de código. A análise detalhada de como a IA lidou com "falsos positivos" e restrições de contagem está documentada no arquivo [resultados_abordagem_A.md](resultados_abordagem_A.md).
* **Abordagem B (Geração de Código):** Em desenvolvimento.

## Tecnologias Utilizadas
* **Python 3.x:** Linguagem principal do pipeline de testes.
* **Z3 Theorem Prover:** Solver utilizado para gerar o gabarito das questões e para executar os códigos gerados pelas LLMs.
* **Google GenAI SDK:** Integração com os modelos Gemini (`gemini-3.1-flash-lite`) para as inferências e avaliações.
