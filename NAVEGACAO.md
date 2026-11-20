# NAVEGACAO.md — TechPath

Mapa de telas, fluxo de navegação e estados de interface do MVP.
Complementa `PRODUCT.md` (escopo) e `PLANO.md` (tarefas). As regras visuais
estão em `DESIGN.md`.

> **Princípio:** toda tela precisa responder a quatro perguntas: o que a
> usuária vê ao abrir, o que ela pode fazer, o que acontece enquanto algo
> carrega e o que aparece quando dá certo ou errado.

---

## 1. Mapa de telas

| # | Tela | Arquivo | Acesso | Objetivo |
|---|------|---------|--------|----------|
| T1 | Login | `login.php` | Visitante | Entrar na conta |
| T2 | Cadastro | `cadastro.php` | Visitante | Criar conta |
| T3 | Áreas | `areas/index.php` | Público | Descobrir áreas da tecnologia |
| T4 | Detalhe da área | `areas/ver.php?id=` | Público | Conhecer uma área |
| T5 | Painel | `painel.php` | Logada | Ver resumo e atalhos |
| T6 | Minhas trilhas | `trilhas/index.php` | Logada | Listar trilhas e modelos |
| T7 | Detalhe da trilha | `trilhas/ver.php?id=` | Dona da trilha | Ver atividades e progresso |
| T8 | Criar trilha | `trilhas/criar.php` | Logada | Criar trilha do zero |
| T9 | Editar trilha | `trilhas/editar.php?id=` | Dona da trilha | Alterar dados da trilha |
| T10 | Personalizar modelo | `trilhas/personalizar.php?modelo=` | Logada | Copiar trilha modelo |
| T11 | Criar atividade | `atividades/criar.php?trilha=` | Dona da trilha | Adicionar atividade |
| T12 | Editar atividade | `atividades/editar.php?id=` | Dona da trilha | Alterar atividade |
| T13 | Sugestões com IA | `ia/solicitar.php?trilha=` | Dona da trilha | Pedir sugestões |
| T14 | Revisar sugestões | `ia/aprovar.php` | Dona da trilha | Aprovar o que entra |
| T15 | Página de erro | `erro.php?codigo=` | Público | Explicar 403/404/500 |
| — | Ações (sem tela) | `logout.php`, `atividades/concluir.php`, `atividades/excluir.php`, `trilhas/excluir.php` | Logada | Ações via POST |

**Telas fora do MVP:** perfil, configurações, rede social, chat, cursos.
Não criar.

### Guarnições de acesso

- Rotas protegidas exigem sessão válida; visitante é enviado para
  `login.php?proximo=<caminho>`.
- `proximo` só aceita caminho interno (começa com `/` e não com `//`), para
  evitar redirecionamento para sites externos.
- Logada que abre `login.php` ou `cadastro.php` vai para `painel.php`.
- Dado de outra usuária responde como **404** (não revela que existe).

---

## 2. Fluxo principal de navegação

```text
                        ┌──────────────┐
                        │  Visitante    │
                        └──────┬───────┘
             ┌─────────────────┼──────────────────┐
             ▼                 ▼                  ▼
        ┌─────────┐      ┌───────────┐      ┌────────────┐
        │ T1 Login│◄────►│ T2 Cadastro│      │ T3 Áreas   │
        └────┬────┘      └───────────┘      └─────┬──────┘
             │ login ok                           │
             ▼                                    ▼
        ┌─────────┐                        ┌────────────┐
        │ T5 Painel│◄──────────────────────│ T4 Detalhe │
        └────┬────┘                        └────────────┘
             │
     ┌───────┼─────────────────┐
     ▼       ▼                 ▼
┌─────────┐ ┌──────────────┐ ┌───────────────┐
│ T6      │ │ Áreas (T3)   │ │ T8 Criar      │
│ Trilhas │ └──────────────┘ │ trilha        │
└────┬────┘                  └───────┬───────┘
     │                               │
     ▼                               ▼
┌──────────────┐  modelo      ┌──────────────┐
│ T7 Trilha    │◄─────────────│ T10 Copiar   │
│ (atividades) │              │ modelo       │
└──┬────┬───┬──┘              └──────────────┘
   │    │   │
   │    │   └──────────────► T13 IA ──► T14 Revisar ──► T7
   │    └──────────────────► T11/T12 Atividade
   └───────────────────────► T9 Editar trilha

Header (sempre): logo → Painel · Áreas · Trilhas · Sair
```

### Passo a passo do fluxo principal

1. **Chegada:** visitante vê `login.php`; pode ir para `cadastro.php` ou
   explorar `areas/index.php` sem conta.
2. **Entrada:** após login ou cadastro, cai no **Painel**.
3. **Exploração:** Painel → Áreas → Detalhe da área.
4. **Primeira trilha:** do detalhe da área, botão "Criar trilha nesta área"
   abre o formulário com a área pré-selecionada; alternativa é usar um
   **modelo** em Minhas trilhas.
5. **Organização:** na trilha, adiciona atividades, edita, conclui.
6. **Acompanhamento:** o progresso atualiza na trilha e no Painel.
7. **IA (opcional):** na trilha, "Sugerir com IA" → formulário → revisão →
   só entra o que for aprovado.
8. **Saída:** "Sair" no header encerra a sessão e volta ao login.

### Fluxos secundários

- **Cadastro:** formulário → sucesso → login com mensagem "Conta criada!".
- **Personalizar modelo:** modelo → confirmação → cópia editável.
- **Excluir:** sempre pede confirmação e nunca é link GET.
- **Sessão expirada:** qualquer rota protegida → login com aviso.
- **Erro:** página amigável com caminho de volta.

---

## 3. Padrões globais de estado

### 3.1 Carregamento

O PHP renderiza a página pronta, então **não há spinner de página**. O
carregamento aparece apenas em ações:

| Situação | Comportamento |
|----------|---------------|
| Envio de formulário | Botão fica `disabled`, texto vira "Salvando…" / "Entrando…" e `aria-busy="true"` no formulário |
| Concluir atividade | Controle desabilitado + "Atualizando…" |
| Chamada de IA | Botão "Gerando sugestões…" + aviso "Isso pode levar alguns segundos." + 3 cartões esqueleto |
| Exclusão | Botão "Excluindo…" |
| Navegação normal | Sem estado artificial; o navegador mostra o carregamento padrão |

> Não usar tela de loading global, overlay bloqueante nem atraso artificial.

### 3.2 Lista vazia

Toda lista tem um estado vazio com: título curto, uma frase de orientação e
**uma ação principal**. Nunca deixar apenas "Nenhum registro".

### 3.3 Sucesso

Mensagem temporária no topo do conteúdo (flash), com `role="status"`,
fechável e tom positivo. Depois de POST bem-sucedido, sempre redirecionar
(POST → Redirect → GET) para evitar reenvio.

### 3.4 Erro

- **Erro de campo:** texto abaixo do campo + borda vermelha + foco no
  primeiro campo inválido.
- **Erro de formulário:** resumo no topo com links para os campos.
- **Erro de ação:** faixa de erro no topo, mantendo os dados digitados.
- **Erro de página:** tela T15 com código, explicação e botão "Voltar".
- Mensagens de erro nunca expõem detalhes técnicos, SQL ou dados de outra
  conta.

### 3.5 Mensagens flash padronizadas

| Chave | Texto | Tipo |
|-------|-------|------|
| `cadastro_ok` | "Conta criada! Entre para começar." | sucesso |
| `login_erro` | "E-mail ou senha incorretos." | erro |
| `sessao_expirada` | "Sua sessão expirou. Entre novamente." | aviso |
| `sem_permissao` | "Você não tem acesso a esta página." | erro |
| `trilha_criada` | "Trilha criada! Agora adicione suas atividades." | sucesso |
| `trilha_editada` | "Alterações salvas." | sucesso |
| `trilha_excluida` | "Trilha excluída." | sucesso |
| `modelo_copiado` | "Trilha criada a partir do modelo. Personalize como quiser." | sucesso |
| `atividade_criada` | "Atividade adicionada à trilha." | sucesso |
| `atividade_editada` | "Atividade atualizada." | sucesso |
| `atividade_excluida` | "Atividade excluída." | sucesso |
| `status_ok` | "Progresso atualizado." | sucesso |
| `ia_adicionada` | "Sugestões adicionadas à trilha." | sucesso |
| `ia_vazia` | "Selecione ao menos uma sugestão." | aviso |
| `erro_generico` | "Não foi possível concluir a ação. Tente novamente." | erro |

---

## 4. Especificação das telas

### T1 — Login (`login.php`)

**Quando aparece:** visitante, sessão expirada ou após logout.

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| E-mail | `email`, `autocomplete="username"` | obrigatório, formato válido |
| Senha | `password`, `autocomplete="current-password"` | obrigatório |

**Botões:** "Entrar" (primário, largura total no celular); link "Criar
conta"; link "Voltar para o início" (opcional).

**Mensagens:** erro genérico de credenciais; aviso de sessão expirada;
sucesso de cadastro.

**Estados**

- **Carregando:** botão "Entrando…" desabilitado.
- **Lista vazia:** não se aplica.
- **Sucesso:** redireciona para o Painel (ou `proximo` validado).
- **Erro:** faixa "E-mail ou senha incorretos."; campos preservados, senha
  limpa; foco no e-mail.
- **Erro de validação:** "Informe um e-mail válido." / "Informe sua senha."

**Wireframe (celular)**

```text
┌──────────────────────────┐
│ TechPath                 │
│                          │
│ Entrar                   │
│ Entre para continuar     │
│                          │
│ E-mail                   │
│ [______________________] │
│ Senha                    │
│ [______________________] │
│                          │
│ [       Entrar       ]   │
│                          │
│ Não tem conta? Criar     │
└──────────────────────────┘
```

### T2 — Cadastro (`cadastro.php`)

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| Nome | texto | obrigatório, 2–80 caracteres |
| E-mail | `email` | obrigatório, válido, único |
| Senha | `password` | mínimo 8 caracteres |
| Confirmar senha | `password` | igual à senha |

**Botões:** "Criar conta" (primário); "Mostrar senhas" (opcional, JS, com
`aria-pressed`); link "Já tenho conta".

**Mensagens:** erro por campo; "Este e-mail já está cadastrado. Tente
entrar."; sucesso "Conta criada! Entre para começar."

**Estados**

- **Carregando:** "Criando conta…" desabilitado.
- **Sucesso:** redireciona para login com flash.
- **Erro:** erros inline + resumo no topo; dados preservados (menos senhas).
- **E-mail duplicado:** erro no campo e-mail, com link para login.

**Wireframe (celular)**

```text
┌──────────────────────────┐
│ TechPath                 │
│ Criar conta              │
│                          │
│ Nome                     │
│ [______________________] │
│ E-mail                   │
│ [______________________] │
│ Senha                    │
│ [______________________] │
│ Confirmar senha          │
│ [______________________] │
│ [ ] Mostrar senhas       │
│                          │
│ [      Criar conta    ]  │
│ Já tenho conta           │
└──────────────────────────┘
```

### T3 — Áreas (`areas/index.php`)

**Campos:** nenhum (sem busca no MVP).

**Botões:** cada cartão é um link "Ver detalhes" para T4. No topo, link
"Minhas trilhas" se logada.

**Mensagens:** vazio "Ainda não há áreas publicadas. Volte em breve.";
erro "Não foi possível carregar as áreas. Tente novamente."

**Estados**

- **Carregando:** não se aplica (renderização no servidor).
- **Lista vazia:** cartão de estado vazio com ação "Ir para o início".
- **Sucesso:** grade de áreas com nome, resumo curto e "Ver detalhes".
- **Erro:** faixa de erro com botão "Tentar novamente" (recarrega).
- **Público:** não exige login.

### T4 — Detalhe da área (`areas/ver.php?id=`)

**Conteúdo:** nome, resumo, e seções "O que é", "Onde atuar",
"Conhecimentos", "Exemplos práticos", "Por onde começar".

**Botões:** "Criar trilha nesta área" (logada; visitante vai para login com
retorno) e "Voltar para áreas". Opcional: "Ver trilhas desta área".

**Mensagens:** "Área não encontrada." (id inválido ou inexistente).

**Estados**

- **Carregando:** não se aplica.
- **Lista vazia:** se uma seção não tiver conteúdo, ocultar a seção (não
  mostrar título vazio).
- **Sucesso:** conteúdo completo + CTA.
- **Erro:** T15 com 404 e botão "Ver todas as áreas".

### T5 — Painel (`painel.php`)

**Conteúdo:** saudação "Olá, {nome}", três cartões de resumo (trilhas em
andamento, trilhas concluídas, atividades pendentes), lista "Continue de
onde parou" (até 5 trilhas com barra de progresso) e atalhos "Explorar
áreas" / "Criar trilha".

**Campos:** nenhum.

**Botões:** "Criar trilha" (primário), "Explorar áreas" (secundário),
"Abrir" em cada trilha.

**Mensagens:** vazio "Você ainda não tem trilhas. Explore as áreas ou crie
sua primeira trilha."; erro "Não foi possível carregar seu resumo."

**Estados**

- **Carregando:** não se aplica.
- **Lista vazia:** estado vazio com os dois botões acima.
- **Sucesso:** números e lista atualizados.
- **Erro:** faixa de erro + "Tentar novamente".
- **Parcial:** se o resumo falhar, ainda mostrar os atalhos (não quebrar a
  página inteira).

### T6 — Minhas trilhas (`trilhas/index.php`)

**Estrutura:** duas abas — "Minhas trilhas" (padrão) e "Modelos"
(`?aba=modelos`).

**Campos:** nenhum. (Filtro por área fica para depois do MVP.)

**Botões:** "Criar trilha"; por cartão: "Abrir", "Editar", "Excluir" (minhas)
ou "Personalizar", "Ver" (modelos).

**Mensagens:** vazio minhas "Você ainda não criou trilhas."; vazio modelos
"Nenhum modelo disponível."; sucesso de exclusão.

**Estados**

- **Carregando:** não se aplica.
- **Aba vazia:** estado vazio com "Criar trilha" / "Usar um modelo".
- **Sucesso:** cartões com título, área (badge), "3 de 8 atividades" e
  progresso.
- **Erro:** faixa de erro.
- **Confirmação de exclusão:** diálogo "Excluir a trilha '{título}'? As
  atividades dela também serão excluídas. Esta ação não pode ser desfeita."
  com "Cancelar" e "Excluir" (perigo).

**Wireframe (computador)**

```text
┌─────────────── header: logo · Painel · Áreas · Trilhas · Sair ───────────────┐
│ Minhas trilhas                                  [ + Criar trilha ]           │
│ [ Minhas trilhas ] [ Modelos ]                                              │
│ ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐     │
│ │ Fundamentos de Web  │ │ Lógica com Python   │ │ …                   │     │
│ │ Frontend · 3 de 8   │ │ Dados · 0 de 5      │ │                     │     │
│ │ ▓▓▓▓▓░░░░░ 38%      │ │ ░░░░░░░░░░ 0%       │ │                     │     │
│ │ Abrir  Editar  Excluir │ │ Abrir  Editar  Excluir │ │                  │     │
│ └─────────────────────┘ └─────────────────────┘ └─────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### T7 — Detalhe da trilha (`trilhas/ver.php?id=`)

**Conteúdo:** título, área (se houver), descrição, progresso ("3 de 8
concluídas — 38%"), lista de atividades e ações.

**Campos:** nenhum (edição é em T9/T12).

**Botões:** "Adicionar atividade" (primário), "Editar trilha", "Sugerir com
IA", "Excluir trilha" (perigo, menos destacado). Por atividade:
"Concluir"/"Reabrir", "Editar", "Excluir", link do material.

**Mensagens:** vazio "Esta trilha ainda não tem atividades. Adicione a
primeira para começar a acompanhar seu progresso."; sucesso ao concluir
"Progresso atualizado."; erro "Não foi possível atualizar o status."

**Estados**

- **Carregando:** ao concluir, controle desabilitado com "Atualizando…".
- **Lista vazia:** estado vazio + "Adicionar atividade"; progresso 0%.
- **Sucesso:** item marcado, contadores e barra recalculados.
- **Erro:** faixa de erro e status anterior mantido.
- **Sem permissão:** T15 com 404.
- **100%:** barra completa, badge "Concluída" e mensagem positiva discreta.

**Wireframe (celular)**

```text
┌──────────────────────────┐
│ ← Trilhas       ⋮ Menu   │
├──────────────────────────┤
│ Fundamentos de Web       │
│ Frontend                 │
│ 3 de 8 concluídas — 38%  │
│ ▓▓▓▓▓░░░░░░░░░           │
│ [ Adicionar atividade ]  │
├──────────────────────────┤
│ ☑ HTML básico            │
│   Concluída · material   │
│ ☐ CSS: layout            │
│   Pendente · material    │
│ ☐ JavaScript: DOM        │
├──────────────────────────┤
│ Editar trilha            │
│ Sugerir com IA           │
│ Excluir trilha           │
└──────────────────────────┘
```

### T8 — Criar trilha (`trilhas/criar.php`)

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| Título | texto | obrigatório, 3–100 caracteres |
| Descrição | textarea | opcional, até 500 caracteres |
| Área | select | opcional; pré-selecionada se veio de T4 |

**Botões:** "Criar trilha" (primário), "Cancelar" (volta sem salvar).

**Mensagens:** "Trilha criada! Agora adicione suas atividades."; erros por
campo; "Não foi possível criar a trilha. Tente novamente."

**Estados**

- **Carregando:** "Criando…" desabilitado.
- **Sucesso:** redireciona para T7 da trilha nova.
- **Erro de validação:** mensagens inline + resumo; dados preservados.
- **Erro de servidor:** faixa de erro; dados preservados.
- **Cancelar:** se houver alterações, confirmar "Descartar o que foi
  digitado?".

### T9 — Editar trilha (`trilhas/editar.php?id=`)

**Campos:** mesmos de T8, preenchidos.

**Botões:** "Salvar alterações" (primário), "Cancelar".

**Mensagens:** "Alterações salvas."; "Trilha não encontrada."; erros de
validação.

**Estados**

- **Carregando:** "Salvando…".
- **Sucesso:** redireciona para T7.
- **Erro:** mensagens inline; dados preservados.
- **Sem permissão:** 404 (não 403, para não revelar existência).

### T10 — Personalizar modelo (`trilhas/personalizar.php?modelo=`)

**Conteúdo:** prévia do modelo (título, descrição, quantidade de atividades)
e aviso "O modelo original não será alterado."

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| Nome da minha trilha | texto | obrigatório, pré-preenchido com "{título} — minha versão" |

**Botões:** "Criar minha cópia" (primário), "Cancelar".

**Mensagens:** "Trilha criada a partir do modelo. Personalize como quiser.";
"Modelo não encontrado."

**Estados**

- **Carregando:** "Criando cópia…".
- **Sucesso:** T7 da cópia, com as atividades já copiadas como pendentes.
- **Erro:** mensagem no topo, sem perder o nome digitado.
- **Modelo vazio:** permitir cópia com aviso "Este modelo ainda não tem
  atividades.".

### T11 — Criar atividade (`atividades/criar.php?trilha=`)

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| Título | texto | obrigatório, 3–120 caracteres |
| Link do material | `url` | opcional, deve começar com `http://` ou `https://` |

**Botões:** "Adicionar atividade" (primário), "Cancelar".

**Mensagens:** "Atividade adicionada à trilha."; "Informe um link começando
com http:// ou https://"; "Não foi possível adicionar a atividade."

**Estados**

- **Carregando:** "Adicionando…".
- **Sucesso:** volta para T7 com flash; nova atividade no fim da lista,
  status "Pendente".
- **Erro:** mensagens inline; dados preservados.
- **Sem permissão:** 404.

### T12 — Editar atividade (`atividades/editar.php?id=`)

**Campos:** título e link, preenchidos. (Status muda pelo controle da T7.)

**Botões:** "Salvar alterações", "Cancelar".

**Mensagens:** "Atividade atualizada."; erros de validação.

**Estados**

- **Carregando:** "Salvando…".
- **Sucesso:** volta para T7.
- **Erro:** mensagens inline.
- **Sem permissão:** 404.

### T13 — Sugestões com IA (`ia/solicitar.php?trilha=`)

**Aviso de privacidade:** "Enviaremos apenas a área, o nível e o objetivo.
Não enviamos seu nome nem seu e-mail."

**Campos**

| Campo | Tipo | Regras |
|-------|------|--------|
| Área de interesse | select | obrigatório; pré-selecionada pela trilha |
| Nível | radio (Iniciante / Intermediário / Avançado) | obrigatório |
| Objetivo | textarea | obrigatório, 10–300 caracteres |

**Botões:** "Gerar sugestões" (primário), "Cancelar".

**Mensagens:** "A IA demorou para responder. Tente novamente."; "Não
encontramos sugestões para esses dados. Ajuste o objetivo e tente
novamente."; "Não foi possível gerar sugestões agora."

**Estados**

- **Carregando:** botão "Gerando sugestões…" + aviso de espera + 3 cartões
  esqueleto; limite de ~30 s.
- **Lista vazia:** mensagem + sugestão de ajustar o objetivo.
- **Sucesso:** redireciona para T14.
- **Erro:** faixa de erro; dados preservados para nova tentativa.
- **Sem permissão:** 404.

### T14 — Revisar sugestões (`ia/aprovar.php`)

**Conteúdo:** lista de sugestões (título + descrição curta), todas
**desmarcadas** por padrão; link "Selecionar todas".

**Campos:** checkbox por sugestão.

**Botões:** "Adicionar selecionadas" (primário), "Descartar sugestões",
"Gerar novamente" (volta para T13).

**Mensagens:** "Sugestões adicionadas à trilha."; "Selecione ao menos uma
sugestão."; "Estas sugestões expiraram. Gere novamente."

**Estados**

- **Carregando:** "Adicionando…".
- **Lista vazia:** "Nenhuma sugestão disponível. Gere novamente."
- **Sucesso:** volta para T7 com as atividades criadas como pendentes.
- **Erro:** faixa de erro; seleção preservada.
- **Expiração:** sugestões ficam só na sessão e somem ao sair da tela.

### T15 — Página de erro (`erro.php?codigo=`)

**Conteúdo:** código (403/404/500), título curto, explicação sem termos
técnicos e ação de saída.

**Botões:** "Voltar ao painel" (logada) ou "Ir para o login" (visitante);
"Ver áreas" (opcional).

**Mensagens por código**

| Código | Título | Texto |
|--------|--------|-------|
| 403 | "Acesso não permitido" | "Você não tem permissão para abrir esta página." |
| 404 | "Página não encontrada" | "O endereço não existe ou o conteúdo foi removido." |
| 500 | "Algo deu errado" | "Tente novamente em instantes. Se continuar, avise o suporte." |

**Estados:** não se aplica carregamento; sempre renderizar com o código
correto no `HTTP` status.

---

## 5. Desenho responsivo (visão geral)

### Celular (até 639px)

- Uma coluna; conteúdo com 16px de margem lateral.
- Header compacto (56px) com logo e menu recolhível (`<details>`).
- Botões de ação principais com largura total e altura ≥ 44px.
- Cada trilha/atividade vira um cartão empilhado (sem tabela).
- Formulários em uma coluna, labels acima dos campos.
- Ações secundárias ("Editar", "Excluir") agrupadas em uma linha discreta.
- Barra de progresso sempre visível no topo da trilha.

### Computador (a partir de 1024px)

- Container central de até 1080px.
- Header em linha única: logo à esquerda, navegação e "Sair" à direita.
- Painel em grade de 3 colunas para os cartões de resumo.
- Listas de trilhas em grade de 2–3 colunas.
- Formulários continuam em coluna única, com largura máxima de 640px
  (legibilidade).
- Área de detalhe com conteúdo principal e coluna lateral opcional para
  ações.

### Tablet (640–1023px)

- Uma ou duas colunas, conforme o componente; nunca espremer três cartões.

---

## 6. Regras de navegação e consistência

1. **Um caminho de volta** em toda tela: link "Voltar" ou migalha simples.
2. **Ação primária única** por tela; o resto é secundário.
3. **Ações destrutivas** sempre em vermelho, com confirmação e nunca como
   link GET.
4. **Sem modais para navegação**; modal só para confirmação curta.
5. **Foco visível** em todos os elementos interativos.
6. **Mensagens no topo** do conteúdo, nunca em alerta do navegador.
7. **Nada de perda silenciosa**: sair de formulário com dados pede
   confirmação.
8. **Mesmo vocabulário** em todas as telas: "trilha", "atividade",
   "concluída", "pendente".
9. **URLs previsíveis**: `?id=` para recurso, `?trilha=` para vínculo,
   `?aba=` para alternância.
10. **Estados testáveis**: toda tela com lista deve ter um dado de teste
    vazio e um preenchido no roteiro de QA.

---

## 7. Checklist de navegação (para validar a implementação)

- [ ] Visitante consegue ver Áreas sem login.
- [ ] Rota protegida sem sessão leva ao login e volta ao destino original.
- [ ] Cadastro bem-sucedido leva ao login com mensagem.
- [ ] Login bem-sucedido leva ao Painel.
- [ ] Toda lista vazia tem texto e ação.
- [ ] Toda ação POST mostra sucesso e redireciona.
- [ ] Erros de formulário mantêm os dados digitados.
- [ ] Nenhuma tela mostra dado de outra usuária.
- [ ] Menu funciona com teclado no celular e no computador.
- [ ] Nenhuma ação destrutiva acontece por GET.
- [ ] Todas as telas têm um caminho de volta.
