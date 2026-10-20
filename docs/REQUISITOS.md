# PROCAPE – Saúde Digital: Bot de WhatsApp + Plataforma de Gestão

Levantamento de requisitos a partir da reunião entre Thompson Cysneiros e
Anna Beatriz Mariz Dias. Este documento é a referência para o desenvolvimento;
itens marcados com **[PENDENTE]** ainda precisam de decisão.

---

## 1. Visão geral

Dois componentes:

1. **Bot de WhatsApp (triagem, não tira dúvidas)** – envia mensagens de
   agendamento, coleta confirmação/remarcação/cancelamento, coleta dados
   cadastrais e encaminha o paciente para a equipe quando necessário.
   O bot **nunca envia informação sensível**.
2. **Plataforma de gestão (web)** – lista de pacientes com indicadores de
   status, cadastro de agendamentos, disparo em lote, fila de atendimento
   humano, chat com o paciente dentro do sistema e visualização de agenda.

Motivação: a equipe hoje é reduzida e em turnos alternados (Anna seg/qua/sex
manhã; outros atendentes à tarde em dias diferentes). A plataforma precisa
concentrar o trabalho e o histórico para que qualquer pessoa da equipe consiga
continuar o atendimento sem depender de um celular específico.

---

## 2. Perfis de paciente (linhas de cuidado)

| Linha de cuidado | Origem dos dados | Dados disponíveis no início |
|---|---|---|
| **Pré-cateterismo – Regulação** | Lista da regulação (~12 pacientes/semana) | Nome, telefone, Cartão SUS, nº da solicitação, data de nascimento |
| **Pré-cateterismo – Mutirão** | Lista de mutirão (pacientes entram na fila e são agendados quando ficam prontos) | Mesmos da regulação |
| **Pós-angioplastia** | Já são pacientes do PROCAPE | Dados completos do sistema (nome, prontuário etc.) |
| **Fibrilação atrial** (pesquisa) | Já fazem acompanhamento | Dados completos; contatos vão para a equipe da pesquisa |

- Pacientes da regulação/mutirão normalmente **ainda não têm prontuário**;
  os dados para abrir o prontuário são coletados pelo bot.
- O número do WhatsApp **não é divulgado**: o primeiro contato é sempre do
  PROCAPE. Ainda assim, o paciente pode voltar a escrever depois (ex.:
  complicação, agradecimento, sangramento), e esse contato espontâneo precisa
  ser tratado (ver 4.4).
- Um procedimento por vez por paciente.

---

## 3. Plataforma de gestão

### 3.1 Lista de pacientes
- Ordenação padrão: **não atendidos primeiro**, depois **mais antigos primeiro**
  (por data); atendidos ao final.
- Filtros: nome, prontuário, CPF/Cartão SUS, linha de cuidado, status
  (cores abaixo), "não atendidos", "solicitou cancelamento",
  "solicitou remarcação", "sem resposta".
- Indicador de status do agendamento mais recente:

| Cor | Significado |
|---|---|
| 🟢 Verde | Paciente **confirmou** |
| 🟡 Amarelo | Paciente **pediu remarcação** (vai para atendimento humano) |
| 🔴 Vermelho | Paciente **cancelou** |
| 🔵 Azul | Paciente **não respondeu** (ver 4.3) – separado da fila de urgência |
| ⚪ Cinza | Mensagem enviada, aguardando resposta |

### 3.2 Agendamentos (múltiplos por paciente)
- Cada paciente tem uma **lista de agendamentos**, sem limite. Botões
  **"Adicionar teleconsulta"** e **"Adicionar procedimento"**.
  - Ex.: teleconsulta pré-cateterismo → cateterismo → teleconsulta de
    acompanhamento → (complicação) novo procedimento.
- Campo de procedimento é opcional (há teleconsultas sem procedimento depois).
- Ao **salvar**, o bot envia automaticamente a mensagem ao paciente
  ("Sua teleconsulta/seu procedimento foi agendado para DD/MM às HH:MM") com
  as opções **Confirmar / Solicitar remarcação / Cancelar**.
- Como as opções vêm na própria mensagem do agendamento, o paciente **não
  precisa escolher qual consulta** está confirmando/cancelando.

### 3.3 Disparo em lote (pré-cateterismo)
- Importar/cadastrar a lista da semana (substitui a planilha manual).
- Selecionar grupos de pacientes e enviar a mesma mensagem para todos de uma
  vez (ex.: "estes 3 para o dia X, estes 3 para o dia Y").
- Regra de data usada hoje: teleconsulta ≈ 7 dias antes do cateterismo
  (cateterismo dia 26 → teleconsulta dia 19). A equipe define **dia e hora**
  da teleconsulta; o paciente só confirma ou pede remarcação.
- **[PENDENTE]** Oferecer horários fixos (ex.: 14h, 14h30, 15h, 15h30) para o
  paciente escolher – ficou como "plus" para depois. Decisão atual: a equipe
  define o horário.

### 3.4 Fila de atendimento humano + notificações
- Quando o paciente pede para falar com a equipe (ou pede remarcação), o
  sistema **notifica** a equipe e o paciente aparece destacado na fila.
- Cada item da fila mostra: paciente, linha de cuidado, motivo, **queixa**
  (texto livre) e quando foi aberto – para a equipe priorizar.
- Fibrilação atrial vai para a fila da **equipe de pesquisa**.

### 3.5 Chat dentro do sistema
- A equipe conversa com o paciente **pela plataforma**, não pelo celular.
- Histórico completo da conversa (bot + equipe) fica salvo e visível.
- Vários usuários podem acompanhar (ex.: coordenação vê os agendamentos
  enquanto a atendente conversa).

### 3.6 Agenda
- Visualização em calendário dos agendamentos confirmados
  (ex.: "Thompson – teleconsulta – dia 26").

### 3.7 Usuários
- Login individual; perfis sugeridos: coordenação, atendente, pesquisa.

---

## 4. Fluxos do bot

### 4.1 Agendamento disparado pela equipe
1. Equipe salva agendamento → bot envia mensagem com Confirmar / Remarcar /
   Cancelar.
2. **Confirmar** → status verde, entra na agenda.
3. **Remarcar** → status amarelo e **encaminha direto para atendimento humano**
   (evita o vai-e-vem de várias datas).
4. **Cancelar** → status vermelho.

### 4.2 Pré-cateterismo (regulação/mutirão) – primeiro contato
1. Bot envia a mensagem padrão ("Você foi regulado para fazer um cateterismo
   no PROCAPE…").
2. Paciente responde → bot solicita os **exames** (anexos) – opcional.
3. Bot coleta os **dados para abertura do prontuário** (endereço etc.).
   **[PENDENTE]** lista exata de campos.
4. Equipe (ou regra automática) envia data/hora da teleconsulta → fluxo 4.1.
- Este primeiro contato **não tem trava de tempo** (o paciente pode responder
  só no dia seguinte).

### 4.3 Paciente que não responde
- Dia 1: mensagem inicial.
- Dia 2 (após 24h sem resposta): lembrete automático ("Oi Fulano, lembrando…").
- Dia 3 (mais 24h sem resposta): marcado como **sem resposta (azul)** para a
  equipe ligar. Não entra na fila de urgência do atendimento humano.

### 4.4 Contato espontâneo do paciente ("Oi")
1. Identificação: **CPF + data de nascimento**.
   - **[PENDENTE]** pacientes da regulação nem sempre têm CPF cadastrado –
     aceitar Cartão SUS como alternativa?
2. Se o telefone for diferente do cadastrado: perguntar se deseja atualizar o
   número; a confirmação de identidade é feita pela **equipe** (pede foto de
   documento). O bot não passa nenhuma informação sensível.
3. Bot pergunta a linha de cuidado (pré-cateterismo, pós-angioplastia,
   fibrilação atrial, outro).
4. Bot pergunta a necessidade: **falar com a equipe**, **solicitar
   teleconsulta**, **solicitar atendimento presencial**.
5. Bot pede a **queixa** (texto livre, ex.: sangramento) – base para triagem e
   para uma futura triagem com IA.
6. Abre item na fila de atendimento humano e notifica a equipe.

### 4.5 Travas de tempo da conversa
| Trava | Regra |
|---|---|
| Inatividade com o bot | Paciente parado por X min → bot encerra a conversa. **[PENDENTE]** valor de X (sugestão: 15 min) |
| Atendimento humano | Assim que a equipe fala, **o bot para de responder**. Após 20–30 min sem movimentação, a conversa é encerrada automaticamente |
| Retorno | Ao voltar a falar após encerramento, o paciente passa de novo pela verificação |
| Exceção | Mensagens ativas de primeiro contato/agendamento **não** expiram |

### 4.6 Lembretes
- Lembrete antes da consulta/procedimento (hoje feito pelo próprio WhatsApp;
  passa a ser feito pelo bot). **[PENDENTE]** antecedência (sugestão: 24h).

---

## 5. Requisitos não funcionais
- **LGPD**: dados de saúde são sensíveis; bot não expõe dados; acesso à
  plataforma só com login; registro (log) das conversas.
- **Hospedagem**: VPS própria do hospital/projeto.
  - **[PENDENTE]** contratação da VPS (cadastro em andamento com os dados da
    Mari; alternativa: outra pessoa ou via TI).
  - Plano B para testes: VPS da empresa do Thompson **somente com dados
    fictícios** e um número de WhatsApp de teste.
- **Prazo**: ~20 dias – ~10 dias de desenvolvimento + ~10 dias de testes e
  correções.

---

## 6. Modelo de dados (rascunho)

Implementado como mock em `db/seeds.json` (json-server) com **dados fictícios**.

- `pacientes` – nome, cpf, cartaoSus, dataNascimento, telefone, prontuario,
  numeroSolicitacao, linhaCuidado, statusCadastro, endereco.
- `agendamentos` – pacienteId, tipo (`teleconsulta` | `procedimento`),
  descricao, dataHora, status (`aguardando` | `confirmado` |
  `remarcacao_solicitada` | `cancelado` | `sem_resposta` | `realizado`),
  lembretesEnviados, ultimoEnvioEm, respondidoEm.
- `campanhas` – disparo em lote: linhaCuidado, data do procedimento, data/hora
  da teleconsulta, pacienteIds, template.
- `conversas` – pacienteId, estado (`bot` | `aguardando_humano` |
  `em_atendimento` | `encerrada`), expira (bool), ultimaAtividadeEm,
  atendenteId.
- `mensagens` – conversaId, autor (`bot` | `paciente` | `atendente`), texto,
  anexo, enviadaEm.
- `solicitacoes` – fila humana: pacienteId, conversaId, tipo, queixa,
  equipe (`assistencial` | `pesquisa`), status.
- `usuarios` – nome, perfil.
- `templates` – textos padrão das mensagens.
- `configuracoes` – tempos das travas e dos lembretes.

---

## 7. Fases sugeridas
1. **MVP (prioridade)**: cadastro/importação de pacientes, agendamentos
   múltiplos, envio automático com Confirmar/Remarcar/Cancelar, lista com
   cores e filtros, disparo em lote, lembrete D+1 e marcação "sem resposta".
2. **Atendimento**: fila humana com notificação, chat na plataforma, travas
   de tempo, contato espontâneo com verificação.
3. **Gestão**: agenda em calendário, perfis de usuário, coleta de dados para
   prontuário e exames.
4. **Futuro**: escolha de horário pelo paciente, triagem por IA da queixa.

---

## 8. Ações combinadas
- **Thompson**: tentar novamente o cadastro da VPS; iniciar desenvolvimento.
- **Anna**: acompanhar a Mari sobre a VPS (ou via TI); revisar esta
  transcrição/requisitos e apontar pendências; enviar o modelo de mensagem
  padrão e a lista de dados para abertura de prontuário.
- Verificar se ainda há acesso ao site feito anteriormente pela Flávia.
