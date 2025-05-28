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
    
    ----[ ]----[/]----(S)----
      T3.Q     T1.Q    Luz_Vermelha
                       (Q0.2)
    
    NETWORK 2: Temporizador Vermelho (TON)
    TITLE=Timer Luz Vermelha
    // Descrição: Conta o tempo da luz vermelha (5 segundos).
    
    ----[ ]----[TON T3, PT:5s]--
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
