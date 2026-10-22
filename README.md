# Sprint 4 – GoodWe Smart Charging Station

## Sobre o projeto

Este projeto foi desenvolvido como solução final integrada da disciplina de **Arquitetura de Computadores**.

O objetivo é simular uma estação inteligente de recarga de veículos elétricos utilizando **Raspberry Pi Pico**, **MicroPython** e o simulador **Wokwi**.

A solução foi inspirada no conceito da **GoodWe Smart Energy Controller**, demonstrando como um sistema computacional pode receber dados, processá-los e tomar decisões automáticas para distribuir uma quantidade limitada de energia entre diferentes veículos.

---

## Objetivo

A estação simula três veículos elétricos conectados simultaneamente.

Cada veículo possui:

- identificação;
- nível de bateria;
- potência solicitada;
- potência recebida;
- estado da recarga.

O sistema compara a **energia disponível na estação** com a **demanda total dos veículos** e define automaticamente como a energia será distribuída.

---

## Tecnologias utilizadas

- Raspberry Pi Pico
- MicroPython
- Wokwi
- LEDs
- Resistores
- Monitor Serial

---

## Veículos simulados

| Veículo | Bateria | Potência solicitada |
|---|---:|---:|
| EV1 | 40% | 2000 W |
| EV2 | 60% | 2500 W |
| EV3 | 25% | 3000 W |

A demanda total da estação é:

```text
2000 + 2500 + 3000 = 7500 W
```

---

## Indicadores de estado

Cada veículo possui três LEDs:

- 🟢 Verde – Recarga ativa
- 🟡 Amarelo – Recarga reduzida ou prioritária
- 🔴 Vermelho – Veículo aguardando ou bloqueado

### Pinos utilizados

| Veículo | Verde | Amarelo | Vermelho |
|---|---:|---:|---:|
| EV1 | GP15 | GP14 | GP13 |
| EV2 | GP12 | GP11 | GP10 |
| EV3 | GP9 | GP8 | GP7 |

---

# Funcionamento do sistema

O programa calcula inicialmente a demanda total dos três veículos.

```python
demanda_total = 0

for veiculo in veiculos:
    demanda_total += veiculo["potencia"]
```

A demanda total é de:

```text
7500 W
```

Em seguida, o Raspberry Pi Pico compara esse valor com a energia disponível na estação.

---

# Cenário 1 – Alta disponibilidade de energia

Energia disponível:

```text
8000 W
```

Demanda total:

```text
7500 W
```

Como existe energia suficiente para todos os veículos, cada um recebe exatamente a potência solicitada.

Resultado:

```text
EV1 -> RECARGA ATIVA
EV2 -> RECARGA ATIVA
EV3 -> RECARGA ATIVA
```

Os três LEDs verdes são acionados.

```text
🟢 EV1
🟢 EV2
🟢 EV3
```

---

# Cenário 2 – Energia limitada

Energia disponível:

```text
5000 W
```

Demanda total:

```text
7500 W
```

Como a energia disponível não é suficiente para atender completamente os três veículos, o sistema realiza uma distribuição proporcional da energia.

O fator utilizado é calculado por:

```python
fator = energia_disponivel / demanda_total
```

Neste caso:

```text
5000 / 7500 ≈ 0,667
```

Assim, cada veículo recebe aproximadamente 66,7% da potência originalmente solicitada.

Exemplo:

```text
EV1
Solicita: 2000 W
Recebe: aproximadamente 1333 W

EV2
Solicita: 2500 W
Recebe: aproximadamente 1666 W

EV3
Solicita: 3000 W
Recebe: 2000 W
```

Resultado:

```text
🟡 EV1
🟡 EV2
🟡 EV3
```

Status:

```text
RECARGA CONTROLADA
```

---

# Cenário 3 – Baixa disponibilidade de energia

Energia disponível:

```text
2000 W
```

Neste cenário, não é possível atender adequadamente os três veículos.

O sistema então identifica automaticamente o veículo com o menor nível de bateria.

Níveis:

```text
EV1 = 40%
EV2 = 60%
EV3 = 25%
```

Como o EV3 possui a menor bateria, ele recebe prioridade.

Resultado:

```text
EV1 -> AGUARDANDO
EV2 -> AGUARDANDO
EV3 -> RECARGA PRIORITÁRIA
```

Indicação:

```text
🔴 EV1
🔴 EV2
🟡 EV3
```

O EV3 recebe os 2000 W disponíveis.

---

# Algoritmo de gerenciamento

A lógica principal pode ser representada da seguinte forma:

```text
ENERGIA DISPONÍVEL
        |
        v
DEMANDA DOS VEÍCULOS
        |
        v
Energia suficiente?
     /        \
   SIM        NÃO
    |           |
Todos         Energia >=
carregam      50% da demanda?
              /          \
            SIM          NÃO
             |             |
        Distribuição     Prioridade
        proporcional     para veículo
                         com menor bateria
```

---

# Arquitetura de Computadores

O projeto aplica diferentes conceitos estudados na disciplina.

## Entrada

Os dados de entrada são:

- energia disponível;
- nível de bateria;
- potência solicitada;
- identificação dos veículos.

---

## Memória

Os dados são armazenados temporariamente em variáveis e estruturas de dados durante a execução do programa.

Exemplo:

```python
veiculos = [
    {
        "nome": "EV1",
        "bateria": 40,
        "potencia": 2000
    }
]
```

---

## Processamento

O Raspberry Pi Pico executa o algoritmo em MicroPython.

O sistema:

1. calcula a demanda total;
2. compara a demanda com a energia disponível;
3. calcula a distribuição de potência;
4. identifica prioridades;
5. define o estado de cada veículo.

---

## Saída

As informações são apresentadas através de:

- LEDs;
- Monitor Serial;
- status dos veículos;
- potência recebida.

---

# Fluxo do sistema

```text
                ENTRADAS
                   |
       +-----------------------+
       | Energia disponível    |
       | Bateria dos veículos  |
       | Potência solicitada   |
       +-----------------------+
                   |
                   v
       +-----------------------+
       | Raspberry Pi Pico     |
       | MicroPython           |
       | Processamento         |
       +-----------------------+
                   |
                   v
       +-----------------------+
       | Algoritmo de          |
       | gerenciamento         |
       +-----------------------+
                   |
          +--------+--------+
          |        |        |
          v        v        v
         EV1      EV2      EV3
          |        |        |
          v        v        v
        LEDs     LEDs     LEDs

              Monitor Serial
```

---

# Representação de dados

O sistema também demonstra diferentes representações numéricas utilizadas pelos computadores.

Exemplo para:

```text
Energia disponível = 5000 W
```

Representações:

```text
Decimal:      5000
Binário:      1001110001000
Hexadecimal:  0x1388
```

No programa:

```python
print("Decimal:", energia_disponivel)
print("Binario:", bin(energia_disponivel))
print("Hexadecimal:", hex(energia_disponivel))
```

---

# Relação com a GoodWe

O projeto não busca reproduzir completamente um equipamento comercial da GoodWe.

A proposta é criar uma solução educacional inspirada no conceito de **gerenciamento inteligente de energia**.

O sistema demonstra como um controlador pode analisar a disponibilidade energética e ajustar automaticamente a potência de recarga de diferentes veículos.

---

# Estrutura do repositório

```text
Sprint-4-GoodWe/
│
├── main.py
├── diagram.json
└── README.md
```

### main.py

Contém o código desenvolvido em MicroPython.

### diagram.json

Contém a configuração do circuito utilizado no Wokwi.

### README.md

Contém a documentação técnica do projeto.

---

# Conclusão

A GoodWe Smart Charging Station demonstra a integração entre **hardware, software e dados**.

O Raspberry Pi Pico recebe informações sobre energia e veículos, armazena os dados em memória, executa um algoritmo de gerenciamento e controla automaticamente a recarga.

Com isso, o projeto demonstra como a automação pode contribuir para o uso mais eficiente da energia disponível em uma estação de recarga de veículos elétricos.
