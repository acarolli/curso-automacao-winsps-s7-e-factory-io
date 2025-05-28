# Exercício Prático 1: Partida Direta de Motor (Esteira)

**Objetivo:** Implementar a lógica fundamental de controle industrial: ligar e desligar um motor de esteira transportadora utilizando botões virtuais (memórias internas) com a clássica lógica de selo.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 1 - From A to B"**. Esta cena contém os elementos básicos necessários: uma esteira (`Belt Conveyor 6m`) e sensores (`Diffuse Sensor`) no início e fim. Para este exercício inicial, focaremos apenas no controle do motor da esteira.

**Lógica Ladder (WinSPS-S7):**

Implementaremos a lógica de Partida-Parada com Selo. Esta é uma das estruturas mais comuns e importantes na programação Ladder.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Botao_Partida`: M0.0 (BOOL) - Botão virtual para ligar a esteira.
    *   `Botao_Parada`: M0.1 (BOOL) - Botão virtual para desligar a esteira (usaremos lógica NF no Ladder).
    *   `Motor_Esteira`: Q0.0 (BOOL) - Saída que controla o motor da esteira.

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Controle Partida-Parada com Selo do Motor da Esteira
    TITLE=Controle Motor Esteira
    // Descrição: Liga a esteira com Botao_Partida e desliga com Botao_Parada.
    // O contato de selo Motor_Esteira mantém a esteira ligada após
    // Botao_Partida ser liberado.
    
          +----[ ]----+----[/]----( )----
          | Botao_Partida |  Botao_Parada  Motor_Esteira
          |   (M0.0)    |   (M0.1)     (Q0.0)
    ------|           |
          | Motor_Esteira |
          |   (Q0.0)    |
          +----[ ]----+
    ```

3.  **Explicação da Lógica:**
    *   A rede começa com um contato Normalmente Aberto (NA) `Botao_Partida` (M0.0) em série com um contato Normalmente Fechado (NF) `Botao_Parada` (M0.1).
    *   Estes dois contatos controlam a bobina de saída `Motor_Esteira` (Q0.0).
    *   Em paralelo com o contato `Botao_Partida`, há outro contato NA `Motor_Esteira` (Q0.0). Este é o **contato de selo**.
    *   **Funcionamento:**
        *   Quando `Botao_Partida` (M0.0) é momentaneamente ativado (vai para TRUE), a lógica passa por ele e pelo contato NF `Botao_Parada` (que está TRUE porque M0.1 está FALSE), energizando a bobina `Motor_Esteira` (Q0.0).
        *   Uma vez que `Motor_Esteira` (Q0.0) está TRUE, o contato de selo `Motor_Esteira` (Q0.0) fecha.
        *   Agora, mesmo que `Botao_Partida` (M0.0) volte para FALSE, a lógica continua a fluir através do contato de selo fechado e do contato NF `Botao_Parada`, mantendo a bobina `Motor_Esteira` energizada.
        *   Para parar, `Botao_Parada` (M0.1) é momentaneamente ativado (vai para TRUE). Isso faz com que o contato NF `Botao_Parada` no Ladder abra (vá para FALSE), interrompendo o fluxo lógico para a bobina `Motor_Esteira`.
        *   Com `Motor_Esteira` (Q0.0) indo para FALSE, o contato de selo também abre, e o sistema permanece desligado até que `Botao_Partida` seja pressionado novamente.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 1 - From A to B".
2.  Vá em **File -> Drivers**.
3.  Selecione o driver de comunicação que você configurou para conectar com o WinSPS-S7 (ex: Modbus TCP/IP Client ou S7-PLCSIM, dependendo da sua configuração do Módulo 5).
4.  Clique em **CONFIGURATION** e verifique se os parâmetros de conexão estão corretos (ex: Endereço IP do WinSPS-S7/localhost, porta).
5.  **Mapeamento de I/O:**
    *   Encontre o atuador da esteira na lista de Outputs do Factory IO (geralmente chamado `Belt Conveyor 6m`).
    *   Arraste e solte (ou selecione na lista suspensa) a tag/endereço correspondente à sua saída `Motor_Esteira` no WinSPS-S7 (Q0.0 ou o endereço Modbus Coil mapeado para Q0.0).
    *   *Nota:* Para este exercício, não precisamos mapear os sensores da cena para as entradas do WinSPS-S7.
6.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile seu programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Abra a janela de monitoramento (Status Variable / Beobachten) para observar M0.0, M0.1 e Q0.0.
2.  **No Factory IO:**
    *   Clique no botão **Connect** no painel Drivers. Verifique se aparece um indicador verde (✓) mostrando que a conexão foi bem-sucedida.
    *   Clique no botão **Play (▶)** para iniciar a simulação da cena.
3.  **Execução do Teste:**
    *   **Partida:** No WinSPS-S7, force a variável `Botao_Partida` (M0.0) para TRUE por um breve momento e depois retorne para FALSE. Observe no monitoramento que Q0.0 (Motor_Esteira) deve ir para TRUE e permanecer TRUE. Observe no Factory IO que a esteira transportadora deve começar a se mover.
    *   **Parada:** No WinSPS-S7, force a variável `Botao_Parada` (M0.1) para TRUE por um breve momento e depois retorne para FALSE. Observe no monitoramento que Q0.0 (Motor_Esteira) deve ir para FALSE. Observe no Factory IO que a esteira transportadora deve parar.
    *   Repita os passos de partida e parada algumas vezes para confirmar que a lógica de selo e a parada estão funcionando corretamente.

**Troubleshooting Comum:**
*   **Esteira não liga:** Verifique a conexão no Factory IO, o mapeamento da saída Q0.0 para o `Belt Conveyor`, se o WinSPS-S7 está em RUN, e se a lógica Ladder está correta (monitorando Q0.0).
*   **Esteira não para:** Verifique se o contato NF `Botao_Parada` está corretamente endereçado (M0.1) e se forçar M0.1 para TRUE realmente interrompe a lógica para Q0.0 no monitoramento.
*   **Conexão Falha:** Reveja os passos de configuração do driver no Módulo 5. Verifique IPs, portas e se o WinSPS-S7 está pronto para a comunicação.

Este primeiro exercício prático estabelece a base para os controles mais complexos que virão. Dominar a lógica de selo é fundamental para a programação Ladder.
