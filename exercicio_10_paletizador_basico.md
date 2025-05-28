# Exercício Prático 10: Paletizador Básico (Movimentos)

**Objetivo:** Simular e controlar a sequência de movimentos básicos de um robô paletizador simples para pegar caixas de uma esteira e empilhá-las (de forma rudimentar) em um local designado (palete).

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 9 - Palletizer (Basic)"**. Esta cena é projetada para este tipo de simulação e geralmente inclui:
    *   Uma esteira de entrada (`Conveyor`) para trazer as caixas.
    *   Um sensor (`Diffuse Sensor`) na posição de pega para detectar a caixa.
    *   Um atuador para o eixo Z (`Elevator - Motor Up/Down` ou similar) para movimento vertical.
    *   Um atuador para o eixo X (`Gantry X - Motor Left/Right` ou similar) para movimento horizontal.
    *   Um atuador de garra (`Clamp` ou `Grabber`).
    *   Sensores de posição para os eixos (ex: `Elevator - At Top`, `Elevator - At Bottom`, `Gantry X - At Home`, `Gantry X - At End`).
    *   Uma área de palete.

**Lógica Ladder (WinSPS-S7):**

Controlaremos a sequência de movimentos usando flags de estado, sensores de posição (se disponíveis e mapeados) ou temporizadores para simular a duração dos movimentos.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_Caixa_Pega`: I0.0 (BOOL) - Sensor na posição de pega.
    *   `Sensor_Z_Alto`: I0.1 (BOOL) - Sensor: Elevador no topo.
    *   `Sensor_Z_Baixo`: I0.2 (BOOL) - Sensor: Elevador na base.
    *   `Sensor_X_Home`: I0.3 (BOOL) - Sensor: Eixo X na posição inicial (pega).
    *   `Sensor_X_Palete`: I0.4 (BOOL) - Sensor: Eixo X na posição do palete.
    *   `Motor_Z_Sobe`: Q0.0 (BOOL) - Comando para subir o elevador.
    *   `Motor_Z_Desce`: Q0.1 (BOOL) - Comando para descer o elevador.
    *   `Motor_X_Avanca`: Q0.2 (BOOL) - Comando para mover eixo X para o palete.
    *   `Motor_X_Recua`: Q0.3 (BOOL) - Comando para mover eixo X para a posição inicial.
    *   `Atuador_Garra`: Q0.4 (BOOL) - Comando para fechar/pegar com a garra.
    *   `Iniciar_Ciclo`: M0.0 (BOOL) - Botão/Flag para iniciar (ou usar Sensor_Caixa_Pega).
    *   `Ciclo_Paletizador_Ativo`: M1.0 (BOOL) - Flag indicando que a sequência está em execução.
    *   `Etapa_Seq`: MW10 (WORD) - Variável para armazenar o número da etapa atual da sequência.
    *   *(Opcional: Timers T1, T2... se não usar sensores de posição)*

2.  **Programa Ladder (OB1):** (Abordagem Sequencial usando Etapas e Comparações)
    ```ladder
    NETWORK 1: Iniciar Ciclo Paletizador
    TITLE=Iniciar Sequência Paletização
    // Descrição: Inicia a sequência (vai para Etapa 10) quando uma caixa é detectada
    // e o ciclo não está ativo.
    
    ----[ ]----[/]----(MOVE 10 -> MW10)----
      Sensor_Caixa_Pega Ciclo_Paletizador_Ativo
      (I0.0)            (M1.0)
    ----[ ]----[/]----(S)----
      Sensor_Caixa_Pega Ciclo_Paletizador_Ativo Ciclo_Paletizador_Ativo
                                                (M1.0)
                                                
    // --- Controle Sequencial Baseado em Etapas (MW10) ---
    
    NETWORK 2: Etapa 10 - Descer Z para Pega
    TITLE=Etapa 10: Z Desce Pega
    // Descrição: Se na Etapa 10, comanda Motor_Z_Desce.
    
    ----[== 10]--------( )----
      Etapa_Seq        Motor_Z_Desce
      (MW10)           (Q0.1)
      
    NETWORK 3: Transição 10 -> 20 (Z Chegou Baixo)
    TITLE=Transição 10->20
    // Descrição: Se na Etapa 10 e Sensor_Z_Baixo ativo, vai para Etapa 20.
    
    ----[== 10]----[ ]----(MOVE 20 -> MW10)----
      Etapa_Seq    Sensor_Z_Baixo
      (MW10)       (I0.2)
      
    NETWORK 4: Etapa 20 - Ativar Garra
    TITLE=Etapa 20: Pega Caixa
    // Descrição: Se na Etapa 20, ativa Atuador_Garra.
    // (Adicionar um pequeno timer T1 aqui para garantir a pega antes de subir)
    
    ----[== 20]--------( )----
      Etapa_Seq        Atuador_Garra
      (MW10)           (Q0.4)
    ----[== 20]----[TON T1, PT:0.5s]--
      Etapa_Seq
      
    NETWORK 5: Transição 20 -> 30 (Garra Ativa, Tempo OK)
    TITLE=Transição 20->30
    // Descrição: Se na Etapa 20 e Timer T1 terminou, vai para Etapa 30.
    
    ----[== 20]----[ ]----(MOVE 30 -> MW10)----
      Etapa_Seq    T1.Q
      (MW10)
      
    NETWORK 6: Etapa 30 - Subir Z com Caixa
    TITLE=Etapa 30: Z Sobe Pega
    // Descrição: Se na Etapa 30, comanda Motor_Z_Sobe e desliga Motor_Z_Desce.
    
    ----[== 30]--------( )----
      Etapa_Seq        Motor_Z_Sobe
      (MW10)           (Q0.0)
    ----[== 30]--------(R)----
      Etapa_Seq        Motor_Z_Desce
      (MW10)           (Q0.1)
      
    NETWORK 7: Transição 30 -> 40 (Z Chegou Alto)
    TITLE=Transição 30->40
    // Descrição: Se na Etapa 30 e Sensor_Z_Alto ativo, vai para Etapa 40.
    
    ----[== 30]----[ ]----(MOVE 40 -> MW10)----
      Etapa_Seq    Sensor_Z_Alto
      (MW10)       (I0.1)
      
    NETWORK 8: Etapa 40 - Avançar X para Palete
    TITLE=Etapa 40: X Avança Palete
    // Descrição: Se na Etapa 40, comanda Motor_X_Avanca e desliga Motor_Z_Sobe.
    
    ----[== 40]--------( )----
      Etapa_Seq        Motor_X_Avanca
      (MW10)           (Q0.2)
    ----[== 40]--------(R)----
      Etapa_Seq        Motor_Z_Sobe
      (MW10)           (Q0.0)
      
    NETWORK 9: Transição 40 -> 50 (X Chegou Palete)
    TITLE=Transição 40->50
    // Descrição: Se na Etapa 40 e Sensor_X_Palete ativo, vai para Etapa 50.
    
    ----[== 40]----[ ]----(MOVE 50 -> MW10)----
      Etapa_Seq    Sensor_X_Palete
      (MW10)       (I0.4)
      
    NETWORK 10: Etapa 50 - Descer Z sobre Palete
    TITLE=Etapa 50: Z Desce Palete
    // Descrição: Se na Etapa 50, comanda Motor_Z_Desce e desliga Motor_X_Avanca.
    
    ----[== 50]--------( )----
      Etapa_Seq        Motor_Z_Desce
      (MW10)           (Q0.1)
    ----[== 50]--------(R)----
      Etapa_Seq        Motor_X_Avanca
      (MW10)           (Q0.2)
      
    NETWORK 11: Transição 50 -> 60 (Z Chegou Baixo Palete)
    TITLE=Transição 50->60
    // Descrição: Se na Etapa 50 e Sensor_Z_Baixo ativo, vai para Etapa 60.
    
    ----[== 50]----[ ]----(MOVE 60 -> MW10)----
      Etapa_Seq    Sensor_Z_Baixo
      (MW10)       (I0.2)
      
    NETWORK 12: Etapa 60 - Desativar Garra
    TITLE=Etapa 60: Solta Caixa
    // Descrição: Se na Etapa 60, desativa Atuador_Garra.
    // (Adicionar Timer T2 para garantir soltura antes de subir)
    
    ----[== 60]--------(R)----
      Etapa_Seq        Atuador_Garra
      (MW10)           (Q0.4)
    ----[== 60]----[TON T2, PT:0.5s]--
      Etapa_Seq
      
    NETWORK 13: Transição 60 -> 70 (Garra Solta, Tempo OK)
    TITLE=Transição 60->70
    // Descrição: Se na Etapa 60 e Timer T2 terminou, vai para Etapa 70.
    
    ----[== 60]----[ ]----(MOVE 70 -> MW10)----
      Etapa_Seq    T2.Q
      (MW10)
      
    NETWORK 14: Etapa 70 - Subir Z do Palete
    TITLE=Etapa 70: Z Sobe Palete
    // Descrição: Se na Etapa 70, comanda Motor_Z_Sobe e desliga Motor_Z_Desce.
    
    ----[== 70]--------( )----
      Etapa_Seq        Motor_Z_Sobe
      (MW10)           (Q0.0)
    ----[== 70]--------(R)----
      Etapa_Seq        Motor_Z_Desce
      (MW10)           (Q0.1)
      
    NETWORK 15: Transição 70 -> 80 (Z Chegou Alto)
    TITLE=Transição 70->80
    // Descrição: Se na Etapa 70 e Sensor_Z_Alto ativo, vai para Etapa 80.
    
    ----[== 70]----[ ]----(MOVE 80 -> MW10)----
      Etapa_Seq    Sensor_Z_Alto
      (MW10)       (I0.1)
      
    NETWORK 16: Etapa 80 - Recuar X para Home
    TITLE=Etapa 80: X Recua Home
    // Descrição: Se na Etapa 80, comanda Motor_X_Recua e desliga Motor_Z_Sobe.
    
    ----[== 80]--------( )----
      Etapa_Seq        Motor_X_Recua
      (MW10)           (Q0.3)
    ----[== 80]--------(R)----
      Etapa_Seq        Motor_Z_Sobe
      (MW10)           (Q0.0)
      
    NETWORK 17: Transição 80 -> 0 (X Chegou Home - Fim Ciclo)
    TITLE=Transição 80->0 / Fim Ciclo
    // Descrição: Se na Etapa 80 e Sensor_X_Home ativo, reseta Etapa_Seq para 0
    // e desliga Ciclo_Paletizador_Ativo.
    
    ----[== 80]----[ ]----(MOVE 0 -> MW10)----
      Etapa_Seq    Sensor_X_Home
      (MW10)       (I0.3)
    ----[== 80]----[ ]----(R)----
      Etapa_Seq    Sensor_X_Home Ciclo_Paletizador_Ativo
                                 (M1.0)
    ----[== 80]----[ ]----(R)----
      Etapa_Seq    Sensor_X_Home Motor_X_Recua // Garante desligar motor X
                                 (Q0.3)
    ```

3.  **Explicação da Lógica:**
    *   A lógica usa uma variável `Etapa_Seq` (MW10) para controlar o fluxo sequencial. Cada número de etapa representa um estado ou ação.
    *   A Network 1 inicia o ciclo (define Etapa=10) quando uma caixa chega e o ciclo não está ativo.
    *   Cada etapa (Networks 2, 4, 6, 8, 10, 12, 14, 16) verifica o número da etapa atual (`[== Etapa_Seq]`) e ativa os motores ou atuadores correspondentes. Alguns motores são desligados (Reset) ao iniciar o movimento do próximo eixo.
    *   As transições entre etapas (Networks 3, 5, 7, 9, 11, 13, 15, 17) verificam a etapa atual e a condição de conclusão (sensor de posição ou timer) para mover para o próximo número de etapa (`MOVE ProxEtapa -> MW10`).
    *   A Network 17 finaliza o ciclo, resetando `Etapa_Seq` para 0 e desligando `Ciclo_Paletizador_Ativo`, pronto para a próxima caixa.
    *   *Nota:* Esta é uma implementação básica. Um paletizador real teria lógica para empilhar em diferentes posições X/Y no palete e controlar camadas Z.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 9 - Palletizer (Basic)".
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver (Digital I/O ou Modbus, dependendo se usará posições analógicas ou apenas sensores digitais de fim de curso).
4.  **Mapeamento de I/O:**
    *   **Entradas:** Mapeie `Sensor - At Pick Point` para I0.0, `Elevator - At Top` para I0.1, `Elevator - At Bottom` para I0.2, `Gantry X - At Home` para I0.3, `Gantry X - At End` para I0.4.
    *   **Saídas:** Mapeie `Elevator - Motor Up` para Q0.0, `Elevator - Motor Down` para Q0.1, `Gantry X - Motor Right` (ou avanço) para Q0.2, `Gantry X - Motor Left` (ou recuo) para Q0.3, `Grabber - Grab` para Q0.4.
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore as entradas (I0.0-I0.4), saídas (Q0.0-Q0.4), `Ciclo_Paletizador_Ativo` (M1.0) e `Etapa_Seq` (MW10).
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   Uma caixa deve vir pela esteira.
    *   Quando atingir `Sensor_Caixa_Pega` (I0.0), a sequência deve iniciar (MW10=10, M1.0=TRUE).
    *   Observe a sequência de movimentos no Factory IO: Z desce, Garra pega, Z sobe, X avança, Z desce, Garra solta, Z sobe, X recua.
    *   Acompanhe a mudança do valor de `Etapa_Seq` (MW10) no WinSPS-S7 a cada transição.
    *   Verifique se os motores e a garra são ativados/desativados nas etapas corretas.
    *   Ao final, MW10 deve voltar a 0 e M1.0 a FALSE, aguardando a próxima caixa.

**Troubleshooting Comum:**
*   **Sequência não inicia ou para:** Verifique a condição de início (Network 1). Verifique se a `Etapa_Seq` (MW10) está avançando corretamente. Verifique se os sensores de posição (I0.1-I0.4) estão sendo ativados no Factory IO e lidos corretamente no WinSPS-S7. Se usar timers em vez de sensores, verifique os tempos.
*   **Movimento incorreto (eixo errado, direção errada):** Verifique o mapeamento das saídas (Q0.0-Q0.4) para os motores/atuadores corretos no Factory IO. Verifique qual saída está sendo ativada em cada etapa na lógica Ladder.
*   **Garra não pega/solta:** Verifique a lógica e o mapeamento de Q0.4. Verifique os timers T1/T2 se adicionados.
*   **Colisões:** A lógica sequencial deve prevenir colisões se os sensores forem usados corretamente. Se usar timers, ajuste os tempos cuidadosamente.
*   **Problemas de Conexão/Mapeamento:** Siga os passos de verificação dos exercícios anteriores.

Este exercício finaliza a seção de práticas intermediárias, introduzindo o controle de múltiplos eixos e sequências mais complexas, típicas de robótica e manipulação automatizada.
