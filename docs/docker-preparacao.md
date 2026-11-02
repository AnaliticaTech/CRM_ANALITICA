# Preparação Docker — revisão estática, sem execução

Dockerfile, compose.yaml e next.config.ts foram preservados. output: standalone
e os headers/configurações de segurança existentes continuam intactos. Nenhuma
instalação, teste, build, migração ou operação de container foi executada.

## Contexto e ambiente

.dockerignore exclui ambientes operacionais, credenciais/chaves, caches, logs,
bancos SQLite e diretórios operacionais na raiz. .env.local.example é a única
exceção de ambiente e contém somente campos vazios. Exclusões não garantem
ausência de segredos embutidos no código e não removem dados já em imagens.
As exclusões de diretórios operacionais estão limitadas à raiz para preservar
fontes como src/data. Se um futuro build precisar de algum caminho excluído,
revisar arquivos específicos sem liberar dados operacionais.

São preservados package.json, package-lock.json, src/, public/, next.config.ts,
tsconfig.json, postcss.config.mjs e demais fontes/configurações necessárias.
Não excluir genericamente *.json, *.sql ou docs/: podem ser fontes legítimas.
A avaliação do contexto é estática; o matcher real do Docker e o build não
foram executados, portanto isso não certifica um contexto construível.

O exemplo lista as dez variáveis de aplicação identificadas e CRM_HOST_PORT.
Campos vazios não são configuração funcional nem credenciais fictícias.
As três NEXT_PUBLIC_* são públicas e exigidas nos argumentos de build do Compose.
.env.local em env_file só entrega variáveis ao container; não alimenta por si
só a interpolação de build.args. Em etapa futura aprovada, fornecer explicitamente
a fonte de interpolação privada (por exemplo --env-file .env.local); não passar
segredos de servidor como ARG nem publicar arquivos de ambiente.
Configuração operacional não foi criada ou copiada.

SUPABASE_SERVICE_ROLE_KEY, ENCRYPTION_KEY, META_APP_SECRET e AUTOMATION_CRON_SECRET
são apenas de servidor. ANON_KEY é pública, mas a proteção depende de RLS e
políticas Supabase revisadas. META_APP_ID, ALLOWED_INVITE_HOSTS e
WHATSAPP_TEMPLATES_DRY_RUN dependem das funcionalidades habilitadas.
Rotação de ENCRYPTION_KEY exige preservar a capacidade de decifrar dados antigos.

## Healthcheck preservado

O healthcheck faz GET http://127.0.0.1:3000/next.svg com Node e considera
r.ok. O arquivo existe em public/. Isso mede apenas disponibilidade de um
recurso estático; redirecionamentos são seguidos por padrão. Não comprova
SSR, login, RLS, conexão Supabase, webhooks, cron ou criptografia. Timeout de
5s é externo ao fetch, controlado pelo Docker; o código não tem timeout próprio.
O Compose isolado mantém interval 30s, timeout 5s, start_period 40s, retries 3.
Não foi executado healthcheck. O estado unhealthy, isoladamente, não garante
reinício por restart: unless-stopped.

Recomendação futura: endpoint de liveness sem dados sensíveis e sem dependência
externa, mais verificação de readiness separada com timeout e erros sanitizados.
Não criar endpoint fictício nem fazer chamadas reais nesta preparação.

## Próxima aprovação

Antes de build/teste: revisão do diff, autorização específica para instalação
(inclusive scripts npm ci), configuração sintética isolada, validação do contexto
e plano de validação funcional. Node 22 é compatível nominalmente com engines
Next >=20.9, mas dependências e chamadas externas não foram validadas em execução.
Commit e PR futuros devem incluir somente o manifesto de arquivos revisados;
nenhum ambiente operacional, segredo, cache ou dado deve ser incluído.
