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
| `APP_CORS_ORIGINS` | Origens permitidas no CORS | sim |
| `SPRING_PROFILES_ACTIVE` | Perfil ativo (`local`/`prod`) | sim |
| `PORT` | Porta HTTP da API | nao (default 8080) |

## Deploy no Render

O blueprint (`render.yaml`) fica na raiz do repositorio da API
(`gerenciador_estoque_api/render.yaml`) e define:

- **Banco:** `gerenciador-estoque-db` (PostgreSQL 16, plano free).
- **API:** `gerenciador-estoque-api` (Java, `mvn clean package -DskipTests`).

Ao conectar o repositorio `ramons02/gerenciador_estoque_api` como **Blueprint** no Render,
ele cria o banco e a API, injetando as credenciais via `fromDatabase` (as variaveis
`DB_*` do `env/prod.env` sao preenchidas automaticamente).

## Uso local (dev)

```bash
# com as variaveis de env/dev.env exportadas (ou defaults do application.yml)
cd ../gerenciador_estoque_api
mvn spring-boot:run
```