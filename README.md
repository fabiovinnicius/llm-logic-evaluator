# llm-logic-evaluator

Avalia se um LLM resolve melhor um problema de satisfatibilidade **raciocinando direto em linguagem natural** ou **traduzindo o problema para código Z3** e deixando o solver decidir.

**Resultado:** com o mesmo modelo (`gemini-3.1-flash-lite`) e as mesmas 20 instâncias, a resposta direta acertou **15/20 (75%)** e a tradução para Z3 acertou **20/20 (100%)**. Todos os erros da resposta direta foram falsos positivos: o modelo respondeu SAT para instâncias impossíveis.

## O problema

Montar uma prova de 6 questões a partir de um banco, onde cada questão tem uma dificuldade e um ou mais tópicos (T1–T4):

- pelo menos 1 questão de cada tópico;
- exatamente 1 difícil, 2 fáceis e 3 médias.

Cada instância é um banco de questões diferente. A pergunta é: existe uma prova válida (SAT) ou não (UNSAT)?

## Metodologia

1. **Gabarito:** o Z3 resolve cada instância e gera a resposta esperada.
2. **Abordagem A — resposta direta:** o problema vai em texto para o Gemini, que responde apenas `SAT` ou `UNSAT`.
3. **Abordagem B — LLM como tradutor:** o Gemini recebe o mesmo problema e escreve um programa Python com Z3. O código é executado em um subprocesso (timeout de 10 s) e a saída dele é a resposta.
4. As duas respostas são comparadas com o gabarito.

As 20 instâncias cobrem 7 casos, escolhidos para ter armadilhas diferentes:

| Caso | Descrição | Esperado | Instâncias | A (direta) | B (Z3) |
|---|---|---|---|---|---|
| 1 | Cenário base | SAT | 4 | 4 | 4 |
| 2 | Faltam questões médias | UNSAT | 3 | **0** | 3 |
| 3 | Cotas OK, mas nenhuma questão cobre T4 | UNSAT | 3 | 3 | 3 |
| 4 | Solução única, todas as 6 questões obrigatórias | SAT | 3 | 3 | 3 |
| 5 | T1 e T4 só cobertos por difíceis diferentes | UNSAT | 3 | 2 | 3 |
| 6 | Contagens OK, mas T2 descoberto | UNSAT | 2 | 1 | 2 |
| 7 | Stress test: 16 questões e muito ruído | SAT | 2 | 2 | 2 |
| | **Total** | | **20** | **15 (75%)** | **20 (100%)** |

## Leitura dos resultados

- **Os erros da Abordagem A foram todos falsos positivos (UNSAT → SAT).** O modelo tende a afirmar que existe solução em vez de verificar cada restrição.
- **O caso 2 é o mais revelador:** basta contar que há só 2 questões médias quando a prova pede 3, e mesmo assim a resposta direta errou as 3 instâncias. Restrições de contagem são o ponto fraco.
- **Na Abordagem B o modelo só precisa traduzir as regras**, e a verificação fica com o solver. Isso eliminou os erros neste conjunto.

## Limitações

- 20 instâncias e uma execução por instância. Não medimos a variância entre execuções; uma rodada anterior da Abordagem A chegou a 80%.
- Um único modelo (`gemini-3.1-flash-lite`).
- A Abordagem B executa código gerado pelo modelo. O subprocesso com timeout não é um sandbox; não rode isso com entradas não confiáveis.

## Como rodar

```bash
pip install -r requirements.txt
echo "GEMINI_API_KEY=sua_chave" > .env
jupyter notebook Logic-evaluator.ipynb
```

Rode as células em ordem: geração das instâncias e gabarito com Z3, Abordagem A e Abordagem B.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `Logic-evaluator.ipynb` | Pipeline completo: modelagem, geração das instâncias, gabarito e as duas abordagens |
| `instancias.json` | As 20 instâncias com o gabarito do Z3 |
| `resultados_gemini_nlp.json` | Abordagem A: prompt enviado e resposta de cada instância |
| `resultados_gemini_z3.json` | Abordagem B: código gerado, saída e resultado de cada instância |

## Tecnologias

Python 3, Z3 Theorem Prover, Google GenAI SDK (Gemini), Jupyter.

## Autores

- Fábio Vinnicius — [@fabiovinnicius](https://github.com/fabiovinnicius)
- Daniel Nobre — [dannobre.com.br](https://dannobre.com.br)
