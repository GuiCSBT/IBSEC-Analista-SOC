# Instalação do Shuffle SOAR no Ubuntu com Podman

## Ambiente
- **OS:** Ubuntu (VirtualBox)
- **Container runtime:** Podman (emulando Docker)
- **Shuffle versão:** latest
- **OpenSearch versão:** 3.2.0

---

## Problemas encontrados e soluções

### 1. Permissão negada no volume do OpenSearch

**Erro:**
```
java.nio.file.AccessDeniedException: /usr/share/opensearch/data/nodes
```

**Causa:** O Podman roda em modo rootless, com mapeamento de namespace de usuário diferente do Docker tradicional. O volume ficou com dono errado.

**Solução:**

Descobrir o caminho real do volume:
```bash
docker inspect shuffle-opensearch | grep -A 10 "Mounts"
```

Corrigir permissões dentro do namespace do Podman:
```bash
podman unshare chown -R 1000:1000 /home/guilherme/.local/share/containers/storage/volumes/shuffle_shuffle-database/_data
```

> ⚠️ O `sudo chown` normal **não funciona** com Podman rootless. É obrigatório usar `podman unshare`.

---

### 2. Bootstrap check falhou — memory locking

**Erro:**
```
[1] bootstrap checks failed
[1]: memory locking requested for opensearch process but memory is not locked
```

**Causa:** O OpenSearch tentava travar memória RAM (`mlockall`), o que não é permitido no ambiente.

**Solução:**

Editar o `docker-compose.yml` e adicionar na seção `environment` do serviço `opensearch`:
```yaml
opensearch:
  environment:
    - bootstrap.memory_lock=false
```

Salvar e reiniciar:
```bash
cd ~/Shuffle
docker compose down
docker compose up -d
```

---

## Comandos úteis

### Subir o Shuffle
```bash
cd ~/Shuffle
docker compose up -d
```

### Parar o Shuffle
```bash
cd ~/Shuffle
docker compose down
```

### Ver status dos containers
```bash
docker ps -a
```

### Ver logs do OpenSearch
```bash
docker logs shuffle-opensearch 2>&1 | tail -50
```

### Corrigir permissões do volume (se necessário após reinstalação)
```bash
podman unshare chown -R 1000:1000 /home/guilherme/.local/share/containers/storage/volumes/shuffle_shuffle-database/_data
```

---

## Acesso

| Serviço | URL |
|---|---|
| Shuffle (HTTP) | http://localhost:3001 |
| Shuffle (HTTPS) | https://localhost:3443 |

> Na primeira vez, crie um usuário admin com email e senha locais. O Shuffle não valida se o email existe de verdade.

---

## Stack do curso IBSEC - Módulo 8

| Ferramenta | Função |
|---|---|
| **Wazuh** | SIEM / EDR |
| **MISP** | Threat Intelligence |
| **Shuffle** | SOAR (automação e resposta) |

---

## Observações importantes

- O Ubuntu desta instalação usa **Podman** no lugar do Docker. Todos os comandos `docker` funcionam, mas o comportamento de volumes e namespaces é diferente.
- Sempre use `podman unshare` para operações de permissão em volumes.
- O OpenSearch precisa de no mínimo **4GB de RAM** disponível.
- Após reiniciar a máquina, rode `docker compose up -d` na pasta `~/Shuffle` para subir tudo novamente.
