# Módulo 5: Integração Prática: WinSPS-S7 e Factory IO

Chegamos a um ponto crucial do nosso curso: a integração entre o ambiente de programação e simulação do CLP (WinSPS-S7) e o ambiente de simulação do processo industrial (Factory IO). É nesta etapa que a teoria e a prática se encontram de forma mais concreta, permitindo-nos ver nossos programas Ladder controlando um sistema virtual dinâmico. Este módulo guiará você através do processo de estabelecer a comunicação entre esses dois softwares, mapear as variáveis e realizar os primeiros testes de controle.

Conectar o WinSPS-S7 ao Factory IO é essencial para validar nossas lógicas de controle em um ambiente que simula o comportamento de sensores e atuadores reais. Existem diferentes maneiras de realizar essa conexão, e a escolha do método pode depender da versão específica do WinSPS-S7 que você está utilizando e das suas funcionalidades de comunicação. Vamos explorar as abordagens mais comuns.

Uma **visão geral das opções de conectividade** revela que a comunicação geralmente se baseia em protocolos de rede padrão ou interfaces de simulação específicas. O Factory IO, como vimos, é flexível e oferece vários drivers. O desafio é encontrar um método compatível com o WinSPS-S7. As duas principais vias que consideraremos são a conexão via um driver compatível com S7 (como S7-PLCSIM, se o WinSPS-S7 puder interagir com ele ou emulá-lo de alguma forma) e a conexão via o protocolo Modbus TCP/IP.

O **Método 1: Conexão via Driver S7-PLCSIM (ou compatível)** seria ideal se o WinSPS-S7 pudesse se comunicar diretamente usando o mesmo protocolo que o simulador PLCSIM da Siemens. No Factory IO, você selecionaria o driver "S7-PLCSIM" ou "S7-PLCSIM Advanced". A configuração no lado do Factory IO geralmente envolve apenas selecionar o driver, pois ele tentará encontrar a instância do PLCSIM rodando na máquina local. No lado do WinSPS-S7, seria necessário garantir que ele esteja configurado para expor sua interface de simulação de uma maneira que o Factory IO possa detectar. No entanto, a compatibilidade direta do WinSPS-S7 com este driver específico do Factory IO pode não ser garantida ou pode exigir configurações específicas ou versões particulares do software. É importante consultar a documentação do WinSPS-S7 ou experimentar para verificar se essa via é funcional. Se for, tende a ser a mais simples, pois o mapeamento de I/O no Factory IO geralmente segue diretamente os endereços S7 (I, Q, M).

O **Método 2: Conexão via Driver Modbus TCP/IP** é frequentemente uma alternativa mais universal e robusta, especialmente se a conexão direta S7 não for viável. O Modbus TCP/IP utiliza a rede Ethernet padrão (mesmo que seja apenas a rede local do seu computador) para a comunicação. Neste cenário, um dos softwares atua como **Servidor Modbus** (aquele que detém os dados e responde às solicitações) e o outro como **Cliente Modbus** (aquele que inicia a comunicação e solicita os dados).

*   **Configuração:** Geralmente, é mais comum configurar o **Factory IO como Cliente Modbus TCP/IP**. No painel de Drivers do Factory IO, você seleciona "Modbus TCP/IP Client". Nas configurações do driver, você precisará especificar o endereço IP do servidor Modbus (que será o endereço IP do seu próprio computador, como 127.0.0.1 para localhost, ou o IP da sua máquina na rede local) e a porta Modbus padrão (normalmente 502). Você também definirá quantos e quais tipos de registradores Modbus (Coils, Discrete Inputs, Holding Registers, Input Registers) o Factory IO lerá e escreverá.
*   **WinSPS-S7 como Servidor Modbus:** O desafio aqui é fazer o WinSPS-S7 atuar como um Servidor Modbus. Algumas versões ou configurações do WinSPS-S7 podem ter essa funcionalidade integrada. Se tiver, você precisará ativá-la e configurá-la, garantindo que ela utilize o mesmo endereço IP e porta configurados no cliente Factory IO. Se o WinSPS-S7 não tiver um servidor Modbus embutido, pode ser necessário usar um **software intermediário** (um gateway ou broker Modbus) que possa ler/escrever nos dados do WinSPS-S7 (talvez via OPC ou outra interface que o WinSPS-S7 suporte) e expor esses dados como um servidor Modbus para o Factory IO se conectar. Esta abordagem adiciona uma camada de complexidade, mas é uma solução comum em cenários de integração diversos.

Independentemente do método de conexão escolhido, a etapa seguinte é crucial: o **Mapeamento de Entradas e Saídas (I/O Mapping)**. É aqui que você diz ao Factory IO qual endereço no CLP (WinSPS-S7) corresponde a qual sensor ou atuador na cena 3D. No painel de Drivers do Factory IO, você verá a lista de sensores (Inputs) e atuadores (Outputs) da cena atual. Para cada um deles, você precisa associar o endereço correspondente no CLP.

*   **Exemplo (Conexão S7):** Se você tem um sensor na cena chamado "Sensor_Caixa_Presente" e quer lê-lo na entrada I0.0 do WinSPS-S7, você mapearia "Sensor_Caixa_Presente" para o endereço I0.0 no Factory IO. Se você tem um atuador "Motor_Esteira" e quer controlá-lo com a saída Q0.0, você mapearia "Motor_Esteira" para Q0.0.
*   **Exemplo (Conexão Modbus):** O mapeamento com Modbus é um pouco diferente. Em vez de endereços S7 diretos (I/Q/M), você usará endereços Modbus. Por exemplo, o Factory IO (cliente) pode ser configurado para ler 10 Discrete Inputs a partir do endereço Modbus 0. O primeiro sensor da lista no Factory IO ("Sensor_Caixa_Presente") seria então associado ao endereço Modbus 0 (que corresponde ao primeiro Discrete Input lido). O segundo sensor seria associado ao endereço Modbus 1, e assim por diante. Similarmente, os atuadores (Coils no Modbus) seriam mapeados. O "Motor_Esteira" poderia ser mapeado para o Coil Modbus 0. No lado do WinSPS-S7 (servidor Modbus ou via gateway), você precisaria garantir que o estado da sua variável interna correspondente ao sensor (ex: M10.0) seja escrito no Discrete Input 0 do Modbus, e que o estado do Coil 0 do Modbus seja lido e usado para controlar sua saída Q0.0.

Agora, vamos a um **Passo a Passo Detalhado** para um exemplo simples. Usaremos a cena "Scene 1 - From A to B" do Factory IO, que tem uma esteira, um sensor no início (Sensor A) e um sensor no fim (Sensor B). Nosso objetivo é criar um programa Ladder no WinSPS-S7 que ligue a esteira quando um botão virtual (que adicionaremos ou usaremos uma memória interna forçada) for pressionado e a desligue quando o mesmo botão for pressionado novamente (lógica de toggle ou partida/parada).

1.  **No WinSPS-S7:**
    *   Crie um novo projeto.
    *   Na Tabela de Símbolos, defina: `Botao_Liga_Desliga` (ex: M0.0), `Motor_Esteira` (ex: Q0.0).
    *   No editor Ladder (OB1), crie a lógica de toggle ou partida/parada usando `Botao_Liga_Desliga` para controlar `Motor_Esteira`. Uma lógica simples de toggle usando detecção de borda positiva (P_TRIG ou equivalente) no botão pode ser:
        ```ladder
        Network 1: Detectar Borda do Botão
        --|P|--( )--
        Botao_Liga_Desliga  Borda_Botao (ex: M0.1)
        
        Network 2: Lógica Toggle para Motor
            +----[ ]----+----( )----
            | Borda_Botao |  Motor_Esteira
        ----|           |------[/]----(R)----
            |           |  Motor_Esteira
            +----[ ]----+ 
              Motor_Esteira
        
        ----[ ]---------[/]----(S)----
          Borda_Botao  Motor_Esteira  Motor_Esteira
        ```
        *(Nota: A implementação exata pode variar. Uma lógica Set/Reset com dois botões virtuais M0.0 e M0.1 seria mais simples inicialmente)*
    *   Compile o programa.
    *   Configure a comunicação (seja S7 ou Modbus Server, conforme a capacidade do seu WinSPS-S7).
    *   Faça o download para o CLP simulado e coloque-o em modo RUN.
2.  **No Factory IO:**
    *   Abra a cena "Scene 1 - From A to B".
    *   Vá em File -> Drivers.
    *   Selecione o driver de comunicação correspondente ao método escolhido (S7-PLCSIM ou Modbus TCP/IP Client).
    *   Clique em CONFIGURATION e insira os parâmetros necessários (ex: endereço IP 127.0.0.1 e porta 502 para Modbus Client se o WinSPS-S7 for o servidor no mesmo PC).
    *   Realize o **Mapeamento de I/O**: arraste o atuador `Belt Conveyor` (ou similar) para a linha correspondente à saída `Motor_Esteira` (Q0.0 ou o endereço Modbus Coil mapeado para Q0.0). Por enquanto, não precisamos mapear os sensores A e B para este exemplo simples, mas você os veria listados como Inputs.
    *   Volte para a janela principal do Factory IO.
3.  **Teste de Comunicação e Lógica:**
    *   No Factory IO, clique no botão "Connect". Se a configuração estiver correta, um indicador verde (geralmente um visto) deve aparecer ao lado do nome do driver, indicando comunicação estabelecida.
    *   No WinSPS-S7, vá para o modo de monitoramento.
    *   Force a variável `Botao_Liga_Desliga` (M0.0) para TRUE por um instante e depois para FALSE (simulando um pulso de botão). Observe no monitoramento do WinSPS-S7 se a saída `Motor_Esteira` (Q0.0) é ativada (Set) e permanece ativa.
    *   Observe no Factory IO se a esteira transportadora começa a se mover.
    *   Force novamente `Botao_Liga_Desliga` (M0.0) para TRUE e FALSE. Observe no WinSPS-S7 se `Motor_Esteira` (Q0.0) é desativada (Reset).
    *   Observe no Factory IO se a esteira para.

Se a esteira ligou e desligou conforme comandado pelo botão virtual no WinSPS-S7, a integração foi bem-sucedida! Se não funcionou, é hora do **Diagnóstico e Solução de Problemas Comuns**:

*   **Falha na Conexão:** Verifique se o driver correto está selecionado no Factory IO. Confirme os parâmetros de comunicação (endereço IP, porta, configurações S7). Certifique-se de que o WinSPS-S7 esteja em modo RUN e que sua interface de comunicação (S7 ou Modbus Server) esteja ativa. Verifique se há firewalls bloqueando a comunicação (especialmente para Modbus TCP/IP).
*   **Mapeamento Incorreto:** Confira duplamente o mapeamento no painel de Drivers do Factory IO. Garanta que o atuador `Belt Conveyor` esteja associado exatamente ao endereço (Q0.0 ou Coil Modbus) que está sendo controlado pela sua lógica no WinSPS-S7. Cuidado com a ordem dos bytes/bits se estiver usando Modbus.
*   **Lógica Incorreta no WinSPS-S7:** Use o modo de monitoramento do WinSPS-S7 para depurar sua lógica Ladder. Verifique se a saída `Motor_Esteira` está realmente sendo ativada/desativada quando você força o botão.
*   **Factory IO não está em modo Play:** Certifique-se de que a simulação no Factory IO esteja rodando (botão Play pressionado) para que os atuadores respondam aos comandos.

Dominar a integração entre WinSPS-S7 e Factory IO abre um leque de possibilidades para testar e visualizar programas de automação complexos. Com esta base, estamos prontos para avançar para os exercícios práticos guiados no próximo módulo.
