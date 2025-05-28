# Estrutura do Curso: Automação Industrial com Ladder, WinSPS-S7 e Factory IO

**Título:** Curso Prático de Automação Industrial: Ladder com WinSPS-S7 e Factory IO

**Público-Alvo:** Estudantes de engenharia, técnicos e profissionais da indústria.

**Nível:** Iniciante a Intermediário

**Objetivo:** Capacitar o aluno a programar CLPs Siemens (simulados via WinSPS-S7) em linguagem Ladder e integrar com o simulador Factory IO para criar e testar soluções de automação industrial, mesmo sem acesso a um CLP físico.

## Módulos do Curso

### Módulo 1: Introdução à Automação Industrial e CLPs

*   **Tópicos:**
    *   O que é Automação Industrial? Conceitos Fundamentais e Aplicações Típicas.
    *   Introdução aos Controladores Lógicos Programáveis (CLPs): História, Arquitetura (CPU, Memória, I/O) e Princípios de Funcionamento.
    *   Visão Geral dos CLPs Siemens: Foco nas famílias S7-300 e S7-1200, suas características e diferenças relevantes para a simulação.
    *   Componentes de Hardware Essenciais: Módulos de Entradas e Saídas (Digitais e Analógicas), Fontes de Alimentação.
    *   Sensores Industriais Comuns (Proximidade, Fotoelétricos, Nível, etc.) e Atuadores (Motores, Válvulas, Cilindros) - Conceitos e simulação no Factory IO.
    *   Linguagens de Programação para CLPs (IEC 61131-3): Visão Geral com Ênfase na Linguagem Ladder (LD).

### Módulo 2: Fundamentos da Linguagem Ladder

*   **Tópicos:**
    *   Lógica de Contatos: Representação de Circuitos Elétricos, Contatos Normalmente Abertos (NA) e Normalmente Fechados (NF), Bobinas Simples.
    *   Instruções Lógicas Fundamentais: Lógicas AND, OR, NOT e suas combinações.
    *   Instruções de Saída: Bobinas Set (Travamento) e Reset (Destravamento).
    *   Temporizadores (Timers): Tipos (TON - On-Delay, TOF - Off-Delay, TP - Pulse) e Aplicações Práticas (Ex: Atraso no acionamento, tempo de permanência).
    *   Contadores (Counters): Tipos (CTU - Up Counter, CTD - Down Counter, CTUD - Up/Down Counter) e Aplicações (Ex: Contagem de peças, controle de ciclos).
    *   Instruções de Comparação (CMP): Igualdade, Diferença, Maior que, Menor que, etc.
    *   Operações Matemáticas Básicas: Adição, Subtração, Multiplicação, Divisão (Conforme disponibilidade no WinSPS-S7).
    *   Estrutura Básica de um Programa Ladder: Redes (Rungs), Comentários.
    *   Introdução aos Blocos de Organização (OBs), Funções (FCs) e Blocos de Função (FBs) - Conceito inicial.
    *   Boas Práticas Iniciais: Nomenclatura de Tags, Comentários Essenciais.

### Módulo 3: Introdução ao Ambiente WinSPS-S7

*   **Tópicos:**
    *   Apresentação do WinSPS-S7: Um Simulador de CLP Siemens para Aprendizagem.
    *   Instalação e Configuração Inicial do Software.
    *   Explorando a Interface Gráfica: Editor Ladder, Tabela de Símbolos (Symbol Table), Janela de Status/Monitoramento.
    *   Criação de um Novo Projeto: Seleção de CPU (Simulada), Configuração Básica.
    *   Programando as Primeiras Instruções Ladder no Editor.
    *   Compilação e Verificação de Erros.
    *   Download do Programa para o CLP Simulado.
    *   Modo de Monitoramento (Online): Visualização do Estado das Variáveis e Lógica em Tempo Real.
    *   Ferramentas Básicas de Debugging: Forçar (Force) Entradas/Saídas.

### Módulo 4: Introdução ao Simulador Factory IO

*   **Tópicos:**
    *   Apresentação do Factory IO: Simulador 3D Interativo de Processos Industriais.
    *   Instalação e Requisitos do Sistema.
    *   Navegando pela Interface: Menu Principal, Seleção de Cenas (Scenes), Biblioteca de Peças (Parts Palette).
    *   Interação no Ambiente 3D: Movimentação da Câmera, Manipulação de Objetos.
    *   Explorando as Cenas Pré-definidas: Exemplos de Aplicações Industriais.
    *   Tipos de Drivers de Comunicação: Visão Geral com Foco nos Drivers S7-PLCSIM (Advanced) e Modbus TCP/IP Client/Server.
    *   Identificação de Sensores e Atuadores nas Cenas do Factory IO.

### Módulo 5: Integração Prática: WinSPS-S7 e Factory IO

*   **Tópicos:**
    *   Visão Geral das Opções de Conectividade entre WinSPS-S7 e Factory IO.
    *   Método 1: Conexão via Driver S7-PLCSIM (ou compatível): Configuração no Factory IO e WinSPS-S7.
    *   Método 2: Conexão via Driver Modbus TCP/IP: Configuração do Servidor Modbus no WinSPS-S7 (se suportado) ou via software intermediário, e configuração do Cliente Modbus no Factory IO.
    *   Mapeamento de Entradas e Saídas (I/O Mapping): Associando Tags do WinSPS-S7 aos Sensores e Atuadores no Factory IO.
    *   Passo a Passo Detalhado: Criar um programa Ladder simples (Ex: Ligar/Desligar esteira com botão) e conectá-lo a uma cena correspondente no Factory IO.
    *   Testes de Comunicação: Verificando se os sinais são trocados corretamente entre os softwares.
    *   Diagnóstico e Solução de Problemas Comuns na Integração (Ex: Falha na conexão, mapeamento incorreto).

### Módulo 6: Exercícios Práticos Guiados (Nível Iniciante)

*   **Foco:** Aplicar os conceitos básicos de Ladder e a integração com Factory IO.
*   **Exercícios:**
    1.  **Partida Direta de Motor:** Ligar e desligar uma esteira transportadora usando botões.
    2.  **Controle de Nível Simples:** Encher um tanque até um nível detectado por um sensor.
    3.  **Semáforo de Processo:** Controlar luzes indicadoras (Verde/Amarelo/Vermelho) com temporizadores.
    4.  **Contagem de Caixas:** Usar um sensor para contar caixas que passam por uma esteira.
    5.  **Acionamento Sequencial Básico:** Ativar atuadores em uma ordem pré-definida.

### Módulo 7: Tópicos Intermediários em Ladder e Automação

*   **Tópicos:**
    *   Trabalhando com Entradas e Saídas Analógicas (Simuladas): Conceito de Escalonamento (Scaling), Instruções para Leitura/Escrita Analógica (se disponíveis no WinSPS-S7).
    *   Manipulação de Dados: Instrução MOVE para transferir valores entre registradores/memórias.
    *   Estruturação Avançada de Programas: Criação e Chamada de Funções (FCs) e Blocos de Função (FBs) para reutilização de código e organização.
    *   Introdução aos Blocos de Dados (DBs): Armazenamento de dados de forma organizada.
    *   Detecção de Bordas (Rising/Falling Edge): Instruções P_TRIG e N_TRIG (ou equivalentes).
    *   Tratamento Básico de Falhas e Intertravamentos.
    *   Dicas de Otimização: Simplificação da lógica, uso eficiente de memória.

### Módulo 8: Exercícios Práticos Guiados (Nível Intermediário)

*   **Foco:** Aplicar conceitos intermediários e resolver problemas mais complexos.
*   **Exercícios:**
    6.  **Separação de Peças:** Classificar peças por altura ou cor usando sensores e desviadores.
    7.  **Linha de Produção com Múltiplas Esteiras:** Controlar a transferência de itens entre diferentes esteiras.
    8.  **Sistema de Envase Simples:** Controlar o enchimento de recipientes em uma esteira.
    9.  **Controle de Nível com Setpoint Analógico:** Manter o nível de um tanque dentro de uma faixa definida (simulando I/O analógico).
    10. **Paletizador Básico:** Empilhar caixas em um palete de forma automatizada (movimentos básicos).

### Módulo 9: Projetos Finais Guiados

*   **Foco:** Integrar múltiplos conhecimentos em projetos mais completos, incluindo documentação.
*   **Projetos:**
    1.  **Linha de Montagem Automatizada (Simplificada):** Simular uma pequena linha que executa algumas etapas sequenciais em um produto (Ex: furar, inspecionar, mover).
        *   Requisitos e Descrição Funcional.
        *   Desenvolvimento do Programa Ladder (com FCs/FBs).
        *   Configuração da Cena no Factory IO.
        *   Testes e Validação.
        *   Documentação: Fluxograma do processo, Lista de I/O mapeada, Comentários no código.
    2.  **Sistema de Armazenamento e Recuperação (AS/RS) Básico:** Simular um sistema que armazena e retira caixas de prateleiras.
        *   Requisitos e Descrição Funcional.
        *   Desenvolvimento do Programa Ladder.
        *   Configuração da Cena no Factory IO.
        *   Testes e Validação.
        *   Documentação: Fluxograma, Lista de I/O, Descrição da lógica.

### Módulo 10: Boas Práticas, Troubleshooting e Próximos Passos

*   **Tópicos:**
    *   Revisão e Consolidação de Boas Práticas de Programação Ladder: Padronização, Comentários detalhados, Estrutura modular, Nomenclatura significativa.
    *   Técnicas Avançadas de Debugging no WinSPS-S7 e Factory IO: Análise de lógicas complexas, uso de tabelas de observação (Watch Table).
    *   Dicas Finais para Otimização e Confiabilidade dos Sistemas Automatizados.
    *   Recursos Adicionais: Fóruns, Documentação Oficial (Siemens), Comunidades Online.
    *   Sugestões para Continuação dos Estudos: Outras linguagens (SCL, FBD), Redes Industriais (Profinet, Profibus), Interfaces Homem-Máquina (IHMs), CLPs de outros fabricantes.

## Recursos Adicionais a serem Incluídos

*   Imagens e Capturas de Tela: Interface do WinSPS-S7, Interface do Factory IO, Exemplos de Diagramas Ladder, Componentes Físicos (ilustrativo).
*   Fluxogramas: Para explicar a lógica de controle dos exercícios e projetos mais complexos.
*   Descrições Detalhadas das Simulações: Como cada exercício/projeto deve se comportar no Factory IO.
*   Checklists: Para configuração da comunicação e passos de debugging.
*   Glossário de Termos Técnicos.

