# Exercício Prático 4: Contagem de Caixas

**Objetivo:** Utilizar um contador para contar um número específico de caixas que passam por um sensor em uma esteira transportadora e parar a esteira automaticamente quando a contagem pré-definida for atingida.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 1 - From A to B"**. 
*   **Modificações Necessárias:**
    1.  Adicione um **`Emitter`** (Emissor) da paleta de peças no início da esteira para gerar caixas automaticamente.
    2.  Adicione um **`Remover`** (Removedor) no final da esteira para destruir as caixas que chegam lá.
    3.  Utilizaremos o sensor já existente na cena, próximo ao início (`Diffuse Sensor`, que chamaremos de Sensor A), para realizar a contagem.

**Lógica Ladder (WinSPS-S7):**

Implementaremos a lógica usando um contador crescente (Up Counter - CTU) e a detecção de borda de subida do sensor para garantir uma contagem precisa.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_Contagem`: I0.0 (BOOL) - Entrada conectada ao Sensor A.
    *   `Motor_Esteira`: Q0.0 (BOOL) - Saída que controla o motor da esteira.
    *   `Contador_Caixas`: C1 (COUNTER) - Contador para as caixas.
    *   `Reset_Contador`: M0.0 (BOOL) - Botão virtual para zerar a contagem.
    *   `Borda_Sensor`: M0.1 (BOOL) - Memória interna para armazenar a borda de subida do sensor.

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Detecção de Borda de Subida do Sensor de Contagem
    TITLE=Detectar Borda Sensor
    // Descrição: Detecta a transição de FALSE para TRUE do Sensor_Contagem
    // e ativa a flag Borda_Sensor por um ciclo de scan.
    // Isso garante que cada caixa seja contada apenas uma vez.
    
    ----|P|------------( )----
      Sensor_Contagem  Borda_Sensor
        (I0.0)           (M0.1)
    // Nota: |P| representa a detecção de borda positiva (Rising Edge).
    // A implementação exata pode variar no WinSPS-S7 (pode ser um bloco R_TRIG).
    
    NETWORK 2: Contagem de Caixas (CTU)
    TITLE=Contador de Caixas
    // Descrição: Incrementa o Contador_Caixas (C1) cada vez que uma
    // borda de subida do sensor é detectada (Borda_Sensor = TRUE).
    // O valor pré-definido (PV) é 5. A saída C1.Q será TRUE quando CV >= PV.
    
    ----[ ]----[CTU C1, PV:5]--
      Borda_Sensor
      (M0.1)
    // CU (Count Up) é ativado por Borda_Sensor.
    
    NETWORK 3: Reset do Contador
    TITLE=Resetar Contagem
    // Descrição: Zera o valor acumulado (CV) do Contador_Caixas (C1)
    // quando o botão virtual Reset_Contador é pressionado.
    
    ----[ ]----(R C1)--
      Reset_Contador
      (M0.0)
    // R (Reset) é ativado por Reset_Contador.
    
    NETWORK 4: Controle do Motor da Esteira
    TITLE=Ligar/Desligar Esteira
    // Descrição: Mantém o Motor_Esteira ligado enquanto a contagem
    // não atingiu o valor pré-definido (C1.Q é FALSE).
    // Quando C1.Q se torna TRUE (contagem >= 5), o motor desliga.
    
    ----[/]------------( )----
      C1.Q           Motor_Esteira
                       (Q0.0)
    // Usa um contato NF associado à saída Q do contador C1.
    ```

3.  **Explicação da Lógica:**
    *   A Network 1 detecta a borda de subida do `Sensor_Contagem` (I0.0), garantindo que cada caixa gere apenas um pulso de contagem (`Borda_Sensor`, M0.1).
    *   A Network 2 usa esse pulso (`Borda_Sensor`) para incrementar o `Contador_Caixas` (C1). O valor pré-definido (PV) é 5.
    *   A Network 3 permite zerar o contador a qualquer momento usando o botão virtual `Reset_Contador` (M0.0).
    *   A Network 4 controla o `Motor_Esteira` (Q0.0). Ela usa um contato NF da saída do contador (`C1.Q`). Enquanto a contagem for menor que 5, `C1.Q` é FALSE, o contato NF está fechado, e o motor funciona. Quando a contagem atinge 5, `C1.Q` se torna TRUE, o contato NF abre, e o motor para.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 1 - From A to B". Adicione o `Emitter` e o `Remover` conforme descrito.
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   **Entrada:** Encontre o sensor `Diffuse Sensor` (Sensor A) na lista de Inputs. Mapeie-o para a entrada `Sensor_Contagem` (I0.0 ou endereço Modbus Discrete Input correspondente).
    *   **Saída:** Encontre o atuador `Belt Conveyor 6m` na lista de Outputs. Mapeie-o para a saída `Motor_Esteira` (Q0.0 ou endereço Modbus Coil correspondente).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore I0.0, M0.1, Q0.0, C1.CV (Valor Atual) e C1.Q (Saída do Contador).
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   **Reset Inicial:** No WinSPS-S7, force `Reset_Contador` (M0.0) para TRUE momentaneamente e depois para FALSE. Verifique se C1.CV zera e C1.Q fica FALSE. A esteira no Factory IO deve começar a se mover (Network 4).
    *   **Contagem:** O `Emitter` começará a gerar caixas. Observe as caixas passando pelo `Diffuse Sensor`. A cada passagem, você deve ver `Sensor_Contagem` (I0.0) pulsar, `Borda_Sensor` (M0.1) piscar brevemente, e `C1.CV` incrementar no WinSPS-S7.
    *   **Parada Automática:** Acompanhe `C1.CV`. Quando ele atingir o valor 5, a saída `C1.Q` deve ir para TRUE. No mesmo instante, o contato NF `C1.Q` na Network 4 deve abrir, e `Motor_Esteira` (Q0.0) deve ir para FALSE. A esteira no Factory IO deve parar.
    *   **Reinício:** Force `Reset_Contador` (M0.0) novamente. O contador deve zerar, `C1.Q` deve ir para FALSE, e a esteira deve voltar a funcionar, iniciando um novo ciclo de contagem.

**Troubleshooting Comum:**
*   **Contagem incorreta (conta mais de uma vez por caixa ou não conta):** Verifique a lógica de detecção de borda (Network 1). Certifique-se de que o sensor está posicionado corretamente no Factory IO para detectar cada caixa de forma clara. Verifique o mapeamento de I0.0.
*   **Esteira não para na contagem correta:** Verifique o valor de PV no bloco CTU (Network 2). Verifique a lógica da Network 4 (contato NF de C1.Q controlando Q0.0). Monitore C1.Q e Q0.0 no WinSPS-S7.
*   **Reset não funciona:** Verifique a lógica da Network 3 e se M0.0 está sendo forçado corretamente.
*   **Problemas de Conexão/Mapeamento:** Siga os passos de verificação dos exercícios anteriores.

Este exercício introduz o uso de contadores e a importância da detecção de bordas para eventos discretos, habilidades essenciais para muitas tarefas de automação como controle de lotes, sequenciamento baseado em contagem, etc.
