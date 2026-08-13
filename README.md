# Meno

Uma pequena stack self-hosted para:

- receitas
- planejamento de refeições
- despensa
- estoque
- compras
- alimentação

## Stack

```text
Tandoor
Grocy
PostgreSQL
Docker Compose
```

## Executar

```bash
cp .env.example .env
docker compose up -d
```

Verificar o estado dos containers:

```bash
docker compose ps
```

O `postgres` tem healthcheck e deve ficar `healthy`. O Tandoor só inicia depois
disso e, no primeiro boot, leva alguns minutos rodando as migrações do banco.

Acompanhar os logs:

```bash
docker compose logs -f
```

Logs de um serviço específico:

```bash
docker compose logs -f tandoor
```

Parar tudo:

```bash
docker compose down
```

## Acesso

As portas do host são definidas no `.env` (`TANDOOR_HTTP_PORT` e
`GROCY_HTTP_PORT`). Com os valores padrão:

| Serviço | Porta do host | Porta interna |
| ------- | ------------- | ------------- |
| Tandoor | 8080          | 80            |
| Grocy   | 9283          | 80            |

- Tandoor: <http://localhost:8080> — crie a conta de administrador na primeira visita.
- Grocy: <http://localhost:9283> — login padrão `admin` / `admin` (troque em seguida).

O PostgreSQL **não** é exposto no host; ele fica acessível apenas pela rede
interna do Compose, sob o nome `postgres`.

No GitHub Codespaces, use a aba **Ports** para abrir as URLs encaminhadas.

## Dados

```text
data/postgres/       banco de dados
apps/tandoor/media/  imagens e arquivos das receitas
apps/grocy/data/     banco SQLite e configuração do Grocy
backups/             espaço reservado para backups
```

Os arquivos estáticos do Tandoor ficam em um volume nomeado gerenciado pelo
Docker (`meno_tandoor_static`), como recomenda a documentação oficial.
