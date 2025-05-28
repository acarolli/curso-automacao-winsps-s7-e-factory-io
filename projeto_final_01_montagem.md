# Projeto Final Guiado 1: Linha de Montagem Automatizada (Simplificada)

**Introdução:**
Este projeto final visa consolidar os conhecimentos adquiridos ao longo do curso, aplicando-os na simulação de uma estação de trabalho automatizada. Você desenvolverá a lógica de controle para uma pequena linha que realiza operações sequenciais em peças, integrando sensores, atuadores, temporizadores e lógica de intertravamento, além de praticar a estruturação modular do programa e a documentação técnica.

**Descrição Geral:**
Simular uma estação onde peças base chegam por uma esteira, são fixadas, passam por uma operação simulada (ex: furação) e são liberadas para uma esteira de saída. O sistema deve operar de forma automática após um comando de partida e incluir sinalização básica.

**Cena Factory IO:**
*   **Recomendada:** Utilize a cena pré-definida **"Scene 11 - Assembly Station"** ou uma similar.
*   **Componentes Essenciais (verificar ou adicionar):**
    *   Esteira de Entrada (`Entry Conveyor`)
    *   Sensor de Posição (`Diffuse Sensor` ou similar na posição de trabalho)
    *   Atuador de Fixação (`Clamp`)
    *   Atuador Vertical (`Elevator` ou `Pneumatic Cylinder` Z)
    *   (Opcional) Sensor de Operação Concluída (pode ser simulado por timer)
    *   Esteira de Saída (`Exit Conveyor`)
    *   Botões virtuais ou físicos (se disponíveis na cena) para Partida/Parada.
    *   Sinalizador Luminoso (`Stack Light` com luzes Verde e Vermelha).

**Requisitos e Descrição Funcional Detalhada:**

1.  **Inicialização e Controle Geral:**
    *   O sistema deve possuir um estado "Parado" e um estado "Operando".
    *   Um botão (virtual M0.0 ou físico I0.x) de "Partida" deve transicionar o sistema de "Parado" para "Operando".
    *   Um botão (virtual M0.1 ou físico I0.y) de "Parada" deve transicionar o sistema de "Operando" para "Parado" a qualquer momento, de forma segura (finalizando o ciclo atual se possível, ou parando imediatamente).
    *   A luz Vermelha do `Stack Light` deve acender quando o sistema estiver "Parado".
    *   A luz Verde do `Stack Light` deve acender quando o sistema estiver "Operando" e pronto para receber peças (ocioso).
    *   (Opcional) A luz Amarela pode piscar durante o processamento da peça.

2.  **Operação das Esteiras:**
    *   A Esteira de Entrada e a Esteira de Saída devem funcionar continuamente *apenas* quando o sistema estiver no estado "Operando" e a estação de trabalho estiver ociosa (não processando uma peça).

3.  **Ciclo de Processamento da Peça:**
    *   Quando o sistema está "Operando" e o `Sensor de Posição` detecta a chegada de uma peça (borda de subida):
        a.  A Esteira de Entrada deve parar imediatamente.
        b.  O `Atuador de Fixação` (Clamp) deve ser acionado para prender a peça.
        c.  Após uma pequena confirmação (timer curto ou sensor de clamp ativo, se houver), o `Atuador Vertical` (Z) deve descer.
        d.  A "operação" (furação) é simulada mantendo o Atuador Z embaixo por um tempo definido (ex: 3 segundos, usar Timer TON).
        e.  Após o tempo da operação, o `Atuador Vertical` (Z) deve subir.
        f.  Após Z atingir a posição superior (confirmado por sensor ou timer), o `Atuador de Fixação` (Clamp) deve ser desativado (soltar a peça).
        g.  Após o clamp soltar (confirmado por timer curto ou sensor), a Esteira de Entrada deve voltar a funcionar para buscar a próxima peça, e a Esteira de Saída (que já deveria estar funcionando) leva a peça processada.
        h.  A estação retorna ao estado ocioso, pronta para a próxima peça.

4.  **Intertravamentos e Segurança:**
    *   O Atuador Z não deve descer se o Clamp não estiver ativo.
    *   O Clamp não deve soltar se o Atuador Z não estiver na posição superior.
    *   A Esteira de Entrada não deve funcionar se o Clamp estiver ativo ou se o Atuador Z não estiver na posição superior.
    *   A lógica de Parada deve garantir que os atuadores parem em uma posição segura.

**Desenvolvimento do Programa Ladder (WinSPS-S7):**

*   **Estruturação (Recomendado):**
    *   Crie um **Bloco de Função (FB)** chamado `FB_EstacaoMontagem`. Este FB encapsulará toda a lógica sequencial da estação.
    *   Crie um **Bloco de Dados de Instância (DB)** para este FB (ex: `DB_Instancia_EstacaoMontagem`).
    *   Dentro do FB, use variáveis `STAT` (estáticas) para armazenar o estado atual da sequência (ex: Ocioso, Fixando, DescendoZ, Operando, SubindoZ, Soltando).
    *   Use parâmetros `IN`, `OUT` e `IN_OUT` no FB para conectar os sinais físicos (sensores, atuadores) e comandos (Partida/Parada) ao bloco.
    *   No **OB1**, crie a lógica de Partida/Parada geral e chame a instância do `FB_EstacaoMontagem`, passando os parâmetros necessários.
*   **Lógica Sequencial:** Implemente a máquina de estados dentro do FB usando comparações com a variável de estado e condições (sensores, timers) para transicionar entre os estados.
*   **Temporizadores:** Utilize timers TON para os atrasos e durações necessárias (confirmação clamp, tempo operação, confirmação soltura).
*   **Sinalização:** Controle as saídas das luzes (Qx.x) com base no estado geral (Operando/Parado) e no estado interno do FB (Ocioso/Processando).
*   **Tags e Comentários:** Use nomes simbólicos claros e comente abundantemente a lógica, especialmente dentro do FB, explicando cada estado e transição.

**Configuração da Cena no Factory IO:**

1.  Carregue a cena "Scene 11 - Assembly Station" ou a que você montou.
2.  Configure o driver de comunicação (S7 ou Modbus) para conectar ao WinSPS-S7.
3.  **Mapeamento de I/O Cuidadoso:**
    *   Mapeie o `Sensor de Posição` para a entrada correspondente no CLP (parâmetro IN do FB).
    *   Mapeie os sensores de fim de curso do Atuador Z (Alto/Baixo), se existirem, para entradas.
    *   Mapeie os botões de Partida/Parada (se físicos) para entradas.
    *   Mapeie o motor da `Entry Conveyor` para a saída correspondente (parâmetro OUT do FB).
    *   Mapeie o motor da `Exit Conveyor` para a saída correspondente.
    *   Mapeie o atuador `Clamp` para a saída correspondente.
    *   Mapeie os comandos do `Atuador Vertical` (Subir/Descer) para as saídas correspondentes.
    *   Mapeie as luzes Verde e Vermelha do `Stack Light` para as saídas correspondentes.

**Testes e Validação:**

1.  **Teste Funcional:**
    *   Inicie o sistema com o botão Partida. Verifique se a Luz Verde acende e as esteiras (entrada/saída) começam a funcionar.
    *   Envie uma peça. Verifique se a sequência de fixação, descida Z, operação (pausa), subida Z e soltura ocorre conforme descrito.
    *   Verifique se a Esteira de Entrada para durante o processamento e reinicia após a soltura.
    *   Teste o botão Parada durante diferentes fases do ciclo e verifique se o sistema para de forma segura.
2.  **Teste de Intertravamentos:**
    *   Tente forçar condições (no WinSPS-S7 ou manipulando a cena) que violem os intertravamentos (ex: tentar soltar o clamp com Z embaixo) e verifique se a lógica impede a ação.
3.  **Monitoramento:** Use o monitoramento online do WinSPS-S7 para acompanhar a variável de estado dentro do FB, os timers e o estado das I/Os durante a execução.

**Documentação Técnica (Entregável):**

1.  **Descrição Funcional Detalhada:** Um texto claro explicando todos os modos de operação, a sequência do ciclo e os intertravamentos (similar ao descrito acima).
2.  **Fluxograma do Ciclo de Processamento:** Um diagrama visual (feito em ferramenta de desenho ou mesmo texto estruturado) mostrando os estados da estação (Ocioso, Fixando, DescendoZ, etc.) e as transições entre eles com as condições (sensores/timers).
3.  **Lista de I/O Mapeada:** Uma tabela organizada listando:
    *   Nome do Componente no Factory IO
    *   Tipo (Sensor/Atuador)
    *   Tag Simbólica no WinSPS-S7
    *   Endereço Físico (I/Q) ou Endereço Modbus
    *   Descrição da Função
4.  **Código Ladder Comentado:** O código-fonte do programa no WinSPS-S7 (exportado ou capturas de tela legíveis), com comentários claros em cada rede e na definição das variáveis e blocos (FC/FB).
5.  **(Opcional) Vídeo Curto da Simulação:** Uma gravação de tela mostrando o sistema funcionando corretamente no Factory IO.

Este projeto oferece uma excelente oportunidade para praticar a programação estruturada e abordar um cenário de automação sequencial realista.
