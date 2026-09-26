# Plano de desenvolvimento — Painel de configurações do projeto

Data: 26/09/2026

Projeto inspecionado: `/home/anderson/T.A.INTERFACES`

Entrega deste documento: planejamento da construção; a tela e o serviço ainda serão implementados.

## 1. Objetivo e resultado esperado

Construir uma nova tela no navegador para consultar, editar, validar, salvar e aplicar todos os parâmetros configuráveis do projeto, incluindo as variáveis de arquivos `.env`. A interface deverá usar componentes reais da Watermelon UI, funcionar em português e oferecer temas claro, escuro e automático conforme o sistema.

O administrador deverá conseguir identificar o que cada configuração faz, de onde vem o valor, qual valor está salvo e quando uma alteração passa a valer. Campos confidenciais terão operações de substituição e remoção sem exposição do conteúdo existente.

### Como acompanhar este plano

- `[ ]`: trabalho pendente, inclusive quando parcialmente iniciado.
- `[x]`: trabalho concluído com evidência registrada.
- Marcar uma etapa somente depois de cumprir seus critérios de aceite.
- Registrar data, arquivos, testes e pendências na seção de acompanhamento.
- Não interpretar uma decisão de arquitetura documentada como funcionalidade implementada.

## 2. Diagnóstico confirmado do repositório

| Evidência | Situação observada | Consequência para a construção |
| --- | --- | --- |
| `README.md` | Oficina de prototipação com servidor MkDocs | Preservar a documentação e seu fluxo de execução |
| `requirements.txt` | `mkdocs==1.6.1` e `mkdocs-material==9.7.6` | Aproveitar a base Python para o serviço administrativo |
| `mkdocs.yml` | Configuração do site, tema, navegação, plugins e extensões | Deve entrar no inventário de parâmetros |
| `docs/stylesheets/workshop.css` | Estilos e cores definidos diretamente em CSS | A aparência não é controlada apenas pelo YAML |
| `.env` e variantes | Nenhum arquivo encontrado na inspeção do projeto | Não inventar variáveis como se já existissem |
| Leitura de ambiente | Não foram encontradas referências a `getenv`, `environ`, `!ENV`, `process.env` ou `import.meta.env` nos arquivos de projeto inspecionados | Novas variáveis precisam de consumidores explícitos |
| Frontend React e API | Não encontrados | A tela exige uma nova aplicação administrativa e um serviço de persistência |
| Temas do site | Paletas `default` e `slate`, com CSS próprio | Separar preferência do painel da aparência publicada do site |
| Publicação | `site_url` aponta para GitHub Pages com prefixo `/oficina-framer/` | A hospedagem estática não pode gravar arquivos locais nem executar a API |

A inspeção excluiu dependências, `.git` e saída gerada `site/`. Não houve leitura de valores secretos. Esta base deve ser reavaliada no início da implementação, pois novos arquivos podem surgir.

**Premissa de escopo:** este plano se aplica ao repositório acima. O “Cardápio digital” citado na conversa é uma proposta de atividade na documentação, não uma aplicação implementada encontrada aqui. Se o alvo for outro sistema, refazer o inventário naquele repositório antes de implementar os adaptadores.

### Diagnóstico concluído

- [x] Inspecionar estrutura e arquivos de configuração existentes.
- [x] Verificar existência de arquivos e consumidores `.env` no código do projeto.
- [x] Identificar diferenças entre site estático e painel com escrita no servidor.
- [x] Consultar documentação e catálogo público da Watermelon UI.
- [ ] Revalidar o diagnóstico no início da implementação.

## 3. Definição verificável de “todos os parâmetros”

O universo de cobertura será a união de:

1. Todas as chaves presentes nos arquivos `.env` e modelos de ambiente do projeto.
2. Todas as variáveis consumidas explicitamente pelo código e pelos scripts de execução do projeto.
3. Todas as propriedades de configuração existentes em `mkdocs.yml`, inclusive objetos, listas e tags YAML.
4. Todos os parâmetros de execução introduzidos para o painel e para o servidor de documentação.
5. Todos os tokens de aparência extraídos do CSS para configuração declarativa.
6. Novas chaves descobertas após a implantação do painel.

“Todos” não significa copiar o ambiente inteiro do sistema operacional, editar dependências instaladas ou transformar cada linha de CSS e cada texto de conteúdo em campo de formulário. Constantes que representem decisões configuráveis deverão ser catalogadas e extraídas; conteúdo editorial continua nos arquivos Markdown.

### Regras de cobertura

- [ ] Criar inventário versionado por chave e origem, sem valores secretos.
- [ ] Associar cada chave a um campo tipado ou editor estruturado avançado.
- [ ] Disponibilizar chaves desconhecidas na seção “Outras configurações”, com tipo textual inicial e tratamento confidencial por padrão para `.env`.
- [ ] Permitir cadastrar, renomear e remover chaves `.env`, com revisão das referências e avisos de impacto.
- [ ] Distinguir chave inexistente, vazia, definida e herdada.
- [ ] Exibir campos derivados e externos como somente leitura, com motivo e origem.
- [ ] Não contar campos somente leitura como cobertura de edição completa; registrar exceções explicitamente.
- [ ] Mostrar total descoberto, editável, somente leitura e ainda sem tratamento.
- [ ] Criar verificação automática de diferenças entre inventário e campos da tela.
- [ ] Bloquear conclusão da entrega se houver chave encontrada e silenciosamente omitida.
- [ ] Para cada variável, comprovar o consumidor; marcar chaves sem consumidor como “salva, sem efeito conhecido”.

**Aceite:** nenhuma chave do universo delimitado fica oculta. Toda restrição de edição possui explicação e procedimento para mudança; nenhuma exceção fica implicitamente fora do denominador.

## 4. Arquitetura proposta

### 4.1 Componentes

| Camada | Proposta | Responsabilidade |
| --- | --- | --- |
| Interface | React + TypeScript, com build Vite | Formulários, revisão, estados e temas |
| Componentes | Watermelon UI, incorporados a partir dos registros oficiais selecionados | Base visual real e rastreável |
| API local | Python + FastAPI, versões fixadas na implementação | Catálogo, validação, persistência, sessão e operações |
| Metadados | Catálogo JSON versionado e validado por esquema | Tipo, descrição, origem, segredo, regras e impacto de cada campo |
| Adaptadores | `.env`, YAML e tokens visuais | Leitura e alteração sem perder estrutura |
| Aplicação | Supervisor local dos processos gerenciados | Rebuild, reinício controlado e verificação de saúde |

Essas tecnologias são decisões propostas para a nova aplicação, não dependências já presentes. A fase de fundação deve validar compatibilidade, versões e custo de manutenção antes de fixá-las.

### 4.2 Fluxo de dados

```text
Navegador /configuracoes
        |
        v
API local autenticada /api/configuracoes
        |
        +--> catálogo + validação + revisão de mudanças
        |
        +--> adaptadores --> .env / mkdocs.yml / tokens visuais
        |
        +--> operações --> rebuild ou reinício --> verificação de saúde
```

- [ ] Servir o build do painel e a API pela mesma origem em execução local.
- [ ] Durante desenvolvimento, usar proxy do Vite para a API e restringir origens.
- [ ] Definir `/configuracoes` como rota do painel e suportar recarga direta.
- [ ] Manter o site MkDocs em processo separado, respeitando `/oficina-framer/`.
- [ ] Usar loopback como endereço padrão do painel administrativo.
- [ ] Implementar o painel sem incluí-lo no artefato público do GitHub Pages.
- [ ] Mostrar link para abrir a documentação e indicar quando ela estiver indisponível.
- [ ] Registrar portas e política de conflito; não encerrar processos de outros projetos.

### 4.3 Estrutura sugerida de arquivos

```text
admin-ui/
  package.json
  src/
    app/
    components/watermelon/
    features/configuracoes/
    services/
    styles/
    tests/
admin-api/
  requirements.txt
  app/
    routes/
    schemas/
    services/
    adapters/
    security/
  tests/
config/
  catalogo-configuracoes.json
  catalogo-configuracoes.schema.json
  aparencia.json
scripts/
  iniciar-painel.sh
  verificar-cobertura-configuracoes.py
docs/operacao/
  painel-configuracoes.md
.env.example
.local/configuracoes/           # estado privado, backups e journal; ignorado no Git
```

## 5. Catálogo e fontes de verdade

Cada campo deve possuir: `id`, `chave`, `origem`, `caminho`, `grupo`, `rotulo`, `descricao`, `tipo`, `obrigatorio`, `padrao`, `opcoes`, `restricoes`, `sensivel`, `editavel`, `motivoSomenteLeitura`, `dependencias`, `consumidores` e `aplicacao`.

`aplicacao` admite: imediata, rebuild, reinício do MkDocs, reinício do painel ou externa/manual. Os valores atuais devem vir dos arquivos e do estado conhecido da execução, nunca do catálogo estático.

### 5.1 Matriz inicial dos parâmetros existentes

| Grupo | Chaves/estruturas existentes | Editor previsto |
| --- | --- | --- |
| Identificação | `site_name`, `site_description`, `site_author`, `site_url`, `copyright` | Texto, área de texto e URL |
| Tema e idioma | `theme.name`, `theme.language`, `theme.font.text`, `theme.font.code` | Seletores ou texto validado |
| Paletas | Todos os itens de `theme.palette`, incluindo `scheme`, `primary`, `accent`, `toggle.icon`, `toggle.name` | Coleção de paletas e campos aninhados |
| Recursos | Todos os itens de `theme.features` | Lista pesquisável com inclusão de valores adicionais |
| Estilos | Todos os itens de `extra_css` | Lista ordenável de arquivos do projeto |
| Redes e links | Todos os itens de `extra.social`: `icon`, `link`, `name` | Coleção de formulários |
| Navegação | Toda a árvore `nav`, nomes e destinos | Árvore editável com ordenação e inclusão/remoção |
| Plugins | Toda a estrutura `plugins`, incluindo `search.lang` | Campos conhecidos e editor estruturado de opções |
| Markdown | Toda a estrutura `markdown_extensions`, incluindo configurações de `pymdownx.emoji` | Lista, opções aninhadas e referências especiais protegidas |
| Aparência própria | Cores, tipografia, espaçamentos e medidas configuráveis do CSS | Tokens extraídos com tipo e unidade |
| Ambiente | Chaves que vierem a existir nos `.env` ou consumidores | Formulário gerado por metadados e seção avançada |

Tags `!!python/name` existentes no YAML precisam ser preservadas sem executar objetos durante a leitura da API. Mudanças em plugins, extensões e referências executáveis exigem validação contra componentes instalados e política explícita de confiança; um formulário não deve instalar ou importar código arbitrário.

### 5.2 Ambiente e precedência

- [ ] Definir `.env` como arquivo local padrão; `.env.example` conterá apenas modelos sem segredos.
- [ ] Descobrir outras variantes existentes, se surgirem, e mostrar qual arquivo é efetivamente consumido.
- [ ] Não introduzir múltiplos perfis até haver um carregador e um caso de uso definido.
- [ ] Para o novo serviço, documentar precedência: ambiente do processo > arquivo selecionado > padrão do catálogo.
- [ ] Para MkDocs, manter YAML como fonte de verdade; só vincular uma chave ao ambiente com consumidor explicitamente implementado.
- [ ] Mostrar quando um valor do processo sobrescreve o arquivo editado e impede efeito imediato.
- [ ] Evitar duas fontes editáveis concorrentes para o mesmo parâmetro.
- [ ] Não carregar `.env` por `source`, shell ou execução de expressões.
- [ ] Definir suporte a aspas, escapes, comentários, `export`, Unicode, quebras de linha e interpolação conforme o parser escolhido.
- [ ] Detectar chaves duplicadas e solicitar resolução antes de gravar, sem decidir silenciosamente qual remover.

### 5.3 Novas variáveis propostas para a infraestrutura

Os nomes abaixo são propostas, ainda inexistentes; só devem ser criados junto com consumidores e testes.

| Variável proposta | Uso | Efeito |
| --- | --- | --- |
| `ADMIN_HOST` | Interface de rede do painel; padrão loopback | Reinício do painel |
| `ADMIN_PORT` | Porta do painel | Reinício do painel |
| `DOCS_HOST` | Endereço do MkDocs gerenciado | Reinício da documentação |
| `DOCS_PORT` | Porta do MkDocs gerenciado | Reinício da documentação |
| `ADMIN_LOG_LEVEL` | Nível dos logs sanitizados | Conforme implementação do logger |
| `CONFIG_BACKUP_RETENTION` | Quantidade de versões privadas mantidas | Próxima limpeza de backups |

Credenciais de sessão devem ser geradas pelo serviço e guardadas no armazenamento privado; o painel não deve depender de um segredo embutido no JavaScript. Alterações que impeçam o próximo início precisam de instrução de recuperação pelo arquivo local.

## 6. Organização e comportamento da tela

### 6.1 Composição

```text
Configurações do projeto                  [Claro/Escuro/Sistema]
Projeto e diretório                       [Estado da aplicação]
[Buscar por nome, chave ou descrição.........................]

Seções                   Campos da seção selecionada
Geral                    Rótulo + chave técnica
Ambiente                 Controle + ajuda + origem + impacto
Documentação             Validação junto ao campo
Aparência do site
Navegação e links         Lista/árvore para estruturas compostas
Plugins e extensões
Servidor
Outras configurações
Histórico

N alterações pendentes    [Descartar] [Validar] [Revisar e salvar]
```

- [ ] Exibir o nome e caminho do projeto para evitar edição no alvo errado.
- [ ] Disponibilizar busca global e filtros: alterados, inválidos, obrigatórios, confidenciais e origem.
- [ ] Manter erros e alterações ao trocar de seção.
- [ ] Mostrar contador de parâmetros por seção e cobertura total.
- [ ] Implementar controles por tipo: texto, senha de substituição, número, booleano, enumeração, lista, objeto, URL e caminho.
- [ ] Permitir edição completa de listas e árvores; não limitar o formulário aos primeiros elementos.
- [ ] Usar editor avançado com validação para estruturas ainda sem controle especializado.
- [ ] Avisar sobre alterações pendentes ao sair, recarregar ou trocar de projeto/contexto.
- [ ] Impedir envio duplicado e preservar o rascunho em erro de rede.
- [ ] Manter rascunhos sensíveis somente em memória e apagá-los ao encerrar a sessão.
- [ ] Diferenciar “restaurar padrão do campo”, “descartar rascunho” e “restaurar versão salva”.
- [ ] Revisar antes/depois e impacto antes da gravação; apresentar apenas estado dos segredos.
- [ ] Mostrar estados de carregamento, vazio, falha, sem permissão, sessão expirada e conflito de versão.
- [ ] Exibir “salvo, aguardando aplicação”, “aplicando”, “aplicado” ou “falha ao aplicar”.

### 6.2 Watermelon UI: seleção e integração

A fonte exigida é a [Watermelon UI](https://ui.watermelon.sh/components), que disponibiliza exemplos React e caminhos de integração. O [catálogo público](https://ui.watermelon.sh/api/docs) informa endpoints de listagem e detalhe. A listagem consultada de blocos retornou referências de registro e comandos de instalação por item. A pesquisa por `settings` não retornou blocos; portanto, este plano não presume a existência de um painel de configurações pronto.

| Necessidade de interface | Seleção a fazer no catálogo |
| --- | --- |
| Estrutura e navegação | Composição de painel, sidebar ou navegação equivalente |
| Formulários | Inputs, seletores, alternadores, botões e agrupamentos |
| Revisão e confirmação | Modal ou painel lateral com foco acessível |
| Mensagens | Alerta, indicador de estado e notificação |
| Histórico | Lista ou tabela com detalhes de uma versão |
| Tema | Controle acessível de preferência e tokens de cor |

Os nomes acima descrevem necessidades, não exports ou componentes já confirmados na biblioteca.

- [ ] Consultar catálogo completo e fontes dos candidatos; validar paginação/limites para não usar apenas a primeira página.
- [ ] Registrar slug, URL de demonstração, URL do registro, revisão ou hash, licença e dependências de cada componente escolhido.
- [ ] Inspecionar código e dependências antes de executar comandos de instalação.
- [ ] Criar prova de integração com um bloco estrutural e um controle de formulário reais da Watermelon.
- [ ] Confirmar compatibilidade com React, TypeScript, estilos, ícones e eventual biblioteca de animação.
- [ ] Guardar componentes incorporados em `components/watermelon/` e registrar adaptações locais.
- [ ] Compor a tela com os componentes selecionados; não entregar apenas uma imitação visual da biblioteca.
- [ ] Quando faltar um controle, documentar a lacuna e compor um componente acessível a partir da base incorporada.
- [ ] Fixar dependências e preservar atribuições/licenças aplicáveis.
- [ ] Validar ambos os temas antes de adotar cada componente na tela inteira.

**Aceite:** a implementação referencia componentes reais selecionados, com procedência e dependências verificáveis; nenhum comando de pacote é inventado.

## 7. Tema claro, escuro e acessibilidade

- [ ] Implementar preferências “Claro”, “Escuro” e “Sistema”.
- [ ] Persistir apenas a preferência visual no navegador e aplicar antes da primeira pintura.
- [ ] Observar mudanças do sistema enquanto o modo automático estiver selecionado.
- [ ] Definir tokens semânticos para fundo, superfície, texto, borda, foco, sucesso, aviso e erro.
- [ ] Usar os tokens também em selects, modais, menus, tooltips, toasts, tabelas e editores.
- [ ] Distinguir a preferência do painel dos valores de `theme.palette` do site.
- [ ] Mostrar prévia dos tokens do site sem gravar ou aplicar automaticamente.
- [ ] Migrar apenas propriedades configuráveis do CSS para uma camada de tokens, preservando os valores atuais como padrão.
- [ ] Verificar contraste de texto e controles, foco visível, labels, descrições e anúncios de erros.
- [ ] Garantir navegação por teclado, fechamento com Escape e retorno de foco após diálogos.
- [ ] Respeitar redução de movimento.
- [ ] Verificar larguras de 360, 768 e 1440 pixels e zoom de 200%, sem perda de ações.
- [ ] Evitar que a barra de ações oculte campos ou mensagens em telas pequenas.

## 8. API e contrato de edição

Prefixo proposto: `/api/configuracoes`. Toda operação exige sessão válida, inclusive leitura do catálogo e do histórico.

| Método/rota | Responsabilidade | Resultado esperado |
| --- | --- | --- |
| `GET /catalogo` | Entregar metadados, agrupamentos e cobertura | Sem valores secretos |
| `GET /valores` | Entregar snapshot dos valores não sensíveis e estados dos segredos | Revisão opaca e origens |
| `POST /validar` | Validar alterações contra a revisão e catálogo | Erros por campo e impactos |
| `POST /revisar` | Calcular diff sanitizado | Inclusões, mudanças, remoções e necessidade de aplicação |
| `PATCH /valores` | Gravar patch validado | Nova revisão e impactos pendentes |
| `POST /aplicar` | Executar aplicação de uma revisão salva | Identificador de operação |
| `GET /operacoes/{id}` | Consultar andamento e saúde final | Estado e mensagens sanitizadas |
| `GET /historico` | Listar revisões privadas sem conteúdo secreto | Data, autor, chaves e resultado |
| `POST /restaurar` | Restaurar versão após revisão e validação | Nova revisão, sem sobrescrever histórico |
| `POST /exportar` | Gerar configuração sanitizada | Segredos omitidos e campos pendentes identificados |
| `POST /importar/validar` | Validar e revisar arquivo importado sem persistência | Mesmo fluxo de revisão e gravação |

- [ ] Definir contrato OpenAPI e usar tipos coerentes no cliente.
- [ ] Enviar mudanças por identificadores do catálogo, nunca caminhos arbitrários de arquivos.
- [ ] Incluir revisão esperada, chave de idempotência e operações explícitas de inclusão/alteração/remoção.
- [ ] Retornar `409` em conflito; oferecer recarga e reconciliação sem perder rascunho local.
- [ ] Definir `401`, `403`, `422` e falhas operacionais com mensagens úteis e sanitizadas.
- [ ] Não retornar corpo de requisição nem valor confidencial em mensagens de erro.
- [ ] Limitar tamanho de payload e validar tipos, valores, relações e disponibilidade de recursos.
- [ ] Implementar exportação sanitizada e importação como revisão, sem aplicação automática.

### Operações de segredo

Representar segredo na leitura como `{ definido: true, sensivel: true }`. No patch, permitir `manter`, `substituir` e `remover`; valor vazio não significa remoção implícita. A interface não deve usar `********` como valor real nem reenviá-lo ao servidor.

## 9. Persistência, segurança e recuperação

### 9.1 Gravação confiável

- [ ] Revalidar no servidor imediatamente antes de salvar.
- [ ] Verificar revisão e adquirir bloqueio de escrita.
- [ ] Preservar comentários, ordem, aspas, tags YAML e chaves desconhecidas não alteradas.
- [ ] Usar parser YAML que preserve estrutura; não desserializar objetos Python arbitrários.
- [ ] Rejeitar edição estrutural que o adaptador não consiga representar sem perda e explicar o campo afetado.
- [ ] Preparar candidatos em arquivos temporários no mesmo filesystem do destino.
- [ ] Validar todos os candidatos antes de substituir qualquer destino.
- [ ] Criar backup privado e journal com estado da operação.
- [ ] Substituir arquivos com operação atômica por arquivo e permissões preservadas.
- [ ] Para alterações em vários arquivos, implementar recuperação de transação; renomear um arquivo não garante atomicidade do conjunto.
- [ ] Simular interrupção entre substituições e recuperar uma revisão consistente na próxima inicialização.
- [ ] Não modificar arquivos gerados em `site/` como fonte de configuração.

### 9.2 Proteção de acesso e segredos

- [ ] Criar autenticação local com credencial inicial de uso único e sessão expirada por inatividade.
- [ ] Proteger mutações com validação de origem e defesa contra CSRF; cookies devem ser HttpOnly e SameSite.
- [ ] Validar cabeçalho Host para reduzir risco de acesso indevido ao serviço local.
- [ ] Bloquear exposição fora de loopback até configurar autenticação e transporte apropriados.
- [ ] Restringir arquivos ao projeto cadastrado; impedir path traversal e escapes por links simbólicos.
- [ ] Proibir edição arbitrária de arquivos do sistema e execução de comandos recebidos do navegador.
- [ ] Não incluir segredos em bundles, prefixos públicos de ambiente, localStorage, URLs, logs ou telemetria.
- [ ] Impedir cache de respostas confidenciais e limitar duração das sessões.
- [ ] Tratar backups como secretos, com permissões restritas e retenção definida.
- [ ] Registrar autor, horário, revisão, nomes das chaves e resultado, sem registrar seus valores sensíveis.
- [ ] Atualizar `.gitignore` para arquivos reais `.env`, estado privado e backups; permitir `.env.example` sanitizado.
- [ ] Verificar que builds do painel e do site não copiam arquivos privados.

### 9.3 Salvar e aplicar são etapas distintas

- [ ] Após salvar, informar quais mudanças já têm efeito e quais dependem de aplicação.
- [ ] Rebuildar documentação usando a revisão salva e capturar resultado sanitizado.
- [ ] Gerenciar somente processos iniciados pelo supervisor do projeto e validar identidade antes de encerrá-los.
- [ ] Usar encerramento gracioso, timeout e verificação HTTP no caminho real da documentação.
- [ ] Para reinício da própria API, delegar ao supervisor externo; a API não deve prometer responder após encerrar a si mesma.
- [ ] Persistir estado da operação para que o navegador se reconecte e consulte o resultado.
- [ ] Ao mudar porta/endereço, exibir destino de reconexão e instruções de recuperação.
- [ ] Em falha de aplicação, conservar backup e mostrar “salvo, aplicação falhou”.
- [ ] Disponibilizar restauração e nova aplicação da revisão anterior.
- [ ] Iniciar o serviço de forma independente da sessão interativa que o lançou e comprovar continuidade após o encerramento dessa sessão.

## 10. Fases de execução e critérios de saída

### Fase 1 — Inventário completo e contratos

Dependência: diagnóstico deste documento.

- [ ] Reinspecionar repositório, instruções locais e configurações sem expor valores.
- [ ] Catalogar todas as chaves e constantes configuráveis.
- [ ] Definir consumidores, precedência, sensibilidade e impacto de aplicação.
- [ ] Especificar catálogo, tipos da API e estrutura dos patches.
- [ ] Registrar lista inicial de cobertura e exceções justificadas.

Saída: inventário revisável sem parâmetros omitidos e contratos suficientes para frontend e backend.

### Fase 2 — Fundação técnica e Watermelon

Dependência: fase 1.

- [ ] Criar frontend, API e comandos locais reproduzíveis.
- [ ] Fixar versões compatíveis e documentar requisitos.
- [ ] Selecionar e integrar componentes Watermelon reais.
- [ ] Implementar sessão local, rota do painel e proxy de desenvolvimento.
- [ ] Criar shell responsivo e os três modos de tema.

Saída: tela acessível no navegador, usando Watermelon, com autenticação e tema persistente.

### Fase 3 — Leitura e formulário completo

Dependência: fase 2 e catálogo.

- [ ] Implementar adaptadores de leitura e inventário incremental.
- [ ] Entregar valores sanitizados e estado dos consumidores.
- [ ] Gerar campos, objetos, listas e árvores a partir do catálogo.
- [ ] Implementar busca, filtros, estados e cobertura.
- [ ] Implementar tratamento de chaves desconhecidas e editor avançado.
- [ ] Aplicar validação local e exibir validação remota.

Saída: todos os parâmetros inventariados aparecem e possuem tratamento explícito.

### Fase 4 — Persistência e revisão

Dependência: fase 3.

- [ ] Implementar revisão das mudanças, segredos e confirmação de remoções.
- [ ] Implementar patch, bloqueio, revisão, idempotência e conflitos.
- [ ] Preservar formatos e comentários nos adaptadores.
- [ ] Implementar backups, journal e recuperação de falha parcial.
- [ ] Implementar descarte, restauração, importação e exportação sanitizada.

Saída: alterações válidas persistem sem perda, e falhas não deixam configuração inconsistente.

### Fase 5 — Aplicação e supervisão

Dependência: fase 4.

- [ ] Implementar carregadores dos novos parâmetros `.env` e comprovar consumidores.
- [ ] Implementar rebuild, reinício gerenciado e reconexão do painel.
- [ ] Implementar consultas de operação, estado efetivo e saúde.
- [ ] Exercitar mudança de porta e restauração após aplicação malsucedida.
- [ ] Validar continuidade dos servidores após o término da sessão de lançamento.

Saída: valores salvos têm efeito comprovado ou pendência de aplicação claramente indicada.

### Fase 6 — Validação integrada e acabamento

Dependência: fases 3 a 5.

- [ ] Executar testes da matriz abaixo com arquivos temporários e dados sintéticos.
- [ ] Validar navegação por teclado, foco, responsividade e ambos os temas.
- [ ] Comparar visualmente componentes incorporados com suas referências Watermelon.
- [ ] Validar build do frontend e build estrito da documentação.
- [ ] Executar auditoria de cobertura sem omissões e verificação de ausência de segredos nos artefatos.
- [ ] Corrigir falhas e registrar limitações reais.

Saída: critérios de aceite comprovados com evidências funcionais e visuais.

### Fase 7 — Documentação e entrega

Dependência: fase 6.

- [ ] Documentar instalação, inicialização, encerramento, login inicial e recuperação.
- [ ] Documentar cadastro de novas variáveis, precedência e efeitos de aplicação.
- [ ] Documentar backup, restauração e recuperação se o painel não iniciar.
- [ ] Atualizar README e `.env.example` conforme o que foi efetivamente implementado.
- [ ] Registrar componentes Watermelon utilizados e adaptações.
- [ ] Preencher checklist final e resumo dos testes.

Saída: execução reproduzível por outra pessoa, sem depender da conversa original.

## 11. Matriz de testes

| Área | Casos obrigatórios | Resultado esperado |
| --- | --- | --- |
| Cobertura | Nova chave `.env`, novo campo YAML, lista com vários itens, estrutura aninhada | Todos aparecem, sem truncamento nem omissão |
| Consumidores | Variável consumida, sem consumidor e sobrescrita pelo processo | Efeito e precedência exibidos corretamente |
| `.env` | Vazio, ausente, espaços, aspas, `#`, `=`, Unicode, multiline e duplicação | Leitura e escrita coerentes com o parser definido |
| YAML | Comentários, ordem, tags especiais, plugins e extensões | Sem perda estrutural ou execução durante parsing |
| Validação | URL inválida, número fora da faixa, dependência ausente, arquivo inexistente | Erro associado ao campo; nenhuma gravação parcial |
| Segredos | Manter, substituir, remover e falhar na validação | Nenhum valor antigo retornado ou registrado |
| Concorrência | Duas abas e alteração manual em disco | Conflito detectado antes de sobrescrever |
| Durabilidade | Sem permissão, disco cheio simulado e interrupção entre arquivos | Recuperação consistente e backup utilizável |
| Idempotência | Repetição da requisição após timeout | Uma gravação/operação efetiva |
| Aplicação | Rebuild válido/inválido, porta ocupada, reinício e reconexão | Estado verdadeiro e possibilidade de restauração |
| Sessão | Expiração, requisição de outra origem e Host inválido | Acesso rejeitado sem vazamento |
| Caminhos | `..`, absoluto externo e symlink externo | Acesso fora do projeto rejeitado |
| Temas | Claro, escuro, sistema, reload, overlays e troca do sistema | Preferência consistente sem cores ilegíveis |
| Acessibilidade | Teclado, foco, leitor de tela, zoom e celular | Formulário e ações utilizáveis |
| Importação/exportação | Configuração válida, desconhecida, inválida e com segredos | Revisão explícita; exportação sem segredos |
| Regressão | Navegação e build da oficina após edição | Site existente continua funcional |

Testes de escrita e falhas devem utilizar diretórios temporários e arquivos sintéticos. Testes visuais deverão registrar capturas nos temas claro e escuro; não declarar fidelidade apenas porque o build passou.

### Comandos de validação previstos

- [ ] Definir scripts de lint, verificação TypeScript, testes e build no frontend.
- [ ] Definir suíte de testes da API e adaptadores.
- [ ] Executar verificador de cobertura do catálogo.
- [ ] Executar `.venv/bin/mkdocs build --strict` após mudanças na documentação/configuração.
- [ ] Executar teste de navegador: entrar → editar → validar → revisar → salvar → aplicar → recarregar → confirmar persistência.
- [ ] Repetir o fluxo com falha controlada e restauração.

Os comandos de frontend e API serão registrados após a criação de seus manifests; ainda não existem no repositório.

## 12. Riscos concretos e respostas planejadas

| Risco | Resposta |
| --- | --- |
| O projeto pretendido ser outro | Revalidar alvo antes de implementar; o inventário atual é específico desta oficina |
| Browser sem capacidade de gravar `.env` do servidor | Introduzir API local com acesso restrito ao projeto |
| Painel publicado junto do site estático | Separar builds e validar os artefatos públicos |
| Campo salvo sem consumidor real | Registrar consumidor e testar efeito de cada variável |
| YAML com tags especiais perder estrutura | Parser com preservação e testes de round-trip |
| CSS fixo sobrescrever valores do painel | Extrair tokens e testar precedência dos estilos |
| Componentes Watermelon incompatíveis ou ausentes | Prova de integração e matriz de procedência antes da composição final |
| API cair durante seu próprio reinício | Supervisor externo, estado persistido e reconexão |
| Edições concorrentes apagarem mudanças | Revisão opaca e conflito antes da gravação |
| Vazamento em backup, log ou bundle | Armazenamento privado, sanitização e inspeção de artefatos |

## 13. Checklist final de aceite

- [ ] A nova tela abre diretamente no navegador em rota documentada.
- [ ] Componentes Watermelon reais estão incorporados com fontes registradas.
- [ ] Claro, escuro e sistema funcionam em todos os componentes.
- [ ] Todos os parâmetros inventariados estão acessíveis, com exceções de edição explícitas.
- [ ] Novas chaves são descobertas e não desaparecem da interface.
- [ ] Valores atuais e origens correspondem às fontes reais.
- [ ] Alterações válidas persistem após recarregar e reiniciar.
- [ ] Consumidores e efeitos de aplicação estão comprovados.
- [ ] Segredos podem ser substituídos/removidos sem exposição dos valores existentes.
- [ ] Validação, conflitos e erros operacionais mantêm dados consistentes.
- [ ] Backups e restauração foram exercitados.
- [ ] Estado salvo e estado aplicado são distinguíveis.
- [ ] Aplicação e reconexão funcionam sem encerrar processos de outros projetos.
- [ ] O site da oficina continua funcionando e o build público não contém segredos.
- [ ] Evidências de navegador, temas, testes e cobertura estão registradas.
- [ ] Instruções de execução e recuperação permitem uso sem esta conversa.

## 14. Registro de execução

| Fase | Estado | Responsável | Data | Evidência/arquivos | Pendências |
| --- | --- | --- | --- | --- | --- |
| Diagnóstico e plano | Concluído | Codex | 26/09/2026 | Este documento e arquivos citados | Revalidar na implementação |
| 1. Inventário e contratos | Pendente | A definir | — | — | — |
| 2. Fundação e Watermelon | Pendente | A definir | — | — | — |
| 3. Leitura e formulário | Pendente | A definir | — | — | — |
| 4. Persistência | Pendente | A definir | — | — | — |
| 5. Aplicação | Pendente | A definir | — | — | — |
| 6. Validação | Pendente | A definir | — | — | — |
| 7. Documentação e entrega | Pendente | A definir | — | — | — |

## 15. Referências e limites da pesquisa

Consultadas em 26/09/2026:

- [Watermelon UI — componentes](https://ui.watermelon.sh/components).
- [Watermelon UI — orientação para desenvolvedores](https://ui.watermelon.sh/llms.txt).
- [Watermelon UI — documentação da API pública](https://ui.watermelon.sh/api/docs).
- [Watermelon UI — catálogo de blocos](https://ui.watermelon.sh/api/v1/catalog/entries?kind=blocks).
- [Watermelon UI — busca de blocos por settings](https://ui.watermelon.sh/api/v1/catalog/entries?kind=blocks&query=settings&limit=5).

A consulta confirma um caminho de descoberta e incorporação de componentes. Compatibilidade técnica, licença de cada item e adequação visual da composição final permanecem como atividades da implementação. Nenhuma instalação de biblioteca ou construção da tela foi realizada durante a preparação deste plano.
