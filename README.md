# Meno

Uma pequena stack self-hosted para:

- receitas
- planejamento de refeições
- despensa
- estoque
- compras
- alimentação
- treino e acompanhamento físico

## Stack

```text
Tandoor
Grocy
wger
PostgreSQL
Redis
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

O `postgres` tem healthcheck e deve ficar `healthy`. Tandoor e wger só iniciam
depois disso e, no primeiro boot, levam alguns minutos rodando as migrações
do banco.

Acompanhar os logs:

```bash
docker compose logs -f
```

Logs de um serviço específico:

```bash
docker compose logs -f tandoor
docker compose logs -f wger
```

Parar tudo:

```bash
docker compose down
```

## Acesso

As portas do host são definidas no `.env` (`TANDOOR_HTTP_PORT`,
`GROCY_HTTP_PORT` e `WGER_HTTP_PORT`). Com os valores padrão:

| Serviço | Porta do host | Porta interna |
| ------- | -------------- | ------------- |
| Tandoor | 8080           | 80            |
| Grocy   | 9283           | 80            |
| wger    | 8000           | 80 (via nginx) |

- Tandoor: <http://localhost:8080> — crie a conta de administrador na primeira visita.
- Grocy: <http://localhost:9283> — login padrão `admin` / `admin` (troque em seguida).
- wger: <http://localhost:8000> — login padrão `admin` / `adminadmin` (troque em seguida).

O PostgreSQL e o Redis **não** são expostos no host; ficam acessíveis apenas
pela rede interna do Compose, sob os nomes `postgres` e `wger_cache`.

No GitHub Codespaces, use a aba **Ports** para abrir as URLs encaminhadas.
Em VMs sem domínio configurado (ex.: Azure), substitua `localhost` pelo IP
público da máquina e libere as portas correspondentes no firewall/NSG.

## Pós-instalação do wger

Na primeira subida, rode estes três comandos (nessa ordem) pra ter o wger
totalmente funcional, com estilo e exercícios carregados:

```bash
# gera o CSS/JS servidos pelo nginx do wger — sem isso a página sobe sem estilo
docker compose exec wger python3 manage.py collectstatic --noinput

# baixa a base de exercícios (nomes, categorias, músculos) do wger.de
docker compose exec wger python3 manage.py sync-exercises

# baixa as imagens dos exercícios — pode demorar alguns minutos
docker compose exec wger python3 manage.py download-exercise-images
```

O `collectstatic` só precisa ser rodado de novo se você atualizar a imagem
do wger (`docker compose pull wger`) — os arquivos ficam persistidos em
`apps/wger/static`, então sobrevivem a restarts normais.

## Dados

```text
data/postgres/        banco de dados (Tandoor)
data/postgres-init/    script que cria o banco do wger no mesmo Postgres
apps/tandoor/media/    imagens e arquivos das receitas
apps/grocy/data/       banco SQLite e configuração do Grocy
apps/wger/static/      arquivos estáticos do wger (CSS/JS), servidos pelo nginx
apps/wger/media/       imagens enviadas no wger (galeria, exercícios)
backups/               espaço reservado para backups
```

Os arquivos estáticos do Tandoor ficam em um volume nomeado gerenciado pelo
Docker (`meno_tandoor_static`), como recomenda a documentação oficial. Os do
wger, diferente do Tandoor, ficam em bind mount (`apps/wger/static`) porque
precisam ser lidos por um container `nginx` separado, que serve o app.