# Projeto Final Guiado 2: Sistema de Armazenamento e Recuperação (AS/RS) Básico

**Introdução:**
Este segundo projeto final desafia você a aplicar seus conhecimentos em um sistema mais complexo de movimentação e gerenciamento de posições: um Sistema Automatizado de Armazenamento e Recuperação (Automated Storage and Retrieval System - AS/RS). Você desenvolverá a lógica para controlar um elevador com garfo/plataforma para armazenar caixas em prateleiras e recuperá-las sob demanda, exigindo controle preciso de múltiplos eixos, gerenciamento de estados e posições, e implementação de sequências distintas para armazenamento e recuperação.

**Descrição Geral:**
Simular um pequeno armazém automatizado onde caixas chegam por uma esteira, são pegas por um transelevador (elevador + eixo horizontal), armazenadas em locais específicos de uma estante e, posteriormente, recuperadas e enviadas para uma esteira de saída.

**Cena Factory IO:**
*   **Recomendada:** Utilize a cena pré-definida **"Scene 10 - Automated Warehouse"**.
*   **Componentes Essenciais (verificar na cena):**
    *   Esteira de Entrada (`Entry Conveyor`)
    *   Sensor na Posição de Transferência da Entrada (`Sensor At Entry`)
    *   Transelevador composto por:
        *   Elevador (Eixo Z) com motor/comando para Subir/Descer.
        *   Transportador Horizontal (Eixo X ou Y) com motor/comando para Avançar/Recuar (Home/Rack).
        *   Plataforma ou Garfo com atuador para Pegar/Soltar (Load/Unload, Clamp/Unclamp).
    *   Sensores de Posição para os eixos (ex: `At Top`, `At Bottom`, `At Home`, `At Rack`, ou sensores analógicos/encoders que fornecem posição numérica).
    *   Estrutura de Prateleiras (Racks) com múltiplas localizações.
    *   Esteira de Saída (`Exit Conveyor`)
    *   Sensor na Posição de Transferência da Saída (`Sensor At Exit`)
    *   (Opcional) Interface simples no Factory IO ou botões virtuais no CLP para selecionar modo (Armazenar/Recuperar) e localização.

**Requisitos e Descrição Funcional Detalhada:**

1.  **Modos de Operação:**
    *   O sistema deve operar em dois modos principais: **Armazenamento** e **Recuperação**.
    *   A seleção do modo pode ser feita por botões virtuais (flags M no CLP) ou inputs.
    *   Um botão/flag de "Iniciar Ciclo" pode ser necessário para cada modo.
    *   Um botão/flag de "Reset/Parada de Emergência" deve estar disponível.

2.  **Modo Armazenamento:**
    *   O sistema aguarda uma caixa no `Sensor At Entry`.
    *   Quando a caixa chega, a Esteira de Entrada para.
    *   O transelevador (Eixo Z e X/Y) se move para a posição de pega na entrada (coordenadas pré-definidas).
    *   A plataforma/garfo é acionada para pegar a caixa.
    *   O sistema determina a próxima localização livre na prateleira (lógica simples: primeira livre, ou endereço fornecido por input virtual).
    *   O transelevador move a caixa para a posição X/Y e Z da prateleira alvo.
    *   A plataforma/garfo deposita a caixa.
    *   O sistema marca a localização como ocupada.
    *   O transelevador retorna a uma posição de espera (ex: Home X/Y, Top Z).
    *   A Esteira de Entrada é liberada para trazer a próxima caixa (se houver).

3.  **Modo Recuperação:**
    *   O usuário (via input virtual/flag) especifica a localização da prateleira de onde deseja recuperar uma caixa.
    *   O sistema verifica se a localização está ocupada.
    *   O transelevador se move para a posição X/Y e Z da prateleira alvo.
    *   A plataforma/garfo pega a caixa.
    *   O sistema marca a localização como livre.
    *   O transelevador move a caixa para a posição de transferência da saída (coordenadas pré-definidas).
    *   A plataforma/garfo deposita a caixa na Esteira de Saída.
    *   A Esteira de Saída é acionada para transportar a caixa.
    *   O transelevador retorna à posição de espera.

4.  **Controle de Movimento:**
    *   O movimento dos eixos Z e X/Y deve ser controlado para atingir as posições desejadas (Entrada, Saída, Prateleiras).
    *   Se a cena usar sensores digitais de fim de curso, o movimento é simples (ligar motor até sensor ativar).
    *   Se a cena usar sensores analógicos/encoders que fornecem posição numérica, a lógica precisará ler o valor atual, compará-lo com a posição alvo e comandar o motor (possivelmente com controle de velocidade simples ou apenas liga/desliga) até atingir a tolerância desejada.

5.  **Gerenciamento de Posições e Estados:**
    *   O sistema precisa manter um registro de quais localizações nas prateleiras estão ocupadas ou livres.
    *   A lógica sequencial para cada modo (Armazenamento/Recuperação) deve ser gerenciada por estados.

6.  **Intertravamentos:**
    *   Não mover o eixo X/Y se o eixo Z não estiver em uma altura segura (ex: totalmente elevado).
    *   Não acionar o garfo/plataforma (pegar/soltar) se o transelevador não estiver na posição correta (X, Y, Z).
    *   Não iniciar um novo ciclo se o anterior não estiver completo ou se houver falha.

**Desenvolvimento do Programa Ladder (WinSPS-S7):**

*   **Estruturação (Altamente Recomendado):**
    *   Use **Blocos de Função (FBs)** extensivamente.
        *   `FB_ControleEixo`: Um FB reutilizável para controlar o movimento de um eixo (Z ou X/Y) para uma posição alvo (recebe Posição Atual, Posição Alvo, comando Iniciar; retorna Em Movimento, Posição Atingida, Erro). Este FB encapsularia a lógica de ligar/desligar motores e verificar sensores/posição.
        *   `FB_GerenciadorASRS`: O FB principal que gerencia os modos (Armazenar/Recuperar), a sequência de operações, chama os FBs de controle de eixo e interage com o gerenciamento de localizações.
    *   Use **Blocos de Dados (DBs)**:
        *   `DB_Posicoes`: Um DB Global para armazenar as coordenadas (valores numéricos para Z, X/Y) de todas as posições relevantes (Entrada, Saída, cada localização da prateleira).
        *   `DB_StatusPrateleiras`: Um DB Global (ex: Array de BOOL) para armazenar o status de ocupação de cada localização.
        *   DBs de Instância para cada chamada dos `FB_ControleEixo` e para o `FB_GerenciadorASRS`.
*   **Lógica de Controle de Eixo (dentro do FB_ControleEixo):**
    *   Receber a `Posicao_Alvo`.
    *   Ler a `Posicao_Atual` (do sensor analógico/encoder ou dos sensores digitais de fim de curso).
    *   Comparar `Posicao_Atual` com `Posicao_Alvo`.
    *   Comandar os motores (Subir/Descer ou Avancar/Recuar) na direção correta.
    *   Parar o motor quando `Posicao_Atual` estiver dentro de uma tolerância da `Posicao_Alvo` (para controle analógico) ou quando o sensor de fim de curso for atingido (para controle digital).
    *   Sinalizar `Posicao_Atingida`.
*   **Lógica Sequencial (dentro do FB_GerenciadorASRS):**
    *   Implementar máquinas de estado separadas para os modos Armazenamento e Recuperação.
    *   Cada estado comandará uma ação (ex: "Mover Eixo Z para Posição X", "Ativar Garfo", "Mover Eixo X para Prateleira Y").
    *   As transições entre estados dependerão da sinalização de conclusão dos FBs de controle de eixo (`Posicao_Atingida`) ou de timers.
*   **Gerenciamento de Localizações:** Ler e escrever no `DB_StatusPrateleiras` para encontrar locais livres e marcar/desmarcar ocupação.
*   **Tags e Comentários:** Nomenclatura extremamente clara é vital devido à complexidade. Comentar cada estado, transição e bloco.

**Configuração da Cena no Factory IO:**

1.  Carregue a cena "Scene 10 - Automated Warehouse".
2.  Configure o driver de comunicação (provavelmente Modbus TCP/IP se usar posições analógicas, ou Digital I/O se apenas fim de curso).
3.  **Mapeamento de I/O Detalhado:**
    *   Mapeie todos os sensores de posição (digitais ou analógicos/numéricos) para as entradas correspondentes no CLP.
    *   Mapeie o sensor da esteira de entrada e saída.
    *   Mapeie todos os comandos de motor (Z Sobe/Desce, X Avança/Recua) para as saídas.
    *   Mapeie o atuador da garra/plataforma para uma saída.
    *   Mapeie os motores das esteiras de entrada e saída.

**Testes e Validação:**

1.  **Teste de Movimento dos Eixos:** Teste cada eixo individualmente. Comande o movimento para diferentes posições (Entrada, Saída, Prateleiras) e verifique se o `FB_ControleEixo` funciona corretamente, atingindo as posições e sinalizando a conclusão.
2.  **Teste Modo Armazenamento:**
    *   Selecione o modo Armazenamento.
    *   Envie caixas pela esteira de entrada.
    *   Verifique se cada caixa é pega, movida para uma prateleira livre (verifique a lógica de seleção de local livre) e depositada corretamente.
    *   Verifique se o `DB_StatusPrateleiras` é atualizado.
3.  **Teste Modo Recuperação:**
    *   Selecione o modo Recuperação.
    *   Comande a recuperação de uma caixa de uma prateleira específica (que você sabe que está ocupada).
    *   Verifique se o transelevador vai até a prateleira correta, pega a caixa, a leva para a esteira de saída e a deposita.
    *   Verifique se a Esteira de Saída é ativada.
    *   Verifique se o `DB_StatusPrateleiras` é atualizado (local fica livre).
4.  **Teste de Intertravamentos:** Tente criar condições que deveriam ser bloqueadas (ex: comandar movimento X enquanto Z não está no topo) e verifique se a lógica impede.
5.  **Monitoramento:** Monitore intensivamente os estados internos do `FB_GerenciadorASRS`, as posições atuais e alvo dos eixos, e o status das prateleiras no `DB_StatusPrateleiras`.

**Documentação Técnica (Entregável):**

1.  **Descrição Funcional Detalhada:** Explicação completa dos modos Armazenamento e Recuperação, controle de movimento e intertravamentos.
2.  **Fluxogramas:** Fluxogramas separados para a sequência lógica do modo Armazenamento e do modo Recuperação.
3.  **Lista de I/O Mapeada:** Tabela completa de mapeamento (Sensores, Atuadores, Motores -> Tags/Endereços CLP).
4.  **Mapa de Memória (DBs):** Descrição detalhada da estrutura dos DBs utilizados:
    *   `DB_Posicoes`: Lista de posições e suas coordenadas (Z, X/Y).
    *   `DB_StatusPrateleiras`: Como o status de cada prateleira é armazenado.
    *   Estrutura dos parâmetros e variáveis estáticas dos FBs (`FB_ControleEixo`, `FB_GerenciadorASRS`).
5.  **Código Ladder Comentado:** Código-fonte completo e bem comentado, destacando a lógica dos FBs e do OB1.
6.  **(Opcional) Vídeo Curto da Simulação:** Demonstração do sistema operando nos dois modos.

Este projeto final é um excelente exercício para simular sistemas robóticos e de logística automatizada, exigindo uma programação estruturada e um bom gerenciamento de dados e estados.
