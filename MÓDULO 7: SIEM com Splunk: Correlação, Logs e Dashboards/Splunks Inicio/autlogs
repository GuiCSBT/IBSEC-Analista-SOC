# 🔍 SIEM com Splunk — Módulo de Estudos

> Documentação prática do meu processo de aprendizado em análise de logs e detecção de ameaças usando Splunk Enterprise em ambiente local (VirtualBox/Ubuntu).

---

## Objetivo

Treinar as habilidades do dia a dia de um analista SOC Tier 1: ingestão de logs, criação de buscas SPL, identificação de comportamentos suspeitos e construção de alertas — tudo em um ambiente real, não simulado.

---

## Ambiente

| Item | Detalhe |
|---|---|
| Plataforma | Splunk Enterprise 10.4.0 |
| Sistema | Ubuntu (VirtualBox) |
| Fonte de dados | `/var/log/auth.log` (logs reais de autenticação) |
| Sourcetype | `linux_secure` |
| Index | `main` |

---

## O que fiz

### 1. Configuração da ingestão de logs

Configurei o Splunk para monitorar continuamente o arquivo de autenticação do sistema:

- Acessei `Settings → Add Data → Monitor → Files & Directories`
- Selecionei `/var/log/auth.log`
- Defini o sourcetype como `linux_secure` e o index como `main`
- Escolhi **Continuously Monitor** para ingestão em tempo real

Com isso, qualquer evento de login, sudo, sessão SSH ou falha de autenticação passa a ser indexado automaticamente.

### 2. Validação da ingestão

Confirmei que os dados estavam chegando corretamente com a busca:

```spl
index=main sourcetype=linux_secure | head 20
```

Resultado: **201 eventos** capturados, com os seguintes processos identificados:

| Processo | Eventos |
|---|---|
| systemd-logind | 57 |
| sudo | 54 |
| gdm-password | 21 |
| pkexec | 16 |
| CRON | 14 |
| mongod-8.0 | 4 |

### 3. Análise de processos ativos

```spl
index=main sourcetype=linux_secure
| stats count by process
| sort -count
```

Essa busca me mostrou quais processos geraram mais eventos de autenticação — útil para entender a linha de base (baseline) do sistema antes de começar a caçar anomalias.

---

## Buscas SPL de segurança implementadas

### Detecção de brute force SSH

Identifica IPs com mais de 10 tentativas de login falhadas:

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?P<ip>\d+\.\d+\.\d+\.\d+)"
| stats count by ip, user
| where count > 10
| sort -count
```

### Login bem-sucedido após múltiplas falhas

Padrão clássico de comprometimento de conta:

```spl
index=main sourcetype=linux_secure
| eval resultado=if(searchmatch("Failed"),"falha","sucesso")
| rex "from (?P<ip>\d+\.\d+\.\d+\.\d+)"
| stats count(eval(resultado="falha")) as falhas,
        count(eval(resultado="sucesso")) as sucessos by ip
| where falhas > 5 AND sucessos > 0
| sort -falhas
```

### Auditoria de uso de sudo

Registra todos os comandos executados com privilégio root:

```spl
index=main sourcetype=linux_secure "sudo"
| rex "(?P<usuario>\w+) : TTY.*COMMAND=(?P<comando>.+)"
| stats count by usuario, comando
| sort -count
```

### Logins fora do horário comercial

Detecta acessos em horários suspeitos (antes das 6h ou após as 20h):

```spl
index=main sourcetype=linux_secure "Accepted"
| eval hora=strftime(_time,"%H")
| where hora < "06" OR hora > "20"
| rex "for (?P<usuario>\w+) from (?P<ip>\d+\.\d+\.\d+\.\d+)"
| table _time, hora, usuario, ip
| sort -_time
```

### Tentativas com usuários inexistentes

Revela o perfil dos bots e scripts de ataque:

```spl
index=main sourcetype=linux_secure "Invalid user"
| rex "Invalid user (?P<usuario_tentado>\w+) from (?P<ip>\d+\.\d+\.\d+\.\d+)"
| stats count by usuario_tentado, ip
| sort -count
```

---

## Conceitos estudados

### CIM — Common Information Model

Esquema de normalização do Splunk que padroniza nomes de campos entre diferentes fontes de dados. Permite usar o mesmo SPL independente do vendor (Palo Alto, Cisco, Windows, Linux).

Data models relevantes para SOC:

- `Authentication` — logins e falhas
- `Network_Traffic` — conexões de rede
- `Endpoint` — processos e arquivos
- `Intrusion_Detection` — alertas de IDS/IPS

### Correlação de eventos

Técnica que cruza múltiplos eventos para detectar padrões que isoladamente não seriam suspeitos. Exemplo: uma falha de login não é alerta — cem falhas seguidas de um sucesso é brute force confirmado.

### Pipeline de segurança (SIEM)

```
Fontes de dados → Normalização (CIM) → Correlação → Notable Event → Investigação
```

---

## Próximos passos

- [ ] Criar dashboard de monitoramento de autenticação em tempo real
- [ ] Configurar alertas automáticos para brute force
- [ ] Ingerir logs de rede (`/var/log/ufw.log`) para análise de tráfego
- [ ] Estudar `transaction` command para correlação avançada
- [ ] Explorar lookup tables com feeds de threat intelligence

---

## Referências

- [Splunk SPL Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [Splunk CIM Documentation](https://docs.splunk.com/Documentation/CIM)
- [MITRE ATT&CK](https://attack.mitre.org/)
