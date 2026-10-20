# PROCAPE – Saúde Digital: Bot de WhatsApp + Plataforma de Gestão

Levantamento de requisitos a partir da reunião entre Thompson Cysneiros e
Anna Beatriz Mariz Dias, complementado pela especificação funcional do bot
enviada depois. Este documento é a referência para o desenvolvimento; itens
marcados com **[PENDENTE]** ainda precisam de decisão.

Documentos relacionados:
- [`BANCO_DE_DADOS.md`](BANCO_DE_DADOS.md) – estrutura proposta do banco.
- [`ARQUITETURA.md`](ARQUITETURA.md) – stack, módulos e estrutura de código.
- [`MENSAGENS.md`](MENSAGENS.md) – catálogo dos textos do bot, gatilhos e
  ajustes sugeridos.

---

## 1. Objetivo

1. **Bot de WhatsApp (primeiro atendimento / triagem)** – identifica o
   paciente e a demanda, coleta as informações necessárias, orienta, registra
   tudo e encaminha para a equipe quando necessário. Não é um bot de dúvidas
   gerais e **nunca envia informação sensível**.
2. **Plataforma de gestão (web)** – lista de pacientes com indicadores de
   status, agendamentos, disparo em lote, fila de atendimento humano, chat com
   o paciente dentro do sistema, agenda e histórico resumido de cada
   atendimento.

Motivação: a equipe é reduzida e trabalha em turnos alternados (Anna
seg/qua/sex manhã; outros atendentes à tarde em dias diferentes). A
plataforma concentra o trabalho e o histórico para que qualquer pessoa da
equipe continue o atendimento sem depender de um celular específico e sem
precisar ler toda a conversa.

---

## 2. Serviços / linhas de cuidado

| Serviço | Origem típica | Dados disponíveis no início | Equipe |
|---|---|---|---|
| **Pré-cateterismo – Regulação** | Lista da regulação (~12/semana) | Nome, telefone, Cartão SUS, nº da solicitação, data de nascimento | Saúde Digital |
| **Pré-cateterismo – Mutirão** | Lista de mutirão | Mesmos da regulação | Saúde Digital |
| **Pós-angioplastia** | Já são pacientes do PROCAPE | Dados completos + prontuário | Saúde Digital |
| **Fibrilação atrial** | Pacientes em acompanhamento (pesquisa) | Dados completos + prontuário | Pesquisa FA |
| **Aconselhamento genético** | **[PENDENTE]** origem e dados | – | **[PENDENTE]** |
| **Outros** | Contato espontâneo | – | Saúde Digital (triagem) |

- A lista de serviços deve ser **cadastrável** na plataforma (não fixa no
  código), cada um com sua equipe responsável e suas regras (permite
  autoagendamento? remarcação automática? exige prontuário?).
- Um paciente pode estar em mais de um serviço ao longo do tempo.
- Um procedimento por vez por paciente.
- O número do WhatsApp **não é divulgado**; o primeiro contato costuma ser do
  PROCAPE, mas o paciente pode voltar a escrever depois (complicação,
  dúvida, agradecimento) e isso precisa ser tratado.

---

## 3. Plataforma de gestão

### 3.1 Lista de pacientes
- Ordenação padrão: **não atendidos primeiro**, depois **mais antigos**.
- Filtros: nome, prontuário, CPF/Cartão SUS, serviço, status, "não
  atendidos", "solicitou cancelamento", "solicitou remarcação",
  "sem resposta", "pendente de prontuário".
- Indicador de status do agendamento mais recente:

| Cor | Significado |
|---|---|
| 🟢 Verde | Confirmou |
| 🟡 Amarelo | Pediu remarcação |
| 🔴 Vermelho | Cancelou |
| 🔵 Azul | Não respondeu / aguardando confirmação após lembrete |
| 🟣 Roxo | Pendente da equipe (ex.: abertura de prontuário) |
| ⚪ Cinza | Mensagem enviada, aguardando resposta |

### 3.2 Ficha do paciente com **resumo do atendimento**
Ao abrir um paciente, a equipe vê em uma única tela (sem ler a conversa toda):
- Dados de identificação, telefones (WhatsApp), prontuário e serviços.
- **Linha do tempo**: contatos, agendamentos, confirmações, remarcações,
  cancelamentos, encaminhamentos e pendências, com data/hora e quem fez
  (bot, paciente ou membro da equipe).
- **Pendências abertas** em destaque.
- Mensagens principais (marcadas pelo bot ou pela equipe) e acesso à
  conversa completa.

### 3.3 Agendamentos (múltiplos por paciente)
- Lista de agendamentos sem limite; botões **"Adicionar teleconsulta"** e
  **"Adicionar procedimento"**.
- Ao salvar, o bot envia automaticamente a mensagem de confirmação (seção
  4.6).
- Remarcação gera um **novo agendamento ligado ao anterior** (histórico
  preservado).

### 3.4 Agenda e disponibilidade
- **Visão de calendário** dos agendamentos por serviço.
- **Cadastro de disponibilidade** (horários livres por serviço/agenda),
  necessário para o autoagendamento e a remarcação automática pelo bot.
- Regra usada hoje no pré-cateterismo: teleconsulta ≈ 7 dias antes do
  cateterismo (cateterismo dia 26 → teleconsulta dia 19, horários 14h, 14h30,
  15h, 15h30). O sistema pode sugerir os horários a partir dessa regra.

### 3.5 Disparo em lote
- Importar/cadastrar a lista da semana (substitui a planilha manual).
- Enviar a mesma mensagem para um grupo de pacientes ("estes 3 para o dia X,
  estes 3 para o dia Y").

### 3.6 Fila de atendimento humano + notificações
- Itens criados quando o paciente pede a equipe ou quando o bot encaminha
  automaticamente (seção 4.10).
- Cada item mostra: paciente, serviço, **motivo**, **queixa**, prioridade,
  origem (paciente pediu / bot encaminhou) e há quanto tempo está aberto.
- Notificação em tempo real para a equipe responsável.
- Filas separadas por equipe (ex.: Pesquisa FA, abertura de prontuário).

### 3.7 Chat dentro do sistema
- A equipe conversa com o paciente **pela plataforma**, não pelo celular.
- Ao assumir a conversa, o bot para de responder.
- Histórico completo (bot + paciente + equipe) salvo.

### 3.8 Área administrativa
Área restrita para gerir o sistema sem depender de desenvolvimento:

| Tela | O que faz | Quem acessa |
|---|---|---|
| Usuários | Criar, desativar, redefinir senha, atribuir nível e equipes | Admin, Supervisão (até o próprio nível, exclusivo) |
| Perfis e permissões | Ajustar o que cada nível pode fazer | Admin |
| Equipes | Equipes e membros | Admin, Supervisão |
| Serviços | Linhas de cuidado e suas regras (autoagendamento, remarcação automática, exige prontuário) | Admin, Supervisão |
| Modelos de mensagem | Editar textos do bot, ver status de aprovação na Meta | Admin, Supervisão |
| Orientações | Respostas cadastradas que o bot pode dar (item 4.15) | Admin, Supervisão |
| Formulários de avaliação | Link do formulário por serviço/tipo de consulta | Admin, Supervisão |
| Horário de atendimento e feriados | Base da mensagem "fora do horário" | Admin, Supervisão |
| Regras e prazos | Lembretes, travas de tempo, momento de envio do link | Admin, Supervisão |
| Agendas / disponibilidade | Horários livres por serviço | Supervisão, Atendente (se liberado) |
| Auditoria | Quem viu/alterou o quê | Admin, Supervisão |
| Relatórios | Indicadores (confirmação, não resposta, tempo de fila) | Admin, Supervisão |
| Integrações (WhatsApp, backups) | Credenciais e configurações técnicas | **Somente Admin** |

### 3.9 Níveis de acesso
Quatro níveis, conforme as funções da equipe. Cada usuário
tem **um nível** e pertence a **uma ou mais equipes**; o nível define *o que*
pode fazer e a equipe define *quais filas/pacientes* vê.

| Nível | Para quem | Resumo |
|---|---|---|
| **1. Administrador** | TI / Thompson | Tudo, inclusive integrações e permissões. Poucos usuários |
| **2. Supervisão** | Coordenação da Saúde Digital (ex.: Mari) | Tudo da operação + área administrativa (exceto integrações), relatórios, auditoria, exportação |
| **3. Atendente / Enfermagem** | Equipe da Saúde Digital (ex.: Anna) | Operação do dia a dia em todas as filas das suas equipes |
| **4. Equipe específica** | Pesquisa FA, abertura de prontuário, aconselhamento genético | Só a própria fila, pendências e pacientes vinculados ao seu serviço |

Matriz de permissões (✅ total · 🔸 só da(s) sua(s) equipe(s) · 👁 leitura · ❌ sem acesso):

| Ação | Admin | Supervisão | Atendente | Equipe esp. |
|---|---|---|---|---|
| Ver lista/ficha de pacientes | ✅ | ✅ | 🔸 | 🔸 |
| Ver CPF/telefone completos | ✅ | ✅ | ✅ | 🔸 |
| Cadastrar/editar paciente | ✅ | ✅ | ✅ | 🔸 |
| Importar planilha / disparo em lote | ✅ | ✅ | ✅ | ❌ |
| Criar/remarcar/cancelar agendamento | ✅ | ✅ | ✅ | 🔸 |
| Marcar consulta como realizada | ✅ | ✅ | ✅ | 🔸 |
| Fila de atendimento e chat | ✅ | ✅ | 🔸 | 🔸 |
| Transferir atendimento entre equipes | ✅ | ✅ | ✅ | ❌ |
| Registrar nº de prontuário | ✅ | ✅ | ✅ | 🔸 (prontuário) |
| Validar troca de telefone | ✅ | ✅ | ✅ | ❌ |
| Agenda (calendário) | ✅ | ✅ | ✅ | 🔸 |
| Disponibilidade de horários | ✅ | ✅ | **[PENDENTE]** | ❌ |
| Relatórios | ✅ | ✅ | 👁 básico | ❌ |
| Exportar dados | ✅ | ✅ | ❌ | ❌ |
| Modelos de mensagem, orientações, formulários | ✅ | ✅ | ❌ | ❌ |
| Serviços, prazos, horário de atendimento | ✅ | ✅ | ❌ | ❌ |
| Usuários e equipes | ✅ | ✅ (níveis 3–4) | ❌ | ❌ |
| Perfis e permissões | ✅ | ❌ | ❌ | ❌ |
| Auditoria | ✅ | ✅ | ❌ | ❌ |
| Integrações / credenciais | ✅ | ❌ | ❌ | ❌ |

Regras:
- Permissões ficam no banco (perfil → lista de permissões), então a matriz
  pode ser ajustada sem desenvolvimento.
- Áreas sem permissão **não aparecem no menu** e são bloqueadas também na
  API (não basta esconder o botão).
- Toda ação em área restrita e toda visualização de paciente vai para a
  auditoria.
- Senha forte e expiração de sessão; **[PENDENTE]** exigir 2FA para Admin e
  Supervisão.

---

## 4. Fluxos do bot

### 4.1 Início da conversa (paciente envia mensagem)
1. **Consentimento/aviso LGPD** no primeiro contato do número
   (**[PENDENTE]** texto).
2. **Identificar o paciente quando possível**: procurar o número do WhatsApp
   no cadastro.
   - Número conhecido → cumprimentar pelo primeiro nome e pedir **validação**
     (data de nascimento) antes de qualquer ação.
   - Número desconhecido → pedir **CPF (ou Cartão SUS)** + **data de
     nascimento**.
     - Encontrado com outro telefone → perguntar se deseja atualizar o número;
       a troca fica **pendente de validação da equipe** (foto de documento).
     - Não encontrado → seguir como paciente novo (coleta na seção 4.4).
3. **Identificar o serviço** (seção 2): Pré-cateterismo, Pós-angioplastia,
   Fibrilação atrial, Aconselhamento genético, Outros.
4. **Apresentar o menu**:

```
Como podemos ajudar?
1. 📅 Agendamento de teleconsulta
2. 🔄 Remarcação de teleconsulta
3. ❌ Cancelamento
4. ✅ Confirmação de consulta
5. 📋 Orientações
6. 👩‍⚕️ Falar com a equipe
```

- "Orientações" entrou com a especificação das mensagens (item 4.15).
- **Falar com a equipe** fica disponível em todas as etapas.
- Opções 2, 3 e 4 só fazem sentido se o paciente tiver agendamento futuro;
  se não tiver, o bot informa e oferece o agendamento ou a equipe.
- Fibrilação atrial: por regra da reunião, os contatos vão para a fila da
  **equipe da pesquisa** (pedir a queixa e encaminhar).

> Limitação do WhatsApp: mensagens com **botões** aceitam no máximo 3 opções.
> Para o menu de 5 opções usar **mensagem de lista** (até 10 itens) ou texto
> numerado. A confirmação (1/2/3) cabe em botões.

### 4.2 Paciente que já recebeu mensagem do PROCAPE
Quando o paciente responde a uma mensagem enviada pela equipe (agendamento,
lembrete, primeiro contato da regulação), o bot **já sabe quem é e de qual
agendamento se trata** – não pede identificação completa nem pergunta qual
consulta, apenas valida (data de nascimento) quando houver ação sensível.

### 4.3 Agendamento (opção 1)
1. Identificar o tipo de teleconsulta / serviço.
2. Nome completo.
3. CPF ou outro identificador definido pela equipe (**[PENDENTE]**: aceitar
   Cartão SUS como alternativa).
4. Data de nascimento.
5. Telefone: **confirmar** o número do WhatsApp em uso ("este é o melhor
   número para contato?") em vez de pedir de novo.
6. Verificar cadastro existente (CPF/Cartão SUS + data de nascimento).
7. Verificar prontuário (seção 4.4).
8. Verificar disponibilidade na agenda do serviço.
9. Apresentar as opções disponíveis (até 3 por vez + "ver outros"/"nenhum
   serve" → equipe).
10. Registrar o agendamento (reserva o horário).
11. Enviar a confirmação (seção 4.6).

Regras:
- Se o serviço **não permitir autoagendamento** ou não houver
  disponibilidade cadastrada, o bot registra o pedido como **solicitação de
  agendamento** e encaminha para a equipe, informando o paciente de que a
  equipe entrará em contato.
- Decisão da reunião para o pré-cateterismo: **a equipe define dia e hora** e
  o paciente apenas confirma ou pede remarcação. O autoagendamento é um
  "plus" a ser ligado por serviço.

### 4.4 Paciente sem prontuário
- Verificação do prontuário: a plataforma registra se o paciente **possui**,
  **não possui**, **está em abertura** ou **ainda não foi verificado**
  (**[PENDENTE]** existe integração com o sistema do hospital? Se não, a
  verificação é manual pela equipe).
- **Possui** → continuar o fluxo.
- **Não possui**:
  1. Coletar os dados para abertura (**[PENDENTE]** lista exata: nome da
     mãe, endereço, CEP, município etc.) e, se for o caso, exames/documentos
     (anexos).
  2. Registrar uma **pendência para a equipe de abertura de prontuário**.
  3. Informar ao paciente que os dados foram recebidos e que **a equipe dará
     continuidade** – o bot **não informa que o agendamento foi realizado**
     enquanto houver etapa dependente da equipe.
  4. Quando a equipe registrar o prontuário na plataforma, o fluxo retoma
     (bot oferece horários ou a equipe agenda) e o paciente é avisado.

### 4.5 Pré-cateterismo (regulação/mutirão) – primeiro contato ativo
Textos em `MENSAGENS.md`.
1. Disparo da mensagem padrão (1.1) – informa que a teleconsulta é online,
   que o link chega pelo WhatsApp e que exige exames de sangue dos últimos
   3 meses; botão **Ver horários disponíveis**.
2. Paciente toca no botão → bot apresenta os **horários disponíveis** (H).
   - "Nenhum serve" → equipe.
3. Bot solicita **exames** (anexos, opcional) e coleta os **dados para
   abertura do prontuário** (seção 4.4), quando necessário.
4. Horário reservado → confirmação (1.3) com data, horário, aviso de que é
   online, que não precisa ir ao PROCAPE e de que o link será enviado.
5. Sem resposta → lembrete 1 → lembrete 2 → equipe (4.9).
6. Paciente quer remarcar → 4.7.
7. Paciente **não pode realizar a consulta** → registra a situação
   (motivo `impossibilitado`) e encaminha para a equipe.
8. Paciente pergunta sobre o cateterismo → apenas **orientações cadastradas**
   (4.15); se não houver, encaminha.
- Este primeiro contato **não tem trava de tempo**.

### 4.5.1 Pós-angioplastia
Textos em `MENSAGENS.md`.
1. Convite para agendamento da consulta de acompanhamento (2.1).
2. Horários disponíveis (H) ou agendamento pela equipe → confirmação (2.2) com
   data, horário, aviso de online e orientações de participação.
3. Lembrete perto da data (3.2) com **confirmação de presença**:
   *Confirmo minha presença* · *Preciso remarcar* · *Cancelar*.
4. Não confirma → lembretes L1/L2 e depois equipe (4.9).
5. Remarcação → 4.7. Cancelamento → 4.8.
6. Link da teleconsulta no momento definido (4.13).
7. Após a teleconsulta → agradecimento + formulário de avaliação (8.1 / 4.14).

### 4.6 Confirmação de teleconsulta / procedimento
Mensagem automática ao salvar o agendamento (modelo genérico 3.1, usado
para qualquer tipo de teleconsulta, ou modelo específico do serviço),
contendo:
- Nome do paciente;
- Tipo de teleconsulta/procedimento;
- Data e horário;
- Aviso de que é online e de que o paciente deve ficar atento ao WhatsApp;
- Orientações (por serviço/tipo, cadastradas em modelos de mensagem).
- O **link não vai na confirmação**: é enviado em mensagem própria no
  momento definido (4.13).

Confirmação de presença: enviada junto do **lembrete** perto da consulta
(3.2). Botões: **Confirmo minha presença · Preciso remarcar · Cancelar**.
- Confirmar → 🟢 e aparece na agenda.
- Remarcar → seção 4.7 (🟡).
- Cancelar → seção 4.8 (🔴).

### 4.7 Remarcação
1. Identificar a consulta atual (automático se veio da mensagem do
   agendamento; se veio do menu e houver mais de uma futura, perguntar qual).
2. Informar que a solicitação será processada.
3. **Se o serviço permitir remarcação automática e houver disponibilidade**:
   apresentar opções → registrar a nova data → atualizar o agendamento (novo
   registro ligado ao anterior) → enviar nova confirmação.
4. **Caso contrário** (ou se nenhuma opção servir): encaminhar para a equipe.

> Decisão da reunião: no pré-cateterismo, a remarcação vai **direto para a
> equipe** (evita o paciente testar várias datas). A remarcação automática
> fica configurável por serviço. Sugestão: limitar a 1 remarcação automática
> por agendamento; a partir da segunda, equipe.

### 4.8 Cancelamento
1. Confirmar qual consulta (automático se veio da mensagem; senão perguntar).
2. Pedir confirmação ("Tem certeza que deseja cancelar…?") e, opcionalmente,
   o motivo.
3. Registrar o cancelamento, atualizar o status (🔴) e liberar o horário.
4. Informar o paciente.
5. Oferecer: "Deseja solicitar um novo agendamento?" → seção 4.3 ou equipe.

### 4.9 Paciente não responde
Fluxo: primeiro contato → aguarda → **lembrete 1** → aguarda →
**lembrete 2** → encaminhamento para a equipe.

| Momento | Ação |
|---|---|
| Dia 0 | Mensagem de confirmação / primeiro contato (⚪) |
| +24h sem resposta | Lembrete 1 |
| +24h sem resposta | Lembrete 2 |
| +24h sem resposta | Status **sem resposta** (🔵), pendência "ligar para o paciente" e notificação para a equipe – separado da fila de urgência |
| Depois | Aplicar a regra da Saúde Digital (**[PENDENTE]**: cancelar automaticamente? liberar o horário? quantos dias antes da consulta?) |

O sistema registra: data do primeiro contato, data de cada lembrete, se
houve resposta (e quando), status do paciente e necessidade de intervenção
da equipe (tabela `comunicacoes` em `BANCO_DE_DADOS.md`).

- Os prazos (intervalos e número de lembretes) ficam em **configurações**
  editáveis.
- Vale para qualquer mensagem que espera resposta (primeiro contato,
  convite, confirmação de presença).

### 4.10 Encaminhamento para a equipe
Opção **👩‍⚕️ Falar com a equipe**:
1. Perguntar o motivo / queixa (texto livre).
2. Registrar o motivo.
3. Informar que a solicitação foi encaminhada (mensagem 11). **Fora do
   horário de atendimento** (seg–sex, 8h–17h, e feriados cadastrados), envia
   também a mensagem 10 – a mensagem fica na fila para o próximo expediente.
4. Colocar na fila da equipe responsável e **parar o atendimento
   automático**.

Encaminhamento **automático** quando:
- O bot não entender a solicitação: na 1ª vez reapresenta o menu de 6
  opções (mensagem 12); na **2ª** vez, encaminha;
- Situação fora do fluxo (ex.: queixa clínica, sangramento);
- Necessidade de decisão da equipe;
- Problema no agendamento (sem horário, erro de reserva);
- Inconsistência nos dados (CPF não confere com data de nascimento, telefone
  diferente);
- Paciente pedir atendimento humano a qualquer momento (palavras como
  "atendente", "humano", "equipe").

Cada encaminhamento registra o **motivo** (lista fixa) para relatórios.

**[PENDENTE]** Palavras de alerta (ex.: "dor no peito", "sangramento",
"falta de ar") devem gerar prioridade **urgente** e uma resposta padrão
orientando a procurar emergência? Requer validação clínica.

### 4.11 Travas de tempo
| Trava | Regra |
|---|---|
| Inatividade com o bot | Paciente parado por X min → bot encerra a conversa. **[PENDENTE]** X (sugestão: 15 min) |
| Atendimento humano | Assim que a equipe fala, o bot para. Após 20–30 min sem movimentação, a conversa é encerrada |
| Retorno | Após encerramento, o paciente passa de novo pela verificação |
| Exceção | Mensagens ativas (primeiro contato/agendamento) **não** expiram |

### 4.12 Lembretes
- Lembrete antes da consulta/procedimento (**[PENDENTE]** antecedência,
  sugestão 24h) com a confirmação de presença.

### 4.13 Link da teleconsulta
- Enviado **automaticamente** no momento definido pela equipe (mensagem 7).
- O momento é configurável por serviço (ex.: 10 min antes) e pode ser
  ajustado em cada agendamento.
- O link é cadastrado no agendamento (ou padrão da agenda).
  Funciona com link fixo por sala ou gerado por consulta; padrão: 10 min
  antes (sugestão adotada – validar).
- Só é enviado se o agendamento estiver confirmado/aguardando (não
  cancelado). Se o link estiver vazio na hora, gera alerta para a equipe.

### 4.14 Após a teleconsulta
1. A equipe marca a consulta como **realizada** (ou **faltou**).
2. Realizada → bot envia **agradecimento + formulário de avaliação** do
   serviço/tipo de consulta (8.1 / 8.2). Sugestão adotada: envio
   imediato (ajustável em Admin › Prazos).
3. **Pendência do paciente**: se a equipe identificar algo que o paciente
   precisa enviar (documento, exame, informação), registra a pendência na
   plataforma e o bot envia a mensagem 8.3. Quando o paciente envia o
   arquivo/resposta, a pendência é marcada como recebida e a equipe é
   notificada. Sem resposta → mesma regra de lembretes (4.9).
4. Faltou → mensagem 8.4 (*Quero remarcar* / *Falar com a equipe*) e,
   sem resposta, regra de lembretes (4.9) e depois equipe.

### 4.15 Orientações
- O bot responde **somente** com orientações cadastradas pela equipe na área
  administrativa (por serviço: ex. preparo para o cateterismo, exames
  necessários, como funciona a teleconsulta).
- Acesso pelo menu (opção 5) ou quando a mensagem do paciente bater com as
  palavras-chave de uma orientação.
- Sem orientação correspondente ou dúvida clínica → encaminha para a equipe.
- O bot **não gera respostas próprias**.

### 4.16 Fora do horário de atendimento
- O bot funciona 24h (agendamento, confirmação, orientações).
- Quando o paciente precisa da equipe fora do horário, recebe a mensagem 10
  e o atendimento fica na fila para o próximo expediente.
- Horário (seg–sex, 8h–17h) e feriados configuráveis por equipe.

### 4.17 Regras do WhatsApp que impactam o fluxo
- Mensagens iniciadas pelo PROCAPE (primeiro contato, confirmação,
  lembretes) e qualquer mensagem enviada **mais de 24h após a última
  mensagem do paciente** precisam ser **modelos (templates) aprovados pela
  Meta**. Os textos precisam ser definidos cedo para aprovação.
- Isso vale também para a equipe no chat: passadas 24h, só é possível
  reabrir a conversa com um template.

---

## 5. Registro das conversas e do atendimento

O sistema guarda, de forma estruturada (não apenas o texto da conversa):

| Informação | Onde fica (ver `BANCO_DE_DADOS.md`) |
|---|---|
| Nome e dados de identificação | `pacientes` |
| Número(s) de WhatsApp | `paciente_telefones` |
| Serviço / tipo de atendimento | `paciente_servicos`, `conversas.servico_id` |
| Data/hora dos contatos | `conversas`, `mensagens` |
| Mensagens (todas) e principais | `mensagens` (`destaque = true`) |
| Agendamento, confirmação, cancelamento, remarcação | `agendamentos` + `agendamento_eventos` |
| Pendências (da equipe e do paciente) | `pendencias` |
| Primeiro contato, lembretes, links, avaliações enviados e respondidos | `comunicacoes` |
| Encaminhamento para a equipe | `atendimentos_humanos` |
| Dados coletados pelo bot | `conversas.dados_coletados`, `paciente_dados_prontuario` |
| Anexos (exames, documentos) | `anexos` |
| Quem fez o quê | `auditoria` |

Tudo isso alimenta a **linha do tempo** da ficha do paciente (3.2).

---

## 6. Requisitos não funcionais
- **LGPD**: dados de saúde são sensíveis; aviso/consentimento; o bot não
  expõe dados; acesso por login; auditoria; anexos em armazenamento privado;
  política de retenção (**[PENDENTE]**).
- **Canal WhatsApp**: recomendada a **API oficial (WhatsApp Business Cloud
  API)** – APIs não oficiais têm risco de bloqueio do número.
  **[PENDENTE]** conta Meta Business do PROCAPE e número dedicado.
- **Hospedagem**: VPS do hospital/projeto (**[PENDENTE]** contratação).
  Plano B para testes: VPS da empresa do Thompson **apenas com dados
  fictícios** e número de WhatsApp de teste.
- **Prazo**: ~20 dias – ~10 de desenvolvimento + ~10 de testes e correções.

---

## 7. Fases sugeridas
1. **MVP**: login com níveis de acesso, cadastro/importação de pacientes,
   agendamentos múltiplos pela equipe, mensagens de primeiro contato,
   confirmação, lembretes (1 e 2), confirmação de presença, link da
   teleconsulta e agradecimento/avaliação, lista com cores e filtros, disparo
   em lote, marcação "sem resposta", encaminhamento para a equipe com fila,
   chat e mensagem de fora do horário. Área administrativa com usuários e
   modelos de mensagem.
2. **Atendimento completo**: início de conversa com identificação, menu de 6
   opções, orientações cadastradas, contato espontâneo, travas de tempo,
   coleta de dados de prontuário, pendências (da equipe e do paciente), ficha
   com linha do tempo, auditoria, demais telas administrativas.
3. **Agenda**: disponibilidade, calendário, autoagendamento e remarcação
   automática por serviço.
4. **Futuro**: triagem da queixa por IA, relatórios/indicadores, integração
   com o sistema do hospital.

---

## 8. Divergências entre a reunião e a especificação (para validar)

| Tema | Reunião | Especificação nova | Proposta |
|---|---|---|---|
| Remarcação | Vai direto para a equipe | Bot remarca se houver agenda | **Adotado:** configurável por serviço; pré-cateterismo = equipe |
| Cancelamento | Opções na própria mensagem, sem perguntar qual consulta | Confirmar qual consulta | Automático quando vier da mensagem; perguntar só se vier do menu com 2+ consultas |
| Agendamento | Equipe define dia e hora | Bot verifica agenda e oferece opções | Ambos; autoagendamento ligado por serviço (fase 3) |
| Telefone | Já é o número do WhatsApp | Solicitar telefone | Confirmar o número em uso |
| Não resposta | Marcar azul no 3º dia | Aguardando confirmação + regra da Saúde Digital | Azul + regra configurável (**[PENDENTE]**) |
| Serviços | 3 linhas | + Aconselhamento genético e Outros | Serviços cadastráveis |
| Envio do link | – | Msg 1.1: e-mail/SMS · Msgs 1.3, 2.2, 3.1, 7: WhatsApp | **Adotado:** WhatsApp (mensagem 7) |
| Disponibilidade (pré-cat.) | Equipe define dia/hora | Msg 1.1 pede "sua disponibilidade" (texto livre) | **Adotado:** botão "Ver horários disponíveis" + lista |
| Confirmação de presença | 1/2/3 (confirmar/remarcar/cancelar) | 1 Confirmo / 2 Preciso remarcar | **Adotado:** 3 botões, incluindo Cancelar |
| Não resposta | 1 lembrete | 2 lembretes e depois equipe | **Adotado:** 2 lembretes (configurável) |
| Menu | 5 opções | 6 opções (+ Orientações) | **Adotado:** 6 opções (lista) |

Os textos das mensagens já foram ajustados conforme essas propostas – ver
`MENSAGENS.md`, seções 4 e 5 (aguardando validação da equipe).

A primeira especificação pulava as seções **9, 10 e 12**; a especificação
de mensagens cobriu orientações, lembretes, avaliação e fora do horário.
Confirmar se ainda falta algo. Na especificação de mensagens, a seção 8
("Após a teleconsulta") tem subitens numerados como 9.x.

---

## 9. Ações combinadas
- **Thompson**: tentar novamente o cadastro da VPS; planejamento técnico.
- **Anna**: acompanhar a Mari sobre a VPS (ou via TI); revisar os
  **[PENDENTE]** deste documento; enviar os textos padrão das mensagens e a
  lista de dados para abertura de prontuário.
- Verificar se ainda há acesso ao site feito anteriormente pela Flávia.
