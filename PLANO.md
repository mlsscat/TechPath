# PLANO.md — TechPath (MVP)

Plano de execução do MVP descrito em `PRODUCT.md`, pensado para quem está
começando. Cada tarefa é pequena, tem um resultado visível e uma forma
clara de verificar se funcionou.

> **Regra geral:** uma tarefa por vez. Só marque `[x]` depois de executar a
> verificação e fazer o commit. Nenhuma tarefa deve durar mais que ~1 hora.

---

## 1. Estrutura de arquivos proposta

Estrutura simples, sem framework e sem Composer. As pastas são criadas aos
poucos, conforme as tarefas avançam (não é preciso criar tudo de uma vez).

```text
meu_projeto_LabD/
├── index.php               # redireciona para login ou painel
├── login.php               # formulário + processamento do login
├── logout.php              # encerra a sessão
├── cadastro.php            # criação de conta
├── painel.php              # resumo de trilhas e progresso
│
├── areas/                  # F2 — catálogo de áreas
│   ├── index.php           # lista de áreas
│   └── ver.php             # detalhe de uma área
│
├── trilhas/                # F3 — trilhas
│   ├── index.php           # minhas trilhas + trilhas modelo
│   ├── ver.php             # trilha + atividades + progresso
│   ├── criar.php
│   ├── editar.php
│   ├── excluir.php         # exclusão via POST
│   └── personalizar.php    # copiar trilha modelo para a usuária
│
├── atividades/             # F4 — atividades
│   ├── criar.php
│   ├── editar.php
│   ├── excluir.php         # exclusão via POST
│   └── concluir.php        # alternar status via POST
│
├── ia/                     # F6 — sugestões com IA
│   ├── solicitar.php       # formulário de sugestão
│   └── aprovar.php         # grava apenas o que a usuária aprovar
│
├── includes/               # código compartilhado (não acessível direto)
│   ├── config.php          # carrega .env
│   ├── db.php              # conexão PDO
│   ├── auth.php            # sessão e exigir_login()
│   ├── functions.php       # helpers: e(), redirecionar(), csrf_token()
│   ├── header.php          # topo + menu
│   └── footer.php          # rodapé + scripts
│
├── assets/
│   ├── css/style.css
│   └── js/app.js           # JS só onde realmente precisar
│
├── database/
│   ├── schema.sql          # estrutura atual do banco
│   └── seed.sql            # áreas e trilhas iniciais
│
├── .env                    # NÃO versionado (segredos)
├── .env.example            # modelo sem valores reais
├── .gitignore
├── README.md
├── PRODUCT.md
├── PLANO.md
└── AGENTS.md
```

**Por que assim?** Cada funcionalidade fica numa pasta com o nome que a
usuária vê na URL (ex.: `/trilhas/ver.php?id=3`). O código repetido fica em
`includes/`. Nada de URL amigável, roteador ou MVC completo nesta fase:
isso adicionaria complexidade sem necessidade para o MVP.

**Proteção básica de pastas:** `includes/` e `database/` não devem ser
abertas pelo navegador. Na Fase 0, criaremos um `.htaccess` simples (ou
configuração equivalente no Apache) para bloquear o acesso direto. Isso não
substitui mover o projeto para uma pasta `public/`, que fica como melhoria
futura.

---

## 2. Fases e tarefas

### Fase 0 — Base do projeto

- [ ] **0.1** Criar `.gitignore` com `.env`, `*.log`, dumps e arquivos de IDE.
  - Verificar: `git status` não mostra `.env` mesmo depois de criá-lo.
- [ ] **0.2** Criar `.env.example` com `APP_ENV`, `DB_HOST`, `DB_PORT`,
      `DB_NAME`, `DB_USER`, `DB_PASS` (sem valores reais).
  - Verificar: o arquivo tem apenas nomes de variáveis e valores de exemplo.
- [ ] **0.3** Criar o banco `techpath` e um usuário de aplicação com
      permissões apenas nesse banco.
  - Verificar: `mariadb -u techpath -p techpath -e "SELECT 1;"` funciona.
- [ ] **0.4** Criar `includes/config.php` que lê o `.env` (parser simples,
      sem dependências) e expõe as configurações.
  - Verificar: `php -l includes/config.php` e um teste rápido via
      `php -r 'require "includes/config.php"; var_dump(DB_NAME);'`.
- [ ] **0.5** Criar `includes/db.php` com conexão PDO (`utf8mb4`, erros em
      exceção).
  - Verificar: script curto via `php -r` retorna a versão do MariaDB.
- [ ] **0.6** Criar `database/schema.sql` com a tabela `users`.
  - Verificar: `mariadb ... techpath < database/schema.sql` roda sem erro e
      `DESCRIBE users;` mostra as colunas esperadas.
- [ ] **0.7** Adicionar `areas` ao schema (nome, slug, descrição, conteúdo).
  - Verificar: `SHOW TABLES;` lista `areas`.
- [ ] **0.8** Adicionar `trails` (dona, área opcional, título, descrição,
      `is_template`, timestamps) com chaves estrangeiras.
  - Verificar: `SHOW CREATE TABLE trails;` mostra as FKs.
- [ ] **0.9** Adicionar `activities` (trilha, título, status, link, ordem)
      com `ON DELETE CASCADE` em relação a `trails`.
  - Verificar: apagar uma trilha de teste remove suas atividades.
- [ ] **0.10** Criar `database/seed.sql` com 3 áreas e 1 trilha modelo.
  - Verificar: `SELECT COUNT(*) FROM areas;` retorna 3.
- [ ] **0.11** Criar `includes/header.php`, `includes/footer.php` e
      `assets/css/style.css` com layout base (menu, container, responsivo).
  - Verificar: página de teste temporária abre no navegador em desktop e
      celular sem erros no console.
- [ ] **0.12** Criar `.htaccess` bloqueando `includes/`, `database/`, `.env`
      e arquivos `.md`.
  - Verificar: acessar `/includes/config.php` no navegador retorna 403.

### Fase 1 — F1: cadastro e autenticação

- [ ] **1.1** Criar `cadastro.php` com formulário (nome, e-mail, senha,
      confirmação) usando o layout base.
  - Verificar: página abre e o HTML é válido.
- [ ] **1.2** Processar o cadastro com validação no servidor (campos
      obrigatórios, e-mail válido, senha mínima, senhas iguais, e-mail
      único) usando `password_hash()`.
  - Verificar: cadastro válido cria linha em `users` com hash começando em
      `$2y$`; e-mail repetido mostra mensagem amigável.
- [ ] **1.3** Criar `login.php` com verificação via `password_verify()`,
      `session_regenerate_id()` e mensagem genérica de erro.
  - Verificar: login correto redireciona; senha errada não revela se o
      e-mail existe.
- [ ] **1.4** Criar `includes/auth.php` com `exigir_login()` e
      `usuaria_logada()`.
  - Verificar: acessar `painel.php` deslogada redireciona para `login.php`.
- [ ] **1.5** Criar `logout.php` (via POST) e `index.php` redirecionando
      conforme o estado da sessão.
  - Verificar: após logout, o botão "voltar" não devolve acesso ao painel.
- [ ] **1.6** Exibir o nome da usuária no header e esconder menu de visitante.
  - Verificar: header muda entre logada e deslogada.
- [ ] **1.7** Implementar token CSRF nos formulários (helper em
      `functions.php`) e validar em todos os POSTs.
  - Verificar: enviar formulário sem token é recusado.

### Fase 2 — F2: catálogo de áreas

- [ ] **2.1** Criar `areas/index.php` listando as áreas do banco.
  - Verificar: as 3 áreas do seed aparecem, sem SQL manual na página.
- [ ] **2.2** Criar `areas/ver.php` exibindo o conteúdo completo de uma área
      (o que é, atuação, conhecimentos, exemplos, por onde começar).
  - Verificar: cada área do seed abre com conteúdo próprio; `id` inexistente
      mostra "não encontrada".
- [ ] **2.3** Adicionar link "Áreas" no menu (logada e visitante, conforme
      decisão de acesso).
  - Verificar: navegação funciona em desktop e celular.
- [ ] **2.4** Escapar toda saída dinâmica com `e()` (`htmlspecialchars`).
  - Verificar: cadastrar área de teste com `<script>` no nome e confirmar que
      aparece como texto, não executa.

### Fase 3 — F3: trilhas

- [ ] **3.1** Criar `trilhas/index.php` com "Minhas trilhas" e "Trilhas
      modelo" (somente leitura).
  - Verificar: usuária nova vê 1 modelo e nenhuma trilha própria.
- [ ] **3.2** Criar `trilhas/criar.php` (título, descrição, área opcional).
  - Verificar: trilha criada aparece na lista e pertence à usuária logada.
- [ ] **3.3** Criar `trilhas/editar.php` com verificação de dona no servidor.
  - Verificar: usuária B recebe 403/404 ao tentar editar trilha de A.
- [ ] **3.4** Criar `trilhas/excluir.php` via POST com confirmação no JS.
  - Verificar: exclusão remove a trilha e suas atividades.
- [ ] **3.5** Criar `trilhas/personalizar.php` copiando um modelo para a
      usuária, sem alterar o original.
  - Verificar: modelo continua intacto após personalizar; cópia é editável.
- [ ] **3.6** Criar `trilhas/ver.php` com dados da trilha e lista de
      atividades (ainda sem progresso).
  - Verificar: trilha sem atividades mostra estado vazio amigável.

### Fase 4 — F4: atividades

- [ ] **4.1** Criar `atividades/criar.php` vinculada a uma trilha da usuária.
  - Verificar: atividade criada aparece na trilha correta.
- [ ] **4.2** Criar `atividades/editar.php` (título, link de material).
  - Verificar: edição persiste após recarregar; trilha de outra usuária é
      bloqueada.
- [ ] **4.3** Criar `atividades/excluir.php` via POST.
  - Verificar: atividade some da lista e do banco.
- [ ] **4.4** Criar `atividades/concluir.php` alternando
      `pendente` ↔ `concluida` via POST.
  - Verificar: status muda na lista imediatamente após recarregar.
- [ ] **4.5** Validar que o link de material é uma URL http(s) e abrir em
      nova aba com `rel="noopener noreferrer"`.
  - Verificar: `javascript:` e texto comum são recusados.

### Fase 5 — F5: progresso

- [ ] **5.1** Criar `calcular_progresso($trailId)` em `includes/functions.php`
      com a fórmula do PRODUCT.md (0% quando não há atividades).
  - Verificar: trilha vazia retorna 0; 1 de 4 concluídas retorna 25.
- [ ] **5.2** Exibir progresso em `trilhas/ver.php` (valor + barra visual).
  - Verificar: concluir/desconcluir atividade atualiza o percentual.
- [ ] **5.3** Criar `painel.php` com resumo: trilhas em andamento,
      concluídas e atividades pendentes.
  - Verificar: números batem com o banco para uma usuária de teste.
- [ ] **5.4** Garantir que painel e progresso mostrem apenas dados da
      usuária logada.
  - Verificar: criar segunda usuária e comparar os painéis.

### Fase 6 — F6: sugestões com IA (dependência externa)

> Só iniciar quando as fases anteriores estiverem estáveis. A chave da API
> fica no `.env` e nunca no código ou no Git.

- [ ] **6.1** Definir provedor, modelo, custo e formato de resposta (JSON com
      lista de atividades). Registrar a decisão no README.
  - Verificar: uma chamada manual de teste retorna JSON no formato esperado.
- [ ] **6.2** Criar `ia/solicitar.php` (área, nível, objetivo) com validação.
  - Verificar: campos inválidos são recusados antes de chamar a API.
- [ ] **6.3** Integrar a chamada no PHP com timeout, tratamento de erro e
      limite de tamanho da resposta.
  - Verificar: desligar a rede/simular erro mostra mensagem amigável, sem
      quebrar a página.
- [ ] **6.4** Exibir sugestões em tela de revisão (nada salvo
      automaticamente).
  - Verificar: recarregar a página não cria atividades.
- [ ] **6.5** Implementar `ia/aprovar.php` gravando apenas itens aprovados.
  - Verificar: aprovar 2 de 3 sugestões cria exatamente 2 atividades.
- [ ] **6.6** Revisar privacidade: enviar o mínimo de dados e não registrar
      prompts/respostas com dados pessoais nos logs.
  - Verificar: logs não contêm e-mail, nome ou objetivo da usuária.

### Fase 7 — Qualidade e entrega

- [ ] **7.1** Revisar segurança: SQL injection, XSS, CSRF, sessão, uploads
      (se houver), mensagens de erro em produção.
  - Verificar: checklist de segurança preenchido no README.
- [ ] **7.2** Revisar acessibilidade básica: labels, foco visível, contraste,
      navegação por teclado.
  - Verificar: percorrer o fluxo principal só com o teclado.
- [ ] **7.3** Testar responsividade nas larguras 360px, 768px e 1280px.
  - Verificar: sem rolagem horizontal e sem elementos sobrepostos.
- [ ] **7.4** Escrever `README.md` com instalação, banco, `.env` e execução.
  - Verificar: seguir o README do zero em uma pasta limpa funciona.
- [ ] **7.5** Executar o roteiro de teste ponta a ponta (cadastro → área →
      trilha → atividades → progresso → IA → logout).
  - Verificar: todas as etapas passam sem erro no log do PHP.
- [ ] **7.6** Revisão final do código e limpeza de arquivos temporários.
  - Verificar: `git status` limpo, sem arquivos de teste esquecidos.

---

## 3. Definição de pronto (por tarefa)

Uma tarefa só é considerada concluída quando:

1. O comportamento descrito funciona no navegador.
2. `php -l` passa nos arquivos alterados.
3. Não há warnings/notices no log do PHP nem erros no console do navegador.
4. A verificação da própria tarefa foi executada.
5. Foi feito um commit pequeno, com mensagem clara.

## 4. Ordem recomendada

Fase 0 → 1 → 2 → 3 → 4 → 5 → 6 → 7.

As fases 2 e 3 podem trocar de ordem, mas a Fase 1 (autenticação) deve vir
primeiro porque todas as demais dependem de saber quem é a usuária.
