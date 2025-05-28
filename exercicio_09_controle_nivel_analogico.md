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
