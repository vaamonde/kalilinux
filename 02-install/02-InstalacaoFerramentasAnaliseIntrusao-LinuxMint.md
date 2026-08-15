# 🐧 Aula 02b — Instalação das Ferramentas de Análise, Monitoramento e Teste de Intrusão Wireless no Linux Mint 22.3 (Zena)

> **Módulo:** Redes Sem-Fio — Preparação do Ambiente de Ataque (Linux Mint)
> **Aula:** 02b — Instalação e Configuração das Ferramentas (aircrack-ng, reaver, hashcat, Wireshark, tcpdump e complementares)
> **Base do sistema:** Linux Mint 22.3 "Zena" (Cinnamon/Xfce/MATE) — base **Ubuntu 24.04.3 LTS "Noble Numbat"**, kernel HWE 6.14+
> **Pré-requisito:** Linux Mint 22.3 já instalado, particionado e com pendrive/boot já concluídos (fora do escopo deste documento)
> **Antenas Wi-Fi externas homologadas:**
> - USB Realtek — chipset **RTL8188EUS**, driver `8188eu` (DKMS via GitHub, ver item 06)
> - USB Qualcomm Atheros — chipset **AR9271**, driver `ath9k_htc` (nativo no kernel)

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 02/08/2026 |
| **Data de atualização** | 02/08/2026 |
| **Versão** | 0.01 |
| **Distribuição testada** | Linux Mint 22.3 "Zena" (base Ubuntu 24.04.3 LTS) |
| **Motivo da mudança de distribuição** | A rede do laboratório possui bloqueio/restrição de acesso aos repositórios oficiais do Kali Linux (`http.kali.org`), inviabilizando `apt update`/`apt install` das ferramentas via os metapacotes nativos do Kali (ver Aula 02 — `02-installkali/01-...`). O Linux Mint 22.3, por ser baseado em **Ubuntu 24.04 (repositórios `noble`, liberados na rede do laboratório)**, permite instalar as mesmas ferramentas de pentest wireless a partir dos repositórios `universe`/`multiverse` do Ubuntu, sem depender da infraestrutura do Kali. |

### 🔗 Links do Autor

| Canal | Link |
|---|---|
| Procedimentos em TI | http://procedimentosemti.com.br |
| Bora para Prática | http://boraparapratica.com.br |
| Site pessoal | http://vaamonde.com.br |
| Facebook — Procedimentos em TI | https://www.facebook.com/ProcedimentosEmTi |
| Facebook — Bora para Prática | https://www.facebook.com/BoraParaPratica |
| Instagram — Procedimentos em TI | https://www.instagram.com/procedimentoem |
| YouTube — Bora para Prática | https://www.youtube.com/boraparapratica |
| LinkedIn — Robson Vaamonde | https://www.linkedin.com/in/robson-vaamonde-0b029028/ |
| GitHub — Procedimentos em TI | https://github.com/vaamonde |

---

## ⚠️ Aviso Legal e Ético — Leia Antes de Iniciar

> Este material é destinado **exclusivamente a fins didáticos**, em **ambiente de rede controlada e isolada** (laboratório), com equipamentos de propriedade da instituição de ensino e consentimento expresso para os testes.
>
> A utilização das ferramentas aqui instaladas contra redes de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de responsabilidade civil (Código Civil, arts. 927 a 943).
>
> Cada grupo deve utilizar as ferramentas **somente** contra o Access Point que lhe foi atribuído no laboratório.

---

## 📑 Sumário

1. [Por que o Linux Mint neste cenário](#01---por-que-o-linux-mint-neste-cenário)
2. [Preparação dos Repositórios e Atualização do Sistema](#02---preparação-dos-repositórios-e-atualização-do-sistema)
3. [Instalação da Suíte Aircrack-ng](#03---instalação-da-suíte-aircrack-ng)
4. [Instalação do Reaver, Pixiewps e Wash (Ataques WPS)](#04---instalação-do-reaver-pixiewps-e-wash-ataques-wps)
5. [Instalação do hcxtools / hcxdumptool (Captura e Ataque PMKID)](#05---instalação-do-hcxtools--hcxdumptool-captura-e-ataque-pmkid)
6. [Drivers das Antenas Externas (Realtek RTL8188EUS e Atheros AR9271)](#06---drivers-das-antenas-externas-realtek-rtl8188eus-e-atheros-ar9271)
7. [Instalação do Hashcat (Aceleração por GPU/CPU)](#07---instalação-do-hashcat-aceleração-por-gpucpu)
8. [Instalação do Wireshark e Tcpdump (Análise de Tráfego)](#08---instalação-do-wireshark-e-tcpdump-análise-de-tráfego)
9. [Ferramentas Complementares (mdk4, crunch, macchanger e utilitários de rede)](#09---ferramentas-complementares-mdk4-crunch-macchanger-e-utilitários-de-rede)
10. [Permissões de Usuário — Uso sem `sudo` em Cada Comando](#10---permissões-de-usuário--uso-sem-sudo-em-cada-comando)
11. [Testes de Validação Final do Ambiente](#11---testes-de-validação-final-do-ambiente)
12. [Tabela-Resumo de Equivalência Kali → Mint](#-tabela-resumo-de-equivalência-kali--mint)

---

## 01 - Por que o Linux Mint neste cenário

| Cenário | Kali Linux (Aula 02 original) | Linux Mint 22.3 (este procedimento) |
|---|---|---|
| Repositório principal | `http.kali.org` (kali-rolling) | `archive.ubuntu.com` / mirrors `noble` (Ubuntu 24.04 LTS) + repositórios próprios do Mint |
| Situação na rede do laboratório | 🔴 Bloqueado/restrito | 🟢 Liberado |
| Ferramentas de pentest wireless | Pré-instaladas nos metapacotes `top10`/`kali-linux-default` | **Não vêm pré-instaladas** — precisam ser instaladas manualmente a partir dos repositórios `universe`/`multiverse` do Ubuntu |
| Ambiente gráfico | Xfce | Cinnamon/Xfce/MATE (conforme a edição instalada) |

> 💡 **Ponto-chave desta aula:** todas as ferramentas usadas nas Aulas 06 a 09 (`04-aircrack-ng/*`) — `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`, `reaver`, `pixiewps`, `wash`, `hcxdumptool`, `hcxpcapngtool`, `hashcat`, `mdk3`/`mdk4` — existem como **pacotes independentes nos repositórios do Ubuntu 24.04**, a mesma base do Mint 22.3. Não é necessário nenhum repositório do Kali para reproduzir os procedimentos já documentados.

---

## 02 - Preparação dos Repositórios e Atualização do Sistema

### 🔧 Procedimento

```bash
# Conferir os repositórios ativos (Menu → Preferências → Fontes de Aplicativos, ou via terminal)
cat /etc/apt/sources.list.d/official-package-repositories.list
```

| Componente esperado | Descrição |
|---|---|
| `main` | Pacotes suportados oficialmente pela Canonical/Mint. |
| `restricted` | Drivers e pacotes com licença restrita (ex.: firmware de hardware). |
| `universe` | Pacotes mantidos pela comunidade — **é aqui que está a maior parte das ferramentas de pentest** (`aircrack-ng`, `reaver`, `hashcat`, `mdk4`, `hcxtools`). |
| `multiverse` | Pacotes com restrições de licenciamento/copyright. |

> ⚠️ **Atenção:** no **Gerenciador de Atualizações** do Mint, confirme em **Editar → Preferências → Espelhos** se o espelho selecionado está acessível na rede do laboratório (caso a rede também filtre determinados mirrors). Prefira o mirror oficial `Mint` ou `Ubuntu Archive` liberado pela equipe de TI.

### 🔄 Atualização Completa

```bash
sudo apt update
sudo apt full-upgrade -y
```

| Comando | Descrição |
|---|---|
| `apt update` | Atualiza a lista de pacotes disponíveis nos repositórios `main`, `universe`, `restricted` e `multiverse`. |
| `apt full-upgrade` | Atualiza todos os pacotes já instalados para a versão mais recente disponível nos repositórios liberados na rede. |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `apt update` conclui sem erros de conexão (`Could not resolve` ou `Connection timed out` indicam bloqueio de rede — comunicar à equipe de TI). |
| 2 | `apt policy` confirma que os componentes `universe` e `multiverse` estão habilitados. |

---

## 03 - Instalação da Suíte Aircrack-ng

A suíte **aircrack-ng** (versão 1.7 ou superior no repositório `universe` do Ubuntu 24.04) é a base de todas as Aulas 06 a 09 — `airmon-ng`, `airodump-ng`, `aireplay-ng` e `aircrack-ng` fazem parte do mesmo pacote.

### 🔧 Procedimento

```bash
sudo apt update
sudo apt install -y aircrack-ng wireless-tools
```

### ✅ Testes de Validação

```bash
aircrack-ng --version
airmon-ng
```

| # | Teste |
|---|---|
| 1 | `aircrack-ng --version` retorna a versão instalada (ex.: `1.7` ou mais recente disponível no repositório `noble`). |
| 2 | `airmon-ng` (sem parâmetros) lista as interfaces de rede sem-fio reconhecidas pelo sistema, sem erro de comando não encontrado. |

> 💡 **Dica:** se a versão exibida por `aircrack-ng --version` estiver muito desatualizada em relação ao [repositório oficial do projeto no GitHub](https://github.com/aircrack-ng/aircrack-ng), isso é esperado — o Ubuntu/Mint empacota uma versão estável testada, geralmente uma ou duas versões menores atrás do `master` do projeto. Para fins didáticos das Aulas 06-09, essa diferença não afeta o funcionamento dos comandos documentados.

---

## 04 - Instalação do Reaver, Pixiewps e Wash (Ataques WPS)

Usados na **Aula 07** (`04-aircrack-ng/02-QuebraSenhaWPS-Kali.md`). No Ubuntu/Mint, o `wash` é instalado junto com o pacote `reaver`.

### 🔧 Procedimento

```bash
sudo apt install -y reaver pixiewps
```

| Pacote | Conteúdo |
|---|---|
| `reaver` | Instala `reaver` **e** `wash` (enumeração de APs com WPS ativo). |
| `pixiewps` | Ferramenta de ataque offline **Pixie Dust**, usada em conjunto com o `reaver -K 1` / `-P`. |

### ✅ Testes de Validação

```bash
reaver --help | head -n 5
wash --help | head -n 5
pixiewps --help | head -n 5
```

| # | Teste |
|---|---|
| 1 | Os três comandos retornam a tela de ajuda (`--help`), sem erro de "comando não encontrado". |
| 2 | `wash -i <interface>mon -s` lista APs com WPS ativo (repetir o procedimento da Aula 07, item 03). |

---

## 05 - Instalação do hcxtools / hcxdumptool (Captura e Ataque PMKID)

Usados na **Aula 08**, item 09 (ataque complementar PMKID, sem necessidade de handshake completo).

### 🔧 Procedimento

```bash
sudo apt install -y hcxtools hcxdumptool
```

| Pacote | Conteúdo |
|---|---|
| `hcxdumptool` | Comando `hcxdumptool`, usado para capturar o PMKID diretamente do primeiro pacote EAPOL. |
| `hcxtools` | Conjunto de utilitários de conversão, incluindo o `hcxpcapngtool`, que converte capturas `.cap`/`.pcapng` para o formato `.22000` usado pelo hashcat. |

### ✅ Testes de Validação

```bash
hcxdumptool --version
hcxpcapngtool --version
```

| # | Teste |
|---|---|
| 1 | Ambos os comandos retornam a versão instalada sem erro. |

---

## 06 - Drivers das Antenas Externas (Realtek RTL8188EUS e Atheros AR9271)

### 📶 06.1 — Antena Qualcomm Atheros AR9271 (`ath9k_htc`)

Assim como no Kali, esta antena é reconhecida **nativamente** pelo kernel do Ubuntu/Mint, sem necessidade de driver adicional.

```bash
lsusb
iwconfig
dmesg | grep -i ath9k
```

| Campo | Valor esperado |
|---|---|
| `lsusb` | Identifica o chipset `Qualcomm Atheros AR9271`. |
| `iwconfig` | Lista uma interface `wlan1` (ou similar) já reconhecida. |
| `dmesg \| grep -i ath9k` | Mensagens de carregamento do driver e firmware `ath9k_htc`. |

### 📶 06.2 — Antena Realtek RTL8188EUS (driver DKMS)

Diferente do Kali (que possui o pacote `realtek-rtl8188eus-dkms` no próprio repositório), o Ubuntu/Mint **não mantém esse driver empacotado oficialmente**. É necessário compilar via DKMS a partir do repositório mantido pela própria comunidade aircrack-ng no GitHub (`github.com` está liberado na configuração de rede deste ambiente — ver `<network_configuration>`).

```bash
# Pré-requisitos para compilação do módulo via DKMS
sudo apt install -y dkms build-essential git linux-headers-$(uname -r)

# Clonar o driver mantido pelo projeto aircrack-ng
git clone https://github.com/aircrack-ng/rtl8188eus.git
cd rtl8188eus

# Instalar via DKMS
sudo cp -r . /usr/src/rtl8188eus-5.3.9
sudo dkms add -m rtl8188eus -v 5.3.9
sudo dkms build -m rtl8188eus -v 5.3.9
sudo dkms install -m rtl8188eus -v 5.3.9

sudo reboot
```

| Campo | Descrição |
|---|---|
| `dkms` | Framework que recompila automaticamente o módulo a cada atualização do kernel — essencial após os `full-upgrade` do item 02. |
| `github.com/aircrack-ng/rtl8188eus` | Repositório com o driver corrigido para suportar **modo monitor e injeção de pacotes**, mantido pela mesma equipe do aircrack-ng — alternativa ao driver genérico `r8188eu` do kernel, que não garante suporte completo a injeção. |

```bash
# Após reiniciar, verificar se o módulo carregou corretamente
lsmod | grep 8188eu
dmesg | grep -i 8188eu
iwconfig
```

> ⚠️ **Conflito de driver:** caso a interface trave em modo gerenciado, pode haver conflito com o módulo genérico `r8188eu` já presente no kernel. Para resolver:
> ```bash
> echo "blacklist r8188eu" | sudo tee -a /etc/modprobe.d/realtek-blacklist.conf
> sudo reboot
> ```

> 💡 **Secure Boot:** se o notebook estiver com Secure Boot habilitado, o módulo DKMS compilado localmente não será carregado (módulo não assinado). Desabilite o Secure Boot na BIOS/UEFI antes deste passo, ou assine o módulo manualmente (fora do escopo desta aula).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe **duas** interfaces sem-fio externas distintas (Realtek e Atheros), além da placa interna do notebook, se houver. |
| 2 | Nenhuma mensagem de firmware ausente em `dmesg` para as duas antenas. |
| 3 | `airmon-ng` (item 03) reconhece ambas as antenas antes de habilitar o modo monitor. |

---

## 07 - Instalação do Hashcat (Aceleração por GPU/CPU)

Usado na **Aula 08**, item 08, como alternativa mais rápida ao `aircrack-ng -w` para dicionários maiores (modo `-m 22000`, WPA-PBKDF2-PMKID+EAPOL).

### 🔧 Procedimento

```bash
sudo apt install -y hashcat
```

> 💡 **GPU vs. CPU:** o pacote `hashcat` do repositório já vem com suporte a OpenCL/CUDA. Em notebooks sem GPU dedicada (caso comum em laboratório), o hashcat roda em modo CPU automaticamente — mais lento que com GPU, mas ainda assim funcional para os dicionários do laboratório (`05-wordlist/*.txt`).

```bash
# Verificar os dispositivos de processamento reconhecidos (CPU/GPU)
hashcat -I
```

### ✅ Testes de Validação

```bash
hashcat --version
hashcat -b -m 22000    # benchmark rápido do modo WPA/WPA2/PMKID
```

| # | Teste |
|---|---|
| 1 | `hashcat --version` retorna a versão instalada sem erro. |
| 2 | `hashcat -I` lista ao menos um dispositivo de backend (CPU, no mínimo). |
| 3 | O benchmark do modo `-m 22000` executa sem falha de driver OpenCL. |

---

## 08 - Instalação do Wireshark e Tcpdump (Análise de Tráfego)

Ferramentas adicionadas a este roteiro para reforçar o **entendimento visual e em linha de comando do tráfego capturado** durante as Aulas 06-09 (ex.: abrir os arquivos `.cap` gerados pelo `airodump-ng` diretamente no Wireshark, ou inspecionar pacotes em tempo real via `tcpdump`).

### 🔧 Procedimento — Wireshark

```bash
sudo apt install -y wireshark
```

Durante a instalação, será exibida a pergunta:

> *"Should non-superusers be able to capture packets?"*

➡️ Selecionar **`<Sim>`** — permite capturar pacotes sem precisar rodar o Wireshark como `root` (ver item 10).

### 🔧 Procedimento — Tcpdump

```bash
sudo apt install -y tcpdump
```

| Ferramenta | Uso nas Aulas 06-09 |
|---|---|
| **Wireshark** | Abrir e analisar visualmente os arquivos `.cap`/`.pcapng` gerados pelo `airodump-ng` (Aulas 06, 08, 09) e pelo `hcxdumptool` (Aula 08, item 09) — útil para identificar quadros de *Beacon*, *Probe Request/Response*, *Deauth* e o próprio *4-way handshake* EAPOL visualmente (filtro `eapol` no Wireshark). |
| **Tcpdump** | Captura rápida em linha de comando, sem interface gráfica — útil para conferir em tempo real, via SSH ou terminal remoto, se uma interface está de fato capturando tráfego antes de abrir o Wireshark. |

### ✅ Testes de Validação

```bash
wireshark --version
tcpdump --version
```

| # | Teste |
|---|---|
| 1 | Wireshark abre normalmente pelo menu gráfico ou via `wireshark &`, sem exigir `sudo`. |
| 2 | Abrir um arquivo `chavewpa-01.cap` (gerado na Aula 08) no Wireshark e aplicar o filtro `eapol` — os 4 pacotes do handshake devem aparecer destacados. |
| 3 | `tcpdump -D` lista as interfaces disponíveis para captura, incluindo `wlan0mon`/`wlan1mon` quando o modo monitor estiver ativo. |

> 💡 **Dica didática:** use o Wireshark para *mostrar* visualmente à turma o que o `airodump-ng` já capturou — por exemplo, aplicar o filtro `wlan.fc.type_subtype == 0x08` para ver apenas *Beacon frames*, ou `wlan.fc.type_subtype == 0x0c` para *Deauthentication*, reforçando o conteúdo teórico das Aulas 06 e 08.

---

## 09 - Ferramentas Complementares (mdk4, crunch, macchanger e utilitários de rede)

### 🔧 Procedimento

```bash
sudo apt install -y mdk4 crunch macchanger net-tools wireless-tools iw aptitude
```

| Pacote | Uso |
|---|---|
| `mdk4` | Sucessor do `mdk3` (descontinuado e removido dos repositórios atuais) — mesma finalidade descrita na Aula 07, item 06 (flood de desautenticação/beacon para contornar *lockout* de WPS). A sintaxe de módulos muda ligeiramente em relação ao `mdk3` (ver quadro abaixo). |
| `crunch` | Geração de wordlists customizadas, citado como ferramenta complementar na Aula 08. |
| `macchanger` | Alteração do endereço MAC da interface — útil em discussões sobre *MAC spoofing* e para variar o MAC da placa atacante entre testes. |
| `net-tools` / `wireless-tools` / `iw` | Comandos legados e atuais de rede (`ifconfig`, `iwconfig`, `iw`), usados ao longo de todas as aulas do módulo. |

### ⚠️ Diferença de sintaxe — `mdk3` (Kali) → `mdk4` (Mint/Ubuntu)

| Ação | `mdk3` (Aula 07) | `mdk4` (equivalente no Mint) |
|---|---|---|
| Autenticação flood | `mdk3 wlan0mon a -a <BSSID>` | `mdk4 wlan0mon a -a <BSSID>` |
| Deauth flood direcionado | `mdk3 wlan0mon x -t <BSSID> -n <ESSID> -s 200` | `mdk4 wlan0mon d -c <CANAL> -B <BSSID>` |
| Beacon flood | `mdk3 wlan0mon b -t <BSSID>` | `mdk4 wlan0mon b -c <CANAL>` |

> 💡 **Atenção ao adaptar:** o `mdk4` reorganizou os módulos de ataque (letras/parâmetros diferentes do `mdk3`). Ao reproduzir o procedimento da Aula 07, item 06, consulte `mdk4 --help` e `mdk4 --fullhelp` para confirmar a sintaxe exata da versão instalada antes de repassar o comando à turma.

### ✅ Testes de Validação

```bash
mdk4 --help | head -n 5
crunch --version 2>&1 | head -n 3
macchanger --version
iw --version 2>/dev/null || iw dev
```

| # | Teste |
|---|---|
| 1 | Todos os comandos retornam saída válida, sem erro de "comando não encontrado". |
| 2 | `macchanger -s wlan1` exibe o MAC atual da interface da antena externa. |

---

## 10 - Permissões de Usuário — Uso sem `sudo` em Cada Comando

No Kali, o usuário padrão historicamente tinha acesso facilitado a essas ferramentas. No Mint/Ubuntu, por padrão, capturas de pacotes e mudança de modo de interface exigem privilégios elevados. Para evitar digitar `sudo` antes de cada comando das Aulas 06-09, ajuste as permissões uma única vez:

### 🔧 Procedimento

```bash
# Adicionar o usuário ao grupo wireshark (permite captura sem sudo)
sudo usermod -aG wireshark $USER

# Conceder capacidades de captura de pacotes aos binários do aircrack-ng
sudo setcap cap_net_raw,cap_net_admin=eip $(which airodump-ng)
sudo setcap cap_net_raw,cap_net_admin=eip $(which aireplay-ng)
sudo setcap cap_net_raw,cap_net_admin=eip $(which tcpdump)

# Efetivar a entrada no grupo (logout/login ou reboot)
sudo reboot
```

> ⚠️ **Atenção didática:** mesmo com essas permissões, comandos como `airmon-ng start`, `reaver` e `mdk4` normalmente ainda exigem `sudo`, pois alteram o modo da interface de rede em nível de sistema. O ajuste acima elimina a necessidade de `sudo` **apenas** para captura passiva (`airodump-ng`, `tcpdump`, Wireshark), mantendo a mesma lógica de privilégios já usada nas Aulas 06-09.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Wireshark abre e lista as interfaces de captura sem pedir senha. |
| 2 | `tcpdump -i wlan1mon` inicia a captura sem exigir `sudo` (executado pelo usuário comum, após reboot). |

---

## 11 - Testes de Validação Final do Ambiente

Repita o fluxo completo de uma aula anterior (ex.: reconhecimento passivo da Aula `03-analytics/01-...`) usando somente as ferramentas instaladas neste procedimento, para confirmar que o ambiente Mint está pronto:

```bash
# 1. Finalizar processos conflitantes
sudo airmon-ng check kill

# 2. Habilitar modo monitor na antena externa
sudo airmon-ng start wlan1

# 3. Reconhecimento passivo
sudo airodump-ng wlan1mon
```

### ✅ Checklist de Validação Final

| # | Teste |
|---|---|
| 1 | `aircrack-ng`, `airmon-ng`, `airodump-ng`, `aireplay-ng` executam sem erro. |
| 2 | `reaver`, `wash`, `pixiewps` executam sem erro. |
| 3 | `hcxdumptool`, `hcxpcapngtool` executam sem erro. |
| 4 | `hashcat -b -m 22000` roda um benchmark completo. |
| 5 | `wireshark` abre um `.cap` de exemplo e reconhece o filtro `eapol`. |
| 6 | `tcpdump -D` lista as interfaces, incluindo a de modo monitor. |
| 7 | `mdk4`, `crunch`, `macchanger` executam sem erro. |
| 8 | Antena Atheros AR9271 reconhecida nativamente; antena Realtek RTL8188EUS reconhecida via DKMS compilado do GitHub. |
| 9 | Modo monitor habilitado com sucesso em ao menos uma das duas antenas (`wlan1mon` ou similar). |

---

## ✅ Checklist Final da Aula

- [ ] Repositórios `universe`/`multiverse` do Ubuntu 24.04 confirmados como ativos e acessíveis na rede do laboratório
- [ ] `apt update && apt full-upgrade` executado com sucesso
- [ ] Suíte `aircrack-ng` instalada e validada (`airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`)
- [ ] `reaver`, `pixiewps` e `wash` instalados e validados
- [ ] `hcxtools`/`hcxdumptool` instalados e validados
- [ ] Driver DKMS da antena Realtek RTL8188EUS compilado e carregado (via `github.com/aircrack-ng/rtl8188eus`)
- [ ] Antena Atheros AR9271 reconhecida nativamente (`ath9k_htc`)
- [ ] `hashcat` instalado, com benchmark do modo `-m 22000` validado
- [ ] `wireshark` instalado, com captura sem `sudo` habilitada (grupo `wireshark`)
- [ ] `tcpdump` instalado e testado
- [ ] `mdk4` instalado — sintaxe comparada com o `mdk3` original das Aulas 06/07
- [ ] `crunch`, `macchanger`, `net-tools`, `wireless-tools`, `iw` instalados
- [ ] Permissões (`setcap` + grupo `wireshark`) configuradas para captura sem `sudo`
- [ ] Fluxo completo de reconhecimento passivo (item 11) reproduzido com sucesso no Mint

---

## 📊 Tabela-Resumo de Equivalência Kali → Mint

| Ferramenta | Pacote no Kali (Aula 02 original) | Pacote no Mint 22.3 / Ubuntu `noble` | Observação |
|---|---|---|---|
| aircrack-ng, airmon-ng, airodump-ng, aireplay-ng | `top10`/`kali-linux-default` (pré-instalado) | `aircrack-ng` | Mesmo comando e sintaxe |
| reaver, wash | pré-instalado | `reaver` | `wash` incluso no mesmo pacote |
| pixiewps | pré-instalado | `pixiewps` | Idêntico |
| hcxdumptool, hcxpcapngtool | `hcxtools`/`hcxdumptool` | `hcxtools` + `hcxdumptool` | Idêntico |
| hashcat | pré-instalado | `hashcat` | Pode exigir configuração extra de driver GPU |
| Driver Realtek RTL8188EUS | `realtek-rtl8188eus-dkms` (pacote Kali) | Compilação manual via DKMS (GitHub `aircrack-ng/rtl8188eus`) | **Principal diferença de esforço** entre as duas distros |
| Driver Atheros AR9271 | nativo (`ath9k_htc`) | nativo (`ath9k_htc`) | Idêntico, nenhuma ação extra |
| mdk3 | pré-instalado | `mdk4` (mdk3 descontinuado) | **Sintaxe de módulos diferente** — ver item 09 |
| crunch | pré-instalado | `crunch` | Idêntico |
| Wireshark | não citado na Aula 02 original | `wireshark` | Adicionado neste procedimento |
| Tcpdump | não citado na Aula 02 original | `tcpdump` | Adicionado neste procedimento |

> 📌 **Próxima aula:** com o ambiente Linux Mint 22.3 equipado com as mesmas ferramentas usadas nas Aulas 06-09 (WEP, WPS, WPA/WPA2 e WPA3), os grupos podem reproduzir integralmente os roteiros de `04-aircrack-ng/*` substituindo apenas o sistema operacional da estação de ataque — os comandos, parâmetros e fluxos permanecem os mesmos documentados, com exceção da sintaxe do `mdk3`/`mdk4` (item 09) e da instalação do driver da antena Realtek (item 06).
