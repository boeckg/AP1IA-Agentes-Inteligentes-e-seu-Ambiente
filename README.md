# Aspirador de Pó – Simulador de Agente Inteligente

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Dependências](https://img.shields.io/badge/depend%C3%AAncias-nenhuma-brightgreen)
![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-success)
![Disciplina](https://img.shields.io/badge/UFSM-CSCDIA00017-informational)

Simulador do ambiente clássico do **mundo do aspirador de pó** (Russell & Norvig, 2013), feito de forma incremental para a Atividade Prática 1 – *Agentes Inteligentes e seu Ambiente*. Um único programa resolve as quatro etapas: varredura simples, sensores com campo de visão, bateria limitada, e obstáculos com sujeira aleatória.

## Índice

- [Descrição](#descrição)
- [Status do projeto](#status-do-projeto)
- [Funcionalidades](#funcionalidades)
- [Demonstração](#demonstração)
- [Instalação](#instalação)
- [Como usar](#como-usar)
- [Arquivo de configuração](#arquivo-de-configuração)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Tecnologias utilizadas](#tecnologias-utilizadas)
- [Roadmap](#roadmap)
- [Suporte](#suporte)
- [Autores](#autores)
- [Licença](#licença)

## Descrição

O ambiente é uma matriz em que `0` é uma célula limpa, `1` uma célula suja e `9` uma parede. O agente anda uma célula por vez nas quatro direções, aspira as sujeiras e, ao final, o simulador mostra as métricas de desempenho: movimentos, aspirações, recargas, total de ações, células sujas, limpas e restantes, entre outras.

O código separa os três componentes pedidos no enunciado:

- **Ambiente** (`ambiente.py`): guarda o estado do mapa e aplica as consequências das ações.
- **Agente** (`agente.py`): escolhe as ações, explora o mapa e guarda na memória o que percebeu.
- **Medição de desempenho** (`main.py`): executa a simulação e registra as métricas.

## Status do projeto

**Concluído**: as quatro etapas da atividade estão implementadas.

## Funcionalidades

- **Etapa 1 – Varredura:** o agente percorre o mapa em zigue-zague e aspira cada célula suja que encontra.
- **Etapa 2 – Sensores (`SIGHT-RANGE`):** o agente explora até ter observado todas as células e depois limpa as sujeiras conhecidas. Há duas estratégias de limpeza:
  - `mais_proxima`: vai sempre para a sujeira mais próxima pelo menor caminho.
  - `ordem_descoberta`: limpa na ordem em que as sujeiras foram vistas.
- **Etapa 3 – Bateria:** antes de cada deslocamento, o agente confere se tem carga para ir, agir e voltar à estação. Se não tiver, volta para recarregar.
- **Etapa 4 – Obstáculos e estocasticidade:** o agente desvia das paredes usando busca em largura (BFS). Novas sujeiras aparecem com probabilidade `CELL-DIRT-PROB`, e o resultado é reproduzível pela `RANDOM-SEED`. A execução para ao atingir `MAX-STEPS`.
- **Detecção automática da etapa** a partir dos parâmetros do arquivo de entrada.

## Demonstração

```console
$ python3 main.py input1.txt
Etapa: 1
Estratégia: zigue-zague
Número de movimentos realizados: 63
Número de ações de aspiração realizadas: 11
Número de recargas efetuadas: 0
Número total de ações: 74
Quantidade de células inicialmente sujas: 11
Quantidade de células efetivamente limpas: 11
Quantidade de células sujas restantes: 0
Novas sujeiras geradas: 0
Bateria restante: 100.0
Células visitadas na exploração: 64
Células observadas na exploração: 64
Células apenas observadas na exploração: 0
Motivo do encerramento: Mapa limpo.
```

## Instalação

**Pré-requisito:** Python **3.10 ou superior**. Não é preciso instalar nada com `pip`, porque o projeto só usa módulos da biblioteca padrão (veja [Tecnologias utilizadas](#tecnologias-utilizadas)).

1. Verifique a versão do Python:
   ```bash
   python3 --version
   ```
2. Extraia o `.zip` e entre na pasta do projeto:
   ```bash
   cd trabIA
   ```

## Como usar

```bash
python3 main.py ARQUIVO.txt [ESTRATEGIA] [ETAPA]
```

| Argumento | Obrigatório | Valores | Padrão |
|---|---|---|---|
| `ARQUIVO.txt` | sim | arquivo de configuração (formato do enunciado) | — |
| `ESTRATEGIA` | não | `mais_proxima` ou `ordem_descoberta` | `mais_proxima` |
| `ETAPA` | não | `1`, `2`, `3` ou `4` | detectada automaticamente |


**Exemplos:**

```bash
python3 main.py input1.txt                      # Etapa 1 – zigue-zague
python3 main.py input2.txt                      # Etapa 2 – sujeira mais próxima
python3 main.py input2.txt ordem_descoberta     # Etapa 2 – ordem de descoberta
python3 main.py input3.txt                      # Etapa 3 – bateria
python3 main.py input4.txt ordem_descoberta 4   # Etapa 4 – forçando a etapa
```

**Detecção automática da etapa** (quando `ETAPA` não é informada):

| Condição no arquivo | Etapa |
|---|---|
| Mapa com `9`, `CELL-DIRT-PROB > 0` ou `MAX-STEPS` definido | 4 |
| `CHARGE-PER-MOVEMENT` ou `CHARGE-PER-VACUUM` > 0 | 3 |
| `SIGHT-RANGE` > 0 | 2 |
| Nenhuma das anteriores | 1 |

Na etapa 1 o argumento de estratégia é ignorado.

## Arquivo de configuração

O arquivo é lido pelo `parser.py` fornecido pelo professor. As linhas com `0`, `1` e `9` formam o mapa, e as outras seguem o formato `CHAVE:valor`. Linhas vazias e linhas que começam com `#` são ignoradas.

```text
00009000
11009010
00000000
START:0,0
SIGHT-RANGE:1
CHARGE-PER-MOVEMENT:0.2
CHARGE-PER-VACUUM:2.0
POWER-STATION-LOC:2,0
CELL-DIRT-PROB:0.02
RANDOM-SEED:42
MAX-STEPS:1000
```

| Parâmetro | Significado | Padrão |
|---|---|---|
| `START:l,c` | posição inicial do agente | `0,0` |
| `SIGHT-RANGE:r` | raio de visão | `0` |
| `CHARGE-PER-MOVEMENT:x` | consumo de bateria por movimento (%) | `0` |
| `CHARGE-PER-VACUUM:x` | consumo de bateria por aspiração (%) | `0` |
| `POWER-STATION-LOC:l,c` | posição da estação de recarga | posição inicial |
| `CELL-DIRT-PROB:p` | probabilidade de surgir sujeira a cada movimento | `0` |
| `RANDOM-SEED:s` | semente do gerador aleatório | `42` |
| `MAX-STEPS:n` | limite de ações (movimentos + aspirações + recargas) | sem limite |

## Estrutura do projeto

```text
trabIA/
├── main.py         # simulador: laço exploração/limpeza, política de bateria, métricas
├── ambiente.py     # classe Ambiente: mapa, ações, sensor (campo de visão), sorteio de sujeira
├── agente.py       # classe Agente: memória, decisões, busca em largura (BFS), bateria
├── parser.py       # leitura do arquivo de configuração (fornecido)
├── input1.txt      # exemplo da etapa 1
├── input2.txt      # exemplo da etapa 2
├── input3.txt      # exemplo da etapa 3
├── input4.txt      # exemplo da etapa 4
├── relatorio.pdf   # relatório com arquitetura, estratégias e resultados
└── README.md
```

## Tecnologias utilizadas

- **Python 3.10+**
- Módulos da biblioteca padrão:
  - `sys`: argumentos de linha de comando (`main.py`)
  - `random`: sorteio de sujeira com semente (`ambiente.py`)
  - `dataclasses` e `pathlib`: leitura da configuração (`parser.py`)

## Roadmap

- [ ] Exploração que pule linhas de acordo com o raio de visão, para visitar menos células.
- [ ] Implementar a terceira estratégia do enunciado: ordenar as sujeiras por coordenadas.
- [ ] Política de recarga menos conservadora.


## Autores

| Nome | Contato |
|---|---|
| Gustavo Boeck da Silva | gustavo.boeck@acad.ufsm.br · |
| Matheus da Silva Dias | matheus-dias.2@acad.ufsm.br |


**Referência:** RUSSELL, S.; NORVIG, P. *Inteligência Artificial*. 3. ed. Rio de Janeiro: Elsevier, 2013.
