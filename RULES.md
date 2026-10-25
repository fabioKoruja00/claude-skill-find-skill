# Regras do repositório

## Idioma dos títulos e textos descritivos

- **Escopo:** obrigatório nos repositórios próprios de `fabioKoruja00` que **não sejam forks**, públicos ou privados. `phpIPAM-grnet` está incluído porque não é fork. **Nenhum fork** recebe esta regra; deve seguir a governança do upstream.
- **Português brasileiro (pt-BR)** nos títulos e textos descritivos escritos por nós: `Issues`, `Pull Requests`, `Releases`, `Discussions`, `Wiki`, `Projects`, mensagens de `Commit` (assunto e corpo), cabeçalhos narrativos e `CHANGELOG`.
- **Termos técnicos permanecem em inglês:** `Issue`, `Pull Request`, `PR`, `Branch`, `Commit`, `Merge`, `Squash`, `Rebase`, `Review`, `Draft`, `Checks`, `Workflow`, `GitHub Actions` e nomes de bibliotecas, tecnologias, protocolos, APIs, comandos, flags, caminhos, identificadores e código.
- **Convenções do GitHub não são traduzidas:** preservar os nomes de seções dos modelos de `Pull Request` (`Summary`, `Related Issues`, `Target Branch`, `Validation`) e mensagens geradas pelo GitHub, como `Merge pull request #N from ...`. O conteúdo descritivo de cada seção deve estar em português.
- **Conventional Commits:** preservar prefixos técnicos como `feat`, `fix`, `docs`, `test`, `chore`, `ci` e `refactor`, mas escrever a descrição após `:` em português. Exemplo: `docs(governanca): padronizar títulos em português`.
- **Pendências:** títulos das `Issues` em português, mantendo níveis de severidade e nomes técnicos; quando adotado no repositório, usar `O que fazer` e `Pronto quando` na descrição.
- **Histórico:** não reescrever `Commits` nem mensagens automáticas de `Merge` já publicados apenas para traduzir texto. Corrigir títulos descritivos editáveis ainda abertos e seguir a regra nas próximas contribuições.

## Estrelas das origens de skills adotadas

- Quando uma skill externa for instalada ou adotada, registrar o nome, a URL canônica do repositório GitHub de origem, o caminho do `SKILL.md` e a versão/commit (quando conhecido) no `SKILLS.md` ou no inventário de skills existente. Não incluir catálogos apenas consultados.
- Preferência explícita do proprietário: reconhecer com uma `Star` cada repositório de origem efetivamente utilizado, apenas pela conta GitHub autenticada do próprio usuário e por ação individual/autorizada. Verificar antes se já existe uma estrela; uma `Star` por repositório basta para várias skills. Não realizar automação massiva, engajamento artificial nem usar contas de terceiros.
- Se não houver ferramenta ou permissão para registrar/verificar a `Star`, acrescentar na `Pull Request` a URL da origem e um checkbox de estrela pendente; nunca declarar a estrela concluída sem confirmação. Não armazenar, copiar ou divulgar tokens e segredos.
- Aplicar este procedimento apenas à governança dos repositórios próprios; forks seguem as regras de seus projetos upstream.
