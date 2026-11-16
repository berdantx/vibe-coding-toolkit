---
name: pre-projeto-app
description: Usar ANTES de começar qualquer aplicativo, sistema, SaaS, painel ou automação com código. Entrevista a pessoa, classifica risco e gera spec, checklist de segurança e plano em fases para o Claude Code, integrado ao vibe-coding-toolkit.
---

# Pré-projeto de aplicativo

Esta skill roda antes de qualquer linha de código. Ela tira da cabeça da pessoa o que o sistema precisa fazer, decide o nível de rigor de segurança e entrega os documentos que o Claude Code vai seguir na construção.

Ela define **o que** construir. O **como** construir vem do [vibe-coding-toolkit](https://github.com/berdantx/vibe-coding-toolkit): brainstorm, plano, subagentes em ondas, revisão multi-agente e quality gates. As referências exatas estão na seção "Referências do toolkit", no fim.

## Regra de ouro (trava)

Não escrever código, não criar projeto, não instalar nada e não criar banco enquanto a spec não for aprovada explicitamente por quem pediu. Se a pessoa pedir para "já ir fazendo", explicar em uma frase que a entrevista leva poucos minutos e evita retrabalho, e seguir com a entrevista. Se ela insistir depois disso, registrar na spec quais pontos ficaram sem resposta como "SUPOSIÇÃO" e pedir aprovação das suposições.

Nunca prometer sistema "sem erro". O compromisso é: pegar o máximo de erros antes do código e testar cada fase antes de avançar.

## Quem faz o quê: esta skill e o Superpowers

- **pre-projeto-app** roda **uma vez**, no início do projeto. Produz spec, segurança e plano de fases.
- **Superpowers** (brainstorm → plano → implementação → revisão) roda **a cada funcionalidade**, dentro das fases do plano. Ele não refaz a entrevista do projeto: parte da `docs/SPEC.md` já aprovada e só esclarece o que for específico daquela funcionalidade.

Se o Superpowers não estiver instalado, usar o prompt portátil [brainstorm até plano](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/prompts/04-brainstorm-to-plan.md) com a mesma disciplina.

## Passo 0: quem está respondendo

Primeira pergunta, sempre: quem está respondendo é o Dan (ou alguém técnico do time) ou é um cliente/parceiro?

- **Modo técnico (Dan):** perguntas diretas, pode usar termos técnicos, pode agrupar 2 ou 3 perguntas relacionadas por vez.
- **Modo leigo (cliente):** uma pergunta por vez, zero jargão, sempre com exemplo. Nunca perguntar "qual o modelo de dados"; perguntar "que informações o sistema precisa guardar? Por exemplo: nome e telefone dos clientes, pedidos, pagamentos". Decisões técnicas não são perguntadas ao cliente: a skill decide e anota na spec para o Dan revisar.

Em qualquer modo: português brasileiro informal, respostas curtas, e a cada bloco devolver um resumo de 2 ou 3 linhas ("entendi que...") para a pessoa confirmar ou corrigir. Quando uma resposta admitir mais de uma leitura razoável, mostrar as leituras lado a lado em vez de escolher uma em silêncio.

## Passo 1: entrevista

Percorrer os blocos na ordem. Pular o que já foi respondido. Se a resposta for vaga, pedir um exemplo concreto ("me conta a última vez que isso aconteceu").

**Bloco A, problema e objetivo**
1. Que problema isso resolve? Como é feito hoje sem o sistema?
2. Como a gente vai saber que deu certo? (um resultado observável)
3. Tem prazo ou data que não pode passar?

**Bloco B, pessoas**
4. Quem vai usar? Quantos tipos de usuário existem (ex.: admin, cliente, equipe)?
5. O que cada tipo pode ver e fazer, e o que NÃO pode?
6. Quantas pessoas usando no começo e daqui a 6 meses?

**Bloco C, fluxo principal**
7. Me descreve o passo a passo da ação mais importante, do começo ao fim.
8. O que acontece quando dá errado no meio (pagamento recusado, dado faltando, pessoa desiste)?
9. Se só pudesse existir UMA função na primeira versão, qual seria?

**Bloco D, dados**
10. Que informações o sistema guarda?
11. Tem dado pessoal (nome, CPF, telefone, e-mail, endereço)? Tem dado sensível (saúde, financeiro, de menor de idade)?
12. De onde vêm os dados: a pessoa digita, importa planilha, vem de outro sistema?
13. Alguém precisa exportar ou apagar os próprios dados?

**Bloco E, integrações e dinheiro**
14. Precisa conversar com outros sistemas (WhatsApp, Eduzz, ManyChat, e-mail, planilhas, IA)?
15. Tem pagamento dentro do sistema? Quem processa?
16. Envia mensagens ou e-mails automáticos? Para quem e quando?

**Bloco F, operação**
17. Quem vai cuidar disso depois de pronto? Quem recebe o aviso se cair?
18. O que acontece com o negócio se ficar 1 hora fora do ar? E 1 dia?
19. Já existe domínio, conta, marca, design ou referência visual?

**Fechamento:** "Tem alguma coisa que eu não perguntei e você acha importante?"

## Passo 2: classificação de risco

Marcar cada item como sim ou não e definir o nível:

- Tem login?
- Guarda dado pessoal?
- Guarda dado sensível ou de menor de idade?
- Tem pagamento ou movimenta dinheiro?
- Tem mais de um tipo de usuário ou mais de um cliente no mesmo sistema (multi-tenant)?
- Dispara mensagens para terceiros (WhatsApp, e-mail)?
- Recebe webhooks ou expõe API pública?
- Aceita upload de arquivo?
- Usa IA com texto vindo do usuário?

**Nível 1 (baixo):** nenhum sim, ou só página/ferramenta interna sem dados de terceiros.
**Nível 2 (médio):** login e/ou dado pessoal comum.
**Nível 3 (alto):** pagamento, dado sensível, multi-tenant, ou disparo em massa.

O nível define quais itens do checklist de segurança são obrigatórios e o rigor da revisão no fim de cada fase. Explicar o nível para a pessoa em uma frase.

## Passo 3: gerar os documentos

Stack padrão: **Next.js + Supabase + Vercel**. Só sugerir outra se a entrevista mostrar um motivo concreto, e nesse caso justificar na spec.

Salvar na pasta do projeto, em `docs/`:

### `docs/SPEC.md`
- Resumo em 3 linhas (problema, para quem, resultado esperado)
- Tipos de usuário e matriz de permissões (tabela: ação x tipo de usuário)
- Escopo do MVP (lista numerada de funcionalidades)
- Fora de escopo (lista explícita do que NÃO entra agora)
- Fluxos principais passo a passo, incluindo os caminhos de erro
- Modelo de dados (tabelas, campos, relações, quem é dono de cada linha)
- Integrações (o quê, direção, credencial necessária)
- Telas (lista com o objetivo de cada uma)
- Critérios de aceite por funcionalidade, no formato "Dado... Quando... Então..."
- Suposições e perguntas em aberto

### `docs/SEGURANCA.md`
Nível de risco e o checklist aplicável (ver abaixo), cada item com caixinha para marcar, a fase em que será verificado e **como** será verificado (teste, regra de lint, hook ou conferência manual).

### `docs/PLANO.md`
Fases pequenas, cada uma entregando algo testável. Dentro de cada fase, tarefas no formato:

```
### T3.2 Validar cupom no endpoint de checkout
Files: src/app/api/checkout/route.ts, src/lib/cupons.ts
Depends-on: T3.1
Verificação: `npm test -- cupons` passa cobrindo cupom válido, expirado e inexistente
```

Regras do plano:
- Toda tarefa tem uma **verificação executável** (comando, teste ou comportamento observável). "Deve funcionar" não vale. Tarefa sem jeito de verificar está vaga demais: quebrar em tarefas menores.
- Toda tarefa tem `Files:` (arquivos que cria ou altera) e `Depends-on:` (IDs das tarefas de que depende, ou `none`). Isso deixa o plano pronto para execução em ondas paralelas. Se houver dúvida sobre o que a tarefa toca, tratar como dependente de tudo que veio antes.

Ordem padrão das fases:
1. **Fundação:** repositório, ambientes, variáveis, banco, auth, e o setup do toolkit (ver "Fase 1" abaixo)
2. **Modelo de dados** com permissões e testes de permissão
3. **Fluxo principal** de ponta a ponta (a função única do MVP)
4. **Funcionalidades secundárias**, uma por fase
5. **Integrações externas**
6. **Revisão de segurança completa**, testes finais, deploy de produção

### `CLAUDE.md` (raiz do projeto)
Partir do [template do toolkit](https://github.com/berdantx/vibe-coding-toolkit/blob/main/templates/CLAUDE.md.template) e preencher:
- **Stack:** TypeScript · Next.js + React · Supabase · Vercel · gerenciador de pacotes do projeto
- **Canonical commands:** os comandos reais de install, lint, typecheck, test, build e dev (preencher depois da Fase 1, nunca chutar)
- **Tabela de agentes especialistas:** manter só as linhas que o projeto usa; `security-reviewer` é obrigatório nos níveis 2 e 3

E acrescentar as regras deste projeto:
- Ler `docs/SPEC.md`, `docs/SEGURANCA.md` e `docs/PLANO.md` antes de qualquer tarefa
- Trabalhar uma fase por vez; não avançar sem as verificações da fase passando
- Mudança de escopo: parar, atualizar a SPEC e pedir aprovação antes de codar
- Nunca colocar segredo em código, commit ou log
- Consultar a documentação atual de Next.js, Supabase e Vercel (via Context7, se instalado) antes de implementar auth, RLS e deploy, em vez de confiar na memória
- Referenciar `.claude/rules/parallel-subagent-driven-development.md` se o projeto usar ondas paralelas
- Ao terminar cada fase, atualizar o PLANO com o que foi feito e o que ficou pendente

## Checklist de segurança (base para o SEGURANCA.md)

**Sempre (nível 1 em diante)**
- Segredos só em variáveis de ambiente; `.env` no `.gitignore`; nada sensível com prefixo `NEXT_PUBLIC_`
- Ambientes separados de desenvolvimento e produção
- HTTPS em tudo; dependências atualizadas e sem alerta crítico
- Validação de toda entrada no servidor (nunca confiar só no formulário)
- Mensagens de erro para o usuário sem detalhes internos

**Nível 2 em diante**
- RLS (Row Level Security) ligado em TODAS as tabelas, com política escrita para cada uma
- Chave `service_role` do Supabase só no servidor, nunca no navegador
- Toda checagem de permissão feita no servidor, validando o usuário da sessão
- Teste automatizado provando que o usuário A não lê nem altera dado do usuário B
- Limite de tentativas (rate limit) em login, cadastro e formulários públicos
- LGPD: coletar só o necessário, texto de consentimento, caminho para exportar e apagar dados
- Backup do banco confirmado e teste de restauração feito ao menos uma vez
- Log de erros e de ações importantes, sem dado pessoal nem segredo no log

**Nível 3**
- Pagamento sempre pelo processador (nunca guardar dado de cartão); valor e status confirmados no servidor
- Webhooks com verificação de assinatura e tratamento de evento repetido (idempotência)
- Isolamento entre clientes (multi-tenant) testado em todas as tabelas e rotas
- Trilha de auditoria para ações administrativas e financeiras
- Disparos em massa com trava de volume, confirmação humana e modo de teste
- Upload: limite de tamanho, tipos permitidos, armazenamento privado com link temporário
- IA com texto do usuário: tratar a saída como não confiável, sem dar acesso a ações destrutivas, e limitar custo por usuário

## Passo 4: aprovação

Apresentar um resumo curto: o que entra no MVP, o que ficou de fora, nível de risco, número de fases e as perguntas em aberto. Perguntar: "Posso seguir com isso ou quer ajustar algo?"

- No modo leigo, o resumo vai sem termos técnicos, e a spec fica marcada como "aguardando revisão técnica do Dan" antes de qualquer construção.
- Só depois do "pode seguir" começar a Fase 1 do PLANO.
- Plano que muda depois do feedback é esperado: reconfirmar só as partes que mudaram.

## Fase 1: fundação com o toolkit

Além de repositório, ambientes, banco e auth, a Fase 1 monta as proteções automáticas. Cada item entra como tarefa no PLANO, com verificação:

1. **Superpowers** instalado (comandos na [página de instalação](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/01-installation.md)). Conferir os comandos na hora: esse ecossistema muda rápido.
2. **Context7** instalado, para o agente consultar documentação atual das bibliotecas ([doc](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/tools/12-context7.md)).
3. **Lint como quality gate**, usando o [prompt de setup completo de ESLint](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/prompts/07-eslint-complete-setup.md). A partir do nível 2, incluir obrigatoriamente:
   - plugin de segurança do ESLint;
   - regra 7.6: toda server action e rota de API sensível precisa chamar a função de checagem de sessão do projeto;
   - regra 7.3: arquivos de interface não importam o cliente do banco direto.
   Limite conhecido da 7.6: ela prova que a checagem foi **chamada**, não que o resultado foi respeitado. Não substitui o teste de "usuário A não acessa dado do B" nem o RLS.
4. **Hook de arquivo de segredos:** aviso antes de editar `.env` e similares, seguindo [hooks, boas práticas](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/tools/10-hooks-best-practices.md). Hook de conveniência falha aberto; hook que impõe limite de segurança falha fechado.
5. **Regra de ondas paralelas** copiada para `.claude/rules/` ([arquivo](https://github.com/berdantx/vibe-coding-toolkit/blob/main/templates/rules/parallel-subagent-driven-development.md)), se o plano tiver tarefas independentes.
6. **`CLAUDE.md`** com os comandos canônicos reais preenchidos.

## Durante a construção

- **Execução:** a sessão principal planeja e coordena; a implementação vai para subagentes especialistas. Tarefas só rodam na mesma onda se não dependem uma da outra e não tocam os mesmos arquivos. Implementadores não fazem commit; quem commita é a sessão principal, uma tarefa por vez.
- **Fim de cada tarefa:** rodar a verificação da tarefa e mostrar o resultado antes de marcar como feita.
- **Fim de cada fase:** rodar lint, typecheck e testes; marcar os itens do `SEGURANCA.md` daquela fase; e rodar a revisão:
  - Nível 1: revisão simples de qualidade.
  - Níveis 2 e 3: painel do [code review multi-agente](https://github.com/berdantx/vibe-coding-toolkit/blob/main/docs/prompts/03-multi-agent-code-review.md), com a lente de segurança obrigatória. Revisores trabalham sem ver o achado um do outro; achado sem cenário concreto de falha é descartado; nada CRITICAL ou HIGH fica aberto ao passar de fase.
- **Fase 6 (nível 3):** a revisão final de segurança é feita por um agente que não participou da construção, conferindo o `SEGURANCA.md` item por item contra o código.
- Se uma verificação falhar, corrigir antes de avançar; não desativar nem pular teste ou regra de lint para "fazer passar".
- Se aparecer algo que a entrevista não cobriu, perguntar em vez de supor.

## Referências do toolkit

Tudo em https://github.com/berdantx/vibe-coding-toolkit

| Para quê | Arquivo |
|---|---|
| Instalação de plugins e CLIs | `docs/01-installation.md` |
| Passo a passo completo do fluxo | `docs/02-playbook-onboarding.md` |
| Brainstorm → plano → implementação → revisão | `docs/tools/01-superpowers.md` |
| Subagentes e ondas paralelas | `docs/tools/02-subagent-orchestration.md` |
| Quality gates de lint | `docs/tools/06-eslint-biome-quality-gates.md` |
| Hooks | `docs/tools/10-hooks-best-practices.md` |
| Documentação atualizada de bibliotecas | `docs/tools/12-context7.md` |
| Prompt: brainstorm até plano | `docs/prompts/04-brainstorm-to-plan.md` |
| Prompt: code review multi-agente | `docs/prompts/03-multi-agent-code-review.md` |
| Prompt: setup de ESLint | `docs/prompts/07-eslint-complete-setup.md` |
| Template do CLAUDE.md | `templates/CLAUDE.md.template` |
| Regra de ondas paralelas | `templates/rules/parallel-subagent-driven-development.md` |
