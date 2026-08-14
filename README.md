# PGL Registry

Página estática (sem build) onde alunos cadastram a senha da sua matrícula
para depois obter um token de acesso ao [pgl_proxy](https://github.com/)
via o pacote [`pgl_auth`](https://pypi.org/project/pgl-auth/)
(`ChatOpenAI(api_key=token)`).

Chama `POST /api/register` no [`pgl_auth_server`](https://github.com/)
diretamente do navegador via `fetch()`. Não tem backend próprio — todo o
cadastro é validado (matrícula precisa estar ativa em `pgl_proxy.students`)
e a senha é gravada (hash bcrypt, em `pgl_auth.students`) pelo
`pgl_auth_server`. Esta página nunca fala diretamente com `pgl_proxy`.

## Antes de fazer deploy

Edite a constante `API_BASE_URL` no topo do `<script>` em
[`index.html`](index.html) para apontar para o `pgl_auth_server` correto:

```js
const API_BASE_URL = "https://pgl-auth-server.vercel.app";
```

## Deploy (Vercel)

Site estático puro — nenhuma variável de ambiente ou build step é
necessária. Basta importar o repositório na Vercel; `vercel.json` só
habilita URLs limpas.

## Configuração necessária no pgl_auth_server

O `pgl_auth_server` precisa de CORS habilitado para a URL final desta
página (ex.: `https://pgl-registry-front.vercel.app`), senão o navegador
bloqueia a chamada mesmo com a URL da API certa. Veja o README do
`pgl_auth_server` para como configurar isso.

## Sobre o "reset de senha"

Esta página só faz o **primeiro cadastro** de senha por matrícula
(`POST /api/register` retorna `409` se a matrícula já tem senha em
`pgl_auth.students`). Não existe reset self-service: como a matrícula
sozinha não é segredo, permitir reset só com a matrícula deixaria qualquer
pessoa sequestrar a conta de outro aluno. Reset de senha esquecida precisa
de um fluxo administrativo no `pgl_auth_server` (ainda não implementado).

## Depois do cadastro

A página só cadastra a senha — ela não faz login nem gera token. Para usar
o proxy, o aluno instala `pgl-auth` (`pip install pgl-auth`) e faz:

```python
from pgl_auth import PGLAuthClient
token = PGLAuthClient().login(matricula="...", senha="...")

from langchain_openai import ChatOpenAI
llm = ChatOpenAI(base_url="https://pgl-proxy.vercel.app/v1", api_key=token)
```
