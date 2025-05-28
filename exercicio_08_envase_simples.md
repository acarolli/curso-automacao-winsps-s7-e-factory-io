# Exercício Prático 8: Sistema de Envase Simples

**Objetivo:** Controlar um processo simples de envase onde recipientes são transportados por uma esteira, param em uma posição específica para serem enchidos por um tempo determinado, e depois continuam na esteira.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 5 - Filling Tank"**. Esta cena é quase perfeita, mas o tanque enche com um líquido. Para simular o envase de recipientes discretos, podemos adaptar:
    *   Mantenha a esteira (`Belt Conveyor`).
    *   Utilize o sensor difuso (`Diffuse Sensor`) posicionado sobre a esteira como nosso `Sensor_Posicao` para detectar a chegada do recipiente.
    *   Utilize a válvula (`Valve`) como nossa `Valvula_Enchimento` (mesmo que ela libere líquido na cena original, vamos usá-la para representar o bico de envase).
    *   Adicione um `Emitter` no início da esteira para gerar os recipientes (ex: caixas ou paletes azuis).
    *   Adicione um `Remover` no final da esteira.

**Lógica Ladder (WinSPS-S7):**

A lógica envolverá detectar o recipiente, parar a esteira, abrir a válvula por um tempo, fechar a válvula e reiniciar a esteira.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_Posicao`: I0.0 (BOOL) - Sensor que detecta o recipiente na posição de envase.
    *   `Motor_Esteira`: Q0.0 (BOOL) - Saída para o motor da esteira.
    *   `Valvula_Enchimento`: Q0.1 (BOOL) - Saída para a válvula/bico de envase.
    *   `Timer_Enchimento`: T1 (TIMER) - Temporizador para a duração do envase.
    *   `Flag_Enchendo`: M0.0 (BOOL) - Flag para indicar que o processo de envase está ativo.
    *   `Borda_Sensor_Pos`: M0.1 (BOOL) - Flag para a borda de subida do sensor.

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Detecção de Borda de Subida do Sensor de Posição
    TITLE=Detectar Chegada Recipiente
    // Descrição: Detecta a borda de subida do Sensor_Posicao para
    // iniciar o ciclo de envase, somente se não estiver já enchendo.
    
    ----|P|----[/]----(S)----
      Sensor_Posicao Flag_Enchendo  Flag_Enchendo
      (I0.0)         (M0.0)       (M0.0)
    // Guarda a borda em M0.1 para uso se necessário, ou usa |P| direto
    ----|P|------------( )----
      Sensor_Posicao   Borda_Sensor_Pos
      (I0.0)           (M0.1)
      
    NETWORK 2: Controle do Motor da Esteira
    TITLE=Ligar/Desligar Esteira
    // Descrição: Mantém a esteira ligada, a menos que o ciclo de
    // envase (Flag_Enchendo) esteja ativo.
    
    ----[/]------------( )----
      Flag_Enchendo    Motor_Esteira
      (M0.0)           (Q0.0)
      
    NETWORK 3: Controle da Válvula de Enchimento e Timer
    TITLE=Abrir Válvula e Temporizar
    // Descrição: Abre a Valvula_Enchimento e inicia o Timer_Enchimento
    // quando a flag Flag_Enchendo está ativa.
    
    ----[ ]------------( )----
      Flag_Enchendo    Valvula_Enchimento
      (M0.0)           (Q0.1)
    ----[ ]----[TON T1, PT:3s]-- // Tempo de envase: 3 segundos
      Flag_Enchendo
      (M0.0)
      
    NETWORK 4: Finalizar Ciclo de Envase
    TITLE=Fechar Válvula e Resetar Flag
    // Descrição: Quando o Timer_Enchimento (T1) termina, fecha a
    // Valvula_Enchimento (implicitamente pela Network 3 quando M0.0 é resetado)
    // e reseta a flag Flag_Enchendo, permitindo que a esteira volte a andar.
    
    ----[ ]------------(R)----
      T1.Q           Flag_Enchendo
                       (M0.0)
    ```

3.  **Explicação da Lógica:**
    *   A Network 1 detecta a chegada de um recipiente (`Sensor_Posicao` indo para TRUE) usando a borda de subida (`|P|`). Se um ciclo de envase não estiver ativo (`Flag_Enchendo` = FALSE), ela ativa a flag `Flag_Enchendo` (Set M0.0).
    *   A Network 2 controla o `Motor_Esteira`. Ele funciona (Q0.0 = TRUE) somente quando `Flag_Enchendo` é FALSE. Assim que `Flag_Enchendo` se torna TRUE, a esteira para.
    *   A Network 3 controla a `Valvula_Enchimento` e o `Timer_Enchimento`. Enquanto `Flag_Enchendo` for TRUE, a válvula estará aberta (Q0.1 = TRUE) e o temporizador T1 contará por 3 segundos.
    *   A Network 4 detecta o fim do tempo de envase (T1.Q = TRUE). Quando isso ocorre, ela reseta a `Flag_Enchendo` (Reset M0.0).
    *   Ao resetar M0.0, a condição na Network 3 deixa de ser verdadeira, fazendo Q0.1 (`Valvula_Enchimento`) ir para FALSE (válvula fecha). Simultaneamente, a condição na Network 2 volta a ser verdadeira (contato NF de M0.0 fecha), fazendo Q0.0 (`Motor_Esteira`) ir para TRUE (esteira volta a andar).

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 5 - Filling Tank" (ou a cena modificada com Emitter/Remover).
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   **Entrada:** Mapeie o `Diffuse Sensor` (que está sobre a esteira) para `Sensor_Posicao` (I0.0).
    *   **Saídas:**
        *   Mapeie o `Belt Conveyor` para `Motor_Esteira` (Q0.0).
        *   Mapeie a `Valve` para `Valvula_Enchimento` (Q0.1).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore I0.0, Q0.0, Q0.1, M0.0 e T1.
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   O `Emitter` (se adicionado) começará a gerar recipientes.
    *   **Chegada:** Observe um recipiente se aproximando do `Diffuse Sensor`. A esteira (`Motor_Esteira` Q0.0) deve estar ligada.
    *   **Parada e Envase:** Quando o recipiente ativa o `Sensor_Posicao` (I0.0), observe no WinSPS-S7 que `Flag_Enchendo` (M0.0) vai para TRUE. No Factory IO, a esteira (Q0.0) deve parar, e a `Valve` (Q0.1) deve abrir (simulando o início do envase). O Timer T1 começa a contar.
    *   **Fim do Envase:** Após 3 segundos (tempo de T1), observe no WinSPS-S7 que T1.Q vai para TRUE, e `Flag_Enchendo` (M0.0) é resetada para FALSE. No Factory IO, a `Valve` (Q0.1) deve fechar, e a esteira (Q0.0) deve voltar a funcionar, levando o recipiente embora.
    *   **Próximo Ciclo:** O sistema está pronto para o próximo recipiente.

**Troubleshooting Comum:**
*   **Esteira não para quando o sensor detecta:** Verifique a lógica da Network 1 (borda, flag) e Network 2 (controle da esteira). Verifique o mapeamento de I0.0.
*   **Válvula não abre ou não fecha:** Verifique a lógica da Network 3 (controle da válvula) e Network 4 (reset da flag). Verifique o mapeamento de Q0.1.
*   **Tempo de envase incorreto:** Verifique o valor de PT no bloco TON T1 (Network 3).
*   **Ciclo não reinicia (esteira não volta a andar):** Verifique se `Flag_Enchendo` (M0.0) está sendo corretamente resetada pela Network 4 e se a Network 2 está respondendo corretamente ao estado de M0.0.
*   **Problemas de Conexão/Mapeamento:** Siga os passos de verificação dos exercícios anteriores.

Este exercício combina detecção de posição, controle de atuadores (motor e válvula) e temporização para simular um processo de envase sequencial, comum em linhas de produção.
