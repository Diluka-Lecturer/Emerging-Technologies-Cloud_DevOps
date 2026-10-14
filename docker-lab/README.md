# NSBM Work Order Container Lab

This lab demonstrates a small containerized application built
from six services:

- **FastAPI** provides the web page and work order API.
- **PostgreSQL** stores work orders.
- **RabbitMQ** receives a `wo.created:<id>` event for every submitted order.
- **Logstash** collects the API container logs.
- **Elasticsearch** stores and indexes those logs.
- **Kibana** provides a browser interface for searching the logs.

Docker Compose creates a private network for the services and starts them in
the required order.

## Prerequisites

Install the following before starting:

1. **Docker with Docker Compose**
   - Windows or macOS: install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
   - Linux: install [Docker Engine](https://docs.docker.com/engine/install/) and
     the Docker Compose plugin.
2. **A terminal**
   - Windows: PowerShell, Command Prompt, Git Bash, or WSL.
   - macOS or Linux: any standard terminal.
3. **A web browser** for the application, RabbitMQ, and Kibana.
4. **curl** for the optional command-line API example.

Confirm that Docker is running:

```console
docker --version
docker compose version
docker info
```

On Windows and macOS, start Docker Desktop first. On Linux, ensure your user can use Docker without a
permission error; follow Docker's official post-install instructions if needed.

> This is a classroom environment. It uses simple passwords and disables
> Elasticsearch security. Do not deploy this configuration to production or
> expose it to the internet.

## Project structure

```text
.
├── app/
│   ├── main.py                    # FastAPI application
│   ├── requirements.txt           # Python packages
│   ├── static/images/             # IFS logo and NSBM tab icon
│   └── templates/index.html       # Browser UI
├── logstash/pipeline/
│   └── logstash.conf              # Logstash input and output
├── Dockerfile                     # Builds the API image
├── docker-compose.yml             # Defines all six services
├── Makefile                       # Optional command shortcuts
└── README.md
```

## Start the lab

### 1. Open the project directory

Download or clone the project, open a terminal, and change to the folder that
contains `docker-compose.yml`:

```console
cd path/to/NSBM
```
Replace `path/to/NSBM` with the actual path on your computer.

### 2. Build and start everything

```console
docker compose up --build -d
```

- `up` creates and starts the containers.
- `--build` builds the API image before starting it.
- `-d` runs the containers in the background and returns the terminal prompt.

The first run downloads several large images and can take a few minutes.
Later runs are faster because Docker reuses downloaded images and build layers.

### 3. Check the containers

```console
docker compose ps
```

Wait until `api`, `db`, `mq`, `elasticsearch`, and `logstash` show as healthy.
Kibana may show as running and can take another minute before its page is
available.

To watch startup messages:

```console
docker compose logs -f
```

Press `Ctrl+C` to stop following the output. This does not stop the containers.

## Service addresses and ports

| Service | Address from the computer | Purpose |
| --- | --- | --- |
| Application | <http://localhost:9000> | Web UI and work order API |
| API health | <http://localhost:9000/healthz> | API health check |
| Kibana | <http://localhost:5601> | Search API logs |
| Elasticsearch | <http://localhost:9200> | Log storage API |
| RabbitMQ UI | <http://localhost:15672> | Queue management (`guest` / `guest`) |
| RabbitMQ AMQP | `localhost:5672` | Application messaging protocol |
| PostgreSQL | `localhost:5433` | Database (`ifs` / `ifs`, database `ifsdb`) |
| Logstash GELF | `localhost:12201/udp` | Receives API container logs |

PostgreSQL, RabbitMQ AMQP, and Logstash GELF are not websites. Opening their
ports in a browser will not display a page.

## Run the demonstration

### 1. Check the API

Open <http://localhost:9000/healthz>. The response should resemble:

```json
{"status":"ok","service":"ifs-workorder-api","env":"dev"}
```

### 2. Create a work order

Open <http://localhost:9000>, complete the **Submit a Work Order** form, and
select **Submit**. The new order appears in the **Work Orders** table.

Alternatively, on macOS, Linux, Git Bash, or WSL, use curl:

```console
curl -X POST http://localhost:9000/workorders \
  -H "Content-Type: application/json" \
  -d '{"id":1001,"title":"Replace pump seal","status":"NEW"}'
```

On Windows PowerShell, use:

```powershell
$body = @{ id = 1001; title = "Replace pump seal"; status = "NEW" } | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:9000/workorders" -Method Post -ContentType "application/json" -Body $body
```

Submitting an existing ID updates that work order rather than adding a
duplicate.

### 3. Verify database-backed data

The **Work Orders** table calls `GET /workorders`, which reads the rows from
PostgreSQL. Select **Refresh** if another person or terminal created the order.

The raw JSON response is also available at:

<http://localhost:9000/workorders>

### 4. Verify the RabbitMQ event

1. Open <http://localhost:15672>.
2. Sign in with username `guest` and password `guest`.
3. Open **Queues and Streams**.
4. Select the automatically created **workorders** queue.
5. Open **Get messages** and request a message.

The payload resembles `wo.created:1001`.

The API automatically creates the durable `workorders` queue, declares the
`ifs.events` fanout exchange, and binds the queue to that exchange.

### 5. View API logs in Kibana

1. Open <http://localhost:5601>.
2. Open **Discover**.
3. If prompted, create a data view with:
   - Name or index pattern: `nsbm-logs-*`
   - Timestamp field: `@timestamp`
4. Search for `workorders`, `1001`, or another request value.

The logging path is:

```text
API standard output → Docker GELF driver → Logstash
                    → Elasticsearch → Kibana
```

## Stop and restart

Stop and remove the containers while retaining stored data:

```console
docker compose down
```

Start them again without rebuilding:

```console
docker compose up -d
```

Rebuild after changing the application, Dockerfile, or static files:

```console
docker compose up --build -d
```

## Clear classroom data

Delete only the Elasticsearch log indices while the stack is running:

```console
curl -X DELETE "http://localhost:9200/nsbm-logs-*"
```

Delete the containers and all Compose-managed volumes:

```console
docker compose down -v
```

The `-v` option permanently removes stored PostgreSQL data, RabbitMQ data, and
Elasticsearch logs for this Compose project. The next start is clean.

## Troubleshooting

### Docker permission denied

On Linux, add your user to the Docker group according to Docker's official
post-install documentation, sign out and back in, and retry `docker info`.
Do not make the Docker socket world-writable.

On Windows, make sure Docker Desktop is running and WSL integration is enabled
for the selected WSL distribution.

### Port already in use

Another application may already use one of the published ports. Stop that
application or change the host side of the mapping in `docker-compose.yml`.
For example, `"9001:9000"` makes the API available on port 9001 while its
container still uses port 9000.

### Kibana is not ready

Kibana and Elasticsearch can take over a minute to start. Check:

```console
docker compose ps
docker compose logs kibana
docker compose logs elasticsearch
```

### Buildx requests `fs.read` privileges

Some recent Docker Compose and Buildx combinations may reject an otherwise
local build. Use the following temporary workaround.

macOS, Linux, Git Bash, or WSL:

```console
BUILDX_BAKE_ENTITLEMENTS_FS=0 docker compose up --build -d
```

Windows PowerShell:

```powershell
$env:BUILDX_BAKE_ENTITLEMENTS_FS = "0"
docker compose up --build -d
```

### API logs do not arrive on native Linux

The GELF logging driver is configured with `host.docker.internal`, which Docker
Desktop provides automatically. If a native Linux Docker Engine cannot resolve
that name, replace it in `docker-compose.yml` with an address reachable by the
Docker daemon, such as the host's bridge address, and recreate the API:

```console
docker compose up --build -d --force-recreate api
```

### Inspect an individual service

```console
docker compose logs api
docker compose logs db
docker compose logs mq
docker compose logs logstash
docker compose logs elasticsearch
docker compose logs kibana
```

The API uses the GELF logging driver, so its application logs are primarily
viewed through Kibana rather than `docker compose logs api`.
