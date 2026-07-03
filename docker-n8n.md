# Guide: Running n8n + Ollama with Docker 🐳

The exact commands from the build. Works on any Linux box (a ~$5/month VPS, a home server, an old laptop). Docker Desktop on Mac/Windows works too — one difference noted below.

## 1. Run n8n

```bash
docker volume create n8n_data

docker run -d --name n8n \
  --restart unless-stopped \
  -p 127.0.0.1:5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e NODES_EXCLUDE='[]' \
  -e GENERIC_TIMEZONE="Europe/Berlin" \
  docker.n8n.io/n8nio/n8n
```

Then open **http://localhost:5678** and create your owner account. This is local — your machine, your data.

What each piece does, and why it's there:

| Flag | Why |
|---|---|
| `-v n8n_data:/home/node/.n8n` | **Persistent volume.** Your workflows, credentials, *and the agent's log file* (`/home/node/.n8n/agent_log.jsonl`) live here and survive container restarts and updates. Without this you lose everything on restart. |
| `-p 127.0.0.1:5678:5678` | **Localhost-only binding.** n8n is reachable from your machine but not from the internet. If you run this on a remote server, access it through an SSH tunnel (`ssh -L 5678:localhost:5678 user@server`) instead of exposing the port. |
| `-e NODES_EXCLUDE='[]'` | **Keeps the Execute Command node available.** Both workflows use *Execute Command* as the agent's memory — appending each processed email to a log file and reading it back for the digest. Some hosting setups and templates exclude this node via `NODES_EXCLUDE` because it can run shell commands inside the container. Setting it explicitly to an empty JSON array (`[]`) means "exclude nothing," so the log nodes work. Only do this on a machine you control — which is the whole point of this build. |
| `-e GENERIC_TIMEZONE=...` | Makes schedule triggers fire in **your** timezone. Set it to yours (e.g. `America/New_York`), then adjust the digest cron (`0 10 * * *` as shipped) so it lands at your 7 AM. |
| `--restart unless-stopped` | The agent comes back by itself after a reboot. Set-and-forget. |

## 2. Install Ollama + pull the model

On the host machine (not inside Docker):

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen2.5:3b-instruct
```

The model needs **~2 GB of free RAM** when loaded. Sanity check it locally:

```bash
ollama run qwen2.5:3b-instruct "Reply with just: ready"
```

## 3. Let the n8n container reach Ollama

This is the step that trips people up. The workflow calls Ollama at:

```text
http://172.17.0.1:11434
```

`172.17.0.1` is the **Docker bridge gateway** — it's how a container (n8n) reaches a service running on the **host** (Ollama). Two setups:

- **n8n in Docker (this guide):** keep `http://172.17.0.1:11434` as shipped — but Ollama must listen on more than localhost. On Linux:

  ```bash
  sudo systemctl edit ollama
  ```

  Add:

  ```ini
  [Service]
  Environment="OLLAMA_HOST=0.0.0.0"
  ```

  Then:

  ```bash
  sudo systemctl restart ollama
  ```

- **n8n NOT in Docker** (npm install, desktop app): edit the URL in the **"Ollama Classify"** node to `http://localhost:11434`.

- **Docker Desktop on Mac/Windows:** the bridge IP differs — use `http://host.docker.internal:11434` in the node instead.

## 4. The "can n8n reach Ollama?" check

Run this before importing anything — it tests the exact path the workflow will use:

```bash
docker exec n8n wget -qO- http://172.17.0.1:11434/api/tags
```

- **You see JSON listing `qwen2.5:3b-instruct`** → you're done, import the workflows.
- **Connection refused / timeout** → Ollama isn't listening on the bridge; recheck the `OLLAMA_HOST=0.0.0.0` step.
- **JSON but no model listed** → run `ollama pull qwen2.5:3b-instruct` again.

## 5. Updating later

```bash
docker pull docker.n8n.io/n8nio/n8n
docker stop n8n && docker rm n8n
# re-run the docker run command from step 1 — the volume keeps all your data
```
