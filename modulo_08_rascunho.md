# Módulo 8: Exercícios Práticos Guiados (Nível Intermediário)

Agora que exploramos tópicos intermediários como I/O analógico simulado, manipulação de dados, modularização com FCs/FBs, detecção de bordas e intertravamentos, é hora de aplicar esses conceitos em cenários de automação mais desafiadores. Este módulo apresenta exercícios práticos guiados de nível intermediário, utilizando o WinSPS-S7 e o Factory IO. O objetivo é integrar múltiplos conhecimentos para resolver problemas que se aproximam mais da complexidade encontrada na indústria, reforçando as habilidades adquiridas nos módulos anteriores.

**Exercício 6: Separação de Peças por Altura**

*   **Objetivo:** Classificar caixas de diferentes alturas (altas e baixas) que chegam em uma esteira, desviando-as para esteiras de destino separadas.
*   **Cena Factory IO:** Utilize a cena pré-definida "Scene 7 - Sorting by Height (Basic)" ou uma similar. Esta cena geralmente inclui uma esteira de entrada, sensores para detectar a presença e a altura das peças (um sensor baixo e um sensor alto), um atuador de desvio (como um braço empurrador ou uma esteira desviadora) e duas esteiras de saída.
*   **Lógica Ladder (WinSPS-S7):** A lógica envolverá a detecção da peça, a identificação de sua altura com base nos sensores e o acionamento do desviador no momento correto.
    *   Defina na Tabela de Símbolos: `Sensor_Entrada` (I0.0 - detecta qualquer peça), `Sensor_Baixo` (I0.1 - detecta peças baixas e altas), `Sensor_Alto` (I0.2 - detecta apenas peças altas), `Motor_Esteira_Entrada` (Q0.0), `Atuador_Desvio` (Q0.1 - para desviar peças altas), `Motor_Esteira_Baixa` (Q0.2), `Motor_Esteira_Alta` (Q0.3).
    *   Lógica Sugerida:
        1.  Ligar as esteiras de saída (`Motor_Esteira_Baixa`, `Motor_Esteira_Alta`) permanentemente ou com um botão de partida geral.
        2.  Ligar a esteira de entrada (`Motor_Esteira_Entrada`) se o sistema estiver habilitado.
        3.  Quando `Sensor_Entrada` detectar uma peça (borda de subida), iniciar a lógica de classificação.
        4.  Memorizar o estado dos sensores `Sensor_Baixo` e `Sensor_Alto` *enquanto* `Sensor_Entrada` estiver ativo (ou usar um temporizador após a detecção inicial para dar tempo da peça passar pelos sensores de altura).
        5.  Se `Sensor_Alto` foi ativado, a peça é alta. Se apenas `Sensor_Baixo` foi ativado, a peça é baixa.
        6.  Usar um temporizador (TON) iniciado pela detecção em `Sensor_Entrada` para calcular o momento em que a peça chegará ao desviador.
        7.  Se a peça for alta, ativar `Atuador_Desvio` quando o temporizador expirar. Manter o atuador ativo por um curto período (outro timer TP ou lógica Set/Reset com timer).
        8.  Se a peça for baixa, não ativar o desviador.
        9.  Considerar intertravamentos: não iniciar a classificação de uma nova peça enquanto a anterior não tiver liberado a área de desvio.
*   **Mapeamento (Factory IO):**
    *   Configure o driver e conecte.
    *   Mapeie os sensores (`Entry Sensor`, `Low Sensor`, `High Sensor`) para as entradas I0.0, I0.1, I0.2.
    *   Mapeie os atuadores (`Entry Conveyor`, `Sorting Actuator` ou `Diverter Belt`, `Low Belt`, `High Belt`) para as saídas Q0.0, Q0.1, Q0.2, Q0.3.
*   **Teste:**
    1.  Execute a simulação. Peças altas e baixas devem ser geradas.
    2.  Observe se a lógica identifica corretamente a altura das peças.
    3.  Verifique se o atuador de desvio é acionado apenas para as peças altas, no momento correto para desviá-las para a esteira correspondente.
    4.  Confirme se as peças baixas seguem reto para a outra esteira.
    5.  Monitore os timers e as variáveis internas no WinSPS-S7.

**Exercício 7: Linha de Produção com Múltiplas Esteiras**

*   **Objetivo:** Controlar a transferência suave de itens entre três esteiras transportadoras conectadas em sequência.
*   **Cena Factory IO:** Utilize a cena "Scene 3 - From A to B with Interlock" ou crie uma cena com três esteiras (Esteira A, B, C) em linha, cada uma com seu motor e um sensor no final (Sensor A, B, C).
*   **Lógica Ladder (WinSPS-S7):** Implementar uma lógica de "semáforo" ou intertravamento para evitar colisões. Uma esteira só pode mover um item para a próxima se a próxima esteira estiver livre (sem item no seu sensor final).
    *   Defina: `Sensor_A` (I0.0), `Sensor_B` (I0.1), `Sensor_C` (I0.2), `Motor_A` (Q0.0), `Motor_B` (Q0.1), `Motor_C` (Q0.2).
    *   Lógica Sugerida:
        ```ladder
        // Network 1: Controle Motor C (Liga se não houver peça no fim)
        // (Pode ter lógica adicional para parar se a saída estiver bloqueada)
        ----[/]------------( )----
          Sensor_C       Motor_C
        
        // Network 2: Controle Motor B (Liga se Motor C está ligado OU Sensor C está livre, E não há peça em Sensor B)
            +----[ ]----+----[/]----( )----
            |  Motor_C  |  Sensor_B   Motor_B
        ----|           |
            |   [/]     |
            +----Sensor_C+
        
        // Network 3: Controle Motor A (Liga se Motor B está ligado OU Sensor B está livre, E não há peça em Sensor A)
            +----[ ]----+----[/]----( )----
            |  Motor_B  |  Sensor_A   Motor_A
        ----|           |
            |   [/]     |
            +----Sensor_B+
        ```
        *Explicação:* A lógica permite que uma esteira funcione se a próxima estiver funcionando ou vazia no final, mas para imediatamente se seu próprio sensor final detectar uma peça (evitando que ela empurre a peça seguinte).
*   **Mapeamento (Factory IO):**
    *   Configure e conecte.
    *   Mapeie os sensores e motores das três esteiras às respectivas I/Os.
*   **Teste:**
    1.  Execute. Coloque itens na Esteira A.
    2.  Observe a transferência: os itens devem passar de A para B e de B para C.
    3.  Teste o intertravamento: bloqueie a saída da Esteira C (colocando um item manualmente ou deixando acumular). A Esteira C deve parar quando `Sensor_C` for ativado. Consequentemente, a Esteira B deve parar quando `Sensor_B` for ativado, e a Esteira A quando `Sensor_A` for ativado. As esteiras devem reiniciar automaticamente quando o bloqueio for removido.

**Exercício 8: Sistema de Envase Simples**

*   **Objetivo:** Controlar o processo de enchimento de recipientes que se movem em uma esteira. A esteira para, o recipiente é enchido por um tempo definido, e a esteira volta a andar.
*   **Cena Factory IO:** Utilize a cena "Scene 5 - Filling Tank" ou modifique uma cena de esteira adicionando um tanque com válvula de enchimento (`Fill Valve`) posicionado sobre a esteira e um sensor (`Position Sensor`) para detectar quando o recipiente está na posição correta para enchimento.
*   **Lógica Ladder (WinSPS-S7):** Usará detecção de posição, temporizador para o enchimento e controle da esteira e válvula.
    *   Defina: `Sensor_Posicao` (I0.0), `Motor_Esteira` (Q0.0), `Valvula_Enchimento` (Q0.1), `Timer_Enchimento` (T1), `Flag_Enchendo` (M0.0).
    *   Lógica Sugerida:
        1.  Motor da esteira (`Motor_Esteira`) ligado por padrão, a menos que esteja enchendo.
        2.  Quando `Sensor_Posicao` detectar um recipiente (borda de subida `---|P|---`), iniciar a sequência de enchimento:
            *   Ligar a flag `Flag_Enchendo` (Set M0.0).
            *   Desligar `Motor_Esteira`.
            *   Ligar `Valvula_Enchimento`.
            *   Iniciar `Timer_Enchimento` (TON, ex: PT 3s).
        3.  Quando `Timer_Enchimento` terminar (T1.Q = TRUE):
            *   Desligar `Valvula_Enchimento`.
            *   Desligar a flag `Flag_Enchendo` (Reset M0.0).
            *   Ligar `Motor_Esteira` novamente.
        4.  Usar a `Flag_Enchendo` para intertravamentos (ex: não detectar nova peça enquanto estiver enchendo).
        ```ladder
        // Network 1: Ligar Esteira (se não estiver enchendo)
        ----[/]------------( )----
          Flag_Enchendo    Motor_Esteira
        
        // Network 2: Detectar Peça e Iniciar Enchimento
        ----|P|----[ ]----(S)----
        Sensor_Posicao Flag_Enchendo(NF) Flag_Enchendo
        
        // Network 3: Ligar Válvula e Timer durante enchimento
        ----[ ]------------( )----
          Flag_Enchendo    Valvula_Enchimento
        ----[ ]----[TON T1, PT:3s]--
          Flag_Enchendo
        
        // Network 4: Terminar Enchimento e Resetar Flag
        ----[ ]------------(R)----
          T1.Q           Flag_Enchendo
        // (Válvula desliga automaticamente pela Network 3, Esteira liga pela Network 1)
        ```
*   **Mapeamento (Factory IO):**
    *   Configure e conecte.
    *   Mapeie o sensor de posição, o motor da esteira e a válvula de enchimento às I/Os correspondentes.
*   **Teste:**
    1.  Execute. Coloque recipientes na esteira.
    2.  Observe: A esteira move o recipiente até o sensor. A esteira para. A válvula abre por 3 segundos (simulando enchimento). A válvula fecha. A esteira volta a andar. O ciclo se repete para o próximo recipiente.

**Exercício 9: Controle de Nível com Setpoint Analógico (Simulado)**

*   **Objetivo:** Manter o nível de líquido em um tanque dentro de uma faixa definida por um setpoint, usando I/O analógico simulado.
*   **Cena Factory IO:** Utilize a cena "Scene 2 - Level Control". Desta vez, usaremos o sensor de nível (`Level Meter`) que fornece uma saída numérica (analógica simulada) representando a altura, e controlaremos a `Fill Valve` e a `Discharge Valve`.
*   **Lógica Ladder (WinSPS-S7):** Ler o valor analógico simulado, compará-lo com setpoints (alto e baixo) e controlar as válvulas.
    *   Defina: `Leitura_Nivel_Analog` (ex: IW0 - Input Word 0, ou endereço Modbus Input Register), `Setpoint_Alto` (ex: MW10 - Memory Word 10), `Setpoint_Baixo` (ex: MW12), `Valvula_Enchimento` (Q0.0), `Valvula_Descarga` (Q0.1).
    *   Lógica Sugerida:
        1.  Carregar valores nos setpoints (ex: usando instrução MOVE no início ou forçando). Assuma que a leitura analógica e os setpoints estão na mesma escala (0-100, por exemplo, após escalonamento se necessário).
        2.  Ler `Leitura_Nivel_Analog`.
        3.  Comparar: Se `Leitura_Nivel_Analog` <= `Setpoint_Baixo`, ligar `Valvula_Enchimento` e desligar `Valvula_Descarga`.
        4.  Comparar: Se `Leitura_Nivel_Analog` >= `Setpoint_Alto`, desligar `Valvula_Enchimento` e ligar `Valvula_Descarga` (ou apenas desligar enchimento se a descarga for manual/outro processo).
        5.  Implementar uma pequena histerese (banda morta) para evitar que as válvulas fiquem ligando/desligando rapidamente perto dos setpoints.
        ```ladder
        // Network 1: Controle de Enchimento (Liga abaixo do SP Baixo, Desliga acima do SP Alto)
        ----[<=]----(S)---- // Liga se Nivel <= SP_Baixo
         Leitura_Nivel_Analog  Valvula_Enchimento
         Setpoint_Baixo
        ----[>=]----(R)---- // Desliga se Nivel >= SP_Alto
         Leitura_Nivel_Analog  Valvula_Enchimento
         Setpoint_Alto
        
        // Network 2: Controle de Descarga (Opcional - Liga acima do SP Alto, Desliga abaixo do SP Baixo)
        ----[>=]----(S)---- // Liga se Nivel >= SP_Alto
         Leitura_Nivel_Analog  Valvula_Descarga
         Setpoint_Alto
        ----[<=]----(R)---- // Desliga se Nivel <= SP_Baixo
         Leitura_Nivel_Analog  Valvula_Descarga
         Setpoint_Baixo
        ```
*   **Mapeamento (Factory IO):**
    *   Configure e conecte (provavelmente via Modbus para I/O analógico simulado).
    *   Mapeie a saída numérica `Level Meter` para a entrada analógica `Leitura_Nivel_Analog` (IW0 ou endereço Modbus Input Register).
    *   Mapeie os atuadores `Fill Valve` e `Discharge Valve` para as saídas digitais Q0.0 e Q0.1 (ou Modbus Coils).
*   **Teste:**
    1.  Execute. Defina valores para `Setpoint_Alto` (ex: 80) e `Setpoint_Baixo` (ex: 20) no WinSPS-S7.
    2.  Observe o tanque encher. Quando atingir 80, a válvula de enchimento deve fechar (e a de descarga abrir, se implementado).
    3.  Se a descarga estiver ativa, observe o nível baixar. Quando atingir 20, a válvula de descarga deve fechar e a de enchimento abrir.
    4.  O nível deve oscilar entre os setpoints.

**Exercício 10: Paletizador Básico (Movimentos)**

*   **Objetivo:** Simular os movimentos básicos de um braço robótico simples para empilhar caixas em um palete.
*   **Cena Factory IO:** Utilize a cena "Scene 9 - Palletizer (Basic)". Esta cena geralmente tem uma esteira de entrada, um sensor para detectar a caixa na posição de pega, um atuador Z (vertical), um atuador X (horizontal) e uma garra (clamp/grab). 
*   **Lógica Ladder (WinSPS-S7):** Controlar a sequência de movimentos: esperar caixa, descer Z, pegar, subir Z, mover X, descer Z, soltar, subir Z, retornar X.
    *   Defina: `Sensor_Caixa_Pega` (I0.0), `Atuador_Z` (Q0.0 - controla subida/descida, pode precisar de duas saídas ou lógica interna), `Atuador_X` (Q0.1 - controla avanço/retorno), `Garra` (Q0.2 - pegar/soltar), mais flags de estado (M) e timers (T) para a sequência.
    *   Lógica Sequencial (complexa, usar flags de etapa como no Exercício 5, mas com mais passos):
        1.  Esperar `Sensor_Caixa_Pega`.
        2.  Etapa 1: Mover Z para baixo (ativar Q0.0 por um tempo T1 ou até sensor de Z baixo).
        3.  Etapa 2: Ativar Garra (Set Q0.2).
        4.  Etapa 3: Mover Z para cima (desativar Q0.0 por tempo T2 ou até sensor de Z alto).
        5.  Etapa 4: Mover X para posição do palete (ativar Q0.1 por tempo T3 ou até sensor X palete).
        6.  Etapa 5: Mover Z para baixo (ativar Q0.0 por tempo T4 ou até sensor Z baixo palete).
        7.  Etapa 6: Desativar Garra (Reset Q0.2).
        8.  Etapa 7: Mover Z para cima (desativar Q0.0 por tempo T5 ou até sensor Z alto).
        9.  Etapa 8: Mover X de volta para posição inicial (desativar Q0.1 por tempo T6 ou até sensor X inicial).
        10. Repetir ciclo.
        *Nota: Esta lógica é simplificada. Um paletizador real envolve contagem de camadas, ajuste de posição X/Y no palete, etc.*
*   **Mapeamento (Factory IO):**
    *   Configure e conecte.
    *   Mapeie o sensor de pega, os atuadores Z, X e a garra às I/Os correspondentes.
*   **Teste:**
    1.  Execute.
    2.  Observe a sequência de movimentos do paletizador virtual quando uma caixa chega à posição de pega.
    3.  Verifique se a caixa é pega, movida e solta no local aproximado do palete.
    4.  Monitore a sequência de etapas e timers no WinSPS-S7.

Completar estes exercícios intermediários demonstra uma compreensão sólida da programação Ladder e sua aplicação em cenários de automação mais elaborados. Você está agora bem preparado para enfrentar os projetos finais no próximo módulo.
