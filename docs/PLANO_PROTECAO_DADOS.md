# Plano de Segurança e Proteção de Dados Pessoais (LGPD)
## PROCAPE – Saúde Digital: Bot de WhatsApp + Plataforma de Gestão

| | |
|---|---|
| **Controlador** | Universidade de Pernambuco (UPE) – PROCAPE, por meio do NUTES / Saúde Digital |
| **Responsável técnico** | Thompson Cysneiros (desenvolvimento, implantação e operação técnica) |
| **Encarregado (DPO)** | [a indicar pela UPE/PROCAPE] |
| **Versão** | 1.0 – planejamento (antes da produção) |
| **Revisão** | Anual, ou a cada mudança relevante no sistema, nos provedores ou na legislação |

> Este plano é um documento técnico-administrativo. Antes da produção deve
> ser **validado pelo encarregado (DPO) e pela assessoria jurídica da UPE**,
> que podem ajustar interpretações legais e prazos.

Documentos relacionados: `NECESSIDADES_TECNICAS.md` (infraestrutura e
backup), `REQUISITOS.md`, `BANCO_DE_DADOS.md`, `MENSAGENS.md`.

---

## Sumário
1. Objetivo e abrangência
2. Marco legal e normativo
3. Papéis e responsabilidades
4. Inventário de dados e registro das operações
5. Princípios da LGPD aplicados
6. Bases legais
7. Transparência
8. Direitos dos titulares
9. Segurança da informação
10. Ciclo de vida e tabela de temporalidade
11. Backup
12. Eliminação, anonimização e descarte
13. Compartilhamento, operadores e transferência internacional
14. Situações especiais
15. Gestão de incidentes de segurança
16. Relatório de Impacto (RIPD)
17. Pessoas: termos, capacitação e desligamento
18. Manutenção e gestão de mudanças
19. Monitoramento, auditoria e melhoria contínua
20. Responsabilização e sanções
21. Checklist de conformidade antes da produção
22. Anexos (modelos)

---

## 1. Objetivo e abrangência

Estabelecer as regras, controles e responsabilidades para que todo
tratamento de dados pessoais realizado pelo sistema – e pelas pessoas que o
operam – cumpra a **Lei nº 13.709/2018 (LGPD)** e normas correlatas,
protegendo os pacientes e evitando riscos à instituição.

Abrange:
- a plataforma de gestão, o bot de WhatsApp, o banco de dados, os anexos,
  os backups e os registros (logs);
- o celular e o chip do WhatsApp institucional;
- as estações de trabalho usadas pela equipe para acessar o sistema;
- a planilha usada na carga inicial e qualquer arquivo exportado;
- os provedores envolvidos (Oracle Cloud, WhatsApp/Meta, GitHub, SMTP, ATI).

---

## 2. Marco legal e normativo

| Norma | Pontos aplicáveis |
|---|---|
| **Lei 13.709/2018 – LGPD** | Arts. 5º (definições), 6º (princípios), 7º e 11 (bases legais), 9º (informação ao titular), 13 (pesquisa em saúde), 14 (crianças e adolescentes), 15–16 (término e conservação), 17–22 (direitos), 23–30 (poder público), 33–36 (transferência internacional), 37–41 (agentes e encarregado), 42–45 (responsabilidade), 46–49 (segurança e sigilo), 50–51 (boas práticas), 52–54 (sanções) |
| Resolução CD/ANPD nº 15/2024 | Regulamento de comunicação de incidente de segurança (prazo e conteúdo) |
| Resolução CD/ANPD nº 18/2024 | Atuação do encarregado |
| Resolução CD/ANPD nº 19/2024 | Transferência internacional de dados |
| Guia orientativo da ANPD para o Poder Público | Tratamento de dados por órgãos públicos |
| Lei 12.527/2011 – LAI | Art. 31: informações pessoais têm acesso restrito |
| Lei 13.787/2018 | Prontuário: guarda mínima de 20 anos (o sistema **não** é o prontuário – ver 10.2) |
| Resolução CFM 2.314/2022 (telemedicina) e normas do COFEN sobre telenfermagem | Registro do atendimento no prontuário e concordância do paciente com o atendimento a distância |
| Código Penal, art. 154; códigos de ética profissional | Sigilo profissional |
| Lei 14.534/2023 | CPF como número único de identificação |
| Normas internas da UPE e do Estado de Pernambuco | Política de privacidade e segurança da informação institucionais |

---

## 3. Papéis e responsabilidades (arts. 5º, 37–41)

| Papel (LGPD) | Quem | Responsabilidades |
|---|---|---|
| **Controlador** (art. 5º, VI) | UPE/PROCAPE (NUTES – Saúde Digital) | Decide finalidades e meios; responde pelo tratamento; aprova este plano; mantém o registro das operações (art. 37) e o RIPD (art. 38) |
| **Encarregado – DPO** (art. 41) | [indicar] | Canal com titulares e ANPD; orienta a equipe; valida este plano, o aviso de privacidade e o RIPD; coordena incidentes |
| **Responsável técnico** | Thompson Cysneiros | Desenvolve e opera o sistema segundo as instruções do controlador; implementa os controles de segurança (art. 46); apoia incidentes e solicitações de titulares. Se atuar como prestador externo, enquadra-se como **operador** (art. 5º, VII; art. 39) e deve assinar termo/contrato com cláusulas de proteção de dados |
| **Supervisão NUTES** | Coordenação da Saúde Digital | Gestão de usuários e permissões; atendimento às solicitações dos titulares; guarda de cópia da chave de backup; revisão periódica de acessos |
| **Usuários** (atendentes, equipes específicas) | Colaboradores | Usar o sistema só para a finalidade do atendimento; cumprir o termo de confidencialidade |
| **Operadores/suboperadores** | Oracle Cloud (hospedagem), provedor SMTP, ATI (fase 2) | Tratar só conforme instruções; segurança contratual (art. 39) |
| **Terceiro independente** | WhatsApp/Meta | Plataforma de comunicação usada para falar com o paciente (ver 13.2) |

---

## 4. Inventário de dados e registro das operações (art. 37)

### 4.1 Titulares
Pacientes do PROCAPE atendidos pela Saúde Digital (pré-cateterismo,
pós-angioplastia, fibrilação atrial, aconselhamento genético e outros),
eventualmente seus **responsáveis/familiares** que falam em nome deles, e os
**colaboradores** usuários do sistema.

### 4.2 Registro das operações de tratamento (ROPA)

| # | Dados | Categoria | Finalidade | Base legal | Origem | Onde fica | Quem acessa | Retenção |
|---|---|---|---|---|---|---|---|---|
| 1 | Nome, CPF, data de nascimento, prontuário (opcional) | Pessoal (identificação) | Identificar o paciente e evitar erro de pessoa | Art. 7º, III e VIII; art. 11, II, "b" e "f" | Paciente (bot), planilha da carga inicial, equipe | Banco (volume criptografado) | Equipe conforme nível/equipe | 10.1 |
| 2 | Telefones (até 3) | Pessoal (contato) | Comunicação sobre o atendimento | idem | idem | Banco | idem | 10.1 |
| 3 | Serviço/linha de cuidado, agendamentos, status, datas de procedimento e teleconsulta | **Sensível – saúde** | Organizar teleconsultas, procedimentos e acompanhamento | Art. 11, II, "f" e "b" | Equipe, paciente | Banco | idem | 10.1 |
| 4 | Conteúdo das conversas (texto, queixa informada) | **Sensível – saúde** | Triagem e atendimento | Art. 11, II, "f" | Paciente | Banco | Equipe da fila correspondente | 10.1 |
| 5 | Anexos (exames, documentos) | **Sensível – saúde** | Preparar a consulta; conferir identidade | Art. 11, II, "f" | Paciente | Volume de anexos criptografado | Equipe da fila | 90 dias após a consulta |
| 6 | Observações da equipe | Sensível (pode conter saúde) | Registrar ocorrências do atendimento | Art. 11, II, "f" | Equipe | Banco | Equipe | 10.1 |
| 7 | Registro de ciência do aviso de privacidade | Pessoal | Prestação de contas (art. 6º, X) | Art. 7º, II e III | Sistema | Banco | Supervisão | Enquanto houver cadastro + 5 anos |
| 8 | Logs de auditoria (quem viu/alterou o quê) | Pessoal (usuários e pacientes) | Segurança e responsabilização | Art. 7º, II; art. 46 | Sistema | Banco | Admin, Supervisão | 5 anos |
| 9 | Logs técnicos (servidor, Nginx) | Pessoal (IP, usuário) | Segurança e diagnóstico | Art. 7º, II e III; art. 46 | Sistema | Volume de boot | Responsável técnico | 90 dias |
| 10 | Dados de usuários (nome, e-mail, nível, 2FA) | Pessoal | Controle de acesso | Art. 7º, II e III | UPE | Banco | Admin, Supervisão | Desligamento + 5 anos (só para auditoria) |
| 11 | Solicitações de titulares e incidentes | Pessoal | Cumprir direitos e a Res. 15/2024 | Art. 7º, II | Titular, sistema | Banco | Supervisão, DPO | 5 anos |
| 12 | Backups (cópias de 1 a 11) | Todas | Continuidade e recuperação | Art. 46 | Sistema | Oracle (SP) e ATI/UPE, criptografados | Responsável técnico, Supervisão (chave) | 35 dias / mensais 6 meses |

O ROPA deve ser mantido atualizado pelo controlador; esta tabela é a versão
inicial.

### 4.3 Dados que **não** são coletados (minimização – art. 6º, III)
Endereço, nome da mãe, e-mail do paciente, raça/cor, religião, dados
financeiros, biometria, localização, fotos que não sejam exames/documentos
solicitados, diagnósticos detalhados e resultados de exames em texto.

---

## 5. Princípios da LGPD aplicados (art. 6º)

| Princípio | Como o sistema cumpre |
|---|---|
| I – Finalidade | Uso exclusivo para organização do atendimento da Saúde Digital; proibido uso para outros fins (marketing, pesquisa sem aprovação etc.) |
| II – Adequação | Cada dado coletado tem justificativa no inventário (4.2) |
| III – Necessidade | Cadastro mínimo (nome, CPF, prontuário opcional, telefones, data de nascimento); bot só envia primeiro nome, data, hora e link; anexos apagados em 90 dias; backups com retenção curta |
| IV – Livre acesso | Canal "meus dados" no WhatsApp e pelo encarregado; resposta em até 15 dias (art. 19) |
| V – Qualidade dos dados | Paciente confirma dados; correção pela equipe com registro em observações; validação de CPF e telefone |
| VI – Transparência | Aviso de privacidade no primeiro contato e publicação no site (art. 23, I) |
| VII – Segurança | Seção 9 |
| VIII – Prevenção | Privacy by design (9.8), testes com dados fictícios, RIPD (16) |
| IX – Não discriminação | Nenhuma decisão automatizada que afete o acesso ao atendimento; urgência apenas eleva prioridade, decisão é humana |
| X – Responsabilização e prestação de contas | Auditoria, ROPA, RIPD, registros de incidentes e solicitações, revisões periódicas |

---

## 6. Bases legais (arts. 7º, 11 e 23)

1. **Dados de saúde (sensíveis):**
   - art. 11, II, **"f"** – tutela da saúde, em procedimento realizado por
     profissionais de saúde ou serviços de saúde;
   - art. 11, II, **"b"** – execução de políticas públicas (atendimento SUS
     regulado);
   - art. 11, II, **"d"** – exercício regular de direitos em processo
     (fundamenta a guarda por 5 anos após o último contato – ver 10.1).
2. **Dados pessoais comuns:** art. 7º, **III** (políticas públicas),
   **VIII** (tutela da saúde) e **II** (obrigação legal/regulatória, ex.:
   registros de segurança).
3. **Poder público (art. 23):** tratamento para a finalidade pública do
   PROCAPE/UPE, com divulgação das hipóteses de tratamento (inciso I) e
   indicação do encarregado (inciso III).
4. **Consentimento não é a base legal.** O aviso de privacidade registra a
   **ciência** do paciente, não o consentimento; o atendimento não depende de
   aceite.
5. **Concordância com o atendimento a distância** (exigência ética da
   telemedicina/telenfermagem) é **outra coisa**: deve ser registrada pela
   equipe no prontuário oficial, conforme as normas do CFM/COFEN – não é
   base legal de tratamento de dados.
6. **Vedação** (art. 11, §4º): os dados de saúde **não** serão comunicados
   ou compartilhados com objetivo de vantagem econômica.

---

## 7. Transparência (arts. 9º, 23 e 41)

- **Aviso de privacidade** enviado pelo bot no primeiro contato do número
  (mensagem "A" de `MENSAGENS.md`) e registro de ciência.
- **Aviso completo** (Anexo A) publicado em página pública do sistema e/ou
  no site do PROCAPE/UPE, contendo os itens do art. 9º: finalidade, forma e
  duração do tratamento, identificação do controlador e do encarregado,
  compartilhamentos, responsabilidades e direitos do titular.
- **Identidade e contato do encarregado** divulgados publicamente (art. 41,
  §1º).
- **Comunicação simples**, em linguagem acessível ao paciente.

---

## 8. Direitos dos titulares (arts. 17–22)

### 8.1 Direitos atendidos
| Direito (art. 18) | Como é atendido |
|---|---|
| I – Confirmação da existência de tratamento | Resposta da Supervisão |
| II – Acesso aos dados | Exportação dos dados do paciente (relatório individual) |
| III – Correção | Equipe corrige; fica registrado em observações e auditoria |
| IV – Anonimização, bloqueio ou eliminação de dados desnecessários/excessivos | Análise da Supervisão + DPO; função de anonimização individual |
| V – Portabilidade | Exportação em formato estruturado (art. 25 – interoperável) |
| VI – Eliminação de dados tratados com consentimento | Não se aplica diretamente (base não é consentimento); pedidos de exclusão são analisados à luz do art. 16 |
| VII – Informação sobre compartilhamentos | Lista da seção 13 |
| VIII e IX – Informação sobre não consentir / revogação | Esclarecimento de que a base legal não é consentimento |
| Art. 20 – Revisão de decisões automatizadas | Não há decisões exclusivamente automatizadas; toda triagem pode ser revista pela equipe |

### 8.2 Procedimento
1. **Entrada:** palavra "meus dados" no WhatsApp, e-mail do encarregado ou
   presencialmente.
2. **Verificação de identidade:** CPF + data de nascimento e, se necessário,
   documento com foto – **antes** de entregar qualquer dado. Pedido feito por
   terceiro exige comprovação de representação.
3. **Registro** da solicitação (data, tipo, prazo) – motivo
   `direitos_titular` na fila da Supervisão.
4. **Prazo:** confirmação/acesso em formato simplificado **imediatamente**
   ou declaração completa em até **15 dias** (art. 19, I e II); demais
   pedidos no menor prazo possível, informando eventual impossibilidade e o
   motivo (art. 18, §4º).
5. **Gratuidade** (art. 18, §5º).
6. **Limites:** dados necessários ao cumprimento de obrigação legal ou à
   defesa da instituição podem ser conservados (art. 16), com explicação ao
   titular.
7. **Comunicação a terceiros** que receberam os dados, quando houver
   correção/eliminação (art. 18, §6º).
8. **Registro da resposta** guardado por 5 anos.

---

## 9. Segurança da informação (arts. 46–49)

### 9.1 Governança e controle de acesso
- 4 níveis de acesso (Administrador, Supervisão, Atendente, Equipe
  específica) e escopo por equipe – **mínimo privilégio**.
- Contas **individuais e intransferíveis**; proibido compartilhar login.
- **2FA obrigatório** para Administrador e Supervisão; recomendado para
  todos.
- Senhas: mínimo de 10 caracteres, hash argon2, bloqueio após 5 tentativas
  por 15 min, sessão expira em 30 min de inatividade.
- **Revisão trimestral de acessos** pela Supervisão (quem tem acesso, a qual
  nível, se ainda precisa).
- **Desligamento:** acesso bloqueado no mesmo dia (seção 17).
- CPF e telefone mascarados para quem está fora do escopo.

### 9.2 Criptografia e chaves
| Onde | Como |
|---|---|
| Em trânsito | HTTPS TLS 1.2+ (preferencialmente 1.3), HSTS; conexões internas locais |
| Em repouso – servidor | Volumes de dados e backup com **LUKS** + criptografia do provedor |
| Credenciais sensíveis no banco | Criptografia de campo (ex.: credenciais da conexão do WhatsApp, segredos de 2FA) |
| Backups | Criptografia na origem (AES-256) antes do envio |
| Chaves | Cofre de senhas; cópia lacrada sob guarda da Supervisão; rotação anual e imediata em caso de suspeita; nunca no repositório de código |

### 9.3 Servidor e rede
- Oracle região São Paulo (fase 1) / ATI (fase 2) – dados no Brasil.
- Modelo híbrido (`NECESSIDADES_TECNICAS.md`, seção 5): banco e serviços de
  dados instalados no sistema operacional com atualização automática;
  aplicação em contêineres sem privilégios e sem portas expostas.
- Entrada só 443/80; SSH apenas com chave, sem root, por IPs autorizados;
  ufw + Security List; fail2ban.
- Atualizações de segurança automáticas; reconstrução semanal das imagens
  da aplicação com verificação de vulnerabilidades.
- Antivírus (ClamAV) em todo anexo recebido; arquivo infectado é bloqueado
  e a equipe é avisada.
- Conta Oracle institucional, MFA, 2 administradores, auditoria da nuvem
  ativada.

### 9.4 Aplicação
- Validação de entradas, proteção contra injeção, XSS e CSRF; limite de
  requisições no login e na API.
- Autorização verificada **no servidor** em toda rota.
- Bot: só envia primeiro nome, data, hora, link; nunca CPF, diagnóstico ou
  resultado de exame.
- Exportação limitada a dados cadastrais e restrita a Admin/Supervisão, com
  registro.
- Sem dados pessoais em e-mails, notificações push ou mensagens de erro.

### 9.5 Registros (logs) e auditoria
- Auditoria de: login/logout, visualização de ficha, alterações,
  exportações, restaurações de backup, mudanças de permissão e configuração.
- Logs técnicos sem conteúdo de mensagens ou dados de saúde.
- Logs protegidos contra alteração (somente acréscimo) e guardados pelos
  prazos da seção 10.

### 9.6 Celular e chip do WhatsApp (responsabilidade do NUTES)
O celular do número institucional também guarda as conversas. Regras:
- aparelho **exclusivo**, sem outros aplicativos além do necessário;
- **bloqueio de tela** com senha forte; criptografia do aparelho ativada;
- **backup das conversas em nuvem pessoal (Google Drive/iCloud)
  desativado**; download automático de mídia para a galeria desativado;
- guardado em **local fechado** do NUTES, ligado à tomada e ao Wi-Fi
  institucional;
- ninguém usa o aparelho para conversar diretamente com pacientes – o
  atendimento é feito pela plataforma;
- conversas com mais de 90 dias apagadas do aparelho (o histórico oficial
  fica no sistema);
- em caso de **perda/roubo**: bloquear o chip na operadora, desconectar os
  aparelhos conectados, avaliar como incidente (seção 15);
- na troca do aparelho: apagar com restauração de fábrica e registrar.

### 9.7 Estações de trabalho da equipe
- Acesso ao sistema só pelo navegador; **proibido** salvar prints, fotos de
  tela, exportações ou anexos em computadores pessoais, pen drives,
  WhatsApp pessoal ou nuvens pessoais.
- Bloquear a tela ao se ausentar; não deixar a sessão aberta em computador
  compartilhado.
- Computadores com sistema atualizado e antivírus.
- **Planilhas antigas**: após a carga inicial, as cópias da planilha de
  pacientes em computadores e drives devem ser **eliminadas** e a eliminação
  registrada pela Supervisão (ou guardadas apenas em local institucional
  controlado, se houver obrigação de guarda).

### 9.8 Privacidade desde a concepção (art. 46, §2º) e desenvolvimento seguro
- Decisões de minimização já tomadas no projeto (4.3, 9.4).
- Repositório de código **sem dados pessoais**; segredos fora do código.
- Ambientes de desenvolvimento e homologação **somente com dados
  fictícios**; proibido copiar dados de produção para teste.
- Revisão de segurança antes de cada versão; dependências atualizadas.
- Toda nova funcionalidade que trate dados pessoais passa por avaliação de
  impacto (atualização do RIPD).

---

## 10. Ciclo de vida e tabela de temporalidade (arts. 15 e 16)

### 10.1 Prazos
| Dado | Prazo | Ação ao fim do prazo | Fundamento |
|---|---|---|---|
| Cadastro, agendamentos, observações e conversas | Enquanto em acompanhamento + **5 anos** após o último contato | **Anonimização** automática (10.3) | Art. 16 e art. 11, II, "d" (prazo prescricional quinquenal contra a Fazenda Pública – Decreto 20.910/1932) |
| Pré-cadastro que nunca interagiu (ex.: regulado que não respondeu) | **12 meses** sem interação | **Eliminação** | Necessidade (art. 6º, III) |
| Anexos (exames/documentos) | **90 dias** após a consulta | Eliminação (após a equipe incorporar o necessário ao prontuário oficial) | Necessidade |
| Logs de auditoria | 5 anos | Eliminação | Prestação de contas (art. 6º, X) |
| Logs técnicos | 90 dias | Eliminação | Necessidade |
| Registro de incidentes | Mínimo **5 anos** | Eliminação | Res. CD/ANPD 15/2024 |
| Solicitações de titulares | 5 anos | Eliminação | Prestação de contas |
| Usuários desligados | Bloqueio imediato; dados mantidos 5 anos só para auditoria | Eliminação | Prestação de contas |
| Backups | Diários 35 dias; mensais 6 meses | Expiração automática | Necessidade |
| Arquivos exportados | Uso imediato; apagar após o uso | Eliminação pelo usuário (registrada) | Necessidade |
| Planilha da carga inicial | Não guardada pelo sistema | – | Necessidade |
| Conversas no celular institucional | 90 dias | Apagar no aparelho | Necessidade |

### 10.2 O sistema não é o prontuário
O prontuário oficial (guarda mínima de 20 anos – Lei 13.787/2018) é o do
hospital. **Toda informação clínica relevante** obtida pelo sistema deve
ser registrada pela equipe no prontuário oficial. Isso permite que o
sistema aplique prazos curtos sem prejuízo assistencial.

### 10.3 Término do tratamento (art. 15)
O tratamento termina quando a finalidade é alcançada, o prazo vence, por
determinação da ANPD ou a pedido do titular quando cabível. A conservação
após o término só ocorre nas hipóteses do **art. 16** (obrigação legal,
pesquisa com anonimização, uso exclusivo do controlador anonimizado).

---

## 11. Backup

Detalhado em `NECESSIDADES_TECNICAS.md`, seção 7. Em resumo, conforme a
LGPD:
- 3 cópias (local, Oracle Object Storage São Paulo, storage ATI/UPE), todas
  **criptografadas na origem** e **no Brasil**;
- chaves separadas dos backups;
- retenção máxima de **6 meses**;
- **registro de eliminações reaplicado após toda restauração** – dado
  eliminado não volta;
- restauração apenas por pessoas autorizadas, registrada;
- testes de restauração em ambiente isolado, destruído ao final; backups
  nunca usados para testes;
- imutabilidade de 35 dias contra ransomware.

---

## 12. Eliminação, anonimização e descarte

### 12.1 Métodos
| Situação | Método |
|---|---|
| Eliminação de registro no banco | Exclusão definitiva (não apenas lógica) + registro no **log de eliminações** (identificador interno e data, sem dado pessoal) |
| Anonimização | Remoção definitiva de nome, CPF, telefones, data de nascimento (mantida só a **faixa etária**), prontuário, conteúdo das conversas, observações e anexos; datas reduzidas a **mês/ano**. Mantêm-se só dados estatísticos (serviço, status, quantidades). Hash de CPF **não** é aceito como anonimização (é pseudonimização) |
| Anexos | Exclusão do arquivo e de suas cópias em cache |
| Backups | Expiração automática; exclusão definitiva dos objetos ao fim da retenção |
| Volumes/servidores (fim do uso, migração) | Exclusão dos volumes, backups de volume e buckets na Oracle; antes, destruição da chave LUKS (eliminação criptográfica) |
| Celular | Restauração de fábrica, remoção do chip, registro |
| Papel | Não se usa papel; se houver impressão excepcional, fragmentação |
| Arquivos exportados / planilhas | Exclusão definitiva (inclusive da lixeira) e registro pela Supervisão |

### 12.2 Encerramento da fase Oracle (migração para a ATI)
1. Backup completo verificado e restaurado no servidor da ATI.
2. Validação do funcionamento e do registro de eliminações.
3. Destruição das chaves LUKS dos volumes Oracle e exclusão de instâncias,
   volumes, backups de volume e buckets.
4. Revogação de todas as credenciais usadas na Oracle.
5. **Termo de encerramento** assinado pelo responsável técnico e pela
   Supervisão, guardado por 5 anos (art. 47 – segurança mesmo após o
   término).

### 12.3 Rotinas automáticas
Tarefa diária no sistema: apaga anexos vencidos, elimina pré-cadastros sem
interação há 12 meses, anonimiza cadastros com 5 anos sem contato e registra
tudo no log de eliminações. Relatório mensal para a Supervisão.

---

## 13. Compartilhamento, operadores e transferência internacional (arts. 26, 27, 33, 39)

### 13.1 Operadores
| Operador | Dados | Garantias |
|---|---|---|
| Oracle Cloud (fase 1) | Todos (criptografados) | Região São Paulo; DPA aceito pela instituição; volumes LUKS; conta institucional com MFA |
| ATI-PE / UPE (fase 2 e cópia 3 do backup) | Todos (criptografados) | Órgão público; termo de cooperação/responsabilidade |
| Provedor SMTP institucional | E-mail dos usuários (sem dados de pacientes) | – |
| GitHub | Código-fonte, **sem dados pessoais** | Repositório privado |
| Responsável técnico (se externo) | Acesso técnico | Contrato/termo com cláusulas LGPD (Anexo B) |

### 13.2 WhatsApp / Meta
- O WhatsApp é o canal escolhido pelo próprio paciente para comunicação; as
  mensagens têm criptografia de ponta a ponta, mas **metadados** (número,
  horários) são tratados pela Meta, possivelmente fora do Brasil.
- Medidas: conteúdo mínimo nas mensagens (primeiro nome, data, hora, link);
  nunca diagnósticos, exames ou CPF; aviso de privacidade informa o uso do
  WhatsApp.
- Esse fluxo deve ser **avaliado no RIPD** quanto à transferência
  internacional (art. 33 e Res. CD/ANPD 19/2024).
- A migração para a **API oficial** (se a ATI viabilizar) traz relação
  contratual direta com a Meta e é preferível do ponto de vista da LGPD.

### 13.3 Compartilhamentos vedados ou restritos
- Nenhum compartilhamento com entidades privadas fora das hipóteses do
  art. 26, §1º e art. 27.
- Nenhum uso para publicidade ou vantagem econômica (art. 11, §4º).
- Pedidos de outros órgãos públicos: só com base legal e aprovação do DPO,
  registrados.

---

## 14. Situações especiais

| Situação | Regra |
|---|---|
| **Pesquisa – fibrilação atrial** (art. 13) | Uso dos dados para pesquisa só com projeto aprovado em Comitê de Ética (CEP/CONEP), dentro do órgão de pesquisa, em ambiente controlado e, sempre que possível, **anonimizados ou pseudonimizados**; resultados nunca revelam dados pessoais. A fila da equipe de pesquisa no sistema serve ao **atendimento**, não à extração de dados para estudo |
| **Crianças e adolescentes** (art. 14) | Atender ao melhor interesse; comunicação com o responsável legal; identificar o responsável no cadastro e nas observações |
| **Familiar/cuidador falando pelo paciente** | Comum em idosos. O telefone é registrado com a **relação** (cuidador, familiar) e o nome de quem atende. Recebe informações só após a verificação do paciente (data de nascimento em telefone validado; CPF + data de nascimento nos demais casos). Dúvida → equipe confirma o vínculo |
| **Número compartilhado** (vários pacientes no mesmo telefone) | O bot lista só **primeiro nome + inicial do sobrenome** e verifica a data de nascimento do escolhido; toda mensagem ativa diz de qual paciente se trata; uma pergunta aberta por vez no número, para não misturar respostas |
| **Troca de número / cadastro novo pelo bot** | Exige **foto de documento oficial com foto e CPF**; o número ou cadastro fica pendente até a equipe conferir; antes disso, nenhum dado é informado |
| **Tentativas de verificação** | 3 falhas → bloqueio de 30 min e aviso à equipe; tentativas registradas sem guardar o valor digitado |
| **Decisões automatizadas** (art. 20) | O bot apenas organiza e prioriza; nenhuma decisão de acesso a atendimento é tomada sem humano |
| **Dados de terceiros nas conversas** | Não são cadastrados; se relevantes, registrados de forma mínima em observação |
| **Pedidos de autoridades** | Encaminhados ao DPO/jurídico; atendidos só com fundamento legal; registrados |

---

## 15. Gestão de incidentes de segurança (art. 48 e Res. CD/ANPD 15/2024)

### 15.1 O que é incidente
Qualquer evento que comprometa a **confidencialidade, integridade ou
disponibilidade** de dados pessoais: acesso indevido, vazamento, envio de
mensagem ao paciente errado, perda/roubo do celular ou de um computador com
dados, invasão, ransomware, perda de backup, exportação indevida.

### 15.2 Fluxo de resposta
| Etapa | Prazo | Responsável | Ações |
|---|---|---|---|
| 1. Detecção e registro | Imediato | Quem identificar | Avisar responsável técnico e Supervisão; abrir registro (Anexo C) |
| 2. Contenção | Até 4 h | Responsável técnico | Bloquear acessos, revogar credenciais, isolar o servidor, desconectar o WhatsApp, preservar evidências e logs |
| 3. Avaliação de risco | Até 24 h | Responsável técnico + DPO | Quais dados, quantos titulares, se há dados sensíveis (saúde → risco elevado), possibilidade de dano relevante |
| 4. Comunicação à ANPD e aos titulares | **Até 3 dias úteis** do conhecimento, quando houver risco ou dano relevante | DPO / controlador | Conteúdo do art. 48, §1º: natureza dos dados, titulares envolvidos, medidas técnicas e de segurança, riscos, motivos de eventual demora, medidas adotadas para reverter ou mitigar |
| 5. Erradicação e recuperação | Conforme o caso | Responsável técnico | Corrigir a falha, restaurar de backup íntegro, reaplicar o log de eliminações |
| 6. Lições aprendidas | Até 30 dias | Todos | Atualizar este plano, o RIPD e os controles |

- **Registro de todos os incidentes**, inclusive os não comunicados, por no
  mínimo **5 anos**.
- Envolver a ATI quando o incidente tocar a infraestrutura estadual.
- Contatos de emergência (responsável técnico, Supervisão, DPO, ATI)
  mantidos atualizados no Anexo C.

---

## 16. Relatório de Impacto à Proteção de Dados – RIPD (art. 38)

Recomendado **antes da produção**, por envolver dados sensíveis de saúde em
volume contínuo, novo canal digital e provedor de nuvem. Conteúdo mínimo
(art. 38, parágrafo único):
1. Descrição dos dados tratados (seção 4).
2. Metodologia de coleta (bot, equipe, carga inicial).
3. Medidas de segurança (seção 9) e salvaguardas.
4. Análise de riscos e mitigação, incluindo:
   | Risco | Probabilidade | Impacto | Mitigação |
   |---|---|---|---|
   | Acesso indevido por usuário interno | Média | Alto | Níveis de acesso, auditoria, revisão trimestral |
   | Vazamento por invasão | Baixa | Alto | Hardening, patches, criptografia, 2FA |
   | Envio de mensagem para pessoa errada | Média | Médio | Validação de telefone, conteúdo mínimo nas mensagens |
   | Bloqueio do número (Baileys) | Média | Médio (disponibilidade) | Cadência de envios, API oficial como alternativa |
   | Perda/roubo do celular | Baixa | Médio | Regras da 9.6 |
   | Perda da conta Oracle | Baixa | Alto (disponibilidade) | 3ª cópia fora da Oracle, migração planejada |
   | Metadados no WhatsApp/Meta (exterior) | Alta | Baixo/Médio | Conteúdo mínimo, aviso de privacidade, API oficial |
   | Cópias locais de planilhas/exportações | Média | Alto | Regras da 9.7, exportação restrita e registrada |
5. Parecer do encarregado e aprovação do controlador.

---

## 17. Pessoas: termos, capacitação e desligamento

- **Termo de confidencialidade e uso** (Anexo B) assinado por todo usuário
  antes de receber acesso, inclusive o responsável técnico.
- **Capacitação** antes do primeiro acesso e **reciclagem anual**: LGPD
  básica, sigilo, phishing, uso do sistema, o que fazer em incidente.
- **Desligamento ou mudança de função:** Supervisão bloqueia o acesso no
  mesmo dia; revisão das sessões ativas; devolução de equipamentos.
- **Responsabilização:** violações podem gerar responsabilidade
  administrativa, civil e penal (seção 20).

---

## 18. Manutenção e gestão de mudanças

- Atualizações de segurança do sistema operacional: automáticas.
- Atualização da aplicação: testada em homologação com dados fictícios,
  implantada fora do horário de atendimento, com possibilidade de retorno à
  versão anterior.
- Toda mudança registrada (o quê, quando, quem, motivo).
- Mudança que altere dados tratados, finalidade ou provedor → revisão do
  ROPA, do aviso de privacidade e do RIPD antes de ir para produção.
- Teste mensal de restauração de backup; verificação semanal de integridade.
- Renovação automática de certificados com alerta de vencimento.

---

## 19. Monitoramento, auditoria e melhoria contínua

| Atividade | Frequência | Responsável |
|---|---|---|
| Revisão de acessos e permissões | Trimestral | Supervisão |
| Revisão dos logs de auditoria (acessos fora do padrão, exportações) | Mensal | Supervisão + responsável técnico |
| Relatório das rotinas de eliminação/anonimização | Mensal | Responsável técnico |
| Teste de restauração de backup | Mensal | Responsável técnico |
| Verificação de vulnerabilidades da aplicação | A cada versão / semanal nas imagens | Responsável técnico |
| Revisão deste plano, ROPA e RIPD | Anual | Controlador + DPO |
| Capacitação | Anual | Supervisão |

Indicadores: solicitações de titulares atendidas no prazo, incidentes,
acessos revogados no prazo, testes de restauração bem-sucedidos,
eliminações executadas.

---

## 20. Responsabilização e sanções (arts. 42–45 e 52)

- O controlador e o operador respondem por danos causados por tratamento em
  desacordo com a lei (art. 42); o operador responde solidariamente quando
  descumprir a lei ou as instruções do controlador (art. 42, §1º, I).
- O tratamento é irregular quando não fornece a segurança que o titular
  pode esperar (art. 44).
- Para órgãos e entidades públicas, a ANPD pode aplicar advertência,
  publicização da infração, bloqueio e eliminação dos dados e suspensão do
  tratamento (art. 52, §3º), sem prejuízo das responsabilidades
  administrativas, de improbidade e da LAI dos agentes públicos.
- Servidores e colaboradores respondem conforme o regime disciplinar e o
  termo de confidencialidade; sigilo profissional (Código Penal, art. 154).

---

## 21. Checklist de conformidade antes da produção

| # | Item | Responsável | ✔ |
|---|---|---|---|
| 1 | Encarregado (DPO) indicado e divulgado | Controlador | ☐ |
| 2 | Este plano validado pelo DPO/jurídico | DPO | ☐ |
| 3 | ROPA (4.2) aprovado | Controlador | ☐ |
| 4 | RIPD elaborado e aprovado | DPO | ☐ |
| 5 | Aviso de privacidade (bot + página pública) aprovado e publicado | DPO / NUTES | ☐ |
| 6 | Conta Oracle institucional, MFA, DPA aceito | NUTES / responsável técnico | ☐ |
| 7 | Volumes LUKS, backups criptografados nas 3 cópias, teste de restauração realizado | Responsável técnico | ☐ |
| 8 | Registro de eliminações e rotinas automáticas testados | Responsável técnico | ☐ |
| 9 | 2FA ativo para Admin/Supervisão | Responsável técnico | ☐ |
| 10 | Termos de confidencialidade assinados por todos os usuários | Supervisão | ☐ |
| 11 | Capacitação inicial realizada | Supervisão | ☐ |
| 12 | Celular/chip configurados conforme 9.6 | NUTES | ☐ |
| 13 | Planilhas antigas eliminadas após a carga inicial (9.7) | Supervisão | ☐ |
| 14 | Fluxo de incidentes e contatos de emergência definidos (Anexo C) | DPO | ☐ |
| 15 | Contrato/termo LGPD com o responsável técnico (se externo) | Controlador | ☐ |
| 16 | Ambiente de testes sem dados reais | Responsável técnico | ☐ |

---

## 22. Anexos (modelos)

### Anexo A – Aviso de privacidade (versão completa para publicação)
> **Aviso de Privacidade – Saúde Digital PROCAPE**
>
> **Quem trata seus dados:** Universidade de Pernambuco – PROCAPE, por meio
> do NUTES / Saúde Digital. **Encarregado:** [nome e contato].
>
> **Quais dados:** nome, CPF, data de nascimento, número de prontuário (se
> houver), telefones, informações sobre seus agendamentos e as mensagens e
> arquivos que você enviar por este WhatsApp.
>
> **Para quê:** organizar e realizar seu atendimento pela Saúde Digital –
> agendamento, confirmação, lembretes, envio do link da teleconsulta,
> orientações e encaminhamento para a equipe.
>
> **Por que podemos tratar:** para a proteção da sua saúde e a execução de
> políticas públicas de saúde (LGPD, arts. 7º e 11). Não usamos seus dados
> para outras finalidades nem os vendemos ou compartilhamos para fins
> comerciais.
>
> **Com quem:** com a equipe do PROCAPE responsável pelo seu atendimento e
> com os provedores de tecnologia que hospedam o sistema no Brasil. As
> mensagens trafegam pelo WhatsApp.
>
> **Por quanto tempo:** pelo período do seu acompanhamento e até 5 anos
> após o último contato; depois, os dados são anonimizados. Arquivos
> enviados são apagados 90 dias após a consulta.
>
> **Segurança:** acesso restrito por perfil, criptografia, registro de
> acessos e cópias de segurança protegidas.
>
> **Seus direitos:** confirmar se tratamos seus dados, acessar, corrigir,
> pedir anonimização ou eliminação de dados desnecessários, saber com quem
> compartilhamos e pedir revisão de decisões. Escreva **"meus dados"** neste
> WhatsApp ou contate o encarregado. Você também pode reclamar à ANPD.

### Anexo B – Termo de confidencialidade e uso (resumo dos compromissos)
O usuário declara que: usará o sistema apenas para o atendimento; não
compartilhará login; manterá sigilo sobre todas as informações de pacientes,
inclusive após o desligamento (art. 47); não copiará dados para meios
pessoais (computador, celular, nuvem, WhatsApp pessoal); comunicará
imediatamente qualquer incidente; está ciente de que seus acessos são
registrados e de que violações geram responsabilidade administrativa, civil
e penal.

### Anexo C – Registro de incidente (campos)
Data/hora da descoberta · quem identificou · descrição · dados e titulares
afetados (quantidade, se há dados de saúde) · causa provável · medidas de
contenção · avaliação de risco · decisão sobre comunicação à ANPD/titulares
(e data, se comunicado) · medidas corretivas · lições aprendidas ·
responsável pelo registro.
Contatos de emergência: responsável técnico [ ] · Supervisão NUTES [ ] ·
DPO [ ] · ATI [ ].

### Anexo D – Formulário de solicitação do titular (campos)
Data de entrada · canal · nome e CPF do titular · solicitante (próprio ou
representante e comprovação) · direito solicitado (art. 18) · verificação
de identidade realizada (como) · prazo · resposta e data · responsável.
