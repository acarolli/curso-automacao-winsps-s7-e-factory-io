# Curso Prático de Automação Industrial: Ladder com WinSPS-S7 e Factory IO





---



[Estrutura Curso](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/estrutura_curso.md)

[Módulo 01](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_01_rascunho.md)

[Módulo 02](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_02_rascunho.md)

[Módulo 03](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_03_rascunho.md)

[Módulo 04](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_04_rascunho.md)

[Módulo 05](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_05_rascunho.md)

[Módulo 06](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_06_rascunho.md)

[Módulo 07](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_07_rascunho.md)

[Módulo 08](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_08_rascunho.md)

[Módulo 09](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_09_rascunho.md)

[Módulo 10](https://github.com/acarolli/curso-automacao-winsps-s7-e-factory-io/blob/main/modulo_10_rascunho.md)

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
    
          +----[ ]--------+-------[/]----------( )----
          | Botao_Partida |  Botao_Parada  Motor_Esteira
          |   (M0.0)      |     (M0.1)        (Q0.0)
    ------|               |
          | Motor_Esteira |
          |   (Q0.0)      |
          +----[ ]--------+
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


---


# Exercício Prático 2: Controle de Nível Simples (Enchimento de Tanque)

**Objetivo:** Implementar um controle básico de nível em um tanque, utilizando um sensor de nível alto para interromper o enchimento quando o nível desejado for atingido.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 2 - Level Control"**. Esta cena é ideal pois já contém um tanque (`Tank`), uma válvula de enchimento (`Fill Valve`), uma válvula de descarga (`Discharge Valve` - não usaremos neste exercício) e um sensor de nível capacitivo (`Capacitive Sensor`) posicionado na parte superior, que atuará como nosso sensor de nível alto.

**Lógica Ladder (WinSPS-S7):**

O controle será direto: a válvula de enchimento permanecerá aberta enquanto o sensor de nível alto não detectar a presença de líquido.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Sensor_Nivel_Alto`: I0.0 (BOOL) - Entrada conectada ao sensor de nível no topo do tanque.
    *   `Valvula_Enchimento`: Q0.0 (BOOL) - Saída que controla a válvula de enchimento do tanque.

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Controle da Válvula de Enchimento Baseado no Nível Alto
    TITLE=Controle Enchimento Tanque
    // Descrição: Mantém a válvula de enchimento aberta enquanto o
    // sensor de nível alto não estiver ativo. Quando o sensor
    // detectar líquido (nível alto), a válvula fecha.
    
    ---------[/]---------------( )----
      Sensor_Nivel_Alto  Valvula_Enchimento
            (I0.0)            (Q0.0)
    ```

3.  **Explicação da Lógica:**
    *   A rede consiste em um único contato Normalmente Fechado (NF) associado à entrada `Sensor_Nivel_Alto` (I0.0) que controla diretamente a bobina `Valvula_Enchimento` (Q0.0).
    *   **Funcionamento:**
        *   Quando o tanque está vazio ou o nível está abaixo do sensor, `Sensor_Nivel_Alto` (I0.0) está FALSE (desligado).
        *   O contato NF `Sensor_Nivel_Alto` no Ladder estará, portanto, fechado (permitindo a passagem da lógica, estado TRUE).
        *   Isso energiza a bobina `Valvula_Enchimento` (Q0.0), abrindo a válvula e permitindo que o tanque comece a encher.
        *   Quando o nível do líquido atinge o sensor, `Sensor_Nivel_Alto` (I0.0) vai para TRUE (ligado).
        *   O contato NF `Sensor_Nivel_Alto` no Ladder abre (vai para FALSE), interrompendo a lógica para a bobina.
        *   `Valvula_Enchimento` (Q0.0) é desenergizada, fechando a válvula e parando o enchimento.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena "Scene 2 - Level Control".
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação para conectar com o WinSPS-S7 (conforme Módulo 5).
4.  **Mapeamento de I/O:**
    *   **Entrada:** Encontre o sensor na lista de Inputs do Factory IO (geralmente chamado `Capacitive Sensor`). Mapeie este sensor para a entrada `Sensor_Nivel_Alto` (I0.0 ou endereço Modbus Discrete Input correspondente) no WinSPS-S7.
    *   **Saída:** Encontre a válvula de enchimento na lista de Outputs (geralmente `Fill Valve`). Mapeie esta válvula para a saída `Valvula_Enchimento` (Q0.0 ou endereço Modbus Coil correspondente) no WinSPS-S7.
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore o estado de I0.0 e Q0.0.
2.  **No Factory IO:**
    *   Clique em **Connect** no painel Drivers e verifique a conexão (✓).
    *   Clique em **Play (▶)** para iniciar a simulação.
3.  **Execução do Teste:**
    *   **Observação Inicial:** Ao iniciar a simulação, como o tanque está vazio, o `Capacitive Sensor` (I0.0) estará FALSE. No WinSPS-S7, o contato NF estará fechado, e Q0.0 (`Valvula_Enchimento`) estará TRUE. No Factory IO, a `Fill Valve` deve estar aberta, e o líquido começará a fluir para o tanque.
    *   **Acompanhe o Enchimento:** Observe o nível do líquido subindo no tanque dentro do Factory IO.
    *   **Nível Atingido:** Quando o líquido atingir o `Capacitive Sensor`, ele mudará seu estado para TRUE no Factory IO. Observe no WinSPS-S7 que a entrada I0.0 também irá para TRUE.
    *   **Válvula Fecha:** Com I0.0 TRUE, o contato NF no Ladder abrirá, desenergizando Q0.0 (`Valvula_Enchimento`). Observe no Factory IO que a `Fill Valve` fechará, interrompendo o fluxo de líquido.
    *   **(Opcional) Esvaziar:** Para repetir o teste, você pode usar a `Discharge Valve` manualmente no Factory IO (clicando nela se a interação estiver habilitada) ou parar/reiniciar a simulação para esvaziar o tanque.

**Troubleshooting Comum:**
*   **Válvula não abre no início:** Verifique o mapeamento de Q0.0 para a `Fill Valve`. Confirme se I0.0 está realmente FALSE no início e se o contato NF no Ladder está corretamente programado.
*   **Válvula não fecha quando o nível atinge o sensor:** Verifique o mapeamento de I0.0 para o `Capacitive Sensor`. Confirme se o sensor no Factory IO está mudando para TRUE e se essa mudança está sendo refletida em I0.0 no WinSPS-S7. Verifique se a lógica NF está correta.
*   **Problemas de Conexão/Comunicação:** Siga os mesmos passos de verificação do Exercício 1 (conexão, IPs, portas, modo RUN).

Este exercício demonstra como usar uma entrada digital (sensor) para controlar diretamente uma saída (válvula), implementando um controle de processo simples e fundamental.


---


# Exercício Prático 3: Semáforo de Processo

**Objetivo:** Simular um ciclo de semáforo industrial simples (Verde -> Amarelo -> Vermelho) utilizando temporizadores para controlar a duração de cada luz e criar uma sequência repetitiva.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Você pode utilizar a cena pré-definida **"Scene 4 - Converging Belts"**, que já inclui um semáforo (`Stack Light`) em uma das esteiras, ou criar uma cena nova e adicionar um componente `Stack Light` da paleta de peças (`Pallete`). O `Stack Light` geralmente possui saídas separadas para cada cor (Verde, Amarela, Vermelha).

**Lógica Ladder (WinSPS-S7):**

Utilizaremos temporizadores On-Delay (TON) para controlar os tempos e lógica de Set/Reset ou contatos para gerenciar a sequência das luzes.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Luz_Verde`: Q0.0 (BOOL) - Saída para a luz verde.
    *   `Luz_Amarela`: Q0.1 (BOOL) - Saída para a luz amarela.
    *   `Luz_Vermelha`: Q0.2 (BOOL) - Saída para a luz vermelha.
    *   `Timer_Verde`: T1 (TIMER) - Temporizador para a duração da luz verde.
    *   `Timer_Amarelo`: T2 (TIMER) - Temporizador para a duração da luz amarela.
    *   `Timer_Vermelho`: T3 (TIMER) - Temporizador para a duração da luz vermelha.
    *   `Flag_Ciclo_Iniciado`: M0.0 (BOOL) - (Opcional) Para iniciar o ciclo apenas uma vez ou com um botão.

2.  **Programa Ladder (OB1):** (Uma abordagem possível usando Set/Reset e as saídas Q dos timers)
    ```ladder
    NETWORK 1: Iniciar Ciclo (Liga Vermelho inicialmente ou com botão)
    TITLE=Iniciar Ciclo Semáforo
    // Descrição: Garante que o ciclo comece com a luz vermelha
    // ou pode ser iniciado por um botão M0.0 (não implementado aqui).
    // Usaremos a saída do Timer Vermelho (T3.Q) para reiniciar o ciclo.
    // O contato NF T1.Q garante que o vermelho só ligue quando verde apagar.
    
    ----[ ]-------[/]--------(S)----
        T3.Q     T1.Q    Luz_Vermelha
                           (Q0.2)
    
    NETWORK 2: Temporizador Vermelho (TON)
    TITLE=Timer Luz Vermelha
    // Descrição: Conta o tempo da luz vermelha (5 segundos).
    
    ------[ ]---------[TON T3, PT:5s]--
      Luz_Vermelha
        (Q0.2)
    
    NETWORK 3: Ligar Verde (quando Vermelho termina), Desligar Vermelho
    TITLE=Transição Vermelho -> Verde
    // Descrição: Usa a saída do Timer Vermelho (T3.Q) para ligar
    // a luz verde e desligar a luz vermelha.
    
    ----[ ]------------(S)----
      T3.Q           Luz_Verde
                      (Q0.0)
    ----[ ]------------(R)----
      T3.Q           Luz_Vermelha
                      (Q0.2)
    
    NETWORK 4: Temporizador Verde (TON)
    TITLE=Timer Luz Verde
    // Descrição: Conta o tempo da luz verde (5 segundos).
    
    ----[ ]----[TON T1, PT:5s]--
      Luz_Verde
      (Q0.0)
    
    NETWORK 5: Ligar Amarelo (quando Verde termina), Desligar Verde
    TITLE=Transição Verde -> Amarelo
    // Descrição: Usa a saída do Timer Verde (T1.Q) para ligar
    // a luz amarela e desligar a luz verde.
    
    ----[ ]------------(S)----
      T1.Q           Luz_Amarela
                       (Q0.1)
    ----[ ]------------(R)----
      T1.Q           Luz_Verde
                       (Q0.0)
    
    NETWORK 6: Temporizador Amarelo (TON)
    TITLE=Timer Luz Amarela
    // Descrição: Conta o tempo da luz amarela (2 segundos).
    
    ----[ ]----[TON T2, PT:2s]--
      Luz_Amarela
      (Q0.1)
    
    NETWORK 7: Reiniciar Ciclo (Desligar Amarelo quando seu timer termina)
    TITLE=Transição Amarelo -> Reinício (Vermelho)
    // Descrição: Usa a saída do Timer Amarelo (T2.Q) para desligar
    // a luz amarela. A Network 1 será responsável por religar a vermelha
    // usando T3.Q (que estará FALSE neste momento, permitindo Set Vermelho).
    
    ----[ ]------------(R)----
      T2.Q           Luz_Amarela
                       (Q0.1)
    // Nota: A Network 1 ligará a Luz_Vermelha quando T3.Q (do ciclo anterior) for TRUE
    // e T1.Q for FALSE. A lógica pode precisar de ajuste fino para garantir
    // um reinício limpo e imediato da luz vermelha após a amarela apagar.
    // Uma alternativa é usar T2.Q para Set(Luz_Vermelha) diretamente aqui.
    ```

3.  **Explicação da Lógica:**
    *   A lógica usa as saídas dos temporizadores (Q) para sinalizar o fim de um estado e iniciar o próximo, utilizando bobinas Set (S) e Reset (R) para controlar as luzes.
    *   O ciclo começa (implicitamente ou forçado) com a Luz Vermelha (Network 1).
    *   O `Timer_Vermelho` (T3) conta 5 segundos (Network 2).
    *   Quando T3 termina (T3.Q = TRUE), ele desliga a Luz Vermelha e liga a Luz Verde (Network 3).
    *   O `Timer_Verde` (T1) conta 5 segundos (Network 4).
    *   Quando T1 termina (T1.Q = TRUE), ele desliga a Luz Verde e liga a Luz Amarela (Network 5).
    *   O `Timer_Amarelo` (T2) conta 2 segundos (Network 6).
    *   Quando T2 termina (T2.Q = TRUE), ele desliga a Luz Amarela (Network 7). A lógica da Network 1 (ou uma modificação) deve então religar a Luz Vermelha para reiniciar o ciclo.
    *   *Observação:* A transição exata para reiniciar o ciclo (garantir que o Vermelho acenda imediatamente após o Amarelo apagar) pode exigir um ajuste fino ou uma abordagem ligeiramente diferente, talvez usando a borda de descida de T2.Q ou usando T2.Q para Set(Luz_Vermelha) diretamente na Network 7 e ajustando a Network 1.

**Integração e Mapeamento (Factory IO):**

1.  **Abra o Factory IO** e carregue a cena escolhida (ex: "Scene 4") ou crie uma com um `Stack Light`.
2.  Vá em **File -> Drivers**.
3.  Selecione e configure o driver de comunicação.
4.  **Mapeamento de I/O:**
    *   Encontre as saídas do semáforo na lista de Outputs do Factory IO (ex: `Stack Light Green Light`, `Stack Light Yellow Light`, `Stack Light Red Light`).
    *   Mapeie `Stack Light Green Light` para a saída `Luz_Verde` (Q0.0 ou endereço Modbus Coil correspondente).
    *   Mapeie `Stack Light Yellow Light` para a saída `Luz_Amarela` (Q0.1 ou endereço Modbus Coil correspondente).
    *   Mapeie `Stack Light Red Light` para a saída `Luz_Vermelha` (Q0.2 ou endereço Modbus Coil correspondente).
5.  Volte para a janela principal do Factory IO.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Faça o download para o CLP simulado.
    *   Coloque o CLP simulado em modo **RUN**.
    *   Monitore Q0.0, Q0.1, Q0.2 e os valores dos timers T1, T2, T3.
2.  **No Factory IO:**
    *   Clique em **Connect** e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   Observe o `Stack Light` no Factory IO.
    *   A sequência de luzes deve ser: Vermelho (por ~5s) -> Verde (por ~5s) -> Amarelo (por ~2s) -> Vermelho (por ~5s) ... e assim por diante, repetindo o ciclo.
    *   Acompanhe no WinSPS-S7 as saídas Q0.0, Q0.1, Q0.2 sendo ativadas e desativadas e os temporizadores contando e zerando conforme a sequência progride.

**Troubleshooting Comum:**
*   **Sequência incorreta ou luzes não acendem/apagam:** Verifique cuidadosamente a lógica Ladder, especialmente as condições de Set e Reset e as transições baseadas nas saídas Q dos timers. Monitore passo a passo no WinSPS-S7.
*   **Tempos incorretos:** Verifique os valores de Preset Time (PT) configurados nos blocos TON (T1, T2, T3).
*   **Ciclo não reinicia corretamente:** Analise a lógica de transição da luz Amarela para a Vermelha (Networks 7 e 1). Pode ser necessário ajustar como a Luz Vermelha é reativada.
*   **Problemas de Mapeamento/Conexão:** Confira o mapeamento das saídas Q0.0, Q0.1, Q0.2 para as luzes corretas no Factory IO e verifique a conexão do driver.

Este exercício introduz o uso de temporizadores para controlar sequências baseadas em tempo, uma técnica fundamental em muitas aplicações de automação.


---


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


---


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


---


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


---


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


---


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


---


# Exercício Prático 9: Controle de Nível com Setpoint Analógico (Simulado)

**Objetivo:** Implementar um controle de nível liga/desliga (on/off) em um tanque, mantendo o nível do líquido entre dois setpoints (alto e baixo) definidos pelo usuário, utilizando uma leitura de nível analógica simulada e controlando válvulas digitais.

**Ferramentas:**
*   WinSPS-S7
*   Factory IO

**Cena Factory IO:**
*   Utilize a cena pré-definida: **"Scene 2 - Level Control"**. Desta vez, focaremos nos seguintes componentes:
    *   `Tank`: O reservatório.
    *   `Level Meter`: Este sensor fornece uma **saída numérica (analógica simulada)** que representa a altura do líquido (geralmente em uma escala, ex: 0-10 ou 0-300). Esta será nossa entrada analógica para o CLP.
    *   `Fill Valve`: Válvula de enchimento (controlada por uma saída digital do CLP).
    *   `Discharge Valve`: Válvula de descarga (controlada por outra saída digital do CLP).

**Lógica Ladder (WinSPS-S7):**

A lógica lerá o valor analógico simulado do nível, o comparará com os setpoints alto e baixo, e controlará as válvulas de enchimento e descarga para manter o nível dentro da faixa desejada. Usaremos lógica Set/Reset para as válvulas, o que inerentemente cria uma histerese (banda morta) e evita ciclos rápidos.

1.  **Definição de Símbolos (Tabela de Símbolos):**
    *   `Leitura_Nivel_Analog`: IW0 (WORD) - Entrada analógica que recebe o valor do `Level Meter`. (O endereço exato, IW0, PIW0, ou um endereço Modbus Input Register, dependerá da configuração de comunicação).
    *   `Setpoint_Alto`: MW10 (WORD) - Memória para armazenar o valor do setpoint alto (ex: 80% da escala).
    *   `Setpoint_Baixo`: MW12 (WORD) - Memória para armazenar o valor do setpoint baixo (ex: 20% da escala).
    *   `Valvula_Enchimento`: Q0.0 (BOOL) - Saída para controlar a `Fill Valve`.
    *   `Valvula_Descarga`: Q0.1 (BOOL) - Saída para controlar a `Discharge Valve`.
    *   *Nota sobre Escala:* Assumiremos aqui que os valores em `Leitura_Nivel_Analog`, `Setpoint_Alto` e `Setpoint_Baixo` estão na mesma escala (seja a escala bruta do sensor 0-10/0-300 ou uma escala percentual 0-100 após escalonamento, se aplicável e suportado pelo WinSPS-S7).

2.  **Programa Ladder (OB1):**
    ```ladder
    NETWORK 1: Carregar Setpoints (Exemplo - Fazer apenas uma vez ou via IHM)
    TITLE=Definir Setpoints
    // Descrição: Carrega valores nos setpoints Alto e Baixo.
    // Em uma aplicação real, isso viria de uma IHM ou parâmetros.
    // Para teste, podemos usar MOVE com um contato de 'primeiro scan' (não mostrado)
    // ou forçar os valores em MW10 e MW12 diretamente no monitoramento.
    // Exemplo: MOVE(INT_TO_WORD(80), MW10), MOVE(INT_TO_WORD(20), MW12)
    
    // (Instruções MOVE omitidas para clareza - definir valores manualmente)
    
    NETWORK 2: Controle da Válvula de Enchimento (Set/Reset)
    TITLE=Controle Válvula Enchimento
    // Descrição: Liga (S) a válvula de enchimento quando o nível cai
    // abaixo ou igual ao Setpoint_Baixo. Desliga (R) quando o nível
    // sobe acima ou igual ao Setpoint_Alto.
    
    // Ligar Enchimento (Set)
    ----[<=]-----------(S)----
      Leitura_Nivel_Analog  Valvula_Enchimento
      (IW0)                 (Q0.0)
      Setpoint_Baixo
      (MW12)
      
    // Desligar Enchimento (Reset)
    ----[>=]-----------(R)----
      Leitura_Nivel_Analog  Valvula_Enchimento
      (IW0)                 (Q0.0)
      Setpoint_Alto
      (MW10)
      
    NETWORK 3: Controle da Válvula de Descarga (Set/Reset)
    TITLE=Controle Válvula Descarga
    // Descrição: Liga (S) a válvula de descarga quando o nível sobe
    // acima ou igual ao Setpoint_Alto. Desliga (R) quando o nível
    // cai abaixo ou igual ao Setpoint_Baixo.
    
    // Ligar Descarga (Set)
    ----[>=]-----------(S)----
      Leitura_Nivel_Analog  Valvula_Descarga
      (IW0)                 (Q0.1)
      Setpoint_Alto
      (MW10)
      
    // Desligar Descarga (Reset)
    ----[<=]-----------(R)----
      Leitura_Nivel_Analog  Valvula_Descarga
      (IW0)                 (Q0.1)
      Setpoint_Baixo
      (MW12)
    ```

3.  **Explicação da Lógica:**
    *   A Network 1 é um placeholder para a definição dos setpoints. Na prática, você forçaria os valores desejados em MW10 e MW12 usando a ferramenta de monitoramento/modificação do WinSPS-S7.
    *   **Válvula de Enchimento (Network 2):**
        *   A instrução Set (`(S)`) é ativada se `Leitura_Nivel_Analog` for menor ou igual a `Setpoint_Baixo`. Isso liga a `Valvula_Enchimento`.
        *   A instrução Reset (`(R)`) é ativada se `Leitura_Nivel_Analog` for maior ou igual a `Setpoint_Alto`. Isso desliga a `Valvula_Enchimento`.
        *   Como Set e Reset operam em momentos diferentes, a válvula permanecerá ligada entre o Setpoint Baixo e o Alto (durante o enchimento) e desligada entre o Alto e o Baixo (durante a descarga).
    *   **Válvula de Descarga (Network 3):**
        *   A lógica é inversa. A instrução Set (`(S)`) é ativada se o nível for maior ou igual ao `Setpoint_Alto`, ligando a `Valvula_Descarga`.
        *   A instrução Reset (`(R)`) é ativada se o nível for menor ou igual ao `Setpoint_Baixo`, desligando a `Valvula_Descarga`.
    *   Essa lógica Set/Reset cria uma **histerese** natural: o enchimento começa em 20 e para em 80; a descarga começa em 80 e para em 20. Isso evita que as válvulas fiquem ligando e desligando rapidamente se o nível flutuar exatamente sobre um único setpoint.

**Integração e Mapeamento (Factory IO):**

*   **Comunicação:** A leitura de valores analógicos do Factory IO para o WinSPS-S7 geralmente requer o driver **Modbus TCP/IP Client** no Factory IO, com o WinSPS-S7 atuando como Servidor Modbus (ou usando um gateway).
1.  **Abra o Factory IO** e carregue a cena "Scene 2 - Level Control".
2.  Vá em **File -> Drivers** e selecione **Modbus TCP/IP Client**.
3.  **Configure o Driver Modbus:**
    *   Em CONFIGURATION, defina o IP do servidor (seu PC, ex: 127.0.0.1) e a porta (502).
    *   Configure a leitura de **Input Registers** (para ler o nível analógico) e a escrita de **Coils** (para controlar as válvulas digitais).
    *   Exemplo: Ler 1 Input Register começando do endereço 0. Escrever 2 Coils começando do endereço 0.
4.  **Mapeamento de I/O:**
    *   **Entrada Analógica:** Encontre o `Level Meter` na lista de Inputs (ele terá um tipo numérico, ex: REAL ou INT). Mapeie-o para o **Input Register 0** (ou o endereço configurado).
    *   **Saídas Digitais:**
        *   Encontre a `Fill Valve` na lista de Outputs (tipo BOOL). Mapeie-a para o **Coil 0**.
        *   Encontre a `Discharge Valve` na lista de Outputs (tipo BOOL). Mapeie-a para o **Coil 1**.
5.  **No WinSPS-S7:**
    *   Configure o WinSPS-S7 para atuar como **Servidor Modbus** (se suportado) ou use um gateway.
    *   Garanta que o valor lido do **Input Register 0** do Modbus seja movido para `Leitura_Nivel_Analog` (IW0 ou outra memória WORD).
    *   Garanta que o estado de `Valvula_Enchimento` (Q0.0) seja escrito no **Coil 0** do Modbus.
    *   Garanta que o estado de `Valvula_Descarga` (Q0.1) seja escrito no **Coil 1** do Modbus.
    *   *(A configuração exata do servidor Modbus no WinSPS-S7 ou gateway varia muito e pode exigir consulta à documentação específica ou experimentação)*.

**Passos para Teste:**

1.  **No WinSPS-S7:**
    *   Compile o programa Ladder.
    *   Configure o Servidor Modbus (ou gateway).
    *   Faça o download para o CLP simulado e coloque em modo **RUN**.
    *   Force os valores desejados para `Setpoint_Alto` (MW10, ex: 80) e `Setpoint_Baixo` (MW12, ex: 20) - *certifique-se que a escala corresponde à lida de IW0*.
    *   Monitore IW0, MW10, MW12, Q0.0, Q0.1.
2.  **No Factory IO:**
    *   Clique em **Connect** no painel Drivers Modbus e verifique a conexão (✓).
    *   Clique em **Play (▶)**.
3.  **Execução do Teste:**
    *   **Enchimento:** Inicialmente, o tanque está vazio (IW0 baixo). `Leitura_Nivel_Analog` <= `Setpoint_Baixo` deve ser TRUE. `Valvula_Enchimento` (Q0.0 / Coil 0) deve ligar (Set). A `Fill Valve` abre no Factory IO, e o nível começa a subir. `Valvula_Descarga` (Q0.1 / Coil 1) deve estar desligada.
    *   **Atingindo Nível Alto:** Observe o valor de `Leitura_Nivel_Analog` (IW0) subir. Quando `Leitura_Nivel_Analog` >= `Setpoint_Alto`, a condição de Reset para Q0.0 e a condição de Set para Q0.1 se tornam TRUE. `Valvula_Enchimento` desliga, e `Valvula_Descarga` liga. O enchimento para, e o esvaziamento começa.
    *   **Descarga:** Observe o nível baixar. `Valvula_Enchimento` permanece desligada, `Valvula_Descarga` permanece ligada.
    *   **Atingindo Nível Baixo:** Quando `Leitura_Nivel_Analog` <= `Setpoint_Baixo`, a condição de Set para Q0.0 e a condição de Reset para Q0.1 se tornam TRUE. `Valvula_Enchimento` liga, e `Valvula_Descarga` desliga. O esvaziamento para, e o enchimento recomeça.
    *   **Ciclo:** O nível do líquido deve oscilar continuamente entre os `Setpoint_Baixo` e `Setpoint_Alto`.

**Troubleshooting Comum:**
*   **Valores Analógicos Incorretos:** Verifique a configuração do Modbus (endereços, tipo de dado - Input Register). Verifique se o valor do `Level Meter` no Factory IO está sendo corretamente transferido para `Leitura_Nivel_Analog` (IW0) no WinSPS-S7. Considere problemas de escala (o `Level Meter` pode fornecer 0-10, 0-300, etc., enquanto os setpoints estão em outra escala).
*   **Válvulas não operam ou operam nos níveis errados:** Verifique a lógica de comparação (`<=`, `>=`) e as instruções Set/Reset. Confirme se os valores dos setpoints (MW10, MW12) estão corretos e na mesma escala que IW0. Verifique o mapeamento dos Coils Modbus para Q0.0 e Q0.1.
*   **Válvulas ciclando muito rápido (sem histerese):** Certifique-se de que está usando a lógica Set/Reset corretamente, pois ela fornece a histerese. Se usasse comparações diretas para ligar/desligar as válvulas (sem S/R), flutuações no nível poderiam causar ciclos rápidos.
*   **Problemas de Conexão Modbus:** Verifique IPs, portas, configuração do Servidor Modbus no WinSPS-S7/gateway.

Este exercício introduz o trabalho com I/O analógico simulado e controle baseado em setpoints com histerese, fundamental para muitos processos contínuos ou de batelada.


---


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
