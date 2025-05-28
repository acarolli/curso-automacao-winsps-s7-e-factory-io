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
    
    ----[/]------------( )----
      Sensor_Nivel_Alto  Valvula_Enchimento
        (I0.0)           (Q0.0)
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
