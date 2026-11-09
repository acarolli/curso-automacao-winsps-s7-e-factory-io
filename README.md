# Curso prático de automação industrial: Ladder com WinSPS-S7 e Factory IO

Aprenda a programar CLPs Siemens S7 em **Ladder** sem precisar de um CLP físico: o programa roda no simulador **WinSPS-S7** e controla plantas industriais em 3D no **Factory IO**. São 10 módulos, 10 exercícios progressivos e 2 projetos finais documentados.

- **Para quem:** estudantes de engenharia, técnicos e profissionais da indústria
- **Nível:** iniciante a intermediário
- **Você vai precisar de:** WinSPS-S7 e Factory IO instalados (as integrações usam o driver S7-PLCSIM ou Modbus TCP)

📘 **Material completo em um arquivo:** [curso_completo.md](curso_completo.md) · 🗺️ **Plano do curso:** [estrutura_curso.md](estrutura_curso.md)

## Módulos

| # | Módulo | Conteúdo |
|---|---|---|
| 1 | [Introdução à automação industrial e CLPs](modulo_01_rascunho.md) | Arquitetura do CLP, famílias S7-300 e S7-1200, sensores, atuadores e IEC 61131-3 |
| 2 | [Fundamentos da linguagem Ladder](modulo_02_rascunho.md) | Contatos NA/NF, lógicas AND/OR/NOT, Set/Reset, temporizadores, contadores e comparações |
| 3 | [Ambiente WinSPS-S7](modulo_03_rascunho.md) | Projeto, tabela de símbolos, compilação, download, monitoramento e forçamento |
| 4 | [Simulador Factory IO](modulo_04_rascunho.md) | Cenas, peças, sensores, atuadores e drivers de comunicação |
| 5 | [Integração WinSPS-S7 + Factory IO](modulo_05_rascunho.md) | S7-PLCSIM e Modbus TCP, mapeamento de I/O e diagnóstico da conexão |
| 6 | [Exercícios guiados: nível iniciante](modulo_06_rascunho.md) | Exercícios 1 a 5 |
| 7 | [Ladder intermediário](modulo_07_rascunho.md) | Sinais analógicos e escalonamento, MOVE, FCs, FBs, DBs, bordas e intertravamentos |
| 8 | [Exercícios guiados: nível intermediário](modulo_08_rascunho.md) | Exercícios 6 a 10 |
| 9 | [Projetos finais guiados](modulo_09_rascunho.md) | Linha de montagem e armazém automático (AS/RS) |
| 10 | [Boas práticas e próximos passos](modulo_10_rascunho.md) | Padronização, troubleshooting, tabelas de observação e o que estudar depois |

## Exercícios práticos

| # | Exercício | O que treina |
|---|---|---|
| 1 | [Partida direta de motor](exercicio_01_partida_direta.md) | Lógica de selo (liga/desliga) |
| 2 | [Controle de nível simples](exercicio_02_controle_nivel.md) | Sensor de nível e válvula |
| 3 | [Semáforo de processo](exercicio_03_semaforo.md) | Temporizadores em sequência |
| 4 | [Contagem de caixas](exercicio_04_contagem_caixas.md) | Contadores e detecção de borda |
| 5 | [Acionamento sequencial](exercicio_05_acionamento_sequencial.md) | Sequência de cilindros com flags de estado e temporizadores |
| 6 | [Separação de peças por altura](exercicio_06_separacao_pecas.md) | Sensores combinados e desviadores |
| 7 | [Múltiplas esteiras](exercicio_07_multiplas_esteiras.md) | Transferência e intertravamento entre esteiras |
| 8 | [Envase simples](exercicio_08_envase_simples.md) | Posicionamento e enchimento temporizado |
| 9 | [Nível com setpoint analógico](exercicio_09_controle_nivel_analogico.md) | Controle liga/desliga entre dois setpoints |
| 10 | [Paletizador básico](exercicio_10_paletizador_basico.md) | Movimentos coordenados em sequência |

## Projetos finais

- 🏭 [**Linha de montagem automatizada**](projeto_final_01_montagem.md): etapas sequenciais em um produto, com programa estruturado em FCs/FBs; o entregável inclui lista de I/O e fluxograma do ciclo.
- 📦 [**Armazém automático (AS/RS)**](projeto_final_02_asrs.md): armazenar e retirar caixas de prateleiras com um transelevador.

Cada projeto traz requisitos, descrição funcional, programa Ladder, configuração da cena, roteiro de testes e documentação.

## Autor

**Ayslan Carolli** · engenheiro eletricista e técnico em automação industrial · [LinkedIn](https://www.linkedin.com/in/ayslan-carolli) · [GitHub](https://github.com/acarolli)
