# Guia de Implantação de Laboratório: Plataforma MISP
## Módulo: Threat Intelligence & CTI (Cyber Threat Intelligence)

Este documento é o registro do processo de subida da plataforma **MISP (Malware Information Sharing Platform)** via Docker em um microlaboratório controlado (**Ubuntu Server** rodando em VM/VirtualBox). Mais do que um tutorial pronto, é o diário de bordo dos erros, diagnósticos e soluções encontrados ao longo do caminho — porque na vida real (e nos laboratórios de aula) raramente tudo sobe de primeira.

O objetivo é documentar as lições aprendidas, as decisões de arquitetura e os comandos de infraestrutura necessários para montar um ambiente funcional de Inteligência de Ameaças, com foco em operações de Blue Team e SOC.

---

## 💡 1. Exploração de Ambiente Local: CLI Server vs. Desktop GUI

Durante o reconhecimento inicial do ambiente, percebi uma diferença comum entre quem está acostumado com Linux Desktop e o que se encontra em servidores de produção:

* **Comportamento observado:** O comando `ls` no diretório inicial (`~`) não retornava nada.
* **Causa raiz:** Diferença estrutural entre distribuições. Enquanto versões *Desktop* criam automaticamente pastas padrão (`Downloads`, `Documents`), o **Ubuntu Server** vem propositalmente limpo — menos overhead, menor superfície de ataque.
* **Comando de diagnóstico:**
  ```bash
  ls -la
  ```
  Mostra os arquivos ocultos (`.bashrc`, `.profile`), confirmando que o terminal está funcionando normalmente — só não tinha nada "visível" mesmo.

---

## 🛠️ 2. Evolução de Configuração: Arquivo `.env` Moderno

Ao copiar o template de variáveis (`cp template.env .env`), me deparei com um arquivo de **394 linhas** — bem mais robusto do que os exemplos curtos que aparecem em tutoriais antigos.

* **Análise:** o `.env` atual do `misp-docker` já vem com variáveis de build, integrações futuras e várias opções desativadas por padrão.
* **Navegação eficiente no Vim:**
  1. Abrir o arquivo: `vi .env`
  2. Pesquisar um termo: `Esc`, depois `/BASE_URL` e `Enter`
  3. Ir para próxima ocorrência: `n` (ou `N` para voltar)
  4. Editar: `i` (modo de inserção)
  5. Salvar e sair: `Esc`, depois `:wq` e `Enter`

---

## 🔄 3. Modernização da Sintaxe: Docker Compose V2

| Geração | Comando | Status |
| :--- | :--- | :--- |
| V1 | `sudo docker-compose build` | Obsoleto (binário isolado) |
| V2 | `sudo docker compose build` | Padrão atual (plugin nativo do Docker) |

**Lição:** abandonar de vez o hífen nos comandos de orquestração — toda a documentação recente já assume a sintaxe V2.

---

## 🚨 4. Diagnóstico Crítico: Falha "Unhealthy" no Banco de Dados (Errcode: 28)

**Sintoma:**
```text
Container misp-docker-db-1         Error
dependency failed to start: container misp-docker-db-1 is unhealthy
```

**Investigação:**
Com `sudo docker logs misp-docker-db-1`, encontrei o erro real:
```
[ERROR] mariadbd: Error writing file './ddl_recovery.log' (Errcode: 28 "No space left on device")
```

* **Causa real:** o instalador do Ubuntu Server usa **LVM (Logical Volume Manager)** e, por padrão, aloca apenas **50% do disco virtual** para a partição raiz (`/`). O download das imagens do MISP estourou os 15 GB iniciais e o banco não conseguia mais escrever.

---

## 📐 5. Procedimento de Engenharia: Expansão de Armazenamento LVM

Depois de aumentar o disco virtual no VirtualBox para **60 GB**, foi preciso esticar o sistema de arquivos do Linux em tempo real:

```bash
# 1. Aumentar a partição 3 dentro do disco sda
sudo growpart /dev/sda 3

# 2. Atualizar o Volume Físico (PV) do LVM
sudo pvresize /dev/sda3

# 3. Estender o Volume Lógico (LV) usando 100% do novo espaço
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

# 4. Redimensionar o sistema de arquivos em tempo real
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Validei com `df -h`: a partição raiz `/` foi de ~15 GB para **43 GB úteis** (29 GB livres).

---

## 🚀 6. Sanetização e Execução Final (Detached Mode)

Como a primeira tentativa gerou arquivos corrompidos no volume do banco, precisei limpar tudo antes de subir de novo:

```bash
# 1. Limpar volumes e contêineres antigos com erro
sudo docker compose down -v

# 2. Subir a infraestrutura em segundo plano de forma definitiva
sudo docker compose up -d
```
O `-v` apaga os volumes corrompidos antigos, e o `-d` libera o terminal enquanto os serviços sobem em background.

---

## 🔐 7. Permissão Negada ao Docker (`permission denied`)

**Sintoma:**
```text
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock
```

* **Causa:** o usuário comum não pertence ao grupo `docker`, então não consegue falar com o socket sem `sudo`.
* **Correção definitiva:**
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker
  ```
  Depois disso, `docker ps` já funciona sem precisar de `sudo` toda vez.

---

## 🌐 8. O Maior Desafio: Acesso Externo à VM

Depois de tudo "Healthy" no `docker ps`, veio a pergunta que pareceu simples mas não era: **como acessar o MISP de fora da VM?**

### 8.1 — Rede NAT não é acessível de fora
Rodando `ip a`, o IP da VM aparecia como `10.0.2.15` — faixa padrão de **NAT do VirtualBox**. Nesse modo, a VM acessa a internet normalmente, mas nenhum outro dispositivo da rede consegue "entrar" nela.

**Solução:** trocar o modo de rede da VM de **NAT** para **Bridged Adapter** (Adaptador em modo Bridge), nas configurações de Rede da VM no VirtualBox, selecionando a placa de rede física do host. Depois de religar a VM, o `ip a` passou a mostrar um IP real da rede local (`192.168.18.234`), acessível por qualquer máquina da mesma rede.

### 8.2 — Redirecionamento forçado para `localhost`
Mesmo com o IP certo e a porta 443 confirmada aberta (validei com `ss -tlnp | grep 443` na VM e `Test-NetConnection -ComputerName <IP> -Port 443` no Windows, que retornou `True`), o navegador insistia em redirecionar para `https://localhost/users/login` — e `localhost`, no navegador do host, aponta para o próprio Windows, não para a VM.

* **Causa:** a variável `BASE_URL` no `.env` estava vazia, e o MISP assume `localhost` como padrão nesse caso.
* **Correção:**
  ```bash
  vi .env
  # localizar a linha:
  BASE_URL=
  # alterar para:
  BASE_URL=https://192.168.18.234
  ```
  Depois, reiniciar os contêineres para aplicar:
  ```bash
  sudo docker compose down
  sudo docker compose up -d
  ```

Após o `misp-core` voltar ao status **healthy**, o acesso via navegador (`https://192.168.18.234`) finalmente funcionou — com o aviso esperado de certificado autoassinado, que é só aceitar/continuar.

---

## ✅ Resultado Final

Com toda a infraestrutura validada (disco redimensionado, permissões de Docker corrigidas, rede em modo Bridge e `BASE_URL` configurada), o MISP ficou acessível via:

```
https://<IP-da-VM>
```

Login padrão inicial:
```
Usuário: admin@admin.test
Senha: admin
```
(senha trocada no primeiro acesso, por segurança)

---

## 📌 Principais Lições do Processo

1. Ambientes Server e Desktop se comportam diferente — não assuma que algo "quebrou" só porque está vazio.
2. Sempre cheque espaço em disco e estrutura LVM antes de subir stacks pesadas como o MISP.
3. Adicionar o usuário ao grupo `docker` evita ficar repetindo `sudo` o tempo todo.
4. Rede NAT é ótima para acesso à internet de dentro da VM, mas inútil para expor serviços — Bridge é o caminho quando se quer simular um "servidor de verdade".
5. Sempre confira variáveis como `BASE_URL` quando o serviço insiste em redirecionar para um endereço que não faz sentido.
