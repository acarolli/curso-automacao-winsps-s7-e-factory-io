# Módulo 9: Projetos Finais Guiados

Chegamos ao ápice prático do nosso curso: os projetos finais. Nestes projetos, você terá a oportunidade de integrar todos os conhecimentos e habilidades adquiridos ao longo dos módulos anteriores – desde os fundamentos da lógica Ladder e o uso do WinSPS-S7 até a interação com o Factory IO e a aplicação de conceitos intermediários como modularização, temporizadores, contadores e tratamento de sinais. Os projetos são mais abrangentes que os exercícios anteriores e exigirão não apenas a programação da lógica de controle, mas também a compreensão do processo simulado e a elaboração de uma documentação técnica básica.

O objetivo destes projetos é simular, de forma simplificada, desafios reais de automação industrial, proporcionando uma experiência consolidada e preparando você para aplicar seus conhecimentos em situações práticas. Cada projeto será guiado, fornecendo os requisitos, a descrição funcional, sugestões para o desenvolvimento e os elementos essenciais da documentação.

**Projeto 1: Linha de Montagem Automatizada (Simplificada)**

*   **Descrição Geral:** Simular uma pequena estação de trabalho em uma linha de montagem onde peças passam por diferentes etapas sequenciais, como furação, inspeção (simulada) e movimentação para a próxima estação.
*   **Cena Factory IO:** Utilize a cena "Scene 11 - Assembly Station" ou construa uma cena similar que contenha:
    *   Uma esteira de entrada para trazer as peças base.
    *   Um sensor para detectar a peça na posição de trabalho.
    *   Um atuador de fixação (clamp) para segurar a peça.
    *   Um atuador vertical (ex: cilindro Z) com uma ferramenta simulada (ex: furadeira).
    *   Um sensor (simulado ou real) para indicar a conclusão da operação (ex: furação completa).
    *   Uma esteira de saída.
*   **Requisitos e Descrição Funcional:**
    1.  O sistema inicia com um botão de Partida geral e pode ser parado por um botão de Parada.
    2.  As esteiras de entrada e saída devem funcionar continuamente enquanto o sistema estiver ligado, a menos que uma peça esteja sendo processada na estação.
    3.  Quando uma peça base chega ao sensor de posição, a esteira de entrada para.
    4.  O atuador de fixação (clamp) é acionado para segurar a peça.
    5.  O atuador Z desce (simulando a aproximação da ferramenta).
    6.  A operação (ex: furação) é simulada por um tempo definido (usar temporizador).
    7.  Após o tempo, o atuador Z sobe.
    8.  O atuador de fixação solta a peça.
    9.  A esteira de entrada volta a funcionar para trazer a próxima peça (se houver) e a esteira de saída leva a peça processada.
    10. Implementar intertravamentos básicos (ex: não subir Z se clamp não estiver ativo, não soltar clamp se Z não estiver no topo).
    11. Incluir sinalização luminosa (verde para operando, vermelho para parado/falha básica).
*   **Desenvolvimento do Programa Ladder (WinSPS-S7):**
    *   **Estrutura:** Recomenda-se fortemente o uso de Funções (FCs) ou Blocos de Função (FBs) para modularizar o controle. Crie um FB para controlar a estação de montagem como um todo, encapsulando sua lógica sequencial e estados.
    *   **Lógica Sequencial:** Implemente a sequência descrita usando flags de estado (memórias internas, preferencialmente dentro do DB de instância do FB) e temporizadores. Use detecção de bordas para os sensores.
    *   **Controle das Esteiras:** Lógica simples de partida/parada intertravada com o estado da estação.
    *   **Sinalização:** Controle as luzes com base no estado geral do sistema (Ligado/Parado) e no estado da estação (Ociosa/Processando).
    *   **Tags:** Use nomes simbólicos claros para todas as entradas, saídas, memórias, timers e blocos.
*   **Configuração da Cena no Factory IO:**
    *   Selecione ou monte a cena com os componentes necessários.
    *   Configure o driver de comunicação (S7 ou Modbus).
    *   Mapeie cuidadosamente todos os sensores (posição, fim de curso Z se houver) e atuadores (motores das esteiras, clamp, atuador Z, luzes) às tags correspondentes no WinSPS-S7.
*   **Testes e Validação:**
    1.  Execute a simulação passo a passo, verificando cada etapa da sequência.
    2.  Teste os botões de Partida e Parada.
    3.  Verifique os intertravamentos (tente forçar condições que deveriam ser bloqueadas).
    4.  Observe a sinalização luminosa.
    5.  Monitore os estados internos (flags, timers) no WinSPS-S7 durante a execução.
*   **Documentação Técnica:**
    1.  **Fluxograma do Processo:** Um diagrama visual mostrando a sequência lógica das operações da estação.
    2.  **Lista de I/O Mapeada:** Tabela mostrando a correspondência entre os componentes do Factory IO (sensores/atuadores) e as tags/endereços do WinSPS-S7.
    3.  **Comentários no Código:** Comente extensivamente o programa Ladder, explicando cada rede, variável importante e a lógica geral das FCs/FBs.
    4.  **Descrição Funcional (Resumo):** Um breve texto descrevendo como o sistema deve operar.

**Projeto 2: Sistema de Armazenamento e Recuperação (AS/RS) Básico**

*   **Descrição Geral:** Simular um sistema automatizado que armazena caixas recebidas de uma esteira em prateleiras de um pequeno armazém e, posteriormente, recupera essas caixas sob comando (simulado) e as envia para uma esteira de saída.
*   **Cena Factory IO:** Utilize a cena "Scene 10 - Automated Warehouse" ou similar. Esta cena tipicamente inclui:
    *   Esteira de entrada.
    *   Sensor na posição de transferência da entrada.
    *   Um elevador (eixo Z) com uma plataforma ou garfo (eixo X ou Y) para pegar/depositar a caixa.
    *   Estrutura de prateleiras (racks) com posições definidas.
    *   Sensores de posição para os eixos Z e X/Y do elevador/garfo.
    *   Esteira de saída.
    *   Sensor na posição de transferência da saída.
*   **Requisitos e Descrição Funcional:**
    1.  O sistema tem dois modos principais: Armazenamento e Recuperação (selecionáveis por botões virtuais ou flags).
    2.  **Modo Armazenamento:**
        *   Quando uma caixa chega ao sensor da esteira de entrada, a esteira para.
        *   O elevador se move para a posição de entrada (X/Y e Z definidos).
        *   A plataforma/garfo pega a caixa.
        *   O sistema determina a próxima prateleira livre (pode ser uma lógica simples sequencial ou baseada em inputs virtuais).
        *   O elevador move a caixa para a posição X/Y e Z da prateleira alvo.
        *   A plataforma/garfo deposita a caixa.
        *   O elevador retorna à posição inicial ou de espera.
        *   A esteira de entrada volta a funcionar.
    3.  **Modo Recuperação:**
        *   Um comando virtual (input ou flag) especifica qual prateleira deve ser acessada.
        *   O elevador se move para a posição X/Y e Z da prateleira alvo.
        *   A plataforma/garfo pega a caixa.
        *   O elevador move a caixa para a posição de transferência da saída (X/Y e Z definidos).
        *   A plataforma/garfo deposita a caixa na esteira de saída.
        *   A esteira de saída é acionada para remover a caixa.
        *   O elevador retorna à posição inicial.
    4.  Implementar controle de movimento preciso usando os sensores de posição dos eixos.
    5.  Incluir intertravamentos (ex: não mover X/Y se Z não estiver na altura correta, não pegar/depositar se não estiver na posição correta).
*   **Desenvolvimento do Programa Ladder (WinSPS-S7):**
    *   **Estrutura:** Altamente recomendado usar FBs. Um FB principal para gerenciar os modos e a sequência geral, e talvez FBs separados para controlar o movimento de cada eixo (Z, X/Y) e a plataforma/garfo.
    *   **Gerenciamento de Posições:** Usar Blocos de Dados (DBs Globais) para armazenar as coordenadas (valores numéricos para os eixos) das posições de entrada, saída e de cada prateleira. Usar um DB ou flags para rastrear quais prateleiras estão ocupadas.
    *   **Controle de Movimento:** Ler os sensores de posição (podem ser digitais para posições fixas ou analógicos simulados para posicionamento variável). Usar instruções de comparação e MOVE para comandar os atuadores dos eixos até atingirem a posição desejada.
    *   **Lógica Sequencial:** Implementar as sequências de armazenamento e recuperação usando flags de estado/etapas dentro dos FBs.
    *   **Tags:** Nomenclatura clara e organizada, especialmente para as posições e estados.
*   **Configuração da Cena no Factory IO:**
    *   Selecione ou monte a cena.
    *   Configure o driver.
    *   Mapeie todos os sensores (posição da caixa, posições dos eixos) e atuadores (motores das esteiras, motores/atuadores dos eixos Z e X/Y, atuador da plataforma/garfo).
*   **Testes e Validação:**
    1.  Teste o modo de armazenamento: envie caixas e verifique se são armazenadas corretamente nas prateleiras (na sequência definida).
    2.  Teste o modo de recuperação: comande a recuperação de caixas de diferentes prateleiras e verifique se são entregues corretamente na esteira de saída.
    3.  Verifique o posicionamento preciso dos eixos.
    4.  Teste os intertravamentos.
    5.  Monitore as variáveis de posição, estado e ocupação das prateleiras no WinSPS-S7.
*   **Documentação Técnica:**
    1.  **Fluxogramas:** Um fluxograma para o modo de armazenamento e outro para o modo de recuperação.
    2.  **Lista de I/O Mapeada:** Tabela completa de mapeamento.
    3.  **Mapa de Memória (DBs):** Descrição da estrutura dos Blocos de Dados usados para posições e status das prateleiras.
    4.  **Comentários no Código:** Comentários detalhados na lógica Ladder, especialmente nas FCs/FBs de controle de movimento e sequência.
    5.  **Descrição Funcional (Resumo):** Texto descrevendo a operação em ambos os modos.

Estes projetos finais representam um desafio significativo, mas ao completá-los, você terá demonstrado uma capacidade sólida de analisar um problema de automação, projetar uma solução lógica, implementá-la em Ladder usando boas práticas de estruturação, integrá-la com um ambiente de simulação e documentar seu trabalho. Esta experiência é extremamente valiosa e forma a base para enfrentar desafios de automação no mundo real.
