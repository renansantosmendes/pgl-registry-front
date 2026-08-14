# PGL Registry

Página estática (sem build) onde alunos cadastram a senha da sua matrícula
para usar o [pgl_proxy](https://github.com/) (`api_key="matricula:senha"`
no `ChatOpenAI`).

Chama `POST /v1/register` no proxy diretamente do navegador via `fetch()`.
Não tem backend próprio — todo o cadastro é validado e gravado no Neon
pelo proxy.

## Antes de fazer deploy

Edite a constante `API_BASE_URL` no topo do `<script>` em
[`index.html`](index.html) para apontar para o proxy correto:

```js
const API_BASE_URL = "https://pgl-proxy.vercel.app";
```

## Deploy (Vercel)

Site estático puro — nenhuma variável de ambiente ou build step é
necessária. Basta importar o repositório na Vercel; `vercel.json` só
habilita URLs limpas.

## Configuração necessária no pgl_proxy

O proxy só aceita chamadas de navegador (CORS) de origens explicitamente
liberadas. Depois do deploy, adicione a URL final desta página (ex.:
`https://pgl-registry-front.vercel.app`) na env var
`ALLOWED_FRONTEND_ORIGINS` do projeto `pgl_proxy` na Vercel, e refaça o
deploy do proxy. Sem isso, o navegador bloqueia a chamada por CORS mesmo
que a URL do proxy esteja certa.

## Sobre o "reset de senha"

Esta página só faz o **primeiro cadastro** de senha por matrícula
(`POST /v1/register` retorna 409 se a matrícula já tem senha). Não existe
reset self-service: como a matrícula sozinha não é segredo, permitir reset
só com a matrícula deixaria qualquer pessoa sequestrar a conta de outro
aluno. Reset de senha esquecida é feito pelo professor via
`python -m scripts.manage_students set-password <matricula>` no repositório
`pgl_proxy`.
