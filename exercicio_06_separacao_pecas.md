# Exercício Prático 6: Separação de Peças por Altura

**Objetivo:** Classificar caixas de duas alturas diferentes (baixas e altas) que chegam por uma esteira principal, desviando as caixas altas para uma esteira secundária e deixando as baixas seguirem reto.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 7 - Sorting by Height (Basic)"**. Esta cena é projetada para este tipo de tarefa e geralmente inclui:
    *   Um emissor (`Emitter`) que gera caixas altas e baixas.
    *   Uma esteira de entrada (`Entry Conveyor`).
    *   Sensores para detectar a presença e altura: um sensor de entrada (`Entry Sensor`), um sensor baixo (`Low Sensor`) e um sensor alto (`High Sensor`).
    *   Um atuador de desvio (`Pusher` ou `Belt Conveyor Diverter`).
    *   Duas esteiras de saída: uma para peças baixas (reto) e outra para peças altas (desviadas).
    *   Removedores (`Remover`) no final das esteiras de saída.

**Lógica Ladder (WinSPS-S7):**

A lógica precisa detectar a peça, identificar sua altura e acionar o desviador no momento certo apenas para as peças altas.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_Entrada`: I0.0 (BOOL) - Detecta a chegada de qualquer peça.
    *   `Sensor_Baixo`: I0.1 (BOOL) - Detecta peças baixas e altas.
    *   `Sensor_Alto`: I0.2 (BOOL) - Detecta apenas peças altas.
    *   `Motor_Esteira_Entrada`: Q0.0 (BOOL) - Controla a esteira de entrada.
    *   `Atuador_Desvio`: Q0.1 (BOOL) - Ativa o mecanismo de desvio (ex: Pusher).
    *   `Motor_Esteira_Baixa`: Q0.2 (BOOL) - Controla a esteira de peças baixas (saída reta).
    *   `Motor_Esteira_Alta`: Q0.3 (BOOL) - Controla a esteira de peças altas (saída desviada).
    *   `Peca_Presente`: M0.0 (BOOL) - Flag indicando que uma peça está na área de sensoriamento/desvio.
    *   `Peca_Alta`: M0.1 (BOOL) - Flag indicando que a peça detectada é alta.
    *   `Timer_Atraso_Desvio`: T1 (TIMER) - Temporizador para atrasar o acionamento do desviador.
    *   `Timer_Duracao_Desvio`: T2 (TIMER) - Temporizador para a duração do pulso do desviador.

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Controle das Esteiras de Saída
    TITLE=Ligar Esteiras de Saída
    // Descrição: Liga as esteiras de saída permanentemente (ou com um botão geral).
    
    // (Lógica para ligar Q0.2 e Q0.3 - pode ser sempre ON ou com botão de partida)
    // Exemplo simples: Sempre ON
    ----( )----
      Motor_Esteira_Baixa
      (Q0.2)
    ----( )----
      Motor_Esteira_Alta
      (Q0.3)
      
    NETWORK 2: Controle Esteira de Entrada
    TITLE=Ligar Esteira de Entrada
    // Descrição: Liga a esteira de entrada (ou com botão geral).
    
    // (Lógica para ligar Q0.0 - pode ser sempre ON ou com botão de partida)
    // Exemplo simples: Sempre ON
    ----( )----
      Motor_Esteira_Entrada
      (Q0.0)
      
    NETWORK 3: Detecção de Peça na Entrada (Borda de Subida)
    TITLE=Detectar Chegada da Peça
    // Descrição: Detecta a borda de subida do Sensor_Entrada e ativa a flag
    // Peca_Presente, mas somente se outra peça não estiver sendo processada.
    
    ----|P|----[/]----(S)----
      Sensor_Entrada Peca_Presente Peca_Presente
      (I0.0)         (M0.0)       (M0.0)
      
    NETWORK 4: Identificação da Altura da Peça
    TITLE=Verificar Altura
    // Descrição: Quando uma peça está presente, verifica o Sensor_Alto.
    // Se Sensor_Alto estiver ativo, marca a flag Peca_Alta.
    
    ----[ ]----[ ]----(S)----
      Peca_Presente Sensor_Alto  Peca_Alta
      (M0.0)        (I0.2)       (M0.1)
      
    NETWORK 5: Iniciar Atraso para Desvio (se Peça Alta)
    TITLE=Timer Atraso Desvio
    // Descrição: Inicia um timer (T1) quando uma peça alta é detectada
    // na entrada para dar tempo dela chegar ao desviador.
    
    ----[ ]----[ ]----[TON T1, PT:1.5s]-- // Ajustar PT conforme velocidade/distância
      Peca_Presente Peca_Alta
      (M0.0)        (M0.1)
      
    NETWORK 6: Acionar Desviador (Peça Alta)
    TITLE=Ativar Desviador
    // Descrição: Quando o Timer T1 termina, ativa o Atuador_Desvio.
    // Inicia também o Timer T2 para controlar a duração do desvio.
    
    ----[ ]------------( )----
      T1.Q           Atuador_Desvio
                       (Q0.1)
    ----[ ]----[TON T2, PT:0.5s]-- // Ajustar PT para duração do pulso
      T1.Q
      
    NETWORK 7: Desativar Desviador e Resetar Flags
    TITLE=Resetar Ciclo de Desvio
    // Descrição: Quando o Timer T2 (duração do desvio) termina,
    // desativa o Atuador_Desvio e reseta as flags Peca_Presente e Peca_Alta,
    // preparando para a próxima peça.
    
    ----[ ]------------(R)----
      T2.Q           Atuador_Desvio
                       (Q0.1)
    ----[ ]------------(R)----
      T2.Q           Peca_Presente
                       (M0.0)
    ----[ ]------------(R)----
      T2.Q           Peca_Alta
                       (M0.1)
                       
    NETWORK 8: Resetar Flags se Peça Baixa Sair da Entrada
    TITLE=Resetar Ciclo Peça Baixa
    // Descrição: Se uma peça estava presente mas não era alta (Peca_Alta=FALSE),
    // reseta a flag Peca_Presente quando ela sai do Sensor_Entrada (borda de descida).
    
    ----|N|----[/]----(R)----
      Sensor_Entrada Peca_Alta    Peca_Presente
      (I0.0)         (M0.1)       (M0.0)
    // Nota: |N| representa a detecção de borda negativa (Falling Edge).
    ```

3.  **Explicação da Lógica:**
    *   As esteiras de saída e entrada são ligadas (Networks 1 e 2 - podem ser aprimoradas com botões gerais de partida/parada).
    *   A chegada de uma peça é detectada pela borda de subida de `Sensor_Entrada`, ativando `Peca_Presente` (Network 3).
    *   Enquanto `Peca_Presente` está ativa, o estado de `Sensor_Alto` é verificado. Se alto, `Peca_Alta` é ativada (Network 4).
    *   Se `Peca_Alta` for TRUE, o `Timer_Atraso_Desvio` (T1) é iniciado (Network 5). O tempo (PT) deve ser ajustado para corresponder ao tempo que a peça leva para viajar do sensor de entrada até o desviador.
    *   Quando T1 termina, o `Atuador_Desvio` é ativado (Q0.1) e o `Timer_Duracao_Desvio` (T2) é iniciado (Network 6).
    *   Quando T2 termina, o `Atuador_Desvio` é desativado, e as flags `Peca_Presente` e `Peca_Alta` são resetadas, finalizando o ciclo para aquela peça (Network 7).
    *   Se a peça detectada não for alta (`Peca_Alta` permanece FALSE), a Network 8 detecta quando a peça sai do `Sensor_Entrada` (borda de descida) e reseta apenas a flag `Peca_Presente`, permitindo que a peça baixa siga reto sem acionar o desvio.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 7 - Sorting by Height (Basic)".
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   **Entradas:**
        *   Mapeie `Entry Sensor` para `Sensor_Entrada` (I0.0).
        *   Mapeie `Low Sensor` para `Sensor_Baixo` (I0.1 - *não usado diretamente na lógica acima, mas bom mapear*).
        *   Mapeie `High Sensor` para `Sensor_Alto` (I0.2).
    *   **Saídas:**
        *   Mapeie `Entry Conveyor` para `Motor_Esteira_Entrada` (Q0.0).
        *   Mapeie o atuador de desvio (ex: `Pusher - Actuate` ou `Belt Conveyor Diverter - Move Left/Right`) para `Atuador_Desvio` (Q0.1).
        *   Mapeie a esteira de saída reta (geralmente a continuação da principal) para `Motor_Esteira_Baixa` (Q0.2).
        *   Mapeie a esteira de saída desviada para `Motor_Esteira_Alta` (Q0.3).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore as entradas (I0.0, I0.2), saídas (Q0.0, Q0.1, Q0.2, Q0.3), flags (M0.0, M0.1) e timers (T1, T2).
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   O `Emitter` começará a gerar caixas altas e baixas aleatoriamente.
    *   **Peça Baixa:** Observe uma peça baixa chegando. `Sensor_Entrada` (I0.0) ativa, `Peca_Presente` (M0.0) ativa. `Sensor_Alto` (I0.2) permanece FALSE, `Peca_Alta` (M0.1) permanece FALSE. A peça passa direto. Quando sai do `Sensor_Entrada`, a borda de descida (Network 8) reseta `Peca_Presente`. O desviador (Q0.1) não deve ser ativado.
    *   **Peça Alta:** Observe uma peça alta chegando. `Sensor_Entrada` (I0.0) ativa, `Peca_Presente` (M0.0) ativa. `Sensor_Alto` (I0.2) ativa, `Peca_Alta` (M0.1) ativa. O Timer T1 começa a contar. Após o tempo de T1, Q0.1 (`Atuador_Desvio`) ativa por 0.5s (tempo de T2). A peça deve ser desviada para a esteira lateral. Após T2, Q0.1 desativa e as flags M0.0 e M0.1 são resetadas.
    *   Verifique se as peças seguem para as esteiras corretas (baixas reto, altas desviadas).

**Troubleshooting Comum:**
*   **Todas as peças são desviadas ou nenhuma é:** Verifique a lógica de identificação de altura (Network 4) e o mapeamento de `Sensor_Alto` (I0.2). Verifique se a flag `Peca_Alta` está sendo ativada/resetada corretamente.
*   **Desvio ocorre no momento errado (muito cedo/tarde):** Ajuste o tempo (PT) do `Timer_Atraso_Desvio` (T1) na Network 5.
*   **Desvio não é eficaz (peça não é totalmente desviada ou o atuador fica ativo por muito tempo/pouco tempo):** Ajuste o tempo (PT) do `Timer_Duracao_Desvio` (T2) na Network 6 e 7. Verifique se o atuador físico no Factory IO é adequado (talvez precise de um pulso mais longo ou um tipo diferente de desviador).
*   **Sistema para após uma peça:** Verifique se as flags (`Peca_Presente`, `Peca_Alta`) estão sendo resetadas corretamente no final de cada ciclo (Networks 7 e 8).
*   **Problemas de Conexão/Mapeamento:** Siga os passos de verificação dos exercícios anteriores.

Este exercício combina detecção de sensores, lógica condicional, flags de estado e temporização para realizar uma tarefa de classificação comum na indústria.
