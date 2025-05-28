# Módulo 6: Exercícios Práticos Guiados (Nível Iniciante)

Após termos explorado os fundamentos teóricos da automação, da linguagem Ladder, e das ferramentas WinSPS-S7 e Factory IO, além de termos aprendido a integrá-las, chegou o momento de consolidar esse conhecimento através da prática. Este módulo é dedicado a exercícios guiados de nível iniciante, projetados para reforçar os conceitos básicos e desenvolver sua confiança na criação de programas Ladder funcionais que interagem com o ambiente simulado do Factory IO. Cada exercício apresentará um problema simples de automação, a cena do Factory IO a ser utilizada, a lógica Ladder necessária e os passos para implementação e teste.

O objetivo aqui não é criar sistemas complexos, mas sim garantir que você domine as instruções fundamentais (contatos, bobinas, set/reset, temporizadores básicos, contadores básicos) e o processo de mapeamento e teste da integração WinSPS-S7/Factory IO. Vamos começar!

**Exercício 1: Partida Direta de Motor (Esteira Transportadora)**

*   **Objetivo:** Implementar a lógica mais fundamental de controle: ligar e desligar um motor (representado por uma esteira transportadora) utilizando botões virtuais.
*   **Cena Factory IO:** Utilize a cena pré-definida "Scene 1 - From A to B". Esta cena contém uma esteira transportadora (`Belt Conveyor`) e sensores no início e fim, mas para este exercício inicial, focaremos apenas no controle da esteira.
*   **Lógica Ladder (WinSPS-S7):** Vamos implementar uma lógica clássica de Partida-Parada com Selo. Precisaremos de dois botões virtuais (memórias internas no WinSPS-S7, que forçaremos manualmente) e uma saída para o motor.
    *   Defina na Tabela de Símbolos: `Botao_Partida` (ex: M0.0), `Botao_Parada` (ex: M0.1 - Lógica NF no Ladder), `Motor_Esteira` (ex: Q0.0).
    *   Crie a seguinte rede em Ladder:
        ```ladder
        // Network 1: Lógica Partida-Parada com Selo
            +----[ ]----+----[/]----( )----
            | Botao_Partida |  Botao_Parada  Motor_Esteira
        ----|           |
            |  Motor_Esteira|
            +----[ ]----+
        ```
        *Explicação:* Pressionar `Botao_Partida` (forçar M0.0 para TRUE) energiza `Motor_Esteira`. O contato de selo `Motor_Esteira` em paralelo mantém a bobina energizada mesmo após soltar `Botao_Partida`. Pressionar `Botao_Parada` (forçar M0.1 para TRUE) abre o contato NF `Botao_Parada` no Ladder, desenergizando `Motor_Esteira`.
*   **Mapeamento (Factory IO):**
    *   Configure o driver de comunicação (S7 ou Modbus, como aprendido no Módulo 5).
    *   Mapeie o atuador `Belt Conveyor` da cena para a saída `Motor_Esteira` (Q0.0 ou endereço Modbus correspondente).
*   **Teste:**
    1.  Conecte o Factory IO ao WinSPS-S7.
    2.  Coloque o WinSPS-S7 em RUN e monitoramento.
    3.  Coloque o Factory IO em modo Play.
    4.  Force `Botao_Partida` (M0.0) para TRUE momentaneamente. A esteira no Factory IO deve começar a se mover.
    5.  Force `Botao_Parada` (M0.1) para TRUE momentaneamente. A esteira deve parar.
    6.  Repita para confirmar o funcionamento.

**Exercício 2: Controle de Nível Simples (Enchimento de Tanque)**

*   **Objetivo:** Controlar o enchimento de um tanque até que um sensor de nível alto seja ativado.
*   **Cena Factory IO:** Utilize a cena pré-definida "Scene 2 - Level Control". Esta cena possui um tanque, uma válvula de enchimento (`Fill Valve`), uma válvula de descarga (`Discharge Valve`) e um sensor de nível (`Level Sensor`). Focaremos na válvula de enchimento e no sensor de nível.
*   **Lógica Ladder (WinSPS-S7):** A lógica é simples: enquanto o sensor de nível não estiver ativo, a válvula de enchimento deve estar aberta.
    *   Defina na Tabela de Símbolos: `Sensor_Nivel_Alto` (ex: I0.0), `Valvula_Enchimento` (ex: Q0.0).
    *   Crie a seguinte rede:
        ```ladder
        // Network 1: Controle da Válvula de Enchimento
        ----[/]------------( )----
          Sensor_Nivel_Alto  Valvula_Enchimento
        ```
        *Explicação:* O contato NF `Sensor_Nivel_Alto` garante que, enquanto o sensor estiver desligado (nível baixo), a `Valvula_Enchimento` estará ligada. Quando o sensor ligar (nível alto), o contato NF abrirá, desligando a válvula.
*   **Mapeamento (Factory IO):**
    *   Configure o driver e conecte.
    *   Mapeie o sensor `Level Sensor` para a entrada `Sensor_Nivel_Alto` (I0.0 ou endereço Modbus correspondente).
    *   Mapeie o atuador `Fill Valve` para a saída `Valvula_Enchimento` (Q0.0 ou endereço Modbus correspondente).
*   **Teste:**
    1.  Conecte e coloque ambos os softwares em modo de execução/play.
    2.  Observe no Factory IO: a válvula de enchimento deve abrir e o nível do líquido no tanque começará a subir.
    3.  Quando o líquido atingir o sensor de nível, o sensor será ativado no Factory IO.
    4.  Observe no WinSPS-S7 que a entrada `Sensor_Nivel_Alto` (I0.0) vai para TRUE.
    5.  A lógica Ladder deve então desativar a saída `Valvula_Enchimento` (Q0.0).
    6.  Observe no Factory IO que a válvula de enchimento fecha, parando o fluxo.

**Exercício 3: Semáforo de Processo**

*   **Objetivo:** Simular um semáforo simples com luzes Verde, Amarela e Vermelha, usando temporizadores para controlar a sequência.
*   **Cena Factory IO:** Você pode usar a cena "Scene 4 - Converging Belts" que possui um semáforo (`Stack Light`) ou criar uma cena simples adicionando um `Stack Light` da paleta.
*   **Lógica Ladder (WinSPS-S7):** Usaremos temporizadores TON para criar a sequência: Verde (5s) -> Amarelo (2s) -> Vermelho (5s) -> repete.
    *   Defina na Tabela de Símbolos: `Luz_Verde` (Q0.0), `Luz_Amarela` (Q0.1), `Luz_Vermelha` (Q0.2), `Timer_Verde` (T1), `Timer_Amarelo` (T2), `Timer_Vermelho` (T3).
    *   Crie as redes (a lógica exata pode variar, esta é uma sugestão):
        ```ladder
        // Network 1: Ligar Verde (se Vermelho terminou)
        ----[ ]------------(S)----
          T3.Q           Luz_Verde // Usa a saída Q do Timer Vermelho
        
        // Network 2: Temporizador Verde (TON)
        ----[ ]----[TON T1, PT:5s]--
          Luz_Verde
        
        // Network 3: Ligar Amarelo (se Verde terminou), Desligar Verde
        ----[ ]------------(S)----
          T1.Q           Luz_Amarela
        ----[ ]------------(R)----
          T1.Q           Luz_Verde
        
        // Network 4: Temporizador Amarelo (TON)
        ----[ ]----[TON T2, PT:2s]--
          Luz_Amarela
        
        // Network 5: Ligar Vermelho (se Amarelo terminou), Desligar Amarelo
        ----[ ]------------(S)----
          T2.Q           Luz_Vermelha
        ----[ ]------------(R)----
          T2.Q           Luz_Amarela
        
        // Network 6: Temporizador Vermelho (TON) - Reinicia o ciclo
        ----[ ]----[TON T3, PT:5s]--
          Luz_Vermelha
        
        // Network 7: Desligar Vermelho (quando T3 termina para recomeçar)
        ----[ ]------------(R)----
          T3.Q           Luz_Vermelha
        ```
*   **Mapeamento (Factory IO):**
    *   Configure o driver e conecte.
    *   Mapeie `Stack Light Green Light` para `Luz_Verde` (Q0.0).
    *   Mapeie `Stack Light Yellow Light` para `Luz_Amarela` (Q0.1).
    *   Mapeie `Stack Light Red Light` para `Luz_Vermelha` (Q0.2).
*   **Teste:**
    1.  Execute a simulação.
    2.  Observe o semáforo no Factory IO. As luzes devem acender na sequência Verde -> Amarela -> Vermelha, com as durações definidas nos temporizadores, e o ciclo deve se repetir.
    3.  Monitore os temporizadores e as saídas no WinSPS-S7 para acompanhar a lógica.

**Exercício 4: Contagem de Caixas**

*   **Objetivo:** Contar caixas que passam por uma esteira usando um sensor e parar a esteira após um número específico de caixas.
*   **Cena Factory IO:** Use a cena "Scene 1 - From A to B". Adicione um emissor (`Emitter`) no início para gerar caixas e um removedor (`Remover`) no fim. Use o `Diffuse Sensor` (Sensor A) para contar.
*   **Lógica Ladder (WinSPS-S7):** Usaremos um contador CTU.
    *   Defina: `Sensor_Contagem` (I0.0), `Motor_Esteira` (Q0.0), `Contador_Caixas` (C1), `Reset_Contador` (M0.0 - botão virtual).
    *   Lógica:
        ```ladder
        // Network 1: Contar Caixas (Borda de Subida do Sensor)
        ----|P|----[CTU C1, PV:5]-- // Conta na borda de subida do sensor
        Sensor_Contagem
        
        // Network 2: Resetar Contador
        ----[ ]----(R C1)-- // Botão virtual para resetar
        Reset_Contador
        
        // Network 3: Ligar Esteira (se contador não atingiu PV)
        ----[ ]------------( )----
          C1.Q (Invertido) Motor_Esteira // Liga se C1.Q for FALSO
        // Ou use um contato NF associado a C1.Q
        ----[/]------------( )----
          C1.Q           Motor_Esteira
        ```
*   **Mapeamento (Factory IO):**
    *   Configure e conecte.
    *   Mapeie `Diffuse Sensor` (Sensor A) para `Sensor_Contagem` (I0.0).
    *   Mapeie `Belt Conveyor` para `Motor_Esteira` (Q0.0).
*   **Teste:**
    1.  Execute.
    2.  Force `Reset_Contador` (M0.0) para TRUE e depois FALSE para garantir que o contador comece em 0.
    3.  A esteira deve ligar (pois C1.Q está FALSO).
    4.  Caixas serão emitidas e passarão pelo sensor.
    5.  Observe o valor CV do `Contador_Caixas` (C1) incrementar no WinSPS-S7 a cada caixa.
    6.  Quando o contador atingir 5 (PV), a saída C1.Q ficará TRUE.
    7.  A lógica da Network 3 deve desligar `Motor_Esteira`.
    8.  A esteira no Factory IO deve parar.
    9.  Force `Reset_Contador` para reiniciar.

**Exercício 5: Acionamento Sequencial Básico**

*   **Objetivo:** Ativar dois atuadores (ex: pistões) em uma sequência simples: Atuador A avança, Atuador A recua, Atuador B avança, Atuador B recua.
*   **Cena Factory IO:** Crie uma cena simples com dois cilindros pneumáticos (`Pneumatic Cylinder`) A e B, cada um com um solenoide para avançar (ex: `Solenoid A+`, `Solenoid B+`) e sensores de fim de curso (avançado/recuado), se disponíveis/necessários para a lógica.
*   **Lógica Ladder (WinSPS-S7):** Usaremos memórias internas (flags) para controlar as etapas da sequência.
    *   Defina: `Iniciar_Ciclo` (M0.0), `Etapa1_Avanca_A` (M1.0), `Etapa2_Recua_A` (M1.1), `Etapa3_Avanca_B` (M1.2), `Etapa4_Recua_B` (M1.3), `Fim_Ciclo` (M1.4), `Solenoid_A_Mais` (Q0.0), `Solenoid_B_Mais` (Q0.1). (Assumindo cilindros de simples ação que recuam sem solenoide, ou adicione Q0.2/Q0.3 para recuo se forem dupla ação).
    *   Lógica Sequencial (exemplo simplificado sem sensores):
        ```ladder
        // Network 1: Iniciar Ciclo e Etapa 1
        ----[ ]----[ ]----(S)----
        Iniciar_Ciclo Fim_Ciclo  Etapa1_Avanca_A
        
        // Network 2: Acionar Solenoide A+ na Etapa 1
        ----[ ]------------( )----
        Etapa1_Avanca_A  Solenoid_A_Mais
        // Adicionar um Timer TON (T1) para duração do avanço
        ----[ ]----[TON T1, PT:2s]--
        Etapa1_Avanca_A
        
        // Network 3: Iniciar Etapa 2 (Recua A) após Timer T1
        ----[ ]----(S)---- Etapa2_Recua_A
        ----[ ]----(R)---- Etapa1_Avanca_A
          T1.Q
        
        // Network 4: (Desligar Solenoide A+ na Etapa 2 - se necessário)
        // Adicionar Timer T2 para duração do recuo
        ----[ ]----[TON T2, PT:2s]--
        Etapa2_Recua_A
        
        // Network 5: Iniciar Etapa 3 (Avança B) após Timer T2
        ----[ ]----(S)---- Etapa3_Avanca_B
        ----[ ]----(R)---- Etapa2_Recua_A
          T2.Q
        
        // Network 6: Acionar Solenoide B+ na Etapa 3
        ----[ ]------------( )----
        Etapa3_Avanca_B  Solenoid_B_Mais
        // Adicionar Timer T3 para duração do avanço
        ----[ ]----[TON T3, PT:2s]--
        Etapa3_Avanca_B
        
        // Network 7: Iniciar Etapa 4 (Recua B) após Timer T3
        ----[ ]----(S)---- Etapa4_Recua_B
        ----[ ]----(R)---- Etapa3_Avanca_B
          T3.Q
        
        // Network 8: (Desligar Solenoide B+ na Etapa 4 - se necessário)
        // Adicionar Timer T4 para duração do recuo
        ----[ ]----[TON T4, PT:2s]--
        Etapa4_Recua_B
        
        // Network 9: Marcar Fim de Ciclo e Resetar Etapa 4
        ----[ ]----(S)---- Fim_Ciclo
        ----[ ]----(R)---- Etapa4_Recua_B
          T4.Q
        
        // Network 10: Resetar Fim_Ciclo para permitir novo início
        ----[ ]----(R)---- Fim_Ciclo
        Iniciar_Ciclo
        ```
*   **Mapeamento (Factory IO):**
    *   Configure e conecte.
    *   Mapeie o atuador do cilindro A (`Solenoid A+` ou similar) para `Solenoid_A_Mais` (Q0.0).
    *   Mapeie o atuador do cilindro B (`Solenoid B+` ou similar) para `Solenoid_B_Mais` (Q0.1).
*   **Teste:**
    1.  Execute.
    2.  Force `Iniciar_Ciclo` (M0.0) momentaneamente.
    3.  Observe no Factory IO: Cilindro A avança, espera, recua, espera. Cilindro B avança, espera, recua, espera. O ciclo deve parar até novo pulso em `Iniciar_Ciclo`.
    4.  Monitore as etapas (M1.0 a M1.4) e timers no WinSPS-S7.

Estes exercícios fornecem uma base prática sólida. Ao completá-los, você terá aplicado os conceitos fundamentais de Ladder e da integração com o Factory IO. Sinta-se à vontade para modificar os tempos, as condições ou experimentar variações. No próximo módulo, avançaremos para tópicos intermediários da programação Ladder.
