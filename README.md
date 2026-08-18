# Gerenciador de Estoque - Infraestrutura

Repositorio de infraestrutura. Fonte da verdade das **variaveis de ambiente** (parametrizacao)
e do deploy.

## Criterio de parametrizacao

1. **Nenhum valor de ambiente fica hardcoded no codigo.** A API le tudo por parametrizacao:
   `${VARIAVEL}` no YAML (com default apenas no perfil local).
2. **Os valores por ambiente ficam aqui, em `env/`:**
   - `env/dev.env` - desenvolvimento local.
   - `env/prod.env` - producao (Render). Valores sensiveis sao injetados pelo Render
     automaticamente (nao commitar credenciais reais).
3. **A API nunca conhece o ambiente** - ela so conhece variaveis. Quem decide o valor e o
   ambiente (este repositorio + Render).
4. **Perfis Spring:** `local` (default, com defaults para dev) e `prod` (sem defaults -
   falha ao iniciar se faltar variavel).

## Variaveis padronizadas

| Variavel | Descricao | Obrigatoria em prod |
|---|---|---|
| `DB_HOST` | Host do PostgreSQL | sim |
| `DB_PORT` | Porta do PostgreSQL | sim |
| `DB_NAME` | Nome do banco | sim |
| `SPRING_DATASOURCE_USERNAME` | Usuario do banco | sim |
| `SPRING_DATASOURCE_PASSWORD` | Senha do banco | sim |
| `DB_SSL_PARAMS` | Parametros extras da JDBC URL (ex: `?sslmode=require` no Neon) | nao (vazio) |
| `APP_CORS_ORIGINS` | Origens permitidas no CORS | sim |
| `SPRING_PROFILES_ACTIVE` | Perfil ativo (`local`/`prod`) | sim |
| `PORT` | Porta HTTP da API | nao (default 8080) |

## Ambientes publicados

| Componente | Onde | URL |
|---|---|---|
| API (Spring Boot) | Render (plano free) | `https://gerenciador-estoque-api-q7fs.onrender.com` |
| App (React/Vite) | Vercel (deployment protegido) | `https://gerenciador-estoque-7dbdmm609-ramons02s-projects.vercel.app` |
| Banco (PostgreSQL) | Neon (projeto `round-water-07779373`) | host `ep-still-bar-aycqhg1n-pooler.c-5.us-east-2.aws.neon.tech` |

O Vercel usa `vercel.json` para reescrever `/api/*` direto para o Render (sem CORS na
navegacao). O deployment Vercel esta **protegido** (exige login) - nao e aberto ao publico.

## Banco de dados (Neon)

O banco de producao e PostgreSQL no **Neon** (SSL obrigatorio, `DB_SSL_PARAMS=sslmode=require`):

- **Plano Free:** 0,5 GB de armazenamento.
- **Plano Launch** (pago): 10 GB de armazenamento.

Para um sistema de gas/agua (vendas, estoque, clientes), 0,5 GB comporta anos de uso.
Plano e uso atual: console.neon.tech -> projeto `round-water-07779373` -> aba *Storage*.

## Deploy no Render e sono do plano free

O blueprint (`render.yaml`) fica na raiz do repositorio da API
(`gerenciador_estoque_api/render.yaml`) e define o servico da API (Java,
`mvn clean package -DskipTests`).

**Limitacao do plano free:** o servico **dorme apos ~15 min sem uso** e o primeiro acesso
leva 30-120s (cold start) para acordar. Para manter a API acordada 24/7:

- O workflow `.github/workflows/keep-alive-render.yml` (este repositorio) faz um ping na
  API a cada 6 minutos via GitHub Actions (gratis).
- Alternativa: monitor externo tipo UptimeRobot (intervalo de 5 min) ou plano pago do
  Render (~US$7/mes, nunca dorme).

Para uma apresentacao ao cliente, prefira o **jar local** (zero latencia) ou garanta que a
API esteja acordada antes de mostrar o sistema.

## Uso local (dev)

```bash
# com as variaveis de env/dev.env exportadas (ou defaults do application.yml)
cd ../gerenciador_estoque_api
mvn spring-boot:run
```