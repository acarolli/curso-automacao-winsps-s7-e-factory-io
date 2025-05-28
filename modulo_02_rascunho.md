# Módulo 2: Fundamentos da Linguagem Ladder

Agora que compreendemos o papel do CLP na automação industrial, é hora de mergulhar na linguagem de programação mais utilizada para esses controladores: a Linguagem Ladder (LD). Como mencionado, sua força reside na sua semelhança com os diagramas elétricos de relés, o que facilita a transição para técnicos e engenheiros já familiarizados com a lógica de comandos elétricos. Neste módulo, desvendaremos os elementos básicos dessa linguagem, construindo a base para a criação de programas de controle eficazes.

O conceito central do Ladder é a **lógica de contatos**. Imagine um circuito elétrico simples com uma fonte de energia, um interruptor e uma lâmpada. O Ladder representa essa lógica de forma gráfica. O programa é organizado em **redes** ou **degraus** (rungs), que são linhas horizontais onde a lógica de controle é implementada. Essas redes são conectadas a duas barras verticais, chamadas de **barras de alimentação** (power rails), que representam as linhas de tensão (geralmente simbolizando 24V e 0V, embora seja uma representação lógica). A energia flui (logicamente) da barra esquerda para a direita, através dos elementos da rede.

Os elementos fundamentais em uma rede Ladder são os **contatos** e as **bobinas**. Os contatos representam as condições de entrada – o estado de sensores, botões ou mesmo resultados de lógicas anteriores. Existem dois tipos básicos de contatos:

1.  **Contato Normalmente Aberto (NA / NO - Normally Open):** Representado por `-[ ]-`. Este contato permite a passagem da lógica (energia) somente quando a variável (entrada física, memória interna, etc.) associada a ele está no estado LIGADO (ou VERDADEIRO, ou nível lógico 1). Pense nele como um botão de pressão que só fecha o circuito quando pressionado.
2.  **Contato Normalmente Fechado (NF / NC - Normally Closed):** Representado por `-[/]-`. Este contato permite a passagem da lógica quando a variável associada a ele está no estado DESLIGADO (ou FALSO, ou nível lógico 0). Ele bloqueia a passagem da lógica quando a variável está LIGADA. Pense nele como um botão de emergência que normalmente mantém o circuito fechado e o abre quando acionado.

As **bobinas** (`-( )-`) representam as saídas do processo lógico – o acionamento de atuadores, lâmpadas ou a definição do estado de memórias internas. Uma bobina é energizada (seu estado vai para LIGADO/VERDADEIRO/1) se houver um caminho lógico contínuo de energia desde a barra de alimentação esquerda até ela, através dos contatos da rede. Se o caminho for interrompido por algum contato aberto (NA desligado ou NF ligado), a bobina é desenergizada (vai para DESLIGADO/FALSO/0).

Combinando contatos, podemos implementar as **instruções lógicas fundamentais**:

*   **Lógica AND:** Para que uma saída seja ativada, *todas* as condições de entrada devem ser verdadeiras. No Ladder, isso é representado colocando contatos em **série** na mesma rede. A lógica só passa se o Contato1 *E* o Contato2 permitirem.
    ```
    --[ ]--[ ]--( )--
     In1   In2   Out1  (Out1 é TRUE se In1 E In2 forem TRUE)
    ```
*   **Lógica OR:** Para que uma saída seja ativada, *pelo menos uma* das condições de entrada deve ser verdadeira. No Ladder, isso é representado criando **ramos paralelos** na rede. A lógica passa se o Contato1 *OU* o Contato2 permitirem.
    ```
        +--[ ]--+
        |  In1  |
    ----|       |--( )--
        |  In2  |  Out1
        +--[ ]--+
          (Out1 é TRUE se In1 OU In2 forem TRUE)
    ```
*   **Lógica NOT:** A condição de saída é o inverso da condição de entrada. Isso é inerentemente implementado pelo uso de contatos NF. Se a entrada `In1` for FALSE, o contato `-[/]-` (associado a `In1`) permitirá a passagem da lógica.

Além da bobina simples (`-( )`), que permanece energizada apenas enquanto a lógica da rede for verdadeira, existem instruções de saída importantes para criar memórias ou travas:

*   **Bobina Set (Travamento) (`-(S)-`):** Quando a lógica da rede que a alimenta se torna verdadeira, esta bobina é energizada e *permanece* energizada, mesmo que a lógica da rede volte a ser falsa. Ela funciona como uma memória que guarda o estado LIGADO.
*   **Bobina Reset (Destravamento) (`-(R)-`):** Quando a lógica da rede que a alimenta se torna verdadeira, esta bobina força a variável associada a ela (geralmente a mesma de uma bobina Set) para o estado DESLIGADO (FALSO/0). Ela é usada para desligar uma saída que foi previamente travada com Set.
    *   **Exemplo de Partida-Parada com Selo:** Uma aplicação clássica usa Set/Reset (ou uma lógica equivalente com bobina simples e contato de selo) para ligar um motor com um botão de partida (NA) e desligá-lo com um botão de parada (NF), mantendo o motor ligado após soltar o botão de partida.

Dois dos blocos funcionais mais essenciais na programação Ladder são os **Temporizadores (Timers)** e **Contadores (Counters)**.

*   **Temporizadores:** Permitem introduzir atrasos ou controlar a duração de eventos. Os tipos mais comuns são:
    *   **TON (Timer On-Delay):** O temporizador começa a contar quando sua entrada de habilitação (EN) é energizada. Após o tempo pré-definido (Preset Time - PT) ter decorrido, e enquanto a entrada EN permanecer energizada, a saída do temporizador (Q) é ativada. Se a entrada EN for desenergizada antes de atingir PT, o tempo acumulado (Elapsed Time - ET) é zerado.
    *   **TOF (Timer Off-Delay):** A saída (Q) é ativada imediatamente quando a entrada (EN) é energizada. Quando a entrada EN é desenergizada, o temporizador começa a contar. A saída Q permanece ativa até que o tempo pré-definido (PT) tenha decorrido após a desenergização de EN.
    *   **TP (Timer Pulse):** Gera um pulso de duração fixa. Quando a entrada (EN) é energizada, a saída (Q) é ativada imediatamente e permanece ativa pelo tempo pré-definido (PT), independentemente do estado de EN após o acionamento inicial. A saída só será reativada por uma nova transição de desligado para ligado na entrada EN após o pulso ter terminado.
*   **Contadores:** Utilizados para contar eventos, como a passagem de peças ou a ocorrência de ciclos.
    *   **CTU (Up Counter):** Conta eventos crescentes. Cada vez que a entrada de contagem (CU - Count Up) recebe um pulso (transição de FALSO para VERDADEIRO), o valor acumulado (Current Value - CV) é incrementado. Quando CV atinge ou excede o valor pré-definido (Preset Value - PV), a saída do contador (Q) é ativada. Uma entrada de Reset (R) zera o valor acumulado (CV).
    *   **CTD (Down Counter):** Conta eventos decrescentes. Começa com um valor pré-definido (PV) carregado na entrada (LD - Load). Cada pulso na entrada de contagem (CD - Count Down) decrementa o valor acumulado (CV). A saída (Q) geralmente é ativada enquanto CV for maior que zero e desativada quando CV chega a zero.
    *   **CTUD (Up/Down Counter):** Combina as funcionalidades do CTU e CTD, possuindo entradas separadas para contagem crescente (CU) e decrescente (CD), além de Reset (R) e Load (LD). Possui saídas que indicam quando o limite superior (QU) ou inferior (QD) é atingido.

Frequentemente, precisamos tomar decisões baseadas na comparação de valores. As **Instruções de Comparação (CMP)** permitem comparar o valor de uma variável (acumulador de contador, valor de temporizador, entrada analógica escalonada, etc.) com uma constante ou outra variável. Exemplos comuns incluem: Igual (== ou EQU), Diferente (!= ou NEQ), Maior que (> ou GRT), Menor que (< ou LES), Maior ou Igual (>= ou GEQ), Menor ou Igual (<= ou LEQ). O resultado da comparação é um valor booleano (VERDADEIRO ou FALSO) que pode ser usado como condição em uma rede Ladder.

Embora o foco do Ladder seja a lógica booleana, muitas vezes precisamos realizar **Operações Matemáticas Básicas**. Instruções como Adição (ADD), Subtração (SUB), Multiplicação (MUL) e Divisão (DIV) permitem manipular valores numéricos armazenados em registradores ou áreas de memória do CLP. Isso é útil para cálculos de escalonamento, ajustes de setpoints, etc. A disponibilidade e a forma de uso dessas instruções podem variar entre diferentes modelos de CLP e softwares de programação como o WinSPS-S7.

A **estrutura básica de um programa Ladder** consiste em múltiplas redes (rungs) executadas sequencialmente pela CPU do CLP, de cima para baixo, repetidamente, como parte do ciclo de scan. É fundamental adicionar **comentários** claros às redes, contatos, bobinas e variáveis (tags) para tornar o programa compreensível e facilitar a manutenção futura.

Conforme os programas crescem em complexidade, torna-se essencial organizá-los de forma modular. Introduzimos aqui os conceitos de **Blocos de Organização (OBs)**, **Funções (FCs)** e **Blocos de Função (FBs)**, que serão explorados com mais detalhes posteriormente. O OB1 é o bloco principal onde o programa do usuário geralmente reside e é chamado ciclicamente pelo sistema operacional do CLP. FCs são blocos de código reutilizáveis que não possuem memória própria (instância). FBs também são reutilizáveis, mas possuem uma memória associada (Bloco de Dados de Instância - DB de Instância), permitindo que mantenham seu estado entre as chamadas. Essa modularidade é crucial para programas bem estruturados.

Por fim, algumas **boas práticas iniciais** são essenciais desde o começo: use nomes significativos e padronizados para suas variáveis (tags), como `Botao_Liga_Motor_Esteira1` em vez de `I0.0`. Adicione comentários explicando a finalidade de cada rede ou seção lógica complexa. Isso não só ajuda os outros a entenderem seu código, mas também auxilia você mesmo no futuro.

Com esses fundamentos da linguagem Ladder, estamos prontos para começar a explorar o ambiente WinSPS-S7 no próximo módulo e colocar esses conceitos em prática.
