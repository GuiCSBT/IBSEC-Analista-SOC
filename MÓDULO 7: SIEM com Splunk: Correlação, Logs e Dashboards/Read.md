# Módulo 7 — SIEM com Splunk

> Este módulo documenta minha jornada prática aprendendo Splunk como ferramenta SIEM, com foco nas habilidades do dia a dia de um analista SOC.

---

## O que é SIEM?

SIEM (Security Information and Event Management) é a tecnologia central de um SOC. Ela faz duas coisas principais:

- **Coleta e centraliza logs** de todas as fontes do ambiente (firewalls, endpoints, servidores, aplicações)
- **Correlaciona eventos** para detectar padrões de ataque que seriam invisíveis analisando cada fonte separadamente

Sem um SIEM, um analista precisaria acessar cada sistema individualmente para investigar um incidente. Com o SIEM, tudo está em um lugar só, pesquisável e correlacionável.

---

## Por que Splunk?

O Splunk é a ferramenta SIEM mais presente no mercado corporativo e uma das mais cobradas em vagas de SOC. Seus diferenciais:

- **SPL** (Search Processing Language) — linguagem poderosa para consultar e transformar logs
- **CIM** (Common Information Model) — normalização de campos entre diferentes fontes de dados
- **Splunk ES** (Enterprise Security) — módulo SIEM com correlation searches, notable events e risk-based alerting
- **Ecossistema** — milhares de apps e integrações prontas no Splunkbase

---

## Estrutura do módulo

```
MÓDULO 7: SIEM com Splunk/
│
├── README.md                  ← você está aqui
│
├── 01-Primeiros-Passos/       ← instalação, ingestão de logs, SPL básico
│   ├── README.md
│   ├── Splunk processos.png
│   └── Splunk security_linux.png
│
├── 02-Deteccao-de-Ameacas/    ← buscas de segurança, casos de uso reais
│   └── README.md
│
├── 03-Dashboards-e-Alertas/   ← visualizações e alertas automáticos
│   └── README.md
│
└── 04-Correlacao-Avancada/    ← transaction, join, subsearches, lookups
    └── README.md
```

---

## Roadmap de aprendizado

### Fase 1 — Fundamentos ✅
- [x] Instalação do Splunk Enterprise em VM local
- [x] Configuração de ingestão de logs (`/var/log/auth.log`)
- [x] Primeiras buscas em SPL
- [x] Análise de processos e eventos de autenticação

### Fase 2 — Detecção de ameaças 🔄
- [ ] Brute force SSH
- [ ] Movimento lateral
- [ ] Exfiltração de dados
- [ ] Uso suspeito de sudo e privilégios

### Fase 3 — Dashboards e alertas
- [ ] Dashboard de monitoramento de autenticação
- [ ] Alertas automáticos com trigger conditions
- [ ] Saved searches e reports

### Fase 4 — SPL avançado
- [ ] `transaction` para correlação de sessões
- [ ] `lookup` com listas de threat intelligence
- [ ] `join` e `append` para cruzar fontes
- [ ] Macros e campos calculados com `eval`

---

## Conceitos-chave do módulo

| Conceito | O que é |
|---|---|
| Index | Repositório onde os eventos são armazenados |
| Sourcetype | Classificação do formato do log (define como o Splunk parseia) |
| SPL | Linguagem de busca do Splunk |
| CIM | Esquema de normalização de campos |
| Correlation Search | Busca agendada que detecta padrões e gera alertas |
| Notable Event | Alerta gerado no Splunk ES para triagem |
| Baseline | Comportamento normal do ambiente, usado como referência |

---

## Ambiente utilizado

| Item | Detalhe |
|---|---|
| Plataforma | Splunk Enterprise 10.4.0 |
| SO | Ubuntu — VirtualBox |
| Logs iniciais | `/var/log/auth.log` |
| Sourcetype | `linux_secure` |
| Index | `main` |

---

## Referências

- [Documentação oficial Splunk](https://docs.splunk.com)
- [SPL Quick Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual)
- [CIM Add-on](https://docs.splunk.com/Documentation/CIM)
- [MITRE ATT&CK](https://attack.mitre.org)
- [Splunkbase — apps gratuitos](https://splunkbase.splunk.com)
