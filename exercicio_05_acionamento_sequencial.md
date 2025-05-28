# Exercício Prático 5: Acionamento Sequencial Básico

**Objetivo:** Implementar uma sequência simples de movimentos para dois atuadores (simulados como cilindros pneumáticos), onde um avança e recua, seguido pelo outro avançando e recuando, utilizando flags de estado e temporizadores.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Crie uma **nova cena vazia** no Factory IO.
*   Adicione dois componentes **`Pneumatic Cylinder`** (Cilindro Pneumático) da paleta de peças. Posicione-os lado a lado ou como preferir.
*   *Nota:* Por padrão, esses cilindros no Factory IO são de dupla ação e requerem sinais separados para avançar e recuar (ex: Solenoid + e Solenoid -). Para simplificar este exercício inicial, podemos controlar apenas o avanço (`Solenoid +`) e assumir que ele recua automaticamente quando o sinal de avanço é removido (configurando o cilindro no Factory IO para "Spring Return" se disponível, ou apenas controlando a saída de avanço). Se quisermos controlar o recuo explicitamente, precisaríamos de saídas adicionais no CLP.
*   Vamos assumir que controlaremos apenas o avanço: Cilindro A (`Solenoid A+`) e Cilindro B (`Solenoid B+`).

**Lógica Ladder (WinSPS-S7):**

Utilizaremos memórias internas (flags) para marcar cada etapa da sequência e temporizadores para definir a duração de cada movimento ou pausa.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Iniciar_Ciclo`: M0.0 (BOOL) - Botão virtual para iniciar a sequência.
    *   `Etapa1_Avanca_A`: M1.0 (BOOL) - Flag: Cilindro A avançando.
    *   `Etapa2_Pausa_A`: M1.1 (BOOL) - Flag: Cilindro A avançado (pausa).
    *   `Etapa3_Recua_A`: M1.2 (BOOL) - Flag: Cilindro A recuando.
    *   `Etapa4_Avanca_B`: M1.3 (BOOL) - Flag: Cilindro B avançando.
    *   `Etapa5_Pausa_B`: M1.4 (BOOL) - Flag: Cilindro B avançado (pausa).
    *   `Etapa6_Recua_B`: M1.5 (BOOL) - Flag: Cilindro B recuando.
    *   `Ciclo_Ativo`: M2.0 (BOOL) - Flag para indicar que a sequência está em andamento.
    *   `Solenoid_A_Mais`: Q0.0 (BOOL) - Saída para avançar Cilindro A.
    *   `Solenoid_B_Mais`: Q0.1 (BOOL) - Saída para avançar Cilindro B.
    *   `Timer_Avanco_A`: T1 (TIMER) - Duração do avanço de A.
    *   `Timer_Pausa_A`: T2 (TIMER) - Duração da pausa de A.
    *   `Timer_Recuo_A`: T3 (TIMER) - Duração do recuo de A (tempo para desativar solenoide).
    *   `Timer_Avanco_B`: T4 (TIMER) - Duração do avanço de B.
    *   `Timer_Pausa_B`: T5 (TIMER) - Duração da pausa de B.
    *   `Timer_Recuo_B`: T6 (TIMER) - Duração do recuo de B.

2.  **Programa Ladder (OB1):** (Lógica sequencial usando flags e timers)
    ```ladder
    NETWORK 1: Iniciar Ciclo
    TITLE=Iniciar Sequência
    // Descrição: Inicia a sequência com o botão Iniciar_Ciclo,
    // somente se o ciclo não estiver ativo.
    // Ativa a flag Ciclo_Ativo e a primeira etapa.
    
    ----[ ]----[/]----(S)----
      Iniciar_Ciclo Ciclo_Ativo  Ciclo_Ativo
      (M0.0)       (M2.0)
    ----[ ]----[/]----(S)----
      Iniciar_Ciclo Ciclo_Ativo  Etapa1_Avanca_A
                               (M1.0)
    
    NETWORK 2: Etapa 1 - Avançar Cilindro A
    TITLE=Etapa 1: Avança A
    // Descrição: Ativa o solenoide de avanço de A e inicia o timer T1.
    
    ----[ ]------------( )----
      Etapa1_Avanca_A  Solenoid_A_Mais
      (M1.0)           (Q0.0)
    ----[ ]----[TON T1, PT:2s]--
      Etapa1_Avanca_A
    
    NETWORK 3: Transição para Etapa 2 (Pausa A)
    TITLE=Transição 1 -> 2
    // Descrição: Quando T1 termina, ativa a Etapa 2 e desativa a Etapa 1.
    
    ----[ ]----(S)----
      T1.Q     Etapa2_Pausa_A
               (M1.1)
    ----[ ]----(R)----
      T1.Q     Etapa1_Avanca_A
               (M1.0)
    
    NETWORK 4: Etapa 2 - Pausa Cilindro A Avançado
    TITLE=Etapa 2: Pausa A
    // Descrição: Mantém o solenoide A ativo (já está) e inicia o timer T2.
    
    ----[ ]----[TON T2, PT:1s]--
      Etapa2_Pausa_A
      (M1.1)
    
    NETWORK 5: Transição para Etapa 3 (Recua A)
    TITLE=Transição 2 -> 3
    // Descrição: Quando T2 termina, ativa a Etapa 3 e desativa a Etapa 2.
    
    ----[ ]----(S)----
      T2.Q     Etapa3_Recua_A
               (M1.2)
    ----[ ]----(R)----
      T2.Q     Etapa2_Pausa_A
               (M1.1)
    
    NETWORK 6: Etapa 3 - Recuar Cilindro A
    TITLE=Etapa 3: Recua A
    // Descrição: Desativa o solenoide de avanço de A e inicia o timer T3.
    
    ----[ ]------------(R)----
      Etapa3_Recua_A   Solenoid_A_Mais
      (M1.2)           (Q0.0)
    ----[ ]----[TON T3, PT:2s]--
      Etapa3_Recua_A
    
    NETWORK 7: Transição para Etapa 4 (Avança B)
    TITLE=Transição 3 -> 4
    // Descrição: Quando T3 termina, ativa a Etapa 4 e desativa a Etapa 3.
    
    ----[ ]----(S)----
      T3.Q     Etapa4_Avanca_B
               (M1.3)
    ----[ ]----(R)----
      T3.Q     Etapa3_Recua_A
               (M1.2)
    
    // --- Lógica similar para as Etapas 4, 5, 6 (Avança B, Pausa B, Recua B) ---
    
    NETWORK 8: Etapa 4 - Avançar Cilindro B
    TITLE=Etapa 4: Avança B
    ----[ ]------------( )----
      Etapa4_Avanca_B  Solenoid_B_Mais
      (M1.3)           (Q0.1)
    ----[ ]----[TON T4, PT:2s]--
      Etapa4_Avanca_B
    
    NETWORK 9: Transição para Etapa 5 (Pausa B)
    TITLE=Transição 4 -> 5
    ----[ ]----(S)----
      T4.Q     Etapa5_Pausa_B
               (M1.4)
    ----[ ]----(R)----
      T4.Q     Etapa4_Avanca_B
               (M1.3)
    
    NETWORK 10: Etapa 5 - Pausa Cilindro B Avançado
    TITLE=Etapa 5: Pausa B
    ----[ ]----[TON T5, PT:1s]--
      Etapa5_Pausa_B
      (M1.4)
    
    NETWORK 11: Transição para Etapa 6 (Recua B)
    TITLE=Transição 5 -> 6
    ----[ ]----(S)----
      T5.Q     Etapa6_Recua_B
               (M1.5)
    ----[ ]----(R)----
      T5.Q     Etapa5_Pausa_B
               (M1.4)
    
    NETWORK 12: Etapa 6 - Recuar Cilindro B
    TITLE=Etapa 6: Recua B
    ----[ ]------------(R)----
      Etapa6_Recua_B   Solenoid_B_Mais
      (M1.5)           (Q0.1)
    ----[ ]----[TON T6, PT:2s]--
      Etapa6_Recua_B
    
    NETWORK 13: Fim do Ciclo
    TITLE=Finalizar Sequência
    // Descrição: Quando T6 termina, desativa a Etapa 6 e a flag Ciclo_Ativo,
    // permitindo que um novo ciclo seja iniciado.
    
    ----[ ]----(R)----
      T6.Q     Etapa6_Recua_B
               (M1.5)
    ----[ ]----(R)----
      T6.Q     Ciclo_Ativo
               (M2.0)
    ```

3.  **Explicação da Lógica:**
    *   A lógica implementa uma máquina de estados simples usando flags (M1.0 a M1.5) para representar cada etapa.
    *   O `Iniciar_Ciclo` (M0.0) só pode iniciar a sequência se `Ciclo_Ativo` (M2.0) for FALSE.
    *   Cada etapa ativa a saída correspondente (se houver) e um temporizador.
    *   A saída Q do temporizador da etapa atual dispara a transição para a próxima etapa, ativando a flag da próxima etapa (Set) e desativando a flag da etapa atual (Reset).
    *   Na Etapa 3 (Recua A) e Etapa 6 (Recua B), a bobina de avanço correspondente é desativada (Reset). Assumimos que o cilindro recua por mola ou gravidade (ou controlariamos uma saída de recuo aqui).
    *   Ao final da Etapa 6, o timer T6 reseta a flag da Etapa 6 e também a flag `Ciclo_Ativo`, permitindo que um novo ciclo seja iniciado com `Iniciar_Ciclo`.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena que você criou com os dois `Pneumatic Cylinder`.
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   Encontre os atuadores dos cilindros na lista de Outputs. Eles podem ter nomes como `Pneumatic Cylinder - Solenoid +`.
    *   Mapeie o solenoide de avanço do primeiro cilindro (Cilindro A) para a saída `Solenoid_A_Mais` (Q0.0 ou endereço Modbus Coil correspondente).
    *   Mapeie o solenoide de avanço do segundo cilindro (Cilindro B) para a saída `Solenoid_B_Mais` (Q0.1 ou endereço Modbus Coil correspondente).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore as flags de etapa (M1.0-M1.5), Ciclo_Ativo (M2.0), as saídas (Q0.0, Q0.1) e os timers.
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   **Iniciar:** No WinSPS-S7, force `Iniciar_Ciclo` (M0.0) para TRUE momentaneamente e depois para FALSE.
    *   **Observar Sequência:** Observe os cilindros no Factory IO e as flags/saídas no WinSPS-S7. A sequência deve ser:
        1.  Cilindro A avança (Q0.0 ON, M1.0 ON).
        2.  Cilindro A pausa avançado (Q0.0 ON, M1.1 ON).
        3.  Cilindro A recua (Q0.0 OFF, M1.2 ON).
        4.  Cilindro B avança (Q0.1 ON, M1.3 ON).
        5.  Cilindro B pausa avançado (Q0.1 ON, M1.4 ON).
        6.  Cilindro B recua (Q0.1 OFF, M1.5 ON).
        7.  Ciclo termina (M2.0 OFF).
    *   **Repetir:** Force `Iniciar_Ciclo` novamente para repetir a sequência.

**Troubleshooting Comum:**
*   **Sequência não inicia:** Verifique se `Ciclo_Ativo` (M2.0) está FALSE antes de forçar M0.0. Verifique a lógica da Network 1.
*   **Sequência para em uma etapa:** Verifique o timer da etapa anterior (ele está terminando?). Verifique a lógica de transição (Set da próxima etapa, Reset da etapa atual). Monitore as flags de etapa.
*   **Cilindro não avança/recua:** Verifique o mapeamento das saídas Q0.0/Q0.1 para os solenoides corretos no Factory IO. Verifique se a bobina correspondente (ou o Reset dela) está sendo ativada na etapa correta no WinSPS-S7.
*   **Tempos incorretos:** Verifique os valores de PT nos blocos TON.
*   **Problemas de Conexão:** Siga os passos de verificação dos exercícios anteriores.

Este exercício demonstra como construir lógicas sequenciais usando flags de estado e temporizadores, uma técnica essencial para controlar máquinas e processos que operam em passos definidos.
