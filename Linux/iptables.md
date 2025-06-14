
# 🔥 Entendendo Conceitos Fundamentais do IPTABLES, NAT e Roteamento

## 🎯 Tabelas no IPTABLES

O `iptables` possui várias tabelas, que definem o tipo de processamento aplicado aos pacotes:

| Tabela     | Função Principal                                                 |
| ---------- | ---------------------------------------------------------------- |
| `filter`   | 🔒 Filtragem de pacotes (default) - permite, bloqueia ou rejeita |
| `nat`      | 🔀 Tradução de endereços de rede (NAT) - altera IP e/ou porta    |
| `mangle`   | 🛠️ Modificação avançada de pacotes (TTL, QoS, marcação, etc.)   |
| `raw`      | ⚙️ Desativa o rastreamento de conexões (`conntrack`)             |
| `security` | 🔐 Regras baseadas em contexto de segurança (SELinux, AppArmor)  |

→ A mais usada para roteamento e redirecionamento é a tabela nat.

---

## 🔄 Chains (Cadeias) - Fluxo dos Pacotes

## ➕ INPUT
- Pacotes destinados à própria máquina.
- Exemplo: SSH, HTTP, ping para o servidor.

---

## ➖ OUTPUT
- Pacotes gerados pela própria máquina.
- Exemplo: A máquina faz ping ou conecta-se a outro servidor.

---

## 🔁 FORWARD
- Pacotes que passam pela máquina, mas não são para ela.
- Usado quando a máquina atua como roteador ou firewall.
- Exemplo: Um roteador Linux repassando tráfego de uma rede interna para a internet.

---

## 🔗 PREROUTING
- Acontece antes do roteamento ser decidido.
- Usado principalmente para:
    - DNAT (Destination NAT): Alterar o IP de destino do pacote.
- 🔥 É onde você faz redirecionamento de portas.

---

## 🔚 POSTROUTING
- Acontece depois do roteamento, antes do pacote sair pela interface.
- Usado principalmente para:
    - SNAT (Source NAT): Alterar o IP de origem do pacote.
- 🔥 É onde você faz compartilhamento de internet (masquerade).

---

## 🌐 NAT (Network Address Translation)
### ✅ O que é NAT?
- É a tradução de endereços de rede.
- Permite que vários dispositivos compartilhem um único IP público ou modifique IPs/portas nos pacotes.

---

## 🚥 Tipos de NAT
## 🔄 SNAT (Source NAT)
- Altera o IP de origem dos pacotes.
- Geralmente usado para quando a máquina compartilha a internet (saídas da rede interna para a externa).
- 🔥 Exemplo clássico:
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
- Isso troca o IP de origem pelo IP da interface `eth0`.

---

## 🔃 DNAT (Destination NAT)
- Altera o IP de destino dos pacotes.
- Usado para redirecionar conexões externas para máquinas internas.
- 🔥 Exemplo de redirecionamento da porta 80 do servidor para um servidor interno:
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080
```

---

## 🗺️ Roteamento
## ✅ O que é Roteamento?
- Processo de decidir para onde um pacote deve ser enviado com base na tabela de rotas do sistema.

→ Roteamento ocorre em máquinas que possuem mais de uma interface de rede (ex.: atuando como firewall, roteador ou gateway).

## 🔥 Ativando roteamento no Linux:

```
sudo sysctl -w net.ipv4.ip_forward=1
```

→ Para tornar permanente, adicione no `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
```

---

## 🔗 Fluxo Simplificado dos Pacotes no IPTABLES

```
       Entrada na interface
               ↓
         [PREROUTING]    → DNAT (Altera o destino, se necessário)
               ↓
     Checar se é para mim? ───────────────→ [INPUT] → Processo local
               │
               ↓
          [FORWARD]  → Encaminhar pacote (se não for para mim)
               ↓
          [POSTROUTING] → SNAT / Masquerade (Altera origem, se necessário)
               ↓
       Saída pela interface
```

---

## 🔥 🔥 Resumo Rápido dos Conceitos

| Conceito        | Função                                                                              |
| --------------- | ----------------------------------------------------------------------------------- |
| **PREROUTING**  | Ações antes de decidir para onde enviar o pacote (geralmente DNAT)                  |
| **POSTROUTING** | Ações após decidir para onde enviar, antes de sair pela interface (geralmente SNAT) |
| **INPUT**       | Tráfego destinado **para a própria máquina**                                        |
| **OUTPUT**      | Tráfego gerado **pela própria máquina**                                             |
| **FORWARD**     | Tráfego que **passa pela máquina** (roteador/firewall)                              |
| **NAT**         | Tabela responsável por modificar IPs e/ou portas (SNAT, DNAT, MASQUERADE)           |
| **DNAT**        | Altera **destino** (ex.: redirecionar porta para outro IP)                          |
| **SNAT**        | Altera **origem** (ex.: mascarar IP interno com IP externo)                         |
| **Roteamento**  | Decidir para onde enviar pacotes com base nas rotas                                 |

---

## 💡 Exemplos Práticos
## 🔄 Habilitar compartilhamento de internet (SNAT/MASQUERADE):

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

## 🔁 Redirecionar porta 80 para um servidor interno (DNAT):

```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080
```

## 🚦 Ativar roteamento no Linux:

```
sudo sysctl -w net.ipv4.ip_forward=1
```

---

# 🔥 Documentação de Redirecionamento de Rede com IPTABLES no Linux

## 📜 Descrição
Este documento tem como objetivo ensinar como utilizar o `iptables` no Linux para realizar redirecionamento de IPs e portas, além de entender como cada comando funciona, como aplicar alterações temporárias ou permanentes, e como remover as regras aplicadas.

## 🚩 Diferença entre `iptables` e `ip route`

| Comando   | Função                                                                                  |
|------------|------------------------------------------------------------------------------------------|
| `iptables` | Firewall, controle de tráfego, bloqueios, NAT (redirecionamento de IP e portas)        |
| `ip route` | Definição de rotas para envio de pacotes (por onde eles devem sair ou chegar)          |

→ **Para redirecionamento de IP ou portas, usamos `iptables`.**

## 🔗 Conceitos básicos do IPTABLES

- **Tabelas**:
  - `filter` → Filtragem de pacotes (padrão).
  - `nat` → Alteração de endereços (NAT, redirecionamento).
  - `mangle` → Alteração de pacotes.
  - `raw` → Manipulação antes do rastreamento de conexões.

- **Chains (Correntes)**:
  - `INPUT` → Tráfego destinado à máquina.
  - `OUTPUT` → Tráfego gerado pela máquina.
  - `FORWARD` → Tráfego que passa pela máquina (roteamento).
  - `PREROUTING` → Antes do roteamento (útil para redirecionamento).
  - `POSTROUTING` → Após o roteamento (útil para mascaramento).

## 🚀 Exemplos de Redirecionamento

### 1️⃣ 🔁 Redirecionar uma porta local para outra porta local

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 3000
```

**Explicação:**
- `-t nat` → Define que a regra será aplicada na tabela NAT (responsável por redirecionamento e alteração de endereços).
- `-A PREROUTING` → Adiciona a regra na chain PREROUTING (antes de decidir o destino final do pacote).
- `-p tcp` → Aplica para o protocolo TCP.
- `--dport 8080` → Porta de destino que chega na máquina.
- `-j REDIRECT` → Ação de redirecionar para a própria máquina.
- `--to-port 3000` → Porta para onde será redirecionado.

✔️ Acessar `http://127.0.0.1:8080` será redirecionado para `http://127.0.0.1:3000`.

---

### 2️⃣ 🔀 Redirecionar requisição a um IP para outro IP (DNAT)
```bash
sudo iptables -t nat -A PREROUTING -d 192.168.1.100 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.200:8080
```
**Explicação:**
- `-d 192.168.1.100` → IP de destino da requisição recebida.
- `--dport 80` → Porta de destino recebida.
- `-j DNAT` → Destination NAT, altera o destino do pacote.
- `--to-destination 192.168.1.200:8080` → Novo IP e porta para onde os pacotes serão encaminhados.

✔️ Acessar `192.168.1.100:80` será enviado para `192.168.1.200:8080`.

---

### 3️⃣ 🌐 Tornar a máquina um roteador (liberar o forward)
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
**Tornar permanente:**
```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
→ Permite que a máquina encaminhe pacotes entre redes (atuar como roteador).

---

### 4️⃣ 🚪 Redirecionar saída da máquina (OUTPUT)
```bash
sudo iptables -t nat -A OUTPUT -d 8.8.8.8 -p tcp --dport 53 -j DNAT --to-destination 1.1.1.1:53
```
**Explicação:**
- Aplica-se no tráfego originado da própria máquina (chain OUTPUT).
- Qualquer tentativa de conexão TCP para `8.8.8.8:53` será redirecionada para `1.1.1.1:53`.

---

## 🗑️ Removendo Regras

Listar todas as regras:
```bash
sudo iptables -L
```

Listar regras PREROUTING:
```bash
sudo iptables -t nat -L -n -v --line-numbers
```

Remover uma regra específica:
```bash
sudo iptables -t nat -D [Chains] [número-da-regra]
```

Exemplo:
```bash
sudo iptables -t nat -D PREROUTING 1
```

Remover todas as regras NAT:
```bash
sudo iptables -t nat -F
```

Remover todas as regras de uma chains:
```bash
sudo iptables -F [Chains]
```

## 🧠 Verificando as Regras Ativas
```bash
sudo iptables -t nat -L -n -v
```

## 💾 Tornando as Configurações Permanentes

Por padrão, as regras do `iptables` são **temporárias** (se perdem ao reiniciar).

### Debian/Ubuntu:
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### CentOS/RHEL:
```bash
sudo service iptables save
```

### Manual (qualquer distro):
```bash
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

## 📚 Referências Rápidas

### ▶️ Sintaxe básica:
```bash
iptables -t [tabela] -A [chain] [condições] -j [ação]
```

### ▶️ Principais ações (targets):
- `ACCEPT` → Aceitar o pacote.
- `DROP` → Descartar o pacote silenciosamente.
- `REJECT` → Rejeitar o pacote com resposta.
- `DNAT` → Altera o destino do pacote (Destination NAT).
- `SNAT` → Altera o IP de origem (Source NAT).
- `MASQUERADE` → Altera o IP de origem dinamicamente (útil para saída dinâmica com IP público).
- `REDIRECT` → Redireciona para a própria máquina.

## 🏗️ Arquitetura do Fluxo no NAT

```plaintext
[PREROUTING] → [ROUTING] → [FORWARD] → [POSTROUTING] → [OUTPUT]
```

- **PREROUTING** → Antes de decidir o destino (útil para redirecionamento de entrada).
- **POSTROUTING** → Após o roteamento (útil para mascaramento/SNAT).
- **OUTPUT** → Tráfego originado na própria máquina.
- **FORWARD** → Pacotes que atravessam a máquina.

## 🔐 Observação Importante
- Alterações incorretas no `iptables` podem impactar conectividade, inclusive acesso SSH remoto.
- Sempre teste localmente antes de aplicar em produção.

---

# 🔐 Documentação de Políticas Padrão (`iptables -P`) no Linux

## 📜 O que são Políticas Padrão?

As **políticas padrão** no `iptables` definem o que deve acontecer com um pacote que **não corresponde a nenhuma das regras definidas** na chain.

👉 Ou seja, se não houver nenhuma regra aplicável ao pacote, a política padrão da chain será aplicada.

---

## 🚥 Chains que suportam políticas padrão:

- `INPUT` → Pacotes **que entram** na máquina.
- `FORWARD` → Pacotes que **passam pela máquina** (se a máquina estiver atuando como roteador).
- `OUTPUT` → Pacotes **gerados pela própria máquina**.

> ⚠️ Chains como `PREROUTING` e `POSTROUTING` não suportam políticas padrão, pois são usadas para NAT e não possuem decisão de aceitar ou rejeitar pacotes.

---

## 🔧 Comando para definir a política padrão:

```bash
sudo iptables -P [CHAIN] [POLÍTICA]
```

### ▶️ Onde:
- `[CHAIN]` → Nome da chain (`INPUT`, `FORWARD` ou `OUTPUT`).
- `[POLÍTICA]` → Ação padrão (`ACCEPT` ou `DROP`).

---

## ✅ Exemplos de Configuração de Políticas

### 🔓 Permitir tudo que entra na máquina (não recomendado para segurança):
```bash
sudo iptables -P INPUT ACCEPT
```

### 🔒 Bloquear todo o tráfego de encaminhamento (roteamento):
```bash
sudo iptables -P FORWARD DROP
```

### 🚀 Permitir tudo que sai da máquina:
```bash
sudo iptables -P OUTPUT ACCEPT
```

---

## 🔐 Exemplo de configuração segura mais comum para servidores:

```bash
sudo iptables -P INPUT DROP       # Bloqueia tudo que entra por padrão
sudo iptables -P FORWARD DROP     # Bloqueia roteamento
sudo iptables -P OUTPUT ACCEPT    # Permite tudo que sai
```

Após definir essas políticas, você deve criar regras específicas para permitir serviços que você deseja, como SSH, HTTP, HTTPS, etc. Por exemplo:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Permite SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Permite HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # Permite HTTPS
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  # Permite respostas
```

---

## 🚫 O que acontece sem uma regra para tráfego de resposta?

Se você fizer:
```bash
sudo iptables -P INPUT DROP
```
Sem adicionar:
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
→ Você não conseguirá nem acessar sites, nem receber respostas de conexões já estabelecidas. Este é um erro muito comum.

---

## 🗑️ Como listar as políticas ativas:

```bash
sudo iptables -L -n -v
```

Saída exemplo:
```
Chain INPUT (policy DROP 100 packets, 0 bytes)
Chain FORWARD (policy DROP 0 packets, 0 bytes)
Chain OUTPUT (policy ACCEPT 200 packets, 50000 bytes)
```

---

## 🔄 Resetar políticas para permitir tudo:

```bash
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

E limpar todas as regras:
```bash
sudo iptables -F
```

---

## 💾 Tornando as Políticas Permanentes

### Debian/Ubuntu:
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### CentOS/RHEL:
```bash
sudo service iptables save
```

### Manual (qualquer distro):
```bash
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## 🚩 Boas Práticas

- 🔒 Use `INPUT DROP` para aumentar a segurança.
- ✅ Permita portas e serviços essenciais manualmente.
- 🌐 Deixe `OUTPUT ACCEPT` em servidores comuns para não bloquear atualizações, DNS e acesso à internet.
- 🚫 Se a máquina não for roteador, use `FORWARD DROP`.

---

## 🏗️ Resumo Visual das Políticas

| Chain   | Função                                  | Política Comum       |
|---------|------------------------------------------|----------------------|
| INPUT   | Entrada na máquina                      | DROP (seguro)        |
| OUTPUT  | Saída da máquina                        | ACCEPT (recomendado) |
| FORWARD | Encaminhamento (roteamento)             | DROP (se não roteia) |

---

## 🔐 Aviso
⚠️ Atenção! Sempre que definir `INPUT DROP`, certifique-se de adicionar antes uma regra permitindo a porta do SSH, ou você perderá acesso remoto à máquina.

---

# 📜 Documentação de Logs no IPTABLES

## 🔍 O IPTABLES gera logs?
✔️ Sim, o iptables pode gerar logs, mas somente se você criar regras específicas para logar.

→ O iptables por padrão não gera logs automáticos de conexões permitidas ou bloqueadas, a não ser que você adicione a ação LOG em uma regra.

---

## 🔥 Como funciona o LOG no IPTABLES?

Quando você adiciona uma regra com o alvo (`target`) **LOG**, o kernel envia uma mensagem para o **syslog** (geralmente `/var/log/kern.log` ou `/var/log/messages`, dependendo da distro).

Esses logs podem ser visualizados com ferramentas como `dmesg`, `journalctl` ou diretamente nos arquivos de log do sistema.

---

## 📦 Sintaxe da Regra de LOG:

```bash
sudo iptables -A [CHAIN] -j LOG [opções]
```

---

## 🛠️ Parâmetros comuns para LOG:

| Parâmetro   | Descrição                                  |
|---------|------------------------------------------|
|`--log-prefix ""`|Texto que será adicionado no início de cada linha de log (máx. 29 caracteres).|
|`--log-level`|Nível de severidade do syslog (ex.: `info`, `warning`, `debug`, `notice`).|

---

## ✅ Exemplos Práticos

## 🔥 Logar e bloquear conexões na porta 22 (SSH) de qualquer IP:

```
sudo iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH ATTEMPT: " --log-level warning
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

→ Isso loga qualquer tentativa de conexão SSH e depois bloqueia.

---

## 🚫 Logar todos os pacotes DROP na chain INPUT:

```
sudo iptables -A INPUT -j LOG --log-prefix "INPUT DROP: " --log-level warning
sudo iptables -A INPUT -j DROP
```

→ Isso não bloqueia sozinho, apenas gera o log. É por isso que geralmente usamos `LOG` seguido de `DROP` ou `REJECT`.

---

## 🌐 Logar tráfego de uma porta específica (ex.: porta 80):

```
sudo iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "HTTP TRAFFIC: " --log-level info
```

---

## 🗒️ Onde os logs aparecem?

## 🔍 Verificar com `dmesg` (buffer do kernel):

```
sudo dmesg | grep "INPUT DROP"
```

## 🔥 Verificar em arquivos de log:
- Debian/Ubuntu:
    - /var/log/kern.log
    - /var/log/syslog

- CentOS/RHEL:
    - /var/log/messages

## 🔎 Usando journalctl (em sistemas com systemd):

```
sudo journalctl -k | grep "INPUT DROP"
```

---

## 🧹 Logs muito verbosos? Cuidado!
- Logs de IPTABLES podem gerar muito volume, principalmente se você estiver logando pacotes DROP ou conexões frequentes.
- Isso pode encher seu disco rapidamente.

---

## 🚩 Dicas para controlar volume de logs:

1. Use log apenas para o que é realmente necessário.

2. Adicione regras específicas para evitar logar tráfego comum.

3. Utilize filtros, por exemplo, logar apenas pacotes de uma origem suspeita:

```
sudo iptables -A INPUT -s 192.168.1.50 -j LOG --log-prefix "SUSPECT: "
```

4. Utilize ferramentas como `logrotate` para gerenciar e rotacionar seus logs automaticamente.

---

## 🏗️ Exemplo Completo de Firewall com Logs

```
# Definir políticas padrão
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Permitir conexões existentes e relacionadas
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Permitir SSH
sudo iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH ATTEMPT: " --log-level warning
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Permitir HTTP e HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Logar e bloquear todo o restante
sudo iptables -A INPUT -j LOG --log-prefix "INPUT DROP: " --log-level warning
sudo iptables -A INPUT -j DROP
```

---

## 💾 Tornando Logs e Regras Permanentes

### Debian/Ubuntu:
```
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### CentOS/RHEL:
```
sudo service iptables save
```

### Manual:
```
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## 🏁 Resumo Rápido

| Comando   | Descrição |
|---------|------------------------------------------|
|`-j LOG`|Define que a ação é gerar um log|
|`--log-prefix "TEXTO"`|Texto que aparece no início do log|
|`--log-level [info]`|warning| 

---

## ⚠️ Aviso
- Nunca logue tudo sem filtro, isso pode gerar gigabytes de dados rapidamente.

- Sempre combine `LOG` com `DROP`, `ACCEPT` ou `REJECT` para garantir o comportamento esperado.

---