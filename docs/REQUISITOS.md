# PROCAPE – Saúde Digital: Bot de WhatsApp + Plataforma de Gestão

Levantamento de requisitos a partir da reunião entre Thompson Cysneiros e
Anna Beatriz Mariz Dias, complementado pela especificação funcional do bot
enviada depois. Este documento é a referência para o desenvolvimento; itens
marcados com **[DEPENDÊNCIA]** não são decisões de sistema: dependem de
ação externa (contratação, textos da equipe, validação jurídica). As
decisões técnicas tomadas para fechar as pendências estão na seção 10.

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
| **Aconselhamento genético** | Contato espontâneo ou cadastro pela equipe | Mesmos da regra geral | Aconselhamento genético |
| **Outros** | Contato espontâneo | – | Saúde Digital (triagem) |

- A lista de serviços deve ser **cadastrável** na plataforma (não fixa no
  código), cada um com sua equipe responsável, modelos de mensagem,
  orientações e formulário de avaliação.
- **Regras definidas para todos os serviços:**
  - **Sem autoagendamento**: o bot não oferece horários; **a equipe define dia
    e hora** na plataforma e o paciente confirma, pede remarcação ou cancela.
  - **Sem remarcação automática**: todo pedido de remarcação vai para a
    equipe.
  - **Prontuário não é obrigatório**: é apenas um campo numérico, preenchido
    quando o paciente tiver; nenhum fluxo depende dele.
- Um paciente pode estar em mais de um serviço ao longo do tempo.
- Um procedimento por vez por paciente.
- O número do WhatsApp **não é divulgado**; o primeiro contato costuma ser do
  PROCAPE, mas o paciente pode voltar a escrever depois (complicação,
  dúvida, agradecimento) e isso precisa ser tratado.

---

## 3. Plataforma de gestão

### 3.1 Lista de pacientes
- Ordenação padrão: **não atendidos primeiro**, depois **mais antigos**.
- Filtros: nome, prontuário, CPF, serviço, status, "não
  atendidos", "solicitou cancelamento", "solicitou remarcação",
  "sem resposta", "aguardando agendamento".
- Indicador de status do agendamento mais recente:

| Cor | Significado |
|---|---|
| 🟢 Verde | Confirmou |
| 🟡 Amarelo | Pediu remarcação |
| 🔴 Vermelho | Cancelou |
| 🔵 Azul | Não respondeu / aguardando confirmação após lembrete |
| 🟣 Roxo | Aguardando a equipe definir data/hora (solicitação de agendamento ou remarcação em andamento) |
| ⚪ Cinza | Mensagem enviada, aguardando resposta |

### 3.2 Ficha do paciente com **resumo do atendimento**
Ao abrir um paciente, a equipe vê em uma única tela (sem ler a conversa toda):
- Dados de identificação, telefones (WhatsApp), prontuário (se houver) e
  serviços.
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

### 3.4 Agenda
- **Visão de calendário** dos agendamentos por serviço.
- **Não há cadastro de disponibilidade**: a equipe escolhe dia e hora ao
  criar o agendamento.
- Ajuda ao agendar: regra usada hoje no pré-cateterismo – teleconsulta ≈ 7
  dias antes do cateterismo (cateterismo dia 26 → teleconsulta dia 19,
  horários 14h, 14h30, 15h, 15h30). Ao informar a data do cateterismo, a
  plataforma **sugere** a data da teleconsulta e mostra os horários já
  ocupados naquele dia, para a equipe escolher.
- **Lista "Aguardando agendamento"**: pacientes que pediram agendamento ou
  remarcação e ainda não têm data (🟣/🟡).

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
- Filas separadas por equipe (ex.: Saúde Digital, Pesquisa FA).

### 3.7 Chat dentro do sistema
- A equipe conversa com o paciente **pela plataforma**, não pelo celular.
- Ao assumir a conversa, o bot para de responder.
- Histórico completo (bot + paciente + equipe) salvo.

### 3.8 Área administrativa
Área restrita para gerir o sistema sem depender de desenvolvimento:

| Tela | O que faz | Quem acessa |
|---|---|---|
| Usuários | Criar, desativar, redefinir senha, atribuir nível e equipes | Admin (todos os níveis); Supervisão (só níveis 3 e 4 – não cria nem altera outro Supervisor ou Admin) |
| Perfis e permissões | Ajustar o que cada nível pode fazer | Admin |
| Equipes | Equipes e membros | Admin, Supervisão |
| Serviços | Linhas de cuidado, equipe responsável, modelos de mensagem, prazos e formulário | Admin, Supervisão |
| Modelos de mensagem | Editar textos e opções numeradas do bot (valem na hora) | Admin, Supervisão |
| Orientações | Respostas cadastradas que o bot pode dar (item 4.15) | Admin, Supervisão |
| Formulários de avaliação | Link do formulário por serviço/tipo de consulta | Admin, Supervisão |
| Horário de atendimento e feriados | Base da mensagem "fora do horário" | Admin, Supervisão |
| Regras e prazos | Lembretes, travas de tempo, momento de envio do link | Admin, Supervisão |
| Auditoria | Quem viu/alterou o quê | Admin, Supervisão |
| Relatórios | Indicadores (confirmação, não resposta, tempo de fila) | Admin, Supervisão |
| Integrações (WhatsApp, backups) | Credenciais e configurações técnicas | Admin, Supervisão |

### 3.9 Níveis de acesso
Quatro níveis, conforme as funções da equipe. Cada usuário
tem **um nível** e pertence a **uma ou mais equipes**; o nível define *o que*
pode fazer e a equipe define *quais filas/pacientes* vê.

| Nível | Para quem | Resumo |
|---|---|---|
| **1. Administrador** | TI / Thompson | Tudo, inclusive integrações e permissões. Poucos usuários |
| **2. Supervisão** | Coordenação da Saúde Digital (ex.: Mari) | Tudo da operação + área administrativa (inclusive integrações e credenciais), relatórios, auditoria, exportação. Só não altera perfis/permissões nem usuários de nível 1–2 |
| **3. Atendente / Enfermagem** | Equipe da Saúde Digital (ex.: Anna) | Operação do dia a dia em todas as filas das suas equipes |
| **4. Equipe específica** | Pesquisa FA, aconselhamento genético | Só a própria fila, pendências e pacientes vinculados ao seu serviço |

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
| Registrar nº de prontuário | ✅ | ✅ | ✅ | 🔸 |
| Validar troca de telefone | ✅ | ✅ | ✅ | ❌ |
| Agenda (calendário) | ✅ | ✅ | ✅ | 🔸 |
| Relatórios | ✅ | ✅ | 👁 básico | ❌ |
| Exportar dados | ✅ | ✅ | ❌ | ❌ |
| Modelos de mensagem, orientações, formulários | ✅ | ✅ | ❌ | ❌ |
| Serviços, prazos, horário de atendimento | ✅ | ✅ | ❌ | ❌ |
| Usuários e equipes | ✅ | ✅ (níveis 3–4) | ❌ | ❌ |
| Perfis e permissões | ✅ | ❌ | ❌ | ❌ |
| Auditoria | ✅ | ✅ | ❌ | ❌ |
| Integrações / credenciais | ✅ | ✅ | ❌ | ❌ |

Regras:
- Permissões ficam no banco (perfil → lista de permissões), então a matriz
  pode ser ajustada sem desenvolvimento.
- Áreas sem permissão **não aparecem no menu** e são bloqueadas também na
  API (não basta esconder o botão).
- Toda ação em área restrita e toda visualização de paciente vai para a
  auditoria.
- Senha forte (mín. 10 caracteres), bloqueio após 5 tentativas erradas por
  15 min e sessão expirada após 30 min sem uso.
- **2FA obrigatório para Admin e Supervisão** (aplicativo autenticador,
  ex.: Google Authenticator), já que ambos acessam credenciais e dados de
  todos os pacientes. Opcional para os demais níveis.
- Credenciais (token do WhatsApp etc.) ficam **criptografadas** no banco e,
  depois de salvas, aparecem **mascaradas** na tela (`••••1234`): é possível
  substituir, mas não ler o valor completo. Toda alteração vai para a
  auditoria.

---

## 4. Fluxos do bot

### 4.1 Início da conversa (paciente envia mensagem)
1. **Aviso de privacidade (LGPD)** no primeiro contato do número (mensagem
   A de `MENSAGENS.md`). É um aviso de **ciência**, não um pedido de
   consentimento: o tratamento de dados de saúde para o atendimento tem base
   legal na tutela da saúde (LGPD, art. 11, II, "f"), então o atendimento
   não depende de o paciente "aceitar".
2. **Identificar o paciente quando possível**: procurar o número do WhatsApp
   no cadastro.
   - Número conhecido → cumprimentar pelo primeiro nome e pedir **validação**
     (data de nascimento) antes de qualquer ação.
   - Número desconhecido → pedir **CPF** + **data de
     nascimento**.
     - Encontrado com outro telefone → perguntar se deseja atualizar o número;
       a troca fica **pendente de validação da equipe** (foto de documento).
     - Não encontrado → seguir como paciente novo (coleta na seção 4.3).
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
- Opção 1 registra uma **solicitação de agendamento** para a equipe (o bot
  não marca horário – seção 4.3).
- Opções 2, 3 e 4 só fazem sentido se o paciente tiver agendamento futuro;
  se não tiver, o bot informa e oferece a solicitação de agendamento ou a
  equipe.
- Fibrilação atrial: por regra da reunião, os contatos vão para a fila da
  **equipe da pesquisa** (pedir a queixa e encaminhar).

> Com o Baileys não há botões/listas confiáveis: o menu e todas as escolhas
> são **opções numeradas**, e o paciente responde com o número (ou a
> palavra, ex.: "cancelar").

### 4.2 Paciente que já recebeu mensagem do PROCAPE
Quando o paciente responde a uma mensagem enviada pela equipe (agendamento,
lembrete, primeiro contato da regulação), o bot **já sabe quem é e de qual
agendamento se trata** – não pede identificação completa nem pergunta qual
consulta, apenas valida (data de nascimento) quando houver ação sensível.

### 4.3 Solicitação de agendamento (opção 1)
O bot **não agenda**: ele coleta os dados e registra a solicitação; a equipe
define dia e hora na plataforma.
1. Identificar o tipo de teleconsulta / serviço.
2. Nome completo.
3. **CPF** – único identificador aceito; o bot valida os dígitos
   verificadores antes de buscar. Quem não souber o CPF é encaminhado para a
   equipe.
4. Data de nascimento.
5. Telefone: **confirmar** o número do WhatsApp em uso ("este é o melhor
   número para contato?") em vez de pedir de novo.
6. Verificar cadastro existente (CPF + data de nascimento); se
   não existir, criar o pré-cadastro.
7. Perguntar, de forma **opcional**, o número do prontuário ("Se você já é
   paciente do PROCAPE e sabe o número do seu prontuário, informe; se não,
   toque em *Não sei / não tenho*").
8. Registrar a **solicitação de agendamento** (status 🟣 `solicitado`) e criar
   a pendência "agendar" para a equipe do serviço.
9. Informar ao paciente que a solicitação foi recebida e que **a equipe
   enviará a data e o horário por este WhatsApp** – o bot **nunca informa
   que o agendamento foi realizado** antes de a equipe definir a data.
10. Quando a equipe salvar o agendamento, o bot envia a confirmação
    (seção 4.6).

### 4.4 Prontuário
- O prontuário é **apenas um número, não obrigatório**: preenchido quando o
  paciente tiver (informado por ele, pela equipe ou já existente na base).
- Nenhum fluxo é bloqueado pela falta do prontuário; não há verificação nem
  etapa de abertura de prontuário no sistema.
- O campo aparece na ficha e pode ser usado como filtro/busca.

### 4.5 Pré-cateterismo (regulação/mutirão) – primeiro contato ativo
Textos em `MENSAGENS.md`.
Duas formas de uso, à escolha da equipe no disparo:
- **A) Disparo já com data e hora** (mais comum, decisão da reunião): a equipe
  define a data no lote e o paciente recebe direto a confirmação (1.3) com
  *Confirmo minha presença* · *Preciso remarcar* · *Cancelar*.
- **B) Disparo sem data** (quando a equipe precisa primeiro do retorno do
  paciente):
  1. Mensagem padrão (1.1) – teleconsulta online, link pelo WhatsApp, exames
     de sangue dos últimos 3 meses; opções **1 Quero agendar** · **2 Falar com a
     equipe**.
  2. Paciente toca em *Quero agendar* → bot pede os **exames** (anexos,
     opcional) e os **dados cadastrais** que faltarem (lista mínima na
     seção 10, item 3)
     → registra a solicitação (🟣) e informa que a equipe enviará data e
     horário.
  3. Equipe define dia e hora na plataforma → confirmação (1.3).

Nos dois casos:
5. Sem resposta → lembrete 1 → lembrete 2 → equipe (4.9).
6. Paciente quer remarcar → 4.7 (sempre equipe).
7. Paciente **não pode realizar a consulta** → registra a situação
   (motivo `impossibilitado`) e encaminha para a equipe.
8. Paciente pergunta sobre o cateterismo → apenas **orientações cadastradas**
   (4.15); se não houver, encaminha.
- Este primeiro contato **não tem trava de tempo**.

### 4.5.1 Pós-angioplastia
Textos em `MENSAGENS.md`.
1. Equipe agenda a consulta de acompanhamento e o bot envia a confirmação
   (2.2) com data, horário, aviso de online e orientações de participação,
   com *Confirmo minha presença* · *Preciso remarcar* · *Cancelar*.
   - Opcional: convite sem data (2.1) com *Quero agendar*, igual à forma B
     do pré-cateterismo.
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
- Opções: **1 Confirmo minha presença · 2 Preciso remarcar · 3 Cancelar**.

O **lembrete** perto da consulta (3.2) repete as opções para quem ainda não
confirmou; para quem já confirmou, é só um lembrete.
- Confirmar → 🟢 e aparece na agenda.
- Remarcar → seção 4.7 (🟡).
- Cancelar → seção 4.8 (🔴).

### 4.7 Remarcação
1. Identificar a consulta atual (automático se veio da mensagem do
   agendamento; se veio do menu e houver mais de uma futura, perguntar qual).
2. Perguntar, opcionalmente, a preferência do paciente (texto livre, ex.:
   "prefiro à tarde") – vai como observação para a equipe.
3. Registrar status 🟡 `remarcacao_solicitada`, abrir atendimento na fila
   (motivo `remarcacao`) e informar que a equipe enviará a nova data.
4. A equipe define a nova data na plataforma → novo agendamento ligado ao
   anterior → o bot envia nova confirmação.

> **Sem remarcação automática** em nenhum serviço: evita o paciente testar
> várias datas e mantém a agenda sob controle da equipe.

### 4.8 Cancelamento
1. Confirmar qual consulta (automático se veio da mensagem; senão perguntar).
2. Pedir confirmação ("Tem certeza que deseja cancelar…?") e, opcionalmente,
   o motivo.
3. Registrar o cancelamento e atualizar o status (🔴).
4. Informar o paciente.
5. Oferecer: "Deseja solicitar uma nova data?" → registra solicitação de
   agendamento para a equipe (4.3).

### 4.9 Paciente não responde
Fluxo: primeiro contato → aguarda → **lembrete 1** → aguarda →
**lembrete 2** → encaminhamento para a equipe.

| Momento | Ação |
|---|---|
| Dia 0 | Mensagem de confirmação / primeiro contato (⚪) |
| +24h sem resposta | Lembrete 1 |
| +24h sem resposta | Lembrete 2 |
| +24h sem resposta | Status **sem resposta** (🔵), pendência "ligar para o paciente" e notificação para a equipe – separado da fila de urgência |
| Depois | **O sistema não cancela automaticamente.** O agendamento fica 🔵 e a equipe decide (ligar, remarcar ou cancelar). Se faltarem **menos de 48h** para a consulta e ela continuar sem confirmação, o paciente sobe para o topo da lista com destaque |

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

**Palavras de alerta** (lista editável em Admin › Prazos; inicial: "dor no
peito", "aperto no peito", "falta de ar", "sangramento", "desmaio",
"desmaiei", "palpitação", "infarto"):
- O bot envia **na hora** a mensagem de orientação de emergência (mensagem U
  de `MENSAGENS.md`: procurar a emergência mais próxima ou ligar 192 – SAMU),
  independentemente da etapa ou do horário;
- abre atendimento com prioridade **urgente** e notifica a equipe;
- o bot **não faz avaliação clínica** – apenas orienta a procurar socorro.
  A equipe pode ajustar as palavras; o texto da mensagem deve ser revisado
  pela equipe de enfermagem/médica antes de entrar em produção.

### 4.11 Travas de tempo
| Trava | Regra |
|---|---|
| Inatividade com o bot | Paciente parado por **15 min** → bot encerra a conversa (configurável) |
| Atendimento humano | Assim que a equipe fala, o bot para. Após **30 min** sem movimentação, a conversa é encerrada (configurável) |
| Retorno | Após encerramento, o paciente passa de novo pela verificação |
| Exceção | Mensagens ativas (primeiro contato/agendamento) **não** expiram |

### 4.12 Lembretes
- Lembrete **24h antes** da consulta/procedimento (configurável por
  serviço), com a confirmação de presença para quem ainda não confirmou.
- Se o agendamento for criado com menos de 24h de antecedência, o lembrete
  não é enviado (a própria confirmação já cumpre esse papel).

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

### 4.17 Regras do canal (Baileys) que impactam o fluxo
- **Sem aprovação de modelos e sem janela de 24h**: qualquer mensagem
  (inclusive da equipe no chat) pode ser enviada a qualquer momento.
- **Sem botões**: opções sempre numeradas.
- **Envio em lote com cadência**: intervalo aleatório entre mensagens (ex.:
  20–60 s) e limite diário configurável, para reduzir o risco de bloqueio
  do número. Um disparo de 12 pacientes leva poucos minutos.
- Mensagens personalizadas (nome, data) – evitar textos idênticos em massa.

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
- **LGPD**: ver seção 6.1.
- **Canal WhatsApp: Baileys** (biblioteca Node.js que conecta como
  "aparelho conectado" do WhatsApp, igual ao WhatsApp Web).
  - **Vantagens:** sem custo por mensagem, sem aprovação de modelos, sem
    janela de 24h, implantação imediata.
  - **Riscos (aceitos):** não é oficial – o número pode ser **bloqueado**
    pelo WhatsApp (principalmente em envios em massa ou denúncias de spam);
    atualizações do WhatsApp podem quebrar a conexão até a biblioteca ser
    atualizada; não há suporte nem contrato com a Meta.
  - **Mitigações:**
    - **chip dedicado e institucional** com **WhatsApp Business** (nome e
      foto do PROCAPE), nunca o número pessoal de alguém;
    - celular desse chip **sempre ligado e com internet** (o aparelho
      conectado cai se o celular ficar muitos dias offline);
    - cadência no envio em lote e limite diário (4.17); uso gradual do número
      novo nas primeiras semanas;
    - tela de **status da conexão** em Admin › Integrações (conectado /
      desconectado, QR code para reconectar) e **alerta** para a Supervisão
      quando cair;
    - versão da biblioteca fixada e atualizada de forma controlada;
    - o bot fala com o WhatsApp através de uma **camada de provedor**: se um
      dia o número for bloqueado ou o hospital preferir a API oficial, troca-se
      só essa camada, sem refazer o sistema.
  - **[DEPENDÊNCIA]** chip/número dedicado e um celular para ele. Para
    testes: um chip de teste qualquer (sem dados reais).
- **Hospedagem**: **[DEPENDÊNCIA]** contratação da VPS do hospital/projeto.
  Requisitos mínimos: 2 vCPU, 4 GB de RAM, 80 GB SSD, Ubuntu 24.04 LTS,
  **datacenter no Brasil** (simplifica a LGPD quanto a transferência
  internacional), IP fixo e domínio com HTTPS. Plano B para testes: VPS da
  empresa do Thompson **apenas com dados fictícios** e número de teste.
- **Backups**: diários, criptografados, guardados fora da VPS, retenção de
  30 dias; teste de restauração mensal.

### 6.1 LGPD – decisões
| Tema | Decisão |
|---|---|
| Base legal | Tutela da saúde (art. 11, II, "f") para o atendimento; não depende de consentimento. O aviso de privacidade informa e registra a ciência |
| Minimização | Só se coleta o necessário para identificar e atender (seção 10, item 3). Nada de dados clínicos pelo bot além da queixa livre informada pelo paciente |
| Bot e WhatsApp | Mensagens com **primeiro nome**, data, hora e link; sem CPF, diagnóstico ou resultado de exame. As mensagens passam pela infraestrutura do WhatsApp, por isso o conteúdo é mínimo |
| Sessão do WhatsApp | Os arquivos de conexão do Baileys equivalem à **senha do número**: ficam criptografados, só na VPS, e só Admin/Supervisão podem reconectar ou desconectar |
| Acesso | Por nível e equipe (3.9), 2FA para Admin/Supervisão, CPF/telefone mascarados fora do escopo |
| Rastreabilidade | Auditoria de visualização e alteração de pacientes, guardada por 5 anos |
| Criptografia | HTTPS em tudo; banco e backups criptografados; credenciais e anexos criptografados |
| Anexos (exames, documentos) | Armazenamento privado. Exames devem ser **incorporados ao prontuário oficial** pela equipe; no sistema, são apagados **90 dias após a consulta realizada** |
| Retenção | Conversas e cadastro: enquanto o paciente estiver em acompanhamento + **5 anos** após o último contato; depois, **anonimização** (mantém só números para relatórios). O sistema **não é o prontuário oficial** – o que for clínico vai para o prontuário do hospital |
| Direitos do titular | Pedido de acesso, correção ou exclusão chega pela fila (motivo `direitos_titular`) e é tratado pela Supervisão em até 15 dias; o sistema permite exportar os dados de um paciente e anonimizá-lo |
| Ambiente de teste | Somente dados fictícios; nunca cópia da produção |
| Incidentes | Registro de incidentes na auditoria; aviso imediato ao encarregado (DPO) do hospital |
| Contratos | A VPS atua como **operadora** (contrato/termos em nome do hospital). O uso do Baileys não tem contrato com a Meta – decisão registrada e aceita pelo hospital |
| Validação | **[DEPENDÊNCIA]** o encarregado (DPO) do hospital revisa o aviso de privacidade e esta tabela antes da produção |
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
   solicitação de agendamento pelo bot, pendências (da equipe e do paciente),
   ficha com linha do tempo, auditoria, demais telas administrativas.
3. **Agenda**: calendário, sugestão de data pela regra do pré-cateterismo e
   lista "aguardando agendamento".
4. **Futuro**: triagem da queixa por IA, relatórios/indicadores, integração
   com o sistema do hospital.

---

## 8. Divergências entre a reunião e a especificação (para validar)

| Tema | Reunião | Especificação nova | Proposta |
|---|---|---|---|
| Remarcação | Vai direto para a equipe | Bot remarca se houver agenda | **Definido:** sempre a equipe (sem remarcação automática) |
| Cancelamento | Opções na própria mensagem, sem perguntar qual consulta | Confirmar qual consulta | Automático quando vier da mensagem; perguntar só se vier do menu com 2+ consultas |
| Agendamento | Equipe define dia e hora | Bot verifica agenda e oferece opções | **Definido:** equipe define dia e hora (sem autoagendamento) |
| Telefone | Já é o número do WhatsApp | Solicitar telefone | Confirmar o número em uso |
| Não resposta | Marcar azul no 3º dia | Aguardando confirmação + regra da Saúde Digital | **Definido:** azul, sem cancelamento automático; destaque se faltar < 48h |
| Serviços | 3 linhas | + Aconselhamento genético e Outros | Serviços cadastráveis |
| Prontuário | Pacientes da regulação não têm | Verificar/abrir prontuário antes de agendar | **Definido:** só um número opcional, sem bloqueio |
| Envio do link | – | Msg 1.1: e-mail/SMS · Msgs 1.3, 2.2, 3.1, 7: WhatsApp | **Adotado:** WhatsApp (mensagem 7) |
| Disponibilidade (pré-cat.) | Equipe define dia/hora | Msg 1.1 pede "sua disponibilidade" (texto livre) | **Definido:** sem texto livre; data já no disparo ou opção "1 – Quero agendar" → equipe define |
| Confirmação de presença | 1/2/3 (confirmar/remarcar/cancelar) | 1 Confirmo / 2 Preciso remarcar | **Adotado:** 3 opções numeradas, incluindo Cancelar |
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
  decisões da seção 10; enviar os textos padrão das mensagens e a
  lista de dados cadastrais que o bot deve coletar dos pacientes novos.
- Verificar se ainda há acesso ao site feito anteriormente pela Flávia.

---

## 10. Pendências resolvidas (decisões técnicas)

Decisões tomadas com base nas necessidades do sistema, segurança e LGPD.
Todas são **configuráveis** na área administrativa quando fizer sentido.

| # | Pendência | Decisão | Motivo |
|---|---|---|---|
| 1 | Identificador do paciente | **Somente CPF**, com validação dos dígitos | Decisão da equipe; CPF é o número único de identificação do cidadão (Lei 14.534/2023) |
| 2 | Paciente ainda sem CPF no cadastro (ex.: lista da regulação traz só Cartão SUS) | Entra como **pré-cadastro**; o bot pede o CPF na primeira interação e completa o cadastro. Cartão SUS fica guardado como dado informativo, mas não serve para identificar | Lista da regulação nem sempre traz CPF |
| 3 | Dados cadastrais que o bot coleta (pacientes novos) | Mínimo: nome da mãe, CEP, endereço (logradouro, número, bairro), município/UF. **Opcionais:** e-mail, prontuário | Minimização (LGPD): o necessário para identificação e contato |
| 4 | Aconselhamento genético | Mesmo fluxo geral (confirmação genérica 3.1), equipe própria; serviço fica **inativo** até a equipe cadastrar orientações e formulário | Não depende de desenvolvimento específico |
| 5 | Aviso LGPD | Aviso de **ciência** (não consentimento), enviado uma vez por número, opção *1 – Entendi* | Base legal de tutela da saúde |
| 5b | Canal WhatsApp | **Baileys**, com as mitigações da seção 6 | Decisão da equipe: sem custo e sem aprovação de modelos |
| 6 | 2FA | Obrigatório para Admin e Supervisão | Ambos acessam credenciais e todos os pacientes |
| 7 | Trava de inatividade do bot | 15 min | Conversa curta de triagem |
| 8 | Trava do atendimento humano | 30 min | Dentro dos 20–30 min da reunião |
| 9 | Lembrete da consulta | 24h antes | Tempo para o paciente reorganizar ou pedir remarcação |
| 10 | Envio do link | 10 min antes | Já adotado |
| 11 | Avaliação | Imediata ao marcar "realizada" | Já adotado |
| 12 | Regra final de não resposta | Sem cancelamento automático; 🔵 + pendência "ligar"; destaque se faltar < 48h | Cancelar sozinho poderia tirar a vaga de um paciente da regulação |
| 13 | Palavras de urgência | Orientação imediata de emergência (192) + prioridade urgente | Segurança do paciente; o bot não avalia clinicamente |
| 14 | Retenção de dados | 5 anos após o último contato → anonimização; anexos apagados 90 dias após a consulta | Minimização; o sistema não é o prontuário oficial |
| 15 | Hospedagem | VPS no Brasil, requisitos mínimos na seção 6 | LGPD e desempenho |
| 16 | Disponibilidade de horários pelo Atendente | Não se aplica (sem autoagendamento) | – |

**Dependências externas (não são decisões de sistema):**
1. Contratar a VPS (Mari / TI).
2. Chip/número institucional dedicado + celular com WhatsApp Business para
   conectar ao Baileys.
3. Equipe validar as mensagens – lista exata do que validar em
   `MENSAGENS.md`, seção 6 (textos ajustados, textos novos e conteúdo das
   Orientações).
4. Equipe de enfermagem/médica revisar a mensagem de urgência (U).
5. Encarregado (DPO) do hospital revisar o aviso de privacidade e a seção 6.1.
6. Links dos formulários de avaliação por serviço.

