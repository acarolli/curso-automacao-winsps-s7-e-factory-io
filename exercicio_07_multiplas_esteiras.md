# Exercício Prático 7: Linha de Produção com Múltiplas Esteiras

**Objetivo:** Controlar a transferência segura e sem colisões de itens entre três esteiras transportadoras conectadas em sequência, implementando uma lógica de intertravamento (interlock) baseada em sensores.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 3 - From A to B with Interlock"**. Esta cena é adequada, pois geralmente apresenta duas ou mais esteiras em sequência com sensores no final de cada uma, projetada para testar lógicas de intertravamento. Alternativamente, crie uma cena com três esteiras curtas (A, B, C) alinhadas, cada uma com seu motor e um sensor difuso (`Diffuse Sensor`) posicionado no final.
*   Adicione um `Emitter` no início da Esteira A e um `Remover` no final da Esteira C.

**Lógica Ladder (WinSPS-S7):**

Implementaremos uma lógica onde uma esteira só pode operar se a próxima esteira estiver funcionando ou se o sensor no final da próxima esteira estiver livre (sem peça). Além disso, cada esteira para se seu próprio sensor final detectar uma peça.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_A`: I0.0 (BOOL) - Sensor no final da Esteira A.
    *   `Sensor_B`: I0.1 (BOOL) - Sensor no final da Esteira B.
    *   `Sensor_C`: I0.2 (BOOL) - Sensor no final da Esteira C.
    *   `Motor_A`: Q0.0 (BOOL) - Saída para o motor da Esteira A.
    *   `Motor_B`: Q0.1 (BOOL) - Saída para o motor da Esteira B.
    *   `Motor_C`: Q0.2 (BOOL) - Saída para o motor da Esteira C.
    *   `Sistema_Ligado`: M0.0 (BOOL) - (Opcional) Botão/Flag geral para ligar/desligar o sistema.

2.  **Programa Ladder (OB1):** (Lógica de intertravamento)
    ```ladder
    NETWORK 1: Controle Motor Esteira C
    TITLE=Controle Motor C
    // Descrição: Liga o Motor C se o sistema estiver ligado e não houver
    // peça detectada no seu próprio sensor final (Sensor_C).
    // (Assumindo que a saída da Esteira C está sempre livre ou controlada externamente)
    
    ----[ ]----[/]----( )----
      Sistema_Ligado Sensor_C     Motor_C
      (M0.0)         (I0.2)      (Q0.2)
      // Nota: Sistema_Ligado (M0.0) pode ser forçado para TRUE para teste simples.
      
    NETWORK 2: Controle Motor Esteira B
    TITLE=Controle Motor B
    // Descrição: Liga o Motor B se o sistema estiver ligado, não houver peça
    // em seu próprio sensor (Sensor_B), E (a Esteira C estiver funcionando OU
    // o Sensor C estiver livre).
    
          +----[ ]----+ 
          |  Motor_C  | 
          |  (Q0.2)   | 
    ------|           |----[/]----( )----
          |   [/]     |  Sensor_B   Motor_B
          +--Sensor_C-+  (I0.1)     (Q0.1)
             (I0.2)
    // Pré-condição: Sistema Ligado
    ----[ ]---------------------- // (Adicionar em série antes do paralelo acima)
      Sistema_Ligado
      (M0.0)
      
    NETWORK 3: Controle Motor Esteira A
    TITLE=Controle Motor A
    // Descrição: Liga o Motor A se o sistema estiver ligado, não houver peça
    // em seu próprio sensor (Sensor_A), E (a Esteira B estiver funcionando OU
    // o Sensor B estiver livre).
    
          +----[ ]----+ 
          |  Motor_B  | 
          |  (Q0.1)   | 
    ------|           |----[/]----( )----
          |   [/]     |  Sensor_A   Motor_A
          +--Sensor_B-+  (I0.0)     (Q0.0)
             (I0.1)
    // Pré-condição: Sistema Ligado
    ----[ ]---------------------- // (Adicionar em série antes do paralelo acima)
      Sistema_Ligado
      (M0.0)
    ```

3.  **Explicação da Lógica:**
    *   A lógica opera de trás para frente: a condição de funcionamento de uma esteira depende do estado da esteira seguinte.
    *   **Esteira C (Network 1):** É a mais simples. Liga se o sistema estiver habilitado (`Sistema_Ligado`) e se não houver uma peça bloqueando sua saída (`Sensor_C` = FALSE). Assume-se que o que acontece depois da Esteira C não causa bloqueio aqui.
    *   **Esteira B (Network 2):** Liga se o sistema estiver habilitado, se não houver peça em seu próprio final (`Sensor_B` = FALSE), E se houver espaço na Esteira C. A condição de espaço na Esteira C é verificada de duas formas (lógica OR): ou a `Motor_C` já está funcionando (indicando que está escoando peças) OU o `Sensor_C` está livre (indicando que há espaço físico mesmo que C esteja parada).
    *   **Esteira A (Network 3):** Segue a mesma lógica da Esteira B, mas dependendo do estado da Esteira B (`Motor_B` ou `Sensor_B`).
    *   O uso do contato NF do próprio sensor da esteira (`[/] Sensor_X`) garante que a esteira pare imediatamente se uma peça chegar ao seu final e não puder avançar, evitando que ela empurre a peça seguinte.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 3" ou a que você criou.
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   **Entradas:**
        *   Mapeie o sensor no final da Esteira A para `Sensor_A` (I0.0).
        *   Mapeie o sensor no final da Esteira B para `Sensor_B` (I0.1).
        *   Mapeie o sensor no final da Esteira C para `Sensor_C` (I0.2).
    *   **Saídas:**
        *   Mapeie o motor da Esteira A para `Motor_A` (Q0.0).
        *   Mapeie o motor da Esteira B para `Motor_B` (Q0.1).
        *   Mapeie o motor da Esteira C para `Motor_C` (Q0.2).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Force `Sistema_Ligado` (M0.0) para TRUE (ou implemente um botão de partida/parada para ele).
    *   Monitore as entradas (I0.0-I0.2) e saídas (Q0.0-Q0.2).
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   **Operação Normal:** O `Emitter` na Esteira A começará a gerar itens. Observe se os itens são transferidos suavemente da Esteira A para a B e da B para a C, e removidos no final.
    *   **Teste de Intertravamento (Bloqueio em C):** Deixe os itens se acumularem no final da Esteira C até que `Sensor_C` (I0.2) seja ativado. Observe:
        *   `Motor_C` (Q0.2) deve desligar (Network 1).
        *   Itens começarão a acumular na Esteira B. Quando um item atingir `Sensor_B` (I0.1), `Motor_B` (Q0.1) deve desligar (Network 2).
        *   Itens começarão a acumular na Esteira A. Quando um item atingir `Sensor_A` (I0.0), `Motor_A` (Q0.0) deve desligar (Network 3).
    *   **Teste de Liberação:** Remova manualmente o bloqueio no final da Esteira C (ou deixe o `Remover` fazer seu trabalho se o bloqueio foi temporário). Observe:
        *   `Sensor_C` fica livre, `Motor_C` religa.
        *   A peça em `Sensor_B` avança para C. `Sensor_B` fica livre, `Motor_B` religa.
        *   A peça em `Sensor_A` avança para B. `Sensor_A` fica livre, `Motor_A` religa.
        *   O sistema deve retornar à operação normal.

**Troubleshooting Comum:**
*   **Esteiras não ligam:** Verifique se `Sistema_Ligado` (M0.0) está TRUE. Verifique os mapeamentos das saídas Q0.0-Q0.2. Verifique se nenhum sensor está bloqueado no início.
*   **Intertravamento não funciona (esteira não para quando deveria):** Verifique a lógica dos contatos NF dos sensores (I0.0, I0.1, I0.2) nas respectivas redes. Verifique a lógica OR (`Motor_X+1` OU `Sensor_X+1` livre) nas Networks 2 e 3.
*   **Esteiras não religam após bloqueio:** Verifique se os sensores estão realmente ficando livres (FALSE) no WinSPS-S7 quando as peças se movem. Verifique se a lógica condicional para religar está correta.
*   **Problemas de Conexão/Mapeamento:** Siga os passos de verificação dos exercícios anteriores.

Este exercício é crucial para entender como implementar lógicas de intertravamento que garantem a operação segura e eficiente de sistemas com múltiplos transportadores ou processos sequenciais interdependentes.
