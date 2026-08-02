# 🐉 Aula 02 — Instalação, Atualização e Configuração Básica do Kali Linux 2026.2 (Xfce)

> **Módulo:** Redes Sem-Fio — Preparação do Ambiente de Ataque (Kali Linux)
> **Aula:** 02 — Instalação, Atualização e Configuração Básica do Kali Linux
> **Equipamento base (notebook):** Dell Inspiron
> **Antenas Wi-Fi externas homologadas:**
> - USB Realtek — chipset **RTL8188EUS**, driver `r8188eu` / `realtek-rtl8188eus-dkms`, 802.11n
> - USB Qualcomm Atheros — chipset **AR9271**, driver `ath9k_htc`, 802.11n

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 31/07/2026 |
| **Data de atualização** | 31/07/2026 |
| **Versão** | 0.01 |
| **Versão do Kali testada** | Kali Linux **2026.2** — Kernel `6.19`, Xfce `4.20.7` |
| **Notebook testado** | Dell Inspiron (instalação bare metal, UEFI) |
| **Antenas testadas** | Realtek RTL8188EUS e Qualcomm Atheros AR9271 |

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

## 📑 Sumário

1. [Requisitos e Especificações do Ambiente](#01---requisitos-e-especificações-do-ambiente)
2. [Download e Verificação da Imagem ISO](#02---download-e-verificação-da-imagem-iso)
3. [Instalação do Kali Linux 2026.2 (Xfce) no Notebook Dell Inspiron](#03---instalação-do-kali-linux-20262-xfce-no-notebook-dell-inspiron)
4. [Atualização do Sistema e Novo Formato de Repositórios APT](#04---atualização-do-sistema-e-novo-formato-de-repositórios-apt)
5. [Configuração Básica Pós-Instalação](#05---configuração-básica-pós-instalação)
6. [Instalação e Verificação das Antenas Wi-Fi Externas](#06---instalação-e-verificação-das-antenas-wi-fi-externas)
7. [Testes de Validação Final](#07---testes-de-validação-final)

---

## 01 - Requisitos e Especificações do Ambiente

### 💻 Notebook

| Item | Valor |
|---|---|
| Modelo | Dell Inspiron (linha padrão do laboratório) |
| Firmware | UEFI (recomendado desabilitar Secure Boot para simplificar o uso de módulos DKMS de terceiros) |
| Espaço em disco | Mínimo 40 GB livres (recomendado 60 GB+) |
| Memória RAM | Mínimo 4 GB (recomendado 8 GB+) |
| Mídia de instalação | Pendrive USB ≥ 8 GB |

> ⚠️ **Atenção:** em alguns modelos Dell Inspiron, o Secure Boot bloqueia o carregamento de módulos de kernel não assinados (como os drivers DKMS das antenas externas). Recomenda-se **desabilitar o Secure Boot na BIOS/UEFI** antes de seguir para a Aula 03 (uso das antenas).

### 📶 Antenas Wi-Fi Externas

| Antena | Chipset | Driver no Kali | Suporta Modo Monitor / Injeção |
|---|---|---|---|
| USB Realtek | `RTL8188EUS` | `r8188eu` (in-kernel) ou `realtek-rtl8188eus-dkms` (pacote Kali) | ✅ Sim, com o driver `realtek-rtl8188eus-dkms` |
| USB Qualcomm Atheros | `AR9271` | `ath9k_htc` (in-kernel, nativo) | ✅ Sim, nativamente, sem necessidade de driver adicional |

> 💡 **Dica:** a antena **AR9271 (Atheros)** é reconhecida "de fábrica" pelo kernel do Kali, sem instalação extra. Já a **RTL8188EUS (Realtek)** normalmente é reconhecida pelo driver genérico `r8188eu`, mas esse driver **não garante bom suporte a modo monitor e injeção de pacotes** — por isso a Aula 06/07 (aircrack-ng/reaver) depende do pacote específico `realtek-rtl8188eus-dkms`, instalado no item 06 deste procedimento.

---

## 02 - Download e Verificação da Imagem ISO

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar a página oficial de download: https://www.kali.org/get-kali/ |
| 2 | Selecionar **Installer Images → Kali Linux 2026.2 (64-bit, Xfce)** |
| 3 | Copiar o hash **SHA256** exibido junto à imagem |

### ✅ Verificação de Integridade (obrigatório)

```bash
# No terminal, dentro da pasta onde a ISO foi baixada
sha256sum kali-linux-2026.2-installer-amd64.iso
```

| Campo | Descrição |
|---|---|
| `sha256sum` | Gera o hash SHA256 do arquivo baixado, que deve ser **idêntico** ao publicado no site oficial do Kali — garante que a imagem não foi corrompida ou adulterada. |

> ⚠️ **Atenção:** nunca utilize imagens ISO de fontes não oficiais (torrents de terceiros, mirrors não confiáveis). Sempre valide o hash antes de gravar no pendrive.

### 💽 Gravação do Pendrive Bootável

| Sistema | Ferramenta recomendada |
|---|---|
| Windows | [Rufus](https://rufus.ie/) (modo DD/ISO) |
| Linux | `dd` ou [Balena Etcher](https://etcher.balena.io/) |
| macOS | Balena Etcher |

```bash
# Exemplo via dd no Linux (substitua /dev/sdX pelo dispositivo correto do pendrive)
sudo dd if=kali-linux-2026.2-installer-amd64.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

> ⚠️ **Atenção:** confirme corretamente o dispositivo (`/dev/sdX`) com `lsblk` antes de rodar o `dd` — um erro aqui pode apagar o disco errado.

---

## 03 - Instalação do Kali Linux 2026.2 (Xfce) no Notebook Dell Inspiron

### 🔧 Procedimento (Instalação Gráfica Padrão)

| Passo | Ação |
|---|---|
| 1 | Inserir o pendrive bootável e ligar o notebook Dell Inspiron |
| 2 | Acessar o **Boot Menu** (geralmente tecla `F12` nos modelos Dell) e selecionar o pendrive USB |
| 3 | No menu do GRUB da mídia, selecionar **Graphical Install** |
| 4 | Selecionar idioma **Português (Brasil)**, localização **Brasil** e layout de teclado **Português (Brasil)** |
| 5 | Definir **hostname**: `kali-grupo-0X` |
| 6 | Definir **domínio de rede**: deixar em branco (uso em laboratório) |
| 7 | Criar usuário padrão e senha forte |
| 8 | Selecionar fuso horário: **(UTC-03:00) Brasília** |

### 💾 Particionamento

| Campo | Valor | Descrição |
|---|---|---|
| Método de particionamento | `Guiado — utilizar o disco inteiro` | Simplifica a instalação em laboratório, criando automaticamente as partições necessárias (`/`, `swap`, `/boot/efi`). |
| Esquema de partição | `Todos os arquivos em uma partição` | Recomendado para notebooks de uso didático/single-user, mais simples de gerenciar. |

➡️ Confirmar **"Sim"** para gravar as alterações no disco

> ⚠️ **Atenção:** este passo **apaga todo o conteúdo do disco** do notebook. Confirme que não há dados importantes antes de prosseguir.

### 📦 Seleção de Software (Metapacotes)

| Metapacote | Status | Descrição |
|---|---|---|
| `desktop-environment` | 🟢 Selecionado | Instala o ambiente gráfico. |
| `xfce4` | 🟢 Selecionado | Ambiente gráfico **Xfce**, mais leve, indicado para notebooks com recursos mais limitados e para as aulas práticas deste curso. |
| `top10` | 🟢 Selecionado | Conjunto das 10 ferramentas mais usadas em pentest (inclui base do `aircrack-ng`). |
| `default` (kali-linux-default) | 🟢 Selecionado | Conjunto padrão de ferramentas do Kali. |

➡️ Aguardar a instalação do sistema base, dos pacotes e do **GRUB** (instalar no disco principal, ex.: `/dev/sda`)

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Notebook reinicia normalmente após a instalação, exibindo o menu do **GRUB**. |
| 2 | Login gráfico no ambiente **Xfce** realizado com sucesso, com o usuário criado. |
| 3 | Comando `hostnamectl` confirma o hostname definido (`kali-grupo-0X`). |

---

## 04 - Atualização do Sistema e Novo Formato de Repositórios APT

> 💡 A partir do Kali 2026.2, o formato tradicional `/etc/apt/sources.list` foi substituído pelo novo padrão **DEB822** (`/etc/apt/sources.list.d/kali.sources`). Em instalações novas via ISO oficial, esse arquivo já vem correto — o procedimento abaixo serve para **conferir** e, se necessário, corrigir.

### 🔧 Procedimento

```bash
# Conferir o arquivo de repositórios (formato DEB822 no 2026.2)
cat /etc/apt/sources.list.d/kali.sources
```

| Campo esperado | Valor |
|---|---|
| `Types` | `deb` |
| `URIs` | `http://http.kali.org/kali` |
| `Suites` | `kali-rolling` |
| `Components` | `main contrib non-free non-free-firmware` |

### 🔄 Atualização Completa

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

| Comando | Descrição |
|---|---|
| `apt update` | Atualiza a lista de pacotes disponíveis nos repositórios configurados. |
| `apt full-upgrade` | Atualiza todos os pacotes instalados para a versão mais recente, incluindo mudanças de dependências (recomendado no Kali, ao invés de `upgrade`). |
| `reboot` | **Obrigatório** após esta atualização — o Kali 2026.2 traz atualizações disruptivas de `polkit` e `xrdp/xorgxrdp` que exigem reinício para funcionar corretamente. |

> ⚠️ **Atenção:** se o notebook possuir GPU **NVIDIA** com driver DKMS instalado, **não** atualize para o kernel `7.0` via `kali-experimental` — permaneça no kernel `6.19` (padrão do 2026.2), pois há incompatibilidade conhecida entre o kernel 7.0 e os drivers DKMS da NVIDIA até o momento desta atualização.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Comando `uname -r` retorna versão de kernel `6.19.x`. |
| 2 | Comando `apt list --upgradable` não retorna mais pacotes pendentes. |

---

## 05 - Configuração Básica Pós-Instalação

### 🔧 Procedimento

```bash
# Criar usuário não-root de uso diário (o Kali 2026.2 já força a criação de um usuário não-root na instalação)
sudo apt install -y curl wget git vim net-tools

# Habilitar e iniciar o NetworkManager (gerenciamento de rede gráfico no Xfce)
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

| Campo | Valor | Descrição |
|---|---|---|
| Hostname | `kali-grupo-0X` | Identifica o notebook do grupo dentro do laboratório/rede. |
| Fuso horário | `America/Sao_Paulo` | Sincroniza logs e timestamps com o horário de Brasília. |
| Ambiente gráfico | `Xfce 4.20.7` | Ambiente leve, ideal para notebooks com recursos limitados nas aulas práticas. |

### 🕒 Conferindo o Fuso Horário

```bash
timedatectl set-timezone America/Sao_Paulo
timedatectl status
```

### 🔐 Configuração do SSH (opcional, para gerenciamento remoto em laboratório)

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

> ⚠️ **Atenção:** o serviço SSH vem **desabilitado por padrão** no Kali (por segurança). Habilite apenas se o laboratório exigir acesso remoto ao notebook, e sempre com senha forte ou autenticação por chave.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `timedatectl status` confirma fuso horário `America/Sao_Paulo` e hora sincronizada. |
| 2 | Ícone de rede do **NetworkManager** aparece na barra do Xfce. |

---

## 06 - Instalação e Verificação das Antenas Wi-Fi Externas

### 📶 06.1 — Antena Qualcomm Atheros AR9271 (`ath9k_htc`)

Esta antena é reconhecida **nativamente** pelo kernel do Kali, sem necessidade de driver adicional.

```bash
# Conectar a antena na porta USB e verificar reconhecimento
lsusb
iwconfig
dmesg | grep -i ath9k
```

| Campo | Valor esperado | Descrição |
|---|---|---|
| `lsusb` | Identifica fabricante/chipset `Qualcomm Atheros AR9271` | Confirma que o hardware foi detectado na porta USB. |
| `iwconfig` | Lista uma interface `wlan1` (ou similar) | Confirma que o driver `ath9k_htc` carregou a interface de rede sem-fio. |
| `dmesg \| grep -i ath9k` | Mensagens de carregamento do módulo `ath9k_htc` e do firmware | Confirma que o firmware da antena foi carregado com sucesso pelo kernel. |

### 📶 06.2 — Antena Realtek RTL8188EUS (`realtek-rtl8188eus-dkms`)

Esta antena é reconhecida pelo driver genérico `r8188eu`/`8188eu`, mas **para uso em pentest (modo monitor e injeção de pacotes)** é necessário instalar o pacote DKMS específico do Kali.

```bash
# Pré-requisitos para compilação do módulo via DKMS
sudo apt update
sudo apt install -y dkms build-essential linux-headers-$(uname -r)

# Instalar o driver oficial do repositório Kali (contrib)
sudo apt install -y realtek-rtl8188eus-dkms

# Reiniciar para carregar o novo módulo
sudo reboot
```

| Campo | Descrição |
|---|---|
| `dkms` | Framework que recompila automaticamente módulos de kernel de terceiros a cada atualização do kernel — essencial para o driver da Realtek continuar funcionando após updates. |
| `linux-headers-$(uname -r)` | Cabeçalhos do kernel em uso, necessários para compilar o módulo DKMS. |
| `realtek-rtl8188eus-dkms` | Pacote mantido pela equipe Kali (repositório `contrib`), com suporte a **modo monitor** e **injeção de pacotes** para o chipset RTL8188EUS. |

```bash
# Após reiniciar, verificar se o módulo carregou corretamente
lsmod | grep 8188eu
dmesg | grep -i 8188eu
iwconfig
```

> ⚠️ **Conflito de driver:** caso a interface da Realtek não apareça corretamente ou trave em modo gerenciado, pode haver conflito entre o módulo genérico `r8188eu` (in-kernel) e o `8188eu` do pacote DKMS. Para resolver, bloqueie o driver in-kernel:
> ```bash
> echo "blacklist r8188eu" | sudo tee -a /etc/modprobe.d/realtek-blacklist.conf
> sudo reboot
> ```

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe **duas** interfaces sem-fio distintas (ex.: `wlan0` Realtek e `wlan1` Atheros), além da placa Wi-Fi interna do notebook (se houver). |
| 2 | Nenhuma mensagem de erro/firmware ausente aparece em `dmesg` para as duas antenas. |
| 3 | Ambas as antenas aparecem listadas ao rodar `airmon-ng` (sem ainda habilitar o modo monitor, apenas para conferência). |

> 💡 **Dica:** identifique e anote qual interface (`wlan0`, `wlan1`, etc.) corresponde a cada antena — essa informação será usada nas Aulas 06 e 07 (`airmon-ng start <interface>`), evitando confundir a placa Wi-Fi interna do notebook com as antenas externas de teste.

---

## 07 - Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Sistema inicia em **Xfce 4.20.7**, com login gráfico funcionando normalmente. |
| 2 | `uname -r` confirma kernel `6.19.x` e `apt full-upgrade` não retorna mais pacotes pendentes. |
| 3 | Fuso horário `America/Sao_Paulo` confirmado via `timedatectl`. |
| 4 | Antena **AR9271** reconhecida nativamente (`ath9k_htc`), sem necessidade de driver adicional. |
| 5 | Antena **RTL8188EUS** reconhecida com o driver `realtek-rtl8188eus-dkms`, pronta para modo monitor. |
| 6 | Notebook conectado à rede do laboratório (Wi-Fi interna ou cabeada) para fins de atualização e acesso à Internet. |

---

## ✅ Checklist Final da Aula

- [ ] Imagem ISO do Kali Linux 2026.2 (Xfce) baixada e hash SHA256 validado
- [ ] Pendrive bootável gravado corretamente
- [ ] Secure Boot desabilitado na BIOS/UEFI do notebook Dell Inspiron (recomendado)
- [ ] Instalação gráfica concluída (idioma, hostname, fuso horário, usuário e senha definidos)
- [ ] Particionamento guiado (disco inteiro) realizado com sucesso
- [ ] Metapacotes `xfce4`, `top10` e `default` selecionados na instalação
- [ ] `apt update && apt full-upgrade` executado com sucesso e sistema reiniciado
- [ ] Fuso horário `America/Sao_Paulo` configurado e validado
- [ ] NetworkManager habilitado e funcionando no Xfce
- [ ] Antena Atheros AR9271 reconhecida nativamente (`ath9k_htc`)
- [ ] Pacote `realtek-rtl8188eus-dkms` instalado e antena Realtek RTL8188EUS reconhecida
- [ ] Interfaces de cada antena identificadas e anotadas (`wlan0`, `wlan1`, etc.)
- [ ] `airmon-ng` reconhece ambas as antenas externas sem erros

> 📌 **Próxima aula:** configuração dos Access Points TP-Link (SSID, segurança, WPS) e, em seguida, testes de invasão wireless com `airmon-ng`, `airodump-ng`, `aircrack-ng` e `reaver` utilizando as antenas configuradas nesta aula.
