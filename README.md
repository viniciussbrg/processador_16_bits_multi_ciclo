# 💻 CPU de 16 Bits Multi-Ciclo

[![Logisim-evolution](https://img.shields.io/badge/Logisim--evolution-3.8.0-blue?style=flat-square&logo=logisim)](https://github.com/logisim-evolution/logisim-evolution)
![Disciplina](https://img.shields.io/badge/Disciplina-Arquitetura_de_Computadores-brightgreen?style=flat-square)
![Avaliação](https://img.shields.io/badge/Avaliação-N2-orange?style=flat-square)

> Projeto acadêmico de uma **CPU de 16 bits multi-ciclo** desenvolvida no simulador **Logisim-evolution**, com suporte completo a memória de dados e instruções, arquitetura de pilha e periféricos mapeados em hardware.

---

## 📌 Índice
- [Visão Geral](#-visão-geral)
- [Demonstração](#-demonstração)
- [Arquitetura da CPU](#arquitetura-da-cpu)
- [Registradores Principais](#registradores-principais)
- [Unidade de Controle](#-unidade-de-controle)
- [Datapath](#datapath)
- [ULA (Unidade Lógica e Aritmética)](#-ula-unidade-lógica-e-aritmética)
- [Execução Multi-Ciclo](#-execução-multi-ciclo)
- [Memória e Periféricos](#-memória-e-periféricos)
- [Como Executar](#-como-executar)
- [Equipe](#-equipe)
- [Referências](#-referências)

---

## 📖 Visão Geral

Este projeto foi construído para a disciplina de Arquitetura de Computadores (Prof. **Vinícius da Silva Borges**). O arquivo principal de desenvolvimento é o `CPU16BIT.circ`. 

**A arquitetura implementada inclui:**
- Arquitetura **multi-ciclo**
- Unidade de controle sequencial baseada em contador
- Registradores de uso geral e dedicados
- Controle explícito de micro-etapas de máquina
- Separação entre Memória de Programa (ROM) e Memória de Dados (RAM)
- Sistema nativo de Pilha (Stack)
- Periféricos com I/O mapeado em memória
- Interação por saída **TTY** e sistema de **Vídeo Matricial**

---

## 🎥 Demonstração

Assista ao nosso vídeo de apresentação com a explicação e a CPU em pleno funcionamento:

[![Vídeo da Apresentação do Projeto](https://img.youtube.com/vi/vjuJkxry6SA/0.jpg)](https://youtu.be/vjuJkxry6SA)

*(Clique na miniatura acima para assistir ao vídeo no YouTube)*

---

<a id="arquitetura-da-cpu"></a>
## ⚙️ Arquitetura da CPU

### Especificações Gerais

| Característica | Detalhes |
|:---|:---|
| **Largura de Dados** | 16 bits |
| **Largura de Endereços** | 16 bits |
| **Espaço de Endereçamento** | 2¹⁶ posições disponíveis |
| **Modelo de Execução** | Múltiplos ciclos por instrução |
| **Sinal de Clock** | Sincronismo Global (`CLK`) |
| **Habilitação (Enable)** | Pino `clk_enb` |

---

<a id="registradores-principais"></a>
## 🗄️ Registradores Principais

O datapath desta CPU é munido com vários registradores que desempenham funções específicas no ciclo de busca e execução:

* **`CONTADOR_DE_PROGRAMA` (PC):** Armazena o endereço da próxima instrução a ser buscada na memória ROM.
* **`REGISTRADOR_DE_INSTRUCAO` (IR):** Armazena a instrução atual durante as etapas de busca, decodificação e execução.
* **`REGISTRADOR_DE_ENDERECO_DE_MEMORIA` (MAR):** Responsável por fornecer à RAM e aos periféricos mapeados o endereço de acesso.
* **`ACUMULADOR` (AC):** O registrador primário para as operações lógico-aritméticas computadas pela ULA.
* **`REGISTRADOR_B` (RB):** O segundo operando da ULA; utilizado também nas transferências de memória e uso da pilha.
* **`REGISTRADORES_Y_E_Z`:** Registradores auxiliares para a retenção de dados temporários durante micro-operações.
* **`FLAGS`:** Conjunto de sinalizadores alterados pela ULA (como **Z**ero e **C**arry) fundamentais para os desvios e testes condicionais executados pela Unidade de Controle.
* **`REGISTRADOR_TTY` / `REGISTER_SAIDA`:** Registradores acoplados à interface de monitoramento e envio de dados para as telas e debug.

---

## 🧠 Unidade de Controle

A CPU possui uma **Unidade de Controle distribuída** orientada por um Contador de Ciclos (`CONTADOR_CICLO`). Em conjunto com o circuito combinacional e o Opcode lido do `IR`, ela atua em cada etapa gerando com precisão os sinais ativadores para todo o sistema.

**Exemplos de Sinais de Controle gerados:**
- **Registradores:** Ativação de carregamento (`*_LOAD`) e saída (`*_EN`).
- **ULA:** Seletores de operação (`ADD`, `SUB`, `A_XOR_B`).
- **Memória:** Configuração de barramento (`ADDRESS_WRITE`, `ADDRESS_SELECT`, `RAM_IN`).
- **Pilha:** Controle dinâmico (`STACK_UP`, `STACK_DOWN`, `STACK_STORE`, `STACK_DATA_OUT`).

---


<a id="datapath"></a>
## 🛤️ Datapath

O componente `CIRCUITO` orquestra o caminho de dados entre as memórias, controladores e lógica funcional.

1. **PC → ROM → IR:** O PC aponta para a instrução na ROM, que devolve uma word de 16 bits para o IR armazenar.
2. **IR → Unidade de Controle:** As linhas de opcode alimentam a Unidade de Controle, despachando os sinais que guiarão as próximas rotas.
3. **Fluxo de Operação (AC / RB / ULA):** Os operandos circulam nos registradores principais (AC, RB, auxiliares) e passam pela ULA. O resultado retorna para retroalimentar os registradores, subir à memória ou enviar artefatos de vídeo para o TTY/Display.
4. **MAR / RAM / Periféricos:** O barramento de endereçamento do sistema é isolado pelo MAR, que coordena leituras e escritas determinando se o destino é a RAM, a placa de vídeo ou registradores dos periféricos de hardware.

---

## 🧮 ULA (Unidade Lógica e Aritmética)

Nossa sub-unidade lógica (`ULA`) tem largura total de 16 bits.

- **Operações Aritméticas Implementadas:** Soma (`ADD`) e Subtração (`SUB`).
- **Operações Lógicas Implementadas:** XOR Lógico (`A_XOR_B`).
- **Comportamento de Flags:** Fornece as flags operacionais ao sistema, como Zero e Carry, permitindo que a CPU decida ramificações de fluxo de maneira condicional (`Branch if Equal`, etc.).

---

## 🔄 Execução Multi-Ciclo

A arquitetura segmenta cada instrução em partes menores ditadas pelo `CONTADOR_CICLO`.

- **1. Fetch:** Leitura da instrução apontada na ROM e salvamento destas informações dentro do IR.
- **2. Decode:** Interpretação isolada dos bits de opcode e levantamento prévio de barramentos internos.
- **3. Execute:** Comandos engatilhados pela decodificação entram em vigor (processamento da ULA, requisições de endereços, alterações no ponteiro de pilha).
- **4. Write-back:** Fase terminal onde o Acumulador e os registradores são alimentados com o resultado final, o PC é atualizado e os ciclos de relógio são reiniciados para buscar a próxima instrução.

---

## 💾 Memória e Periféricos

### Memórias
* **ROM (Memória de Programa):** Memória exclusiva de leitura (16-bit word / 16-bit addressing). As instruções devem ser gravadas pelo Logisim (*clique com o botão direito* ➔ *Edit Contents*).
* **RAM (Memória de Dados):** Arquitetura encarregada de manter as variáveis dinâmicas, o espaço Stack e manter mapeadas as regiões relativas aos componentes periféricos controlados pelo `MAR`.

### Pilha (Stack Pointer)
O sistema provê suporte nativo a controle de pilha. A manipulação do ponteiro na RAM ocorre dinamicamente por intermédio dos sinais `STACK_UP`, `STACK_DOWN`, `STACK_STORE` e `STACK_DATA_OUT`.

### Sistemas Externos (Vídeo & TTY)
* **TTY:** Terminal mapeado no `REGISTRADOR_TTY` para impressão sequencial de texto e depuração.
* **Sistema de Vídeo:** Uma matriz de display simulada (8x8) sustentada pelos circuitos `PLACA_DE_VIDEO` e `BUFFER_VIDEO`. O datapath pode injetar pixels diretamente no Framebuffer alterando a visualização gráfica durante o código.

---

## 🚀 Como Executar

**Pré-Requisito:** `Logisim-evolution` (versão 3.8.0 ou superior).

Siga as instruções passo a passo para carregar e iniciar a CPU:

### 1. Abrir o projeto
1. Abra o Logisim-evolution.
2. Acesse `Arquivo > Abrir` e carregue o `CPU16BIT.circ`.
3. Na árvore à esquerda, clique e expanda sobre o projeto, e abra o circuito principal chamado **`FINAL_BUILD`**.

### 2. Configurar a RAM (Sistema Base)
1. Procure pelo componente de **RAM principal** na parte superior do circuito.
2. Clique com o botão direito nela e selecione a opção **`Carregar imagem...`**.
3. Localize e selecione o arquivo: `FINAL_ASSEMBLY_OS_WITH_TETRIS_RAM_FILE`.

### 3. Configurar o Microcódigo nas ROMs
Na seção inferior do `FINAL_BUILD`, carregue as matrizes de controle:
1. Clique com o botão direito na **primeira ROM**, escolha **`Carregar imagem...`** e selecione `OPCODE_ROM_MICROCODE_1`.
2. Clique com o botão direito na **segunda ROM**, escolha **`Carregar imagem...`** e selecione `OPCODE_ROM_MICROCODE_2`.
3. Clique com o botão direito na **terceira ROM**, escolha **`Carregar imagem...`** e selecione `OPCODE_ROM_MICROCODE_3`.

### 4. Inicializar a Simulação
1. Habilite o sinal do Clock acendendo a chave/botão **`clk_enb`** localizada na porção inferior direita do projeto.
2. Configure a frequência do Clock: No menu do topo vá em `Simular` > `Frequência de pulso` e escolha **`2.0 kHz`** *(esta é a velocidade ideal testada)*.
3. Ative os pulsos de clock pressionando as teclas **`CTRL + K`**.
4. Abaixo do terminal **TTY**, utilize a ferramenta "Teclado" para interagir com o terminal simulado!

---

## 👥 Equipe

| Integrante | RA (Registro Acadêmico) |
|:---|:---|
| **Emanuel Carneiro da Silva** | `082230017` |
| **João Vitor Maciel Nai** | `082230004` |
| **José Gabriel de Morais Souza** | `082230001` |

---

## 🔗 Referências

- 🎥 **Vídeo de Inspiração:** [16-BIT CPU with RegisterFile Tutorial! Part 1 - The ALU.](https://www.youtube.com/watch?v=GRM1Oc3erew)