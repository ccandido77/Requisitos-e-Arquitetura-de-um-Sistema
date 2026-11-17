# Agenda TEA — Levantamento de Requisitos e Arquitetura Cliente-Servidor

**Disciplina:** Projetos de Sistemas
**Instituição:** CIESA — Centro Universitário de Ensino Superior do Amazonas
**Curso:** Análise e Desenvolvimento de Sistemas
**Aluno:** Carlos Cândido · 25000656@estudante.ciesa.br
**Modalidade:** trabalho individual
**Data:** 17 de novembro de 2026

---

## Nota sobre o objeto do trabalho

O enunciado admite um negócio fictício situado em Manaus. Este trabalho usa, em lugar disso, um
**negócio real e em desenvolvimento**: o **Agenda TEA**, cuja comunidade parceira é o **Centro de
Educação Especial Maria das Graças, em Manaus** (90 alunos e 19 profissionais). Todos os requisitos,
decisões e tecnologias descritos aqui foram extraídos do próprio repositório do projeto
(`docs/PROBLEMA.md`, `docs/REQUISITOS.md`, `docs/ARQUITETURA.md`, `docs/SEGURANCA.md`, os
`package.json` das aplicações e o código em `apps/` e `packages/`), e não de suposição de produto.
A numeração de requisitos, regras e decisões deste documento é própria do trabalho acadêmico; a
correspondência com os identificadores do repositório está na seção 2.6. Onde o modelo do enunciado
pede um elemento que o sistema real não possui — o módulo de Pagamentos do Nível 3 —, a ausência
está declarada e justificada na seção 2.1.3, em vez de preenchida com ficção.

---

## Identificação do negócio

| Item                        | Definição                                                                                                                               |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Nome do negócio/sistema** | **Agenda TEA**                                                                                                                          |
| **Natureza**                | SaaS B2B por assinatura, com escopo de dados isolado por unidade escolar (_multi-tenant_) e identidade visual adaptável (_white-label_) |
| **Cliente pagante**         | A instituição (escola ou centro de educação especial), não a família                                                                    |
| **Praça inicial**           | Manaus (AM) — comunidade parceira: Centro de Educação Especial Maria das Graças                                                         |

### Proposta de valor

Substituir a agenda de papel da educação especial por um **registro diário estruturado, datado,
atribuído a um autor identificado e auditável**, que (a) reproduz fielmente o formulário de papel
que a escola já usa, para não criar trabalho duplicado; (b) acrescenta o que o papel não consegue
fazer — confirmação real de administração de medicação, registro estruturado de crise, confirmação
de leitura individual por responsável e alerta que persiste até a confirmação; e (c) nasce em
conformidade com o Art. 14 da Lei nº 13.709/2018 (LGPD), tratando dado de saúde de criança sob o
regime mais exigente da lei. O diferencial declarado é **declarar a lacuna**: o sistema informa
explicitamente o que _não_ foi registrado, em vez de omitir.

### A dor local que o sistema resolve

Na agenda de papel, _"não comeu"_ e _"ninguém anotou"_ produzem exatamente o mesmo resultado visível
para a família: um campo em branco. Entre **25% e 30%** das crianças com TEA são não verbais ou têm
fala minimamente funcional — para elas, o registro do adulto da escola é a **única versão existente
do dia**, porque a família não pode perguntar ao filho como foi. Quando o caderno volta incompleto,
ilegível, esquecido na escola ou molhado na mochila — risco concreto no clima de Manaus —, decisões
de cuidado (ajuste de rotina, ida ao neuropediatra, mudança de medicação) passam a ser tomadas por
adivinhação. Para a instituição de Manaus, acumula-se uma segunda dor: ela não tem **prova** de que
a informação crítica chegou à família, porque a rubrica no caderno nunca demonstrou leitura efetiva.
E não há, nesse perfil de instituição, equipe de TI para operar software complexo — o que torna o
custo de adoção, e não a funcionalidade, o verdadeiro gargalo.

### O vazio de mercado ocupado

Plataformas de comunicação escolar têm boa usabilidade, mas não modelam medicação, crise ou
evacuação. Sistemas clínicos têm rigor, mas nenhum professor os opera em dois minutos no fim do
expediente, com uma criança em cada mão. O Agenda TEA ocupa a interseção: **rigor clínico na mão de
quem tem dois minutos**.

---

# 📌 PARTE 1 — LEVANTAMENTO DE REQUISITOS

## 1.1 Descrição do sistema

O **Agenda TEA** é um sistema de agenda digital integrada, vendido por assinatura a escolas e
centros de educação especial de Manaus, que substitui a agenda de papel usada como único canal
diário de comunicação entre a instituição e a família de crianças com Transtorno do Espectro Autista
(TEA), transformando o registro manuscrito em um registro estruturado, datado, atribuído a um autor
identificado e auditável, com confirmação de administração de medicação, registro estruturado de
crise e confirmação de leitura por responsável. O sistema é operado por **sete tipos de usuários**,
que correspondem aos sete papéis do código (`PapelUsuario`):

- o **responsável** (`RESPONSAVEL` — pai, mãe ou tutor legal, que consome o registro e confirma a
  leitura);
- o **professor** (`PROFESSOR` — docente da turma, que produz o registro diário no fim do
  expediente);
- o **pedagogo** (`PEDAGOGO` — apoio pedagógico, que consulta padrões, orienta a equipe e também
  pode registrar sobre as crianças da turma a que está vinculado);
- o **terapeuta** (`TERAPEUTA` — terapeuta ocupacional, fonoaudiólogo ou psicólogo da instituição,
  vinculado diretamente às crianças que atende, sobre as quais registra rotina, crise e medicação);
- a **coordenação** (`COORDENACAO` — gestão da unidade, que opera no escopo da própria unidade:
  cadastra turmas, crianças, contas, vínculos e planos de medicação, personaliza a identidade visual
  da escola e pode registrar e consultar os registros das crianças da unidade);
- o **administrador** da unidade (`ADMIN` — gestão técnica local de contas e vínculos e atendimento
  aos pedidos de exportação e exclusão de dados do titular; não produz conteúdo clínico, e toda
  leitura que faz de registro de criança é gravada na trilha de auditoria);
- o **superadministrador** da plataforma SaaS (`SUPER_ADMIN` — operador global que provisiona novas
  instituições e configura seus módulos e sua marca, sem acesso a dado de criança nem à trilha de
  auditoria).

O neuropediatra e a demais equipe de saúde **não são usuários do sistema**: recebem da família o
relatório consolidado exportado.

## 1.2 Requisitos Funcionais (RF)

| #        | Requisito funcional                                                                                                                                                                                                                                                                                                                         |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RF01** | O sistema deve permitir que **o usuário** autentique-se com e-mail e senha e receba apenas as permissões do seu perfil, dos seus vínculos e dos módulos contratados pela sua unidade.                                                                                                                                                       |
| **RF02** | O sistema deve permitir que **o professor** registre, em um único formulário, todos os campos da agenda diária de uma criança da sua turma — entrada, saída, estado geral, alimentação, água, sono, evacuação, febre, terapias e anotações livres.                                                                                          |
| **RF03** | O sistema deve permitir que **o professor** registre uma crise informando horário, gatilho percebido, intensidade, duração e a estratégia que acalmou a criança.                                                                                                                                                                            |
| **RF04** | O sistema deve permitir que **o professor** confirme a administração de uma dose prevista de medicação, informando o horário e o responsável pela administração.                                                                                                                                                                            |
| **RF05** | O sistema deve permitir que **o professor** justifique, com motivo obrigatório, a não administração de uma dose prevista de medicação.                                                                                                                                                                                                      |
| **RF06** | O sistema deve permitir que **o professor** corrija um registro de sua própria autoria, com justificativa obrigatória após a janela de correção direta de 24 horas.                                                                                                                                                                         |
| **RF07** | O sistema deve permitir que **o terapeuta** registre a rotina, a crise e a medicação do dia da criança que atende por vínculo direto, com os mesmos formulários usados pelo professor.                                                                                                                                                      |
| **RF08** | O sistema deve permitir que **o responsável** aceite o termo de consentimento de tratamento de dados antes de qualquer visualização de dado da criança.                                                                                                                                                                                     |
| **RF09** | O sistema deve permitir que **o responsável** visualize o resumo do dia da criança vinculada, incluindo a lista explícita dos campos que não foram registrados.                                                                                                                                                                             |
| **RF10** | O sistema deve permitir que **o responsável** confirme a leitura do resumo diário, gerando um registro datado e atribuído que substitui a assinatura no caderno.                                                                                                                                                                            |
| **RF11** | O sistema deve permitir que **o responsável** consulte a linha do tempo e o histórico de registros da criança por período.                                                                                                                                                                                                                  |
| **RF12** | O sistema deve permitir que **o responsável** exporte o relatório consolidado da semana para levar à equipe de saúde.                                                                                                                                                                                                                       |
| **RF13** | O sistema deve permitir que **o responsável** solicite a exportação e a exclusão dos dados dos quais é titular ou representante legal.                                                                                                                                                                                                      |
| **RF14** | O sistema deve permitir que **o profissional vinculado à criança (professor, pedagogo ou terapeuta) e a coordenação** aprovem ou rejeitem o texto gerado por inteligência artificial antes do envio à família.                                                                                                                              |
| **RF15** | O sistema deve permitir que **a coordenação** cadastre turmas, crianças, contas e vínculos da própria unidade escolar, bem como personalize o nome de exibição, as cores e o logotipo da instituição.                                                                                                                                       |
| **RF16** | O sistema deve permitir que **a coordenação** importe turmas, crianças e responsáveis em lote a partir de uma planilha CSV, com pré-visualização antes da gravação.                                                                                                                                                                         |
| **RF17** | O sistema deve permitir que **a coordenação** envie convite por e-mail com token de uso único, para que o convidado defina a própria senha.                                                                                                                                                                                                 |
| **RF18** | O sistema deve permitir que **a coordenação** consulte o log de auditoria das ações realizadas na sua unidade.                                                                                                                                                                                                                              |
| **RF19** | O sistema deve permitir que **o administrador** gerencie contas e vínculos da sua unidade e atenda aos pedidos de exportação e exclusão de dados do titular, enquanto o **superadministrador** provisiona novas unidades e gerencia seus módulos ativos.                                                                                    |
| **RF20** | O sistema deve permitir que **o responsável** receba no próprio aparelho notificação por Web Push de crise registrada — prioritária quando a crise é severa — e alerta de dose prevista de medicação não confirmada após o horário, com lembrete por e-mail enquanto a leitura de um resumo com medicação ou crise severa estiver pendente. |
| **RF21** | O sistema deve permitir que **o professor** anexe fotos ao registro do dia da criança da sua turma.                                                                                                                                                                                                                                         |

> **Conformidade com o mínimo exigido:** 21 requisitos funcionais, todos no formato _"O sistema deve
> permitir que [usuário] [ação]"_, distribuídos entre os perfis descritos em 1.1.

> **Situação no código em 17/11/2026.** Estão implementados RF01 a RF12, RF14 a RF17 e a parte de
> RF19 que trata de contas, vínculos e provisionamento. Dependem de construção, programada pela
> Etapa 2 para depois da virada de chave: **RF13** e a parte de **RF19** sobre pedidos do titular
> (sem rota de exportação nem de exclusão), **RF18** (a permissão existe na matriz, mas não há rota
> de consulta da trilha) e **RF21** (a tabela `anexo` existe no esquema e o módulo `ANEXO_FOTOS`
> vem desligado, mas não há adaptador de armazenamento de objetos). O professor é o ator nominal de
> RF02 a RF06, mas a matriz de permissões concede as mesmas ações ao pedagogo, ao terapeuta e à
> coordenação, cada um no seu caminho de vínculo.

## 1.3 Requisitos Não Funcionais (RNF)

Todos os RNF abaixo são **mensuráveis**: cada um declara limite numérico, percentual, prazo ou norma
técnica verificável. Nenhum usa formulação vaga.

| #         | Categoria                         | Requisito não funcional mensurável                                                                                                                                                                                                                                                                                                                                                                                                                              |
| --------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RNF01** | **Desempenho**                    | No percentil 95, salvar um registro deve concluir em **menos de 500 ms**; carregar o resumo do dia, em **menos de 2 s em rede 4G**; consultar o histórico de 30 dias, em **menos de 1 s**.                                                                                                                                                                                                                                                                      |
| **RNF02** | **Usabilidade**                   | O preenchimento completo da agenda diária de uma criança deve ser concluído em **no máximo 2 minutos**, operável **com uma mão**, e obter nota **SUS ≥ 75** em teste com no mínimo 5 profissionais da instituição parceira.                                                                                                                                                                                                                                     |
| **RNF03** | **Acessibilidade**                | A interface deve atender à **WCAG 2.1 nível AA**: contraste mínimo de **4,5:1** em texto normal (inclusive nas paletas personalizadas por escola, que a API recusa quando não atingem o contraste), alvos de toque de no mínimo **44 × 44 px**, navegação completa por teclado e **0 violação de severidade crítica** reportada pelo axe-core na integração contínua.                                                                                           |
| **RNF04** | **Segurança**                     | Senhas armazenadas com **argon2id**; tráfego exclusivamente sobre **TLS 1.2 ou superior**; _access token_ com validade de **15 minutos** e _refresh token_ de **7 dias** com rotação a cada uso; no máximo **5 tentativas de autenticação por minuto por endereço IP**, respondidas com **HTTP 429**; isolamento _multi-tenant_ obrigatório no ORM (_fail-closed_); os **10 itens do OWASP Top 10** documentados item a item com a mitigação correspondente.    |
| **RNF05** | **Disponibilidade**               | Disponibilidade mínima de **99% na janela escolar de 07h às 19h em dias úteis** — o que admite no máximo **7,2 minutos** de indisponibilidade por dia útil. A indisponibilidade de **100%** do provedor de inteligência artificial **não** pode impedir a entrega do resumo diário estruturado.                                                                                                                                                                 |
| **RNF06** | **Compatibilidade**               | O cliente deve funcionar como **PWA**, sem instalação por loja de aplicativos, nas **duas últimas versões maiores** de Chrome, Safari e Edge, em **Android 10+** e **iOS 16+**, em telas a partir de **320 px** de largura.                                                                                                                                                                                                                                     |
| **RNF07** | **Conformidade e auditabilidade** | **100%** das ações críticas devem gravar autor, ação e resultado na **mesma transação atômica** do dado que as originou; retenção de dados declarada de **5 anos**, com anonimização ao fim do prazo; **0** ocorrência de dado pessoal identificável direto transmitido a provedor externo de modelo de linguagem.                                                                                                                                              |
| **RNF08** | **Manutenibilidade**              | Cobertura de testes de **no mínimo 70%** no pacote de domínio (limite configurado no `vitest.config` do domínio e medido por `pnpm test:coverage`; medição de 17/11/2026: **98%** das instruções e **92,8%** dos ramos); **0 erro** de análise estática e de checagem de tipos no repositório; a suíte completa (**981 testes** na medição de 17 de novembro de 2026) executada como portão obrigatório na integração contínua a cada alteração.                |
| **RNF09** | **Escalabilidade**                | A arquitetura deve suportar **no mínimo 50 unidades escolares** e **4.500 crianças** sem alteração estrutural, com isolamento de dados resolvido no servidor por unidade via `AsyncLocalStorage` e processos **sem estado de sessão em memória** — o único estado em memória admitido é cache descartável (configuração do locatário por 30 s, lista de origens de CORS) e o contador de tentativas, cujo efeito com várias réplicas está declarado no ADR-002. |

> **Cobertura de categorias exigida:** o enunciado pede no mínimo 4 das 5 categorias listadas; este
> documento cobre **todas as 5** — desempenho (RNF01), usabilidade/acessibilidade (RNF02 e RNF03),
> segurança (RNF04), disponibilidade (RNF05) e compatibilidade (RNF06) — e acrescenta conformidade
> (RNF07), manutenibilidade (RNF08) e escalabilidade (RNF09).

## 1.4 Regras de Negócio (RN)

As regras abaixo **existem independentemente do sistema**: todas vigoram hoje, na agenda de papel, na
rotina da instituição ou na legislação brasileira. O software apenas as **passa a impor**; nenhuma
delas é uma funcionalidade.

| #        | Regra de negócio                                                                                                                                                                                                                                                                                                                                                                                                                                           | Por que existe sem o sistema                                                                                                                                                                                                                                                                                           |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RN01** | Medicação só pode ser administrada a uma criança na instituição se houver **prescrição médica vigente** e **autorização escrita do responsável legal**, com dose e horário definidos pelo prescritor — nunca pela escola.                                                                                                                                                                                                                                  | Decorre da responsabilidade civil e sanitária da instituição; hoje se materializa no envelope de receita que a família entrega na secretaria.                                                                                                                                                                          |
| **RN02** | O tratamento de dados de saúde de uma criança exige **consentimento específico e destacado de ao menos um dos responsáveis legais** e deve ocorrer sempre no melhor interesse da criança.                                                                                                                                                                                                                                                                  | Art. 14 da Lei nº 13.709/2018 (LGPD). Vigora com ou sem software; hoje é o termo assinado na matrícula.                                                                                                                                                                                                                |
| **RN03** | Uma criança pode ter **mais de um responsável legal**, e todos têm **igual direito** de ser informados sobre o dia dela — o direito de um não se extingue porque o outro foi informado.                                                                                                                                                                                                                                                                    | Decorre do poder familiar (Código Civil) e é rotina em famílias com guarda compartilhada; hoje gera o conflito do caderno que só vai para a casa de um dos pais.                                                                                                                                                       |
| **RN04** | Somente o **profissional vinculado** à criança relata o dia dela: quem não a atendeu não produz registro sobre ela. O vínculo do professor e do pedagogo é a turma; o do terapeuta, a própria criança; o da coordenação, a unidade que ela coordena.                                                                                                                                                                                                       | É a responsabilidade profissional pelo que se afirma — o professor assina o que viu. Vigora na agenda de papel, onde quem preenche é quem esteve com a turma.                                                                                                                                                          |
| **RN05** | Um registro de cuidado **não é apagado**: a correção posterior preserva o valor anterior e exige justificativa de quem corrigiu.                                                                                                                                                                                                                                                                                                                           | Prática de prontuário e de registro escolar — no caderno, corrige-se riscando e rubricando ao lado, nunca apagando.                                                                                                                                                                                                    |
| **RN06** | A função **administrativa** da instituição (secretaria, TI, operador da plataforma) **não produz conteúdo clínico ou pedagógico** sobre a criança; isso cabe ao profissional habilitado que a atendeu. O administrador local (`ADMIN`) só **consulta** esse conteúdo com cada leitura registrada na trilha de auditoria; a coordenação atua no escopo da própria unidade; o operador da plataforma (`SUPER_ADMIN`) não tem acesso algum a dado de criança. | Separação de atribuições profissionais da instituição; a secretária não escreve na agenda da criança hoje, e quem abre a pasta de um aluno o faz por um motivo que pode ser cobrado.                                                                                                                                   |
| **RN07** | Evento com **risco à integridade física** da criança — crise severa, febre, uso de medicação de resgate — deve ser comunicado ao responsável **no mesmo dia**, não no boletim do mês.                                                                                                                                                                                                                                                                      | Dever de cuidado e de informação da instituição; hoje se cumpre com uma ligação telefônica. No sistema, a crise gera notificação imediata, prioritária se severa, e a dose não confirmada gera alerta (RF20); febre e medicação de resgate chegam pelo resumo do dia, publicado no mesmo dia, sem notificação própria. |
| **RN08** | O registro do acompanhamento da criança deve ser **guardado por 5 anos**, prazo declarado no consentimento, para fins pedagógicos e médicos.                                                                                                                                                                                                                                                                                                               | Prazo de guarda documental já praticado pela instituição para a documentação escolar e para a continuidade do cuidado em saúde.                                                                                                                                                                                        |

## 1.5 Tabela única de requisitos

| Código | Tipo                             | Descrição                                                                                                                                                                   | Prioridade |
| ------ | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| RF01   | Funcional                        | O sistema deve permitir que o usuário autentique-se com e-mail e senha e receba apenas as permissões do seu perfil, vínculos e módulos contratados.                         | Alta       |
| RF02   | Funcional                        | O sistema deve permitir que o professor registre, em um único formulário, todos os campos da agenda diária de uma criança da sua turma.                                     | Alta       |
| RF03   | Funcional                        | O sistema deve permitir que o professor registre uma crise com horário, gatilho, intensidade, duração e estratégia de contenção.                                            | Alta       |
| RF04   | Funcional                        | O sistema deve permitir que o professor confirme a administração de uma dose prevista de medicação, com horário e responsável.                                              | Alta       |
| RF05   | Funcional                        | O sistema deve permitir que o professor justifique, com motivo obrigatório, a não administração de uma dose prevista.                                                       | Alta       |
| RF06   | Funcional                        | O sistema deve permitir que o professor corrija registro de sua própria autoria, com justificativa após a janela de 24 h de correção direta.                                | Média      |
| RF07   | Funcional                        | O sistema deve permitir que o terapeuta registre rotina, crise e medicação do dia da criança que atende por vínculo direto.                                                 | Média      |
| RF08   | Funcional                        | O sistema deve permitir que o responsável aceite o termo de consentimento antes de qualquer visualização de dado da criança.                                                | Alta       |
| RF09   | Funcional                        | O sistema deve permitir que o responsável visualize o resumo do dia, incluindo a lista do que não foi registrado.                                                           | Alta       |
| RF10   | Funcional                        | O sistema deve permitir que o responsável confirme a leitura do resumo diário, com registro datado e atribuído.                                                             | Alta       |
| RF11   | Funcional                        | O sistema deve permitir que o responsável consulte a linha do tempo e o histórico por período.                                                                              | Média      |
| RF12   | Funcional                        | O sistema deve permitir que o responsável exporte o relatório consolidado da semana.                                                                                        | Média      |
| RF13   | Funcional                        | O sistema deve permitir que o responsável solicite exportação e exclusão dos dados de que é titular ou representante.                                                       | Alta       |
| RF14   | Funcional                        | O sistema deve permitir que o profissional vinculado (professor, pedagogo ou terapeuta) e a coordenação aprovem ou rejeitem o texto gerado por IA antes do envio à família. | Média      |
| RF15   | Funcional                        | O sistema deve permitir que a coordenação cadastre turmas, crianças, contas e vínculos da própria unidade e personalize sua marca.                                          | Alta       |
| RF16   | Funcional                        | O sistema deve permitir que a coordenação importe turmas, crianças e responsáveis em lote por planilha CSV, com pré-visualização.                                           | Média      |
| RF17   | Funcional                        | O sistema deve permitir que a coordenação envie convite por e-mail com token de uso único para definição da própria senha.                                                  | Alta       |
| RF18   | Funcional                        | O sistema deve permitir que a coordenação consulte o log de auditoria da sua unidade.                                                                                       | Média      |
| RF19   | Funcional                        | O sistema deve permitir que o administrador local gerencie contas/vínculos e atenda pedidos LGPD, e o superadministrador provisione unidades e módulos.                     | Alta       |
| RF20   | Funcional                        | O sistema deve permitir que o responsável receba push de crise registrada (prioritário se severa) e alerta de dose prevista não confirmada.                                 | Alta       |
| RF21   | Funcional                        | O sistema deve permitir que o professor anexe fotos ao registro do dia da criança da sua turma.                                                                             | Baixa      |
| RNF01  | Não funcional — Desempenho       | p95 < 500 ms para salvar registro; < 2 s em 4G para o resumo do dia; < 1 s para histórico de 30 dias.                                                                       | Alta       |
| RNF02  | Não funcional — Usabilidade      | Preenchimento da agenda em ≤ 2 minutos, operável com uma mão, nota SUS ≥ 75 com ao menos 5 profissionais.                                                                   | Alta       |
| RNF03  | Não funcional — Acessibilidade   | WCAG 2.1 AA: contraste ≥ 4,5:1 (validado em paletas customizadas), alvos ≥ 44 × 44 px, navegação por teclado, 0 violação crítica no axe-core.                               | Alta       |
| RNF04  | Não funcional — Segurança        | argon2id; TLS 1.2+; token de 15 min e refresh de 7 dias com rotação; máx. 5 tentativas/min por IP com HTTP 429; isolamento fail-closed; OWASP Top 10 documentado.           | Alta       |
| RNF05  | Não funcional — Disponibilidade  | ≥ 99% entre 07h e 19h em dias úteis (≤ 7,2 min/dia útil); falha total da IA não impede o resumo estruturado.                                                                | Alta       |
| RNF06  | Não funcional — Compatibilidade  | PWA sem loja de aplicativos; duas últimas versões maiores de Chrome, Safari e Edge; Android 10+, iOS 16+; telas ≥ 320 px.                                                   | Alta       |
| RNF07  | Não funcional — Conformidade     | 100% das ações críticas auditadas na mesma transação; retenção de 5 anos com anonimização; 0 PII direta enviada a LLM externo.                                              | Alta       |
| RNF08  | Não funcional — Manutenibilidade | Cobertura ≥ 70% no domínio (medida: 98% das instruções, 92,8% dos ramos); 0 erro de lint e de tipos; 981 testes como portão obrigatório por alteração.                      | Média      |
| RNF09  | Não funcional — Escalabilidade   | ≥ 50 unidades escolares e 4.500 crianças sem alteração estrutural; sem estado de sessão em memória; isolamento por unidade no servidor via ALS.                             | Média      |
| RN01   | Regra de negócio                 | Medicação exige prescrição médica vigente e autorização escrita do responsável legal, com dose e horário definidos pelo prescritor.                                         | Alta       |
| RN02   | Regra de negócio                 | Tratamento de dado de saúde de criança exige consentimento específico e destacado de ao menos um responsável legal (LGPD, Art. 14).                                         | Alta       |
| RN03   | Regra de negócio                 | Uma criança pode ter vários responsáveis legais, e todos têm igual direito de ser informados.                                                                               | Alta       |
| RN04   | Regra de negócio                 | Somente o profissional vinculado à criança (por turma, por atendimento ou pela unidade que coordena) produz registro sobre ela.                                             | Alta       |
| RN05   | Regra de negócio                 | Registro de cuidado não é apagado; a correção preserva o valor anterior e exige justificativa.                                                                              | Alta       |
| RN06   | Regra de negócio                 | A função administrativa não produz conteúdo clínico; o ADMIN local só o lê com leitura auditada, e o SUPER_ADMIN não o acessa.                                              | Média      |
| RN07   | Regra de negócio                 | Evento com risco à integridade física deve ser comunicado ao responsável no mesmo dia.                                                                                      | Alta       |
| RN08   | Regra de negócio                 | O registro do acompanhamento é guardado por 5 anos, prazo declarado no consentimento.                                                                                       | Média      |

---

# 🏗️ PARTE 2 — DOCUMENTO DE ARQUITETURA CLIENTE-SERVIDOR

**Convenção dos diagramas.** Os três níveis mostram a arquitetura-alvo. O que **ainda não está
construído** em 17/11/2026 aparece com contorno ou seta **tracejados** e a marca _"a construir"_ ou
_"escopo futuro"_, com o pacote da Etapa 2 que o entrega. Todo o restante foi conferido contra o
código.

## 2.1 Diagramas C4

### 2.1.1 Nível 1 — Diagrama de Contexto

```mermaid
---
title: "C4 Nível 1 — Contexto do Sistema Agenda TEA"
---
flowchart TB
    resp["👤 Responsável<br/><i>pai, mãe ou tutor legal</i>"]
    prof["👤 Professor<br/><i>docente da turma</i>"]
    ped["👤 Pedagogo<br/><i>apoio pedagógico</i>"]
    ter["👤 Terapeuta<br/><i>TO, fono ou psicólogo da instituição</i>"]
    coord["👤 Coordenação<br/><i>gestão da unidade escolar</i>"]
    adm["👤 Administrador & SuperAdmin<br/><i>gestão local e plataforma SaaS</i>"]

    sistema["<b>AGENDA TEA</b><br/>Sistema de agenda digital integrada<br/>para acompanhamento de crianças com TEA"]

    email["📧 Serviço de E-mail (SMTP)<br/><i>sistema externo</i>"]
    push["🔔 Serviço de Web Push (VAPID)<br/><i>sistema externo</i>"]
    llm["🤖 Provedor de LLM<br/><i>sistema externo — opcional</i>"]
    s3["🗄️ Armazenamento de Objetos (S3)<br/><i>sistema externo — escopo futuro (RF21)</i>"]

    prof -->|"registro da agenda diária, crise,<br/>confirmação de dose e aprovação do texto de IA"| sistema
    ter -->|"rotina, crise e medicação<br/>da criança atendida"| sistema
    ped -->|"registro da turma e<br/>aprovação do texto gerado por IA"| sistema
    coord -->|"turmas, crianças, contas, vínculos,<br/>CSV, marca e aprovação do texto de IA"| sistema
    adm -->|"provisionamento de unidades,<br/>módulos e pedidos LGPD"| sistema
    resp -->|"aceite do termo de consentimento<br/>e confirmação de leitura"| sistema

    sistema -->|"resumo do dia, lacunas declaradas,<br/>histórico e relatório da semana"| resp
    sistema -->|"lista de doses previstas<br/>e pendências do dia"| prof
    sistema -->|"padrões da turma e histórico"| ped
    sistema -->|"histórico para ajuste de conduta"| ter
    sistema -->|"visão agregada da unidade<br/>e log de auditoria"| coord

    sistema -->|"convite com token de uso único<br/>e lembrete de leitura pendente"| email
    sistema -->|"alerta de crise (prioritário se severa)<br/>e de dose não confirmada"| push
    sistema -->|"registros pseudonimizados<br/><b>sem nome, documento ou endereço</b>"| llm
    sistema -.->|"foto do dia via URL assinada<br/><i>escopo futuro</i>"| s3
    push -->|"entrega do alerta ao aparelho"| resp
    email -->|"mensagem de convite e de lembrete"| resp

    style sistema fill:#1565c0,stroke:#0d47a1,color:#fff
    style resp fill:#e8eaf6,stroke:#3f51b5
    style prof fill:#e8eaf6,stroke:#3f51b5
    style ped fill:#e8eaf6,stroke:#3f51b5
    style ter fill:#e8eaf6,stroke:#3f51b5
    style coord fill:#e8eaf6,stroke:#3f51b5
    style adm fill:#e8eaf6,stroke:#3f51b5
    style email fill:#eceff1,stroke:#607d8b
    style push fill:#eceff1,stroke:#607d8b
    style llm fill:#eceff1,stroke:#607d8b,stroke-dasharray: 5 5
    style s3 fill:#eceff1,stroke:#607d8b,stroke-dasharray: 5 5
```

**Legenda do Nível 1**

| Elemento                           | Significado                                                                                          |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| 🔵 Caixa azul central              | O sistema em projeto (Agenda TEA), tratado como caixa-preta neste nível                              |
| 👤 Caixa lilás                     | Pessoa (ator humano) — os perfis de usuário definidos no RF01 e na seção 1.1                         |
| ⬜ Caixa cinza                     | Sistema externo, fora do escopo de desenvolvimento e mantido por terceiros                           |
| → Seta contínua                    | Fluxo de informação, rotulado com **o dado que trafega**                                             |
| ╌╌ Contorno tracejado (LLM)        | Dependência **opcional**: sem credencial do provedor, o resumo estruturado continua completo (RNF05) |
| ╌╌ Contorno e seta tracejados (S3) | **Escopo futuro** (RF21, Etapa 2, pacote 3.3): ainda não há adaptador nem serviço contratado         |
| Fora do diagrama                   | O **neuropediatra** não é usuário: recebe da família o relatório exportado no RF12                   |

**Rastreabilidade:** RF01, RF08–RF13, RF20 (Responsável) · RF02–RF06 e RF21 (Professor) · RF07
(Terapeuta) · RF14 (Professor, Pedagogo, Terapeuta e Coordenação) · RF15–RF18 (Coordenação) · RF19
(Administrador / SuperAdmin) · RNF07 (seta pseudonimizada para o LLM) · RN02 (aceite do termo antes
de qualquer visualização) · RN07 (seta de alerta via Web Push). A consulta ao log de auditoria
(RF18) e os pedidos LGPD (RF13, RF19) estão no contexto como requisito; a construção deles é o
pacote 3.5 da Etapa 2.

---

### 2.1.2 Nível 2 — Diagrama de Contêineres

```mermaid
---
title: "C4 Nível 2 — Contêineres do Agenda TEA"
---
flowchart TB
    resp["👤 Responsável"]
    equipe["👤 Equipe escolar<br/><i>professor, pedagogo, terapeuta, coordenação</i>"]

    subgraph limite["FRONTEIRA DO SISTEMA — AGENDA TEA"]
        direction TB
        pwa["<b>Aplicação Cliente (PWA)</b><br/>Next.js 16 · React 19 · TypeScript 5.7<br/><i>interface white-label adaptável</i>"]
        sw["<b>Service Worker</b><br/>API Web Push · Cache de app shell<br/><i>não cacheia dado de saúde</i>"]
        subgraph procApi["CONTÊINER DA API — um processo Node.js 20"]
            direction LR
            api["<b>Servidor de Aplicação / API</b><br/>NestJS 11 · TypeScript 5.7<br/><i>controllers, guards, casos de uso e domínio puro</i>"]
            worker["<b>Processador Assíncrono</b><br/>laços periódicos no mesmo processo<br/>(tarefas-periodicas.ts) · outbox + advisory lock<br/><i>notificações e geração de texto por IA</i>"]
        end
        bd[("<b>Banco de Dados</b><br/>PostgreSQL 16<br/>Prisma ORM 6.19.3<br/><i>26 tabelas versionadas ·<br/>imagens da marca em bytea</i>")]
    end

    email["📧 Serviço de E-mail<br/><i>externo</i>"]
    push["🔔 Serviço de Web Push<br/><i>externo</i>"]
    llm["🤖 Provedor de LLM<br/><i>externo — opcional</i>"]
    s3[("🗄️ Armazenamento de Objetos<br/>S3-compatível · <i>externo — escopo futuro</i>")]

    resp -->|"HTTPS / HTML + JS"| pwa
    equipe -->|"HTTPS / HTML + JS"| pwa
    pwa -->|"<b>HTTPS / JSON</b><br/>cookie httpOnly SameSite=Strict<br/>+ conferência de Origin"| api
    pwa -->|"registro da assinatura<br/>HTTPS / JSON"| sw
    api -->|"<b>TCP / SQL</b> via Prisma<br/>TLS, pool de conexões"| bd
    worker -->|"<b>TCP / SQL</b> via Prisma<br/>advisory lock de transação"| bd
    api -.->|"HTTPS / URL pré-assinada<br/><i>RF21 — escopo futuro</i>"| s3
    pwa -.->|"<b>HTTPS</b> direto, por URL assinada<br/><i>RF21 — escopo futuro</i>"| s3
    worker -->|"<b>HTTPS / Web Push</b><br/>protocolo VAPID"| push
    worker -->|"<b>SMTP sobre TLS</b> (porta 587)"| email
    worker -.->|"<b>HTTPS / JSON</b><br/>payload pseudonimizado"| llm
    push -->|"<b>Web Push</b> / mensagem cifrada"| sw
    sw -->|"notificação do sistema operacional"| resp

    style pwa fill:#bbdefb,stroke:#1565c0
    style sw fill:#bbdefb,stroke:#1565c0
    style api fill:#90caf9,stroke:#0d47a1
    style worker fill:#90caf9,stroke:#0d47a1
    style procApi fill:#e3f2fd,stroke:#0d47a1,stroke-dasharray: 3 3
    style bd fill:#c8e6c9,stroke:#2e7d32
    style limite fill:#fafafa,stroke:#1565c0,stroke-width:2px
    style llm stroke-dasharray: 5 5
    style s3 stroke-dasharray: 5 5
```

**Legenda do Nível 2**

| Elemento                   | Significado                                                                                                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🟦 Azul claro              | Contêiner de **apresentação** executado no dispositivo do usuário                                                                                                                   |
| 🟦 Azul escuro             | Contêiner de **execução no servidor**                                                                                                                                               |
| Moldura "Contêiner da API" | API e Processador Assíncrono são **dois contêineres lógicos no mesmo processo e na mesma imagem**: os laços de `tarefas-periodicas.ts` sobem com a API quando `WORKERS_ATIVOS=true` |
| 🟩 Verde (cilindro)        | Contêiner de **persistência**                                                                                                                                                       |
| Retângulo pontilhado       | Fronteira do sistema: o que está dentro é desenvolvido e implantado pelo projeto                                                                                                    |
| → Seta rotulada            | Chamada entre contêineres, rotulada com o **protocolo** e o formato de dados                                                                                                        |
| ╌╌ Seta tracejada          | Chamada opcional (LLM) ou de escopo futuro (S3, RF21)                                                                                                                               |

**Decisões visíveis neste nível**

1. **O cliente nunca fala com o banco ou com o LLM.** Todo acesso passa pela API, que é o único
   lugar onde o vínculo e o escopo de locatário (`AsyncLocalStorage`) são verificados e a auditoria
   é escrita — exigência direta de RN04, RNF04 e RNF07.
2. **O Processador Assíncrono existe por causa de RNF01 e RNF05:** nenhuma requisição HTTP espera
   por provedor externo, e um processo que cai não perde evento de notificação (padrão _outbox_ —
   a notificação é gravada na mesma transação do registro e enviada depois). No piloto ele roda
   **no mesmo processo e contêiner da API**; a separação em réplica própria é configuração
   (`WORKERS_ATIVOS=false` nas réplicas HTTP), não mudança de código, e a exclusão mútua entre
   réplicas já está resolvida pelo _advisory lock_ de transação (`TravaPostgres`). É por isso que o
   custo da Etapa 2 precifica **dois contêineres** (API e PWA), e não três.
3. **Imagens: o que existe e o que é futuro.** Hoje as únicas imagens do sistema são as da
   identidade visual de cada escola (logotipo e ícones): são processadas com `sharp`, gravadas em
   `bytea` no PostgreSQL e servidas pela rota pública com ETag e cache imutável. A foto do dia
   (RF21) é escopo futuro: quando entrar, a API emitirá URL pré-assinada e o PWA falará direto com
   o armazenamento de objetos, para que o binário não atravesse o servidor e a latência de RNF01 não
   dependa do tamanho do arquivo.
4. **O Service Worker não cacheia dado de saúde** — um aparelho perdido no transporte público de
   Manaus não pode virar vazamento (RNF04, RNF07).

---

### 2.1.3 Nível 3 — Diagrama de Componentes (interior do Servidor de Aplicação)

```mermaid
---
title: "C4 Nível 3 — Componentes do Servidor de Aplicação (API NestJS), por camada"
---
flowchart TB
    pwa["<b>Aplicação Cliente (PWA)</b><br/><i>contêiner externo a este diagrama</i>"]

    subgraph apiBox["CONTÊINER: SERVIDOR DE APLICAÇÃO — NestJS 11 / Node.js 20"]
        direction TB

        subgraph cApres["CAMADA DE APRESENTAÇÃO — entrada HTTP, sem regra de negócio"]
            direction LR
            csp["<b>Cabeçalhos de segurança</b><br/>Helmet 8<br/><i>a CSP com nonce é emitida pelo PWA</i>"]
            rate["<b>Limite de Tentativas</b><br/>5/min por IP · HTTP 429<br/><i>rotas de autenticação</i>"]
            dto["<b>Validação de Borda</b><br/>esquemas zod 4 em pipe global"]
            ctrl["<b>Controllers REST</b><br/>domínio, gestão e plataforma"]
        end

        subgraph cNeg["CAMADA DE NEGÓCIO — a regra mora aqui, nunca no framework"]
            direction TB
            autent["<b>Autenticação & Contexto ALS</b><br/>login, rotação de refresh, detecção de reuso,<br/>conferência de Origin · argon2id"]
            autoriz["<b>Autorização</b><br/>PapelGuard (7 perfis)<br/>e VinculoGuard"]
            conform["<b>Conformidade LGPD</b><br/>ConsentimentoGuard global<br/><i>exportação, exclusão e anonimização:<br/>a construir (Etapa 2, 3.5)</i>"]
            modulo["<b>Módulos contratados</b><br/>ModuloGuard · @ExigeModulo<br/><i>módulo ausente responde 404</i>"]
            casos["<b>Casos de Uso</b><br/>registros, resumos, relatórios,<br/>medicação, crise, gestão e plataforma"]
            dominio["<b>Domínio Puro</b><br/>packages/domain · sem framework<br/><i>399 testes · 98% das instruções</i>"]
            audit["<b>Auditoria</b><br/>trilha na transação + interceptor<br/>autor + unidade + ação + resultado<br/><i>append-only por disciplina de código</i>"]
        end

        subgraph cDados["CAMADA DE ACESSO A DADOS — adaptadores das portas"]
            direction LR
            repos["<b>Repositórios Escopados (14)</b><br/>Prisma 6.19.3 com extensão fail-closed"]
            adapIa["<b>AdaptadorAnthropic</b><br/>implements AiPort"]
            adapPush["<b>AdaptadorWebPush</b><br/>implements PushPort"]
            adapMail["<b>AdaptadorEmailSmtp</b><br/>implements EmailPort"]
            adapImg["<b>ProcessadorDeImagemSharp</b><br/>implements ProcessadorDeImagemPort"]
            adapTrava["<b>TravaPostgres</b><br/>implements TravaPort"]
            adapS3["<b>AdaptadorS3</b><br/>implements StoragePort<br/><i>a construir — RF21 (3.3)</i>"]
        end
    end

    bd[("<b>PostgreSQL 16</b>")]
    ext["<b>Serviços externos</b><br/>LLM · Web Push · SMTP · S3 (futuro)"]

    pwa -->|"HTTPS / JSON"| csp
    csp --> rate
    rate --> autent
    autent --> autoriz
    autoriz --> conform
    conform --> modulo
    modulo -->|"identidade, vínculo, termo<br/>e módulo resolvidos"| dto
    dto -->|"DTO tipado e validado"| ctrl
    ctrl --> casos
    casos -->|"invoca regra pura"| dominio
    casos -->|"grava na mesma transação"| audit
    casos -->|"interface de porta, nunca SQL"| repos
    casos -.->|"interface de porta"| adapIa
    casos -.->|"interface de porta"| adapPush
    casos -.->|"interface de porta"| adapMail
    casos -->|"interface de porta"| adapImg
    casos -.->|"escopo futuro"| adapS3
    casos --> adapTrava
    audit --> repos
    repos -->|"TCP / SQL"| bd
    adapTrava -->|"advisory lock"| bd
    adapIa --> ext
    adapPush --> ext
    adapMail --> ext
    adapS3 -.-> ext

    style cApres fill:#fff8e1,stroke:#f9a825,stroke-width:2px
    style cNeg fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style cDados fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style dominio fill:#a5d6a7,stroke:#1b5e20,stroke-width:2px
    style apiBox fill:#fafafa,stroke:#424242,stroke-width:2px
    style pwa fill:#bbdefb,stroke:#1565c0
    style bd fill:#c8e6c9,stroke:#2e7d32
    style adapS3 stroke-dasharray: 5 5
```

**Legenda do Nível 3**

| Elemento                         | Significado                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| 🟨 Bloco amarelo                 | **Camada de Apresentação** — traduz HTTP em objeto validado; não contém regra de negócio                              |
| 🟩 Bloco verde                   | **Camada de Negócio** — casos de uso, guards e o domínio puro; é onde a regra vive                                    |
| 🟦 Bloco azul                    | **Camada de Acesso a Dados** — adaptadores que implementam as portas declaradas no negócio                            |
| 🟩 Verde escuro (`Domínio Puro`) | Núcleo sem nenhuma dependência de framework, banco ou rede                                                            |
| → Seta contínua                  | Dependência obrigatória, sempre **de fora para dentro** (regra de dependência da Clean Architecture)                  |
| ╌╌ Seta tracejada                | Dependência opcional (sem credencial, a fábrica entrega a versão desligada da porta) ou escopo futuro (`AdaptadorS3`) |

**Ordem de execução de toda requisição autenticada**, conforme `configurar-aplicacao.ts` e os
guards globais de `app.module.ts`:
`cabeçalhos (Helmet) → abertura do store ALS → autenticação (cookie httpOnly, conferência de Origin nas escritas, conta recarregada do banco) → papel (RBAC) → vínculo com aquela criança → consentimento → módulo contratado → validação de esquema na borda → caso de uso no domínio puro → auditoria e persistência escopada na mesma transação`

O limite de tentativas não está na corrente de toda requisição: é um guard aplicado às rotas de
autenticação. A trilha de auditoria tem duas origens: os casos de uso críticos gravam a linha dentro
da própria transação, e um interceptor grava uma linha genérica para a escrita que não declarou a
sua — o esquecimento fecha, não abre.

**Observação obrigatória sobre o módulo de Pagamentos**

O modelo do enunciado prevê um componente de _Pagamentos_ na camada de negócio. **O Agenda TEA não
possui esse componente, e a ausência é uma decisão de modelo de negócio, não uma omissão de
projeto:** sendo um SaaS **B2B** por assinatura mensal por unidade escolar, a cobrança ocorre por
contrato e faturamento junto à instituição, fora do produto — não há transação iniciada por usuário
dentro do sistema. Introduzir um gateway de pagamento significaria tratar dado financeiro ao lado de
dado de saúde de criança no mesmo contêiner, ampliando sem necessidade a superfície de conformidade
exigida por RNF04 e RNF07. O lugar que o enunciado reserva a Pagamentos é ocupado, nesta arquitetura,
pelo componente de **Conformidade LGPD**, que é o requisito crítico equivalente neste domínio.

---

## 2.2 Matriz de rastreabilidade requisito → arquitetura

| Requisito                             | Contêiner (Nível 2)                         | Componente (Nível 3)                                                                                                           | ADR relacionado  |
| ------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------------- |
| RF01, RNF04                           | API                                         | Autenticação, Limite de Tentativas, Autorização, Módulos contratados, Repositórios Escopados                                   | ADR-002          |
| RF02–RF07                             | PWA, API, Banco                             | Controllers, Casos de Uso, Domínio Puro, Repositórios                                                                          | ADR-001          |
| RF08, RNF07                           | API, Banco                                  | Conformidade LGPD (ConsentimentoGuard), Auditoria                                                                              | ADR-001          |
| RF13, RF18, RF19 (pedidos LGPD), RN08 | API, Banco                                  | Conformidade LGPD e Auditoria — **a construir** (Etapa 2, 3.5)                                                                 | ADR-001          |
| RF09–RF12                             | PWA, API, Banco                             | Casos de Uso (resumos, relatórios), Repositórios                                                                               | ADR-001          |
| RF14                                  | PWA, API, Processador Assíncrono            | Casos de Uso, AdaptadorAnthropic                                                                                               | ADR-003          |
| RF15–RF17, RF19 (contas e plataforma) | PWA, API, Banco                             | Autorização, Módulos contratados, Casos de Uso (gestão e plataforma), ProcessadorDeImagemSharp                                 | ADR-002          |
| RF20, RN07                            | Processador Assíncrono, Service Worker      | AdaptadorWebPush, AdaptadorEmailSmtp, TravaPostgres                                                                            | ADR-003          |
| RF21                                  | PWA, API, Armazenamento de Objetos (futuro) | AdaptadorS3 — **a construir** (Etapa 2, 3.3)                                                                                   | ADR-003          |
| RNF01                                 | API, Processador Assíncrono                 | separação síncrono/assíncrono, outbox                                                                                          | ADR-003          |
| RNF02, RNF03, RNF06                   | PWA, Service Worker                         | interface white-label com contraste validado; PWA sem loja                                                                     | ADR-003          |
| RNF05                                 | Todos                                       | resumo estruturado independente do LLM                                                                                         | ADR-003          |
| RNF08                                 | Todos                                       | Domínio Puro sem framework; integração contínua                                                                                | ADR-001          |
| RNF09, RN04                           | API, Banco                                  | Contexto ALS, Autorização (vínculo), Repositórios Escopados (_fail-closed_)                                                    | ADR-001, ADR-003 |
| RN01, RN05                            | Banco                                       | `plano_medicacao` separado de `registro_medicacao`; correção como nova versão na trilha (append-only por disciplina de código) | ADR-001          |
| RN02                                  | API, Banco                                  | termo versionado imutável com gatilho no banco; ConsentimentoGuard                                                             | ADR-001          |
| RN03                                  | Banco, Processador Assíncrono               | vínculo N:N com chave estrangeira; uma notificação por responsável                                                             | ADR-001          |
| RN06                                  | API                                         | Autorização: `ADMIN` só lê com trilha; `SUPER_ADMIN` sem acesso a criança                                                      | ADR-002          |

---

## 2.3 Registros de Decisão Arquitetural (ADRs)

### ADR-001 Adoção de banco de dados relacional PostgreSQL em lugar de banco não-relacional

```markdown
# ADR-001 Adoção de banco de dados relacional PostgreSQL em lugar de banco não-relacional

Status: Aceita
Data: 2026-08-25

## Contexto

As seguintes exigências determinam esta decisão:

- **RN01** — medicação exige prescrição vigente e autorização do responsável: a _dose prevista_
  (prescrição transcrita) e o _fato administrado_ são entidades distintas que precisam ser
  relacionadas sem permitir estado ambíguo.
- **RN03** — uma criança pode ter N responsáveis legais, todos com igual direito de acesso: é uma
  relação muitos-para-muitos com integridade obrigatória, pois um vínculo perdido é um direito legal
  violado.
- **RN05** — registro de cuidado não é apagado; a correção preserva o valor anterior.
- **RNF07** — 100% das ações críticas devem gravar autor, ação e resultado **na mesma transação
  atômica** do dado que as originou.
- **RF16** — a importação em lote de uma planilha CSV da escola deve ser atômica: ou entra tudo, ou
  não entra nada.

O ponto decisivo é RNF07 combinado com RF16: sem transação ACID que abranja o dado e sua trilha de
auditoria, existe uma janela em que o registro de saúde está gravado e a prova de quem o gravou não
está — exatamente o estado que o sistema foi construído para eliminar.

## Alternativas consideradas

### Alternativa A: PostgreSQL (relacional) com Prisma ORM

- A favor: transações ACID multi-tabela atendem RNF07 e RF16 diretamente; chaves estrangeiras
  tornam o vínculo de RN03 uma garantia do banco e não do código de aplicação; chaves estrangeiras
  compostas `(crianca_id, unidade_id)` garantem integridade multi-tenant no banco; _migrations_
  versionadas e revisáveis em _pull request_; restrições `CHECK` e gatilhos permitem escrever
  regras no próprio banco (o termo de consentimento publicado é imutável por gatilho, e a conta de
  plataforma sem unidade é garantida por `CHECK`); _advisory lock_ de transação resolve a produção
  de alertas entre réplicas sem tabela extra; tipos derivados do esquema evitam divergência entre
  banco e aplicação.
- Contra: esquema rígido — toda mudança de campo da agenda exige _migration_; escala horizontal de
  escrita é mais trabalhosa que em bancos documentais; exige conhecimento de modelagem relacional
  para manutenção.

### Alternativa B: MongoDB (documental, não-relacional)

- A favor: o registro diário é naturalmente um documento, e a agenda de papel tem campos que variam
  entre instituições — o esquema flexível acomodaria isso sem _migration_; leitura do dia inteiro em
  uma única consulta, sem junções, favorecendo RNF01; escala horizontal de escrita mais simples.
- Contra: transação multi-documento exige _replica set_ e continua mais frágil e custosa do que uma
  transação relacional, o que coloca RNF07 em risco permanente; a integridade do vínculo de RN03
  passaria a ser responsabilidade do código de aplicação, sem rede de segurança do banco; RN05
  dependeria apenas de disciplina de escrita, sem a opção de o banco recusar um `update`
  destrutivo; RF16 atômico exigiria compensação manual em caso de falha parcial.

## Decisão

Escolhemos a **Alternativa A (PostgreSQL com Prisma ORM)**, porque **RNF07 exige que autor, ação e
resultado sejam gravados na mesma transação atômica do dado que os originou** — garantia que, em um
banco relacional, é propriedade do mecanismo de armazenamento, e não convenção que o código precisa
lembrar de respeitar a cada _endpoint_. Em um sistema cujo propósito declarado é ser prova de que a
informação existiu e chegou, a trilha de auditoria não pode ser _best effort_. A integridade
referencial de RN03 reforça a escolha: o vínculo entre criança e responsável é um direito legal, e
colocá-lo sob chave estrangeira significa que nenhum erro de aplicação pode silenciosamente apagá-lo.
O relacional também é o único dos dois que permite, quando adotado, que o próprio banco recuse a
sobrescrita proibida por RN05 (`REVOKE` e gatilho, como já ocorre com o termo).

## Consequências

- Positivas:
  - RNF07 e RF16 são atendidos por transação nativa, sem código de compensação.
  - RN02 e RN03 passam a ser aplicados pelo banco (gatilho de imutabilidade do termo publicado e
    chave estrangeira do vínculo), não apenas pela aplicação.
  - As 26 tabelas têm _migrations_ versionadas e revisáveis, o que torna a evolução do esquema
    auditável como o próprio dado. Duas delas (`anexo` e `mensagem`) já foram criadas para escopo
    futuro e ainda não têm código que as use.
  - O _advisory lock_ do PostgreSQL (`TravaPostgres`) resolve a exclusão mútua na produção de
    alertas sem introduzir outra peça de infraestrutura.
- Negativas:
  - **RN05 ainda não é imposto pelo banco.** O caráter _append-only_ de `log_auditoria` é garantido
    por disciplina de código e RBAC — nenhum caminho da aplicação faz `UPDATE` ou `DELETE` na
    trilha, e a correção grava nova versão em vez de sobrescrever —, mas uma conexão com a
    credencial da aplicação ainda poderia reescrevê-la. O `REVOKE UPDATE, DELETE` com gatilho, como
    o que já protege o termo, é o passo registrado em `docs/SEGURANCA.md` § 11.
  - **Toda alteração nos campos da agenda exige _migration_ e novo _deploy_.** Como o formulário de
    papel varia entre instituições, cada nova escola cliente pode demandar trabalho de esquema —
    atrito direto contra o modelo de negócio de "uma escola nova entra em minutos" e um custo real
    de escala comercial.
  - A escala horizontal de escrita é mais trabalhosa; ao atingir o limite de RNF09, a saída será
    réplica de leitura e particionamento, não adição de nós.
  - A dependência fixada em Prisma 6.19.3 (e não em `latest`) é dívida técnica registrada: a
    migração para a linha 7.x fica pendente até que CLI e cliente estejam realinhados.
```

---

### ADR-002 Sessão do navegador em cookies `httpOnly` com JWT curto e refresh rotativo

```markdown
# ADR-002 Sessão do navegador em cookies httpOnly com JWT curto e refresh rotativo

Status: Aceita
Data: 2026-09-10

## Contexto

As seguintes exigências determinam esta decisão:

- **RNF04** — senhas em argon2id, _access token_ de 15 minutos, _refresh token_ de 7 dias com
  rotação a cada uso, e os 10 itens do OWASP Top 10 mitigados item a item. Isso inclui A03
  (injeção/XSS) e A07 (falhas de identificação e autenticação).
- **RNF07** — nenhum dado pessoal identificável direto deve circular fora do necessário; o _payload_
  da sessão não pode conter dado sensível.
- **RNF09** — processos sem estado de sessão em memória, para permitir múltiplas réplicas.
- **RN04 e RN06** — a decisão de quem pode registrar sobre qual criança, e de quem não produz
  conteúdo clínico, é regra da instituição: precisa ser resolvida no servidor, a partir do vínculo
  persistido, nunca a partir de papel informado pelo cliente.

O problema concreto: o público usuário acessa o PWA de aparelhos pessoais, muitas vezes
compartilhados na família, em uma instituição sem equipe de TI. Uma única falha de XSS que exponha a
sessão daria acesso ao histórico de saúde de uma criança.

## Alternativas consideradas

### Alternativa A: Sessão em servidor com identificador em cookie (sessão clássica)

- A favor: revogação imediata e central — encerrar a sessão é apagar uma linha; nada de valor
  trafega no cookie; modelo mental simples e bem conhecido.
- Contra: viola RNF09, pois exige estado compartilhado entre réplicas (armazenamento de sessão
  dedicado), acrescentando uma peça de infraestrutura a operar e monitorar; cada requisição paga uma
  consulta ao armazenamento de sessão, pressionando a meta de p95 < 500 ms de RNF01.

### Alternativa B: JWT guardado em `sessionStorage` e enviado em cabeçalho `Authorization`

- A favor: imune a CSRF por construção, já que o navegador não anexa o cabeçalho sozinho; servidor
  totalmente sem estado, atendendo RNF09; implementação direta e portável para clientes que não são
  navegador.
- Contra: **qualquer XSS lê o token e leva a sessão embora** — risco inaceitável diante de RNF04 e
  do dado em jogo; a revogação antes da expiração é difícil; obriga o código do cliente a manipular
  o segredo, ampliando a superfície de erro.

### Alternativa C: Cookies `httpOnly` com JWT curto, refresh rotativo e conferência de `Origin`

- A favor: JavaScript não alcança o token, de modo que um XSS não rouba a sessão; `SameSite=Strict`
  mais a verificação de `Origin` contra a lista branca cobre CSRF sem estado adicional no servidor;
  servidor sem estado atende RNF09; a rotação do _refresh_ permite **detectar reuso** — apresentar um
  token já substituído revoga a cadeia inteira e gera auditoria, que é a assinatura clássica de token
  roubado; o _payload_ carrega apenas `sub`, `papel` e `jti`, sem dado sensível (RNF07).
- Contra: PWA e API precisam estar no mesmo site e em HTTPS, o que restringe topologias de
  implantação; a revogação do _access token_ não é instantânea — há até 15 minutos de janela; exige
  um segundo modo de autenticação para clientes que não são navegador.

## Decisão

Escolhemos a **Alternativa C (cookies `httpOnly` com JWT curto, refresh rotativo e conferência de
`Origin`)**, porque **RNF04 exige a mitigação documentada de A03 e A07 do OWASP Top 10 sobre dado de
saúde de criança, e a Alternativa B falha justamente ali: nela, um único XSS converte-se em acesso
integral ao histórico da criança**. A Alternativa A protegeria igualmente, mas ao custo de estado
compartilhado que RNF09 proíbe e de uma consulta por requisição que pressiona RNF01. A rotação com
detecção de reuso é o que fecha a exigência de A07: o sistema não apenas resiste ao roubo de token,
ele o **percebe** e derruba a cadeia, gravando o evento na trilha de auditoria de RNF07.
Complementarmente, a autorização continua sendo resolvida no servidor a partir do vínculo
persistido, de modo que o cookie prova _quem_ é o usuário, mas nunca _o que_ ele pode fazer —
atendendo RN04 e RN06.

## Consequências

- Positivas:
  - XSS deixa de ser caminho para roubo de sessão: o token não é alcançável por JavaScript.
  - CSRF é coberto sem token sincronizado nem estado extra no servidor.
  - O reuso de _refresh token_ é detectado e revoga a cadeia completa, com registro em auditoria.
  - O servidor permanece sem estado de sessão, compatível com múltiplas réplicas (RNF09).
  - O _payload_ da sessão não contém dado sensível, reduzindo o impacto de um vazamento de token.
- Negativas:
  - **A revogação não é imediata: um _access token_ já emitido continua válido por até 15 minutos
    após o encerramento da sessão.** Em um incidente real — aparelho perdido no transporte público —
    existe uma janela de até 15 minutos de acesso ao dado da criança, e esse é um risco aceito
    conscientemente em troca de não manter estado de sessão no servidor.
  - PWA e API passam a ter de estar no mesmo site, sob HTTPS, o que elimina topologias de
    implantação em domínios separados e torna a configuração de CORS e de proxy reverso parte do
    requisito de segurança, não apenas de infraestrutura. No ambiente de testes do Render, em que
    os dois serviços recebem domínios de sites diferentes, a solução foi servir a API pelo próprio
    PWA, no caminho `/api` (rewrite do Next): o navegador conversa com uma origem só.
  - Mantém-se um segundo modo de autenticação (token no corpo) para clientes que não são navegador e
    para a suíte de testes, ou seja, dois caminhos de sessão a manter e testar.
  - Os contadores de limite de tentativas vivem na memória do processo: com mais de uma réplica, o
    limite efetivo de 5 tentativas por minuto de RNF04 multiplica-se pelo número de réplicas.
  - O rewrite `/api` faz todas as requisições chegarem à API com o IP do servidor do PWA. Por isso o
    ambiente de testes opera com **50 tentativas por minuto** (`render.yaml`), e não com as 5 de
    RNF04: com um IP só para a escola inteira, 5/min seriam 5 logins por minuto para todos. Em
    produção, o IP do aparelho precisa ser propagado até a API para que o limite por IP volte a ter
    o sentido que RNF04 exige.
```

---

### ADR-003 Implantação em nuvem gerenciada e containerizada, em lugar de servidor próprio na escola

```markdown
# ADR-003 Implantação em nuvem gerenciada e containerizada, em lugar de servidor próprio na escola

Status: Aceita
Data: 2026-10-05

## Contexto

As seguintes exigências determinam esta decisão:

- **RNF05** — disponibilidade de no mínimo 99% na janela escolar de 07h às 19h em dias úteis, o que
  admite no máximo 7,2 minutos de indisponibilidade por dia útil.
- **RNF09** — suportar no mínimo 50 unidades escolares e 4.500 crianças sem alteração estrutural,
  com processos sem estado e isolamento de dados por unidade resolvido no servidor.
- **RNF01** — metas de latência no percentil 95, que dependem de rede e de capacidade de banco.
- **RNF04 e RNF07** — TLS 1.2+ obrigatório, criptografia em repouso, retenção de 5 anos com
  anonimização ao fim do prazo e trilha de auditoria preservada.
- **RNF06** — PWA sem loja de aplicativos, acessível por qualquer navegador.
- Restrição de contexto: a instituição parceira em Manaus tem 19 profissionais e **nenhuma equipe de
  TI**. O modelo de negócio é SaaS B2B por assinatura, com custo marginal baixo por nova escola.

O ponto decisivo é a combinação de RNF05 com a realidade local: Manaus tem interrupções de energia e
de enlace de internet com frequência não desprezível, e o perfil de cliente não dispõe de gerador,
nobreak dimensionado, enlace redundante ou pessoa de plantão. Um servidor dentro da escola colocaria
a meta de 99% sob variáveis que o projeto não controla e que a instituição não tem como administrar.

## Alternativas consideradas

### Alternativa A: Servidor próprio (_on-premise_) em cada escola cliente

- A favor: o dado de saúde da criança nunca sai fisicamente da instituição, o que é um argumento
  forte de conformidade e de confiança para a família; funciona na rede local mesmo com a internet
  da escola fora do ar; custo previsível e sem recorrência em moeda estrangeira.
- Contra: inviabiliza RNF05, pois a disponibilidade passaria a depender da energia, do enlace e do
  hardware de cada escola, sem redundância nem plantão; inviabiliza RNF09, já que cada nova escola
  exigiria aquisição, instalação e manutenção de equipamento, destruindo o custo marginal baixo do
  modelo SaaS; as responsabilidades de RNF04 e RNF07 (atualização de segurança, cópia de segurança,
  criptografia em repouso, retenção de 5 anos) recairiam sobre uma instituição sem equipe de TI, o
  que na prática significa que não seriam cumpridas; 50 escolas significariam 50 ambientes de
  produção distintos a diagnosticar.

### Alternativa B: Nuvem gerenciada com aplicação containerizada e banco gerenciado

- A favor: atende RNF05 com infraestrutura redundante de energia e de rede, fora do alcance das
  intermitências locais; atende RNF09 porque uma nova escola é um registro de unidade no banco
  _multi-tenant_, sem hardware a adquirir nem ambiente a provisionar; cópia de segurança,
  criptografia em repouso e TLS gerenciados atendem RNF04 e RNF07 sem exigir equipe de TI na escola;
  imagens Docker _multi-stage_ garantem que o ambiente de produção seja idêntico ao verificado em
  integração contínua; segredos externos à imagem; a escala vertical e a réplica de leitura são
  mudanças de configuração, não de arquitetura.
- Contra: cria dependência de fornecedor e custo recorrente proporcional ao uso; o dado de saúde
  passa a residir fora da instituição e, possivelmente, fora do país, o que exige cláusula
  contratual e transparência no termo de consentimento; a escola sem internet perde o acesso
  completamente, pois o PWA não cacheia dado de saúde por decisão de RNF04; a latência até Manaus
  depende da região de hospedagem escolhida.

## Decisão

Escolhemos a **Alternativa B (nuvem gerenciada com aplicação containerizada e banco gerenciado)**,
porque **RNF05 fixa 99% de disponibilidade na janela escolar e RNF09 exige 50 unidades sem alteração
estrutural — e a Alternativa A torna as duas metas impossíveis de sustentar no contexto real de
Manaus**, onde o cliente não dispõe de energia redundante, enlace redundante nem pessoa de plantão
para cumprir 7,2 minutos de indisponibilidade por dia útil. O argumento decisivo é de
responsabilidade: transferir a uma instituição de 19 profissionais sem equipe de TI os deveres de
atualização de segurança, cópia de segurança, criptografia em repouso e retenção de 5 anos exigidos
por RNF04 e RNF07 equivale a declarar esses requisitos no documento e não cumpri-los na prática. A
nuvem gerenciada mantém essas obrigações com quem tem competência técnica para honrá-las, e preserva
o custo marginal baixo por escola que o modelo de assinatura exige.

## Consequências

- Positivas:
  - RNF05 passa a depender de infraestrutura redundante, e não da rede elétrica do bairro.
  - Uma nova escola cliente é criada em minutos, por cadastro de unidade e importação CSV, sem
    hardware nem provisionamento (RNF09, RF16).
  - Cópia de segurança, criptografia em repouso, renovação de TLS e correções de segurança ficam sob
    responsabilidade de quem tem competência para executá-las (RNF04, RNF07). A criptografia em
    repouso é do volume e do banco gerenciado, não do código da aplicação.
  - A imagem containerizada _multi-stage_ garante paridade entre o que a integração contínua
    verificou e o que roda em produção, sustentando RNF08.
  - Processos sem estado de sessão permitem acrescentar réplicas sem redesenhar a aplicação; o
    processador assíncrono, que hoje roda no contêiner da API, separa-se por configuração.
- Negativas:
  - **Sem internet, a escola perde o sistema por completo.** Como o Service Worker não cacheia dado
    de saúde (decisão de RNF04), uma queda de enlace durante a janela escolar deixa a equipe sem
    acesso ao plano de medicação do dia — e esse é precisamente o momento em que o dado é crítico. A
    mitigação prevista é o procedimento de contingência em papel para o plano de medicação, o que
    significa admitir que o papel não desaparece por inteiro.
  - Dependência de fornecedor e custo recorrente que cresce com o número de escolas, com exposição à
    variação cambial.
  - O dado de saúde de criança passa a residir fora da instituição, exigindo cláusula de
    transferência no contrato e declaração explícita no termo de consentimento de RN02 — o que
    aumenta o atrito da venda junto a famílias e gestores mais conservadores.
  - A latência até Manaus fica condicionada à região de hospedagem, e a meta de RNF01 precisa ser
    medida com clientes reais na cidade, não apenas em laboratório.
```

---

## 2.4 Síntese da arquitetura

| Dimensão                     | Definição adotada                                                                                                                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Estilo arquitetural**      | Cliente-servidor em três camadas, com Clean Architecture e padrão _Ports & Adapters_ no servidor                                                                                                           |
| **Cliente**                  | PWA _white-label_ em Next.js 16 / React 19, com design tokens por escola injetados no servidor, sem instalação obrigatória por loja de aplicativos (RNF06)                                                 |
| **Servidor**                 | API REST em NestJS 11 sobre Node.js 20, com domínio puro em pacote próprio e sem framework                                                                                                                 |
| **Processamento assíncrono** | Laços periódicos no mesmo processo da API (outbox de notificações, alertas sob _advisory lock_, texto por IA), separáveis por configuração                                                                 |
| **Persistência**             | PostgreSQL 16 via Prisma ORM 6.19.3, com 26 tabelas, FKs compostas multi-tenant e _migrations_ versionadas; imagens da marca em `bytea`, processadas com `sharp`                                           |
| **Mídia da criança**         | Escopo futuro (RF21): armazenamento de objetos compatível com S3 e URL assinada                                                                                                                            |
| **Comunicação**              | HTTPS/JSON com cookies `httpOnly` e conferência de `Origin`; TCP/SQL para o banco; Web Push (VAPID) e SMTP sobre TLS para notificação                                                                      |
| **Isolamento**               | _Multi-tenant_ por unidade escolar com contexto por requisição via `AsyncLocalStorage` e extensão Prisma _fail-closed_ (RNF04, RNF09)                                                                      |
| **Implantação**              | Nuvem gerenciada, imagens Docker _multi-stage_, segredos externos à imagem                                                                                                                                 |
| **Verificação**              | 981 testes automatizados (399 domínio, 419 API, 163 PWA); cobertura do domínio de 98% das instruções e 92,8% dos ramos; 0 erro de lint e de checagem de tipos; integração contínua como portão obrigatório |

---

## 2.5 O fluxo de uma requisição nesta arquitetura

Para amarrar os três níveis, este é o caminho de uma requisição real: a professora registra uma
crise de uma criança da sua turma (RF03). A ordem é a declarada na seção 2.1.3 — a requisição só
chega à regra de negócio depois de provar quem é, o que o papel permite, qual o vínculo com aquela
criança e que a escola tem o módulo de que a rota precisa.

1. **Cliente.** O PWA envia `POST /criancas/{criancaId}/registros/crise` com o corpo em JSON, por
   HTTPS. O navegador anexa o _cookie_ `httpOnly` da sessão; o código da página não o alcança
   (ADR-002).
2. **Borda HTTP.** A API aplica os cabeçalhos de segurança (Helmet) e abre o _store_ de
   `AsyncLocalStorage` da requisição, que vai carregar a unidade da conta até a persistência.
3. **Autenticação.** Por ser uma escrita autenticada por _cookie_, o guard de autenticação confere
   primeiro se a `Origin` está na lista permitida — origem estranha é recusada antes de qualquer
   regra. Depois verifica a assinatura e a validade do _token_ de acesso e **recarrega a conta do
   banco**: papel, unidade e situação valem pelo que está persistido, não pelo que o _token_
   afirma. Sem sessão válida, `401`.
4. **Papel (RBAC).** A matriz de permissões responde se o papel pode registrar crise. Um
   responsável é recusado aqui, com `403`, sem consulta de vínculo.
5. **Vínculo.** O servidor confere se a professora está vinculada à turma atual daquela criança
   (RN04). Sem vínculo, `403` e uma linha de acesso negado na trilha de auditoria. Criança
   inexistente ou de outra unidade responde igual, para não revelar quem está matriculado.
6. **Consentimento.** O guard global de consentimento só barra o responsável que ainda não aceitou o
   termo vigente (RF08, RN02). Para a professora, passa — e, por rodar depois da autorização, nunca
   esconde uma negação de vínculo.
7. **Módulo contratado.** O `ModuloGuard` confere se a rota exige um módulo opcional que a escola
   não contratou. O registro de crise é núcleo do produto e não exige nenhum; uma rota de módulo
   desligado responderia `404`, porque para aquela escola a funcionalidade não existe.
8. **Validação de borda.** Só então o corpo é validado pelo esquema zod: horário, intensidade,
   duração. Dado malformado recebe `400` apontando o campo.
9. **Caso de uso e domínio puro.** O caso de uso aplica as regras de negócio com código sem
   _framework_ e fala com o banco apenas pela interface do repositório.
10. **Persistência e auditoria.** O repositório escopado grava, **na mesma transação**, o registro
    (carimbado com a unidade do contexto pela extensão _fail-closed_), a linha de auditoria — autor,
    ação e resultado (RNF07, ADR-001) — e uma notificação pendente para **cada** responsável
    vinculado (RN03), marcada como prioritária se a crise for severa. Se uma das gravações falhar,
    nenhuma existe.
11. **Resposta.** A API devolve `201 Created` com o registro.
12. **Depois da resposta.** O Processador Assíncrono, no mesmo processo da API, lê o _outbox_ e
    entrega o Web Push aos responsáveis (RF20, RN07), com e-mail como canal de reserva quando a
    entrega por push falha. A professora não espera por isso.

---

## 2.6 Correspondência de identificadores com o repositório

A numeração deste trabalho foi reorganizada para o formato do enunciado. A tabela abaixo liga cada
identificador acadêmico ao que existe em `docs/REQUISITOS.md` (RF, RNF e RN), em
`docs/ARQUITETURA.md` § 11 (ADR) e no código, para que a rastreabilidade chegue até a
implementação.

**Requisitos funcionais**

| Este documento | `docs/REQUISITOS.md` e código                                                                              | Observação                                                                                 |
| -------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| RF01           | RF01; matriz § 3.2; `ModuloGuard` (ADR-21)                                                                 | Módulos contratados vêm do plano multi-tenant                                              |
| RF02           | RF02; RN-DIA-01                                                                                            | —                                                                                          |
| RF03           | RF03                                                                                                       | —                                                                                          |
| RF04           | RF04; ADR-07                                                                                               | —                                                                                          |
| RF05           | RF04 (justificativa da não administração)                                                                  | —                                                                                          |
| RF06           | RN-EDICAO-01                                                                                               | Janela de 24 h                                                                             |
| RF07           | Matriz § 3.2, coluna `TERAPEUTA` (vínculo direto)                                                          | Não há entidade "sessão de terapia"                                                        |
| RF08           | RF11, primeiro critério                                                                                    | —                                                                                          |
| RF09           | RF06                                                                                                       | —                                                                                          |
| RF10           | RN-LEITURA-01, RN-LEITURA-02; ADR-19                                                                       | —                                                                                          |
| RF11           | RF09                                                                                                       | —                                                                                          |
| RF12           | RF10 (exportação do relatório aprovado)                                                                    | Exportação em PDF é o pacote 3.4 da Etapa 2                                                |
| RF13           | RF11, critérios de exportação e exclusão                                                                   | **Pendente** no código                                                                     |
| RF14           | RF10; RN-IA-01; matriz `APROVAR_SAIDA_IA`                                                                  | —                                                                                          |
| RF15           | Matriz `GERIR_USUARIOS_E_VINCULOS`, `GERIR_PLANOS_MEDICACAO`, `GERIR_PERSONALIZACAO`; ARQUITETURA § 8.1    | Sem RF próprio no repositório                                                              |
| RF16           | `gestao/interface/importacao.controller.ts`                                                                | Sem RF próprio no repositório                                                              |
| RF17           | `gestao/interface/convites.controller.ts`                                                                  | Sem RF próprio no repositório                                                              |
| RF18           | RF12; matriz `CONSULTAR_LOG_AUDITORIA`                                                                     | **Pendente**: sem rota de consulta                                                         |
| RF19           | Matriz `GERIR_USUARIOS_E_VINCULOS`, `EXPORTAR_OU_EXCLUIR_DADOS`, `PROVISIONAR_LOCATARIOS`; ADR-21 a ADR-29 | Pedidos do titular **pendentes**                                                           |
| RF20           | RF07; RN-LEITURA-03; ADR-09, ADR-18                                                                        | —                                                                                          |
| RF21           | RF05                                                                                                       | **Escopo futuro**; `SEGURANCA.md` § 11                                                     |
| —              | RF06-IA (texto do resumo por IA)                                                                           | Coberto por RNF05 e RNF07 deste documento                                                  |
| —              | RF08 (canal família ↔ equipe)                                                                              | Sem requisito neste documento; tabela `mensagem` e módulo `CHAT` existem só como estrutura |

**Requisitos não funcionais, regras de negócio e decisões**

| Este documento         | Repositório                                                       | Observação                                                                                   |
| ---------------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| RNF01 Desempenho       | RNF04                                                             | —                                                                                            |
| RNF02 Usabilidade      | RNF02                                                             | —                                                                                            |
| RNF03 Acessibilidade   | RNF02                                                             | —                                                                                            |
| RNF04 Segurança        | RNF05; `SEGURANCA.md` § 3                                         | OWASP Top 10 item a item                                                                     |
| RNF05 Disponibilidade  | RNF03                                                             | —                                                                                            |
| RNF06 Compatibilidade  | RNF02 (mobile-first); ARQUITETURA § 8                             | —                                                                                            |
| RNF07 Conformidade     | RNF01; RF12; RN-IA-02                                             | —                                                                                            |
| RNF08 Manutenibilidade | RNF07                                                             | —                                                                                            |
| RNF09 Escalabilidade   | RNF06                                                             | —                                                                                            |
| RN01                   | ADR-07; matriz `GERIR_PLANOS_MEDICACAO`                           | —                                                                                            |
| RN02                   | RNF01; RF11                                                       | —                                                                                            |
| RN03                   | RN-VINCULO-01; RN-LEITURA-02                                      | —                                                                                            |
| RN04                   | RN-ACESSO-02                                                      | —                                                                                            |
| RN05                   | RN-EDICAO-01; RF12; ADR-08                                        | —                                                                                            |
| RN06                   | RN-ACESSO-01; matriz `LER_REGISTROS`                              | —                                                                                            |
| RN07                   | RF07; RN-LEITURA-03                                               | —                                                                                            |
| RN08                   | RN-RETENCAO-01                                                    | **Pendente**: job de anonimização                                                            |
| ADR-001                | ADR-03, ADR-04, ADR-05 a ADR-08, ADR-18, ADR-26                   | A comparação com MongoDB é registro deste trabalho                                           |
| ADR-002                | ADR-16, ADR-17; ARQUITETURA § 5.1                                 | —                                                                                            |
| ADR-003                | `docs/DEPLOY_RENDER.md`, `docker-compose.prod.yml`, `render.yaml` | Sem ADR numerado no repositório; a comparação com servidor próprio é registro deste trabalho |

---

## Referências

- BRASIL. **Lei nº 13.709, de 14 de agosto de 2018** — Lei Geral de Proteção de Dados Pessoais
  (LGPD). Brasília, 2018.
- BROWN, Simon. **The C4 model for visualising software architecture**. Disponível em:
  <https://c4model.com>.
- MARTIN, Robert C. **Clean Architecture: A Craftsman's Guide to Software Structure and Design**.
  Boston: Prentice Hall, 2017.
- NYGARD, Michael. **Documenting Architecture Decisions**, 2011.
- OPEN WORLDWIDE APPLICATION SECURITY PROJECT. **OWASP Top 10**. Disponível em:
  <https://owasp.org/www-project-top-ten/>.
- ORGANIZAÇÃO MUNDIAL DA SAÚDE. **Autism** (fact sheet). Disponível em:
  <https://www.who.int/news-room/fact-sheets/detail/autism-spectrum-disorders>.
- MAENNER, M. J. et al. **Prevalence and Characteristics of Autism Spectrum Disorder Among Children
  Aged 8 Years — Autism and Developmental Disabilities Monitoring Network**. MMWR Surveillance
  Summaries, 2023.
- TAGER-FLUSBERG, H.; KASARI, C. **Minimally verbal school-aged children with autism spectrum
  disorder: the neglected end of the spectrum**. Autism Research, v. 6, n. 6, p. 468–478, 2013.
- WORLD WIDE WEB CONSORTIUM. **Web Content Accessibility Guidelines (WCAG) 2.1**, 2018.
