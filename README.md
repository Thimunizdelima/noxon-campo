# Noxon Campo — app e API

App de celular para o **Planejamento semanal** e o **Comunicado semanal de ações (Realizado)** da equipe de campo Noxon, com backend no Supabase.

- **App:** `index.html` (publicado pelo GitHub Pages)
- **API:** `https://zdydkrjkoqhodzjafxau.supabase.co` (REST gerada pelo Supabase / PostgREST)
- **Chave pública (publishable):** `sb_publishable_XxYsr-KMTiB7vtzeWjLuLg_rBmhre9N`

A chave pública pode ficar no código: ela sozinha não dá acesso a nenhum dado. Toda leitura e gravação exige login, e as regras do banco (RLS) definem o que cada pessoa vê.

## Perfis de acesso

| Papel | O que pode fazer |
|---|---|
| `pendente` | Conta criada, sem acesso até o gestor liberar |
| `vendedor` | Lê clientes, produtos e objetivos; cria e edita só os próprios planejamentos e relatórios |
| `gestor` | Vê os lançamentos de todos, escreve os comentários do gestor, libera acessos e edita os catálogos |

O primeiro gestor é definido no painel do Supabase: **Table Editor → perfis →** coluna `papel` = `gestor`.

## Endpoints

Todos ficam em `/rest/v1/<tabela>` e aceitam `GET`, `POST`, `PATCH` e `DELETE` conforme o papel.

| Endpoint | Conteúdo |
|---|---|
| `clientes` | Revendas por rede (`rede`, `nome`, `cidade`, `uf`, `info`) |
| `produtos` | Catálogo Noxon (`categoria`, `nome`, `descricao`) |
| `objetivos` | Base de objetivos/atividades (`grupo`, `nome`, `descricao`) |
| `perfis` | Usuários (`nome`, `email`, `regional`, `papel`) |
| `planejamentos` | Cabeçalho do planejamento (`periodo`, `regional`) |
| `visitas_planejadas` | Visitas por semana (`semana` 1–5, `data`, `cliente`, `cidade`, `objetivos[]`, `detalhe`, `observacao`) |
| `relatorios` | Cabeçalho do realizado (`mes`, `regiao`, cafés da manhã, trabalhos de campo, `comentario_gestor`) |
| `acoes_realizadas` | Ações (`data`, `tipo_visita` revenda/campo, `revenda`, `propriedade`, `cidade`, `uf`, `atividades[]`, `produtos[]`, `valor`, `venda_tipo` C/B/PT, `comentario_gestor`) |
| `v_acoes` | Ações com nome do vendedor, mês e região (para relatórios) |
| `v_visitas` | Visitas com nome do vendedor, período e regional |

## Exemplos

**1. Entrar e obter o token**

```bash
curl -X POST "https://zdydkrjkoqhodzjafxau.supabase.co/auth/v1/token?grant_type=password" \
  -H "apikey: sb_publishable_XxYsr-KMTiB7vtzeWjLuLg_rBmhre9N" \
  -H "Content-Type: application/json" \
  -d '{"email":"gestor@empresa.com","password":"SUA_SENHA"}'
```

Guarde o `access_token` da resposta (vale 1 hora; use o `refresh_token` para renovar).

**2. Ações de outubro com valor, por vendedor**

```bash
curl "https://zdydkrjkoqhodzjafxau.supabase.co/rest/v1/v_acoes?select=vendedor_nome,data,revenda,cidade,produtos,valor,venda_tipo&data=gte.2026-10-01&data=lt.2026-11-01&order=data" \
  -H "apikey: sb_publishable_XxYsr-KMTiB7vtzeWjLuLg_rBmhre9N" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

**3. Visitas planejadas que incluem um objetivo**

```bash
curl "https://zdydkrjkoqhodzjafxau.supabase.co/rest/v1/v_visitas?objetivos=cs.{\"Visita à loja\"}" \
  -H "apikey: sb_publishable_XxYsr-KMTiB7vtzeWjLuLg_rBmhre9N" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

**4. Gestor comenta uma ação**

```bash
curl -X PATCH "https://zdydkrjkoqhodzjafxau.supabase.co/rest/v1/acoes_realizadas?id=eq.ID_DA_ACAO" \
  -H "apikey: sb_publishable_XxYsr-KMTiB7vtzeWjLuLg_rBmhre9N" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"comentario_gestor":"Priorizar Zol na próxima visita"}'
```

Filtros, paginação e seleção de colunas seguem o padrão PostgREST: https://supabase.com/docs/guides/api

## Regras garantidas pelo banco

- Vendedor só enxerga e altera os próprios lançamentos; não consegue lançar em nome de outro.
- Só o gestor altera `comentario_gestor` e o `papel` dos usuários.
- Cadastros novos entram como `pendente`.
- Clientes, produtos e objetivos só são alterados pelo gestor (pelo painel ou pela API).

## Configuração recomendada no painel do Supabase

- **Authentication → URL Configuration → Site URL:** coloque o endereço do GitHub Pages, para que o link de confirmação de e-mail volte para o app.
- Se preferir que só você crie contas, desative **Allow new users to sign up** em Authentication → Sign In / Providers e cadastre a equipe em Authentication → Users.
