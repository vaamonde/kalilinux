# 🦜 Aula 02c — Instalação, Atualização e Configuração Básica do Parrot Security OS 7.3 (Xfce)

> **Módulo:** Redes Sem-Fio — Preparação do Ambiente de Ataque (Parrot Security OS)
> **Aula:** 02c — Instalação, Atualização e Configuração Básica do Parrot Security OS
> **Equipamento base (notebook):** Dell Inspiron e Latitude
> **Antenas Wi-Fi externas homologadas:**
> - USB Realtek — chipset **RTL8188EUS**, driver `r8188eu` / DKMS `aircrack-ng/rtl8188eus`, 802.11n
> - USB Qualcomm Atheros — chipset **AR9271**, driver `ath9k_htc`, 802.11n
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|-------|-----------|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 05/08/2026 |
| **Data de atualização** | 19/08/2026 |
| **Versão** | 0.02 |
| **Versão do Parrot testada** | Parrot Security OS **7.3** — instalador Calamares (base **Parrot 7.2 "Echo"**), Xfce |
| **Notebook testado** | Dell Inspiron e Latitude (instalação bare metal, UEFI) |
| **Antenas testadas** | Realtek RTL8188EUS e Qualcomm Atheros AR9271 |
| **Motivo da inclusão desta distro** | Alternativa ao Kali Linux (Aula 02) e ao Linux Mint (Aula 02b) para os grupos/notebooks em que o Parrot Security já é a distribuição padrão fornecida pela instituição — mantém as mesmas ferramentas de pentest wireless usadas nas Aulas 06-10 (`04-aircrack-ng/*`), a maioria já **pré-instalada** na edição Security. |

### 🔗 Links do Autor

| Canal | Link |
|-------|------|
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
> A utilização das técnicas aqui descritas contra redes de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de responsabilidade civil (Código Civil, arts. 927 a 943).
>
> Cada grupo deve atacar **somente** o Access Point que lhe foi atribuído no laboratório.
---

## 📑 Sumário

1. [Requisitos e Especificações do Ambiente](#01---requisitos-e-especificações-do-ambiente)
2. [Download e Verificação da Imagem ISO](#02---download-e-verificação-da-imagem-iso)
3. [Instalação do Parrot Security OS 7.3 (Calamares) no Notebook Dell Inspiron](#03---instalação-do-parrot-security-os-73-calamares-no-notebook-dell-inspiron)
4. [Atualização do Sistema (APT)](#04---atualização-do-sistema-apt)
5. [Configuração Básica Pós-Instalação](#05---configuração-básica-pós-instalação)
6. [Verificação das Ferramentas de Pentest Wireless (Pré-instaladas no Parrot Security)](#06---verificação-das-ferramentas-de-pentest-wireless-pré-instaladas-no-parrot-security)
7. [Instalação e Verificação das Antenas Wi-Fi Externas](#07---instalação-e-verificação-das-antenas-wi-fi-externas)
8. [Testes de Validação Final](#08---testes-de-validação-final)
9. [Tabela-Resumo de Equivalência Kali → Parrot](#-tabela-resumo-de-equivalência-kali--parrot)
---

## 01 - Requisitos e Especificações do Ambiente

### 💻 Notebook Dell Inspiron e Latitude

| Item | Valor |
|------|-------|
| **Modelo** | `Dell Inspiron e Latitude` (linha padrão do laboratório) |
| **Firmware** | `UEFI` (recomendado desabilitar `Secure Boot` para simplificar o uso de módulos DKMS de terceiros) |
| **Espaço em disco** | `Mínimo 40 GB livres` (recomendado 60 GB+) |
| **Memória RAM** | `Mínimo 4 GB` (recomendado 8 GB+) |
| **Mídia de instalação** | `Pendrive USB ≥ 8 GB` |
---

> ⚠️ **Atenção:** assim como no Kali (Aula 02), em alguns modelos Dell Inspiron o Secure Boot bloqueia o carregamento de módulos de kernel não assinados (drivers DKMS das antenas externas). Recomenda-se **desabilitar o Secure Boot na BIOS/UEFI** antes de seguir para a Aula 03 (uso das antenas).

### 📶 Antenas Wi-Fi Externas

| Antena | Chipset | Driver no Parrot | Suporta Modo Monitor / Injeção |
|--------|---------|------------------|--------------------------------|
| **USB Realtek** | `RTL8188EUS` | `r8188eu` (in-kernel) ou DKMS `aircrack-ng/rtl8188eus` (compilação manual, ver item 07) | ✅ Sim, com o driver DKMS |
| **USB Qualcomm Atheros** | `AR9271` | `ath9k_htc` (in-kernel, nativo) | ✅ Sim, nativamente, sem necessidade de driver adicional |
---

> 💡 **Dica:** assim como no Kali, a antena **AR9271 (Atheros)** é reconhecida "de fábrica" pelo kernel do Parrot, sem instalação extra. A **RTL8188EUS (Realtek)** normalmente é reconhecida pelo driver genérico `r8188eu`, mas esse driver **não garante bom suporte a modo monitor e injeção de pacotes** — recomenda-se o driver DKMS mantido pela própria equipe do aircrack-ng (item 07).
---

## 02 - Download e Verificação da Imagem ISO do Parrot

| Passo | Ação |
|-------|------|
| 1 | Acessar a página oficial de download: https://www.parrotsec.org/download/ |
| 2 | Selecionar **Parrot Security 7.3 (64-bit)** |
| 3 | Copiar o hash **SHA256** exibido junto à imagem |
---

### ✅ Verificação de Integridade (obrigatório)

```bash
# No terminal, dentro da pasta onde a ISO foi baixada utilizando o Linux
sha256sum Parrot-security-7.3_amd64.iso
```

| Campo | Descrição |
|-------|-----------|
| `sha256sum` | Gera o hash SHA256 do arquivo baixado, que deve ser **idêntico** ao publicado no site oficial do Parrot — garante que a imagem não foi corrompida ou adulterada. |
---

> ⚠️ **Atenção:** nunca utilize imagens ISO de fontes não oficiais (torrents de terceiros, mirrors não confiáveis). Sempre valide o hash antes de gravar no pendrive.

### 💽 Gravação do Pendrive Bootável

| Sistema | Ferramenta recomendada |
|---------|------------------------|
| Windows | [Rufus](https://rufus.ie/) (modo DD/ISO) |
| Linux | `dd` ou [Balena Etcher](https://etcher.balena.io/) |
| macOS | Balena Etcher |
---

```bash
# Exemplo via dd no Linux (substitua /dev/sdX pelo dispositivo correto do pendrive)
sudo dd if=Parrot-security-7.3_amd64.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

> ⚠️ **Atenção:** confirme corretamente o dispositivo (`/dev/sdX`) com `lsblk` antes de rodar o `dd` — um erro aqui pode apagar o disco errado.
---

## 03 - Instalação do Parrot Security OS 7.3 (Calamares) no Notebook Dell Inspiron e Latitude 

Diferente do instalador do Kali (Debian Installer, em modo texto/gráfico próprio), o Parrot utiliza o **Calamares**, um instalador gráfico que roda a partir do próprio ambiente live (Xfce) do pendrive. O procedimento abaixo documenta, tela por tela, a instalação realizada no notebook Dell Inspiron e Latitude do laboratório.

### 🔧 Procedimento (Instalação Gráfica Padrão)

| # | Tela | Ação |
|---|------|------|
| 01 | Tela de boot (GRUB da mídia live) | Selecionar **`Try / Install`** |
| 02 | Tela de Login do ambiente live | Clicar em **`Log In`** |
| 03 | Ambiente gráfico Xfce (live) | Clicar no atalho **`Install Parrot`** na área de trabalho |
| 04 | *Welcome to the Calamares installer for Parrot 7.2 (Echo)* | Selecionar idioma **`Português (Brasil)`** → clicar em **`<Próximo>`** |
| 05 | Localização | Região: **`America`** · Área: **`São Paulo`** → clicar em **`<Próximo>`** |
| 06 | Teclado | Modelo: **`Portuguese (Brasil) - Default`** → clicar em **`<Próximo>`** |
| 07 | Partições | Selecionar o dispositivo de armazenamento (HD/SSD) · marcar **`Formatar o disco`** · **`Sem Swap`** → clicar em **`<Próximo>`** |
| 08 | Usuários | Ver tabela detalhada abaixo → clicar em **`<Próximo>`** |
| 09 | Resumo | Clicar em **`<Instalar>`** → confirmar **`<Instalar Agora>`** → aguardar a instalação |
| 10 | Concluir | Clicar em **`<Concluído>`** |
| 11 | — | O sistema reinicializa automaticamente |
| 12 | Tela de Boot (GRUB do disco) | Selecionar **`Parrot OS 7 GNU/Linux`** |
| 13 | Tela de Login | Selecionar o usuário **`senac`** e autenticar |
---

### 👤 Detalhamento da Tela 08 — Criação de Usuário

| Campo do Calamares | Valor | Descrição |
|--------------------|-------|-----------|
| **Qual é o seu nome?** | `SENAC Lapa Tito` | Nome completo exibido no perfil do usuário (não é o login). |
| **Qual nome você quer usar para entrar?** | `senac` | *Username* usado no login e no prompt do terminal (`senac@grupo-0x`). |
| **Qual é o nome deste computador?** | `grupo-0x` | *Hostname* do notebook — **substituir `0x` pelo número real do grupo** (ex.: `grupo-01`), mesma convenção já usada no hostname dos Access Points (Aula 02, `ap-grupo-0X`). |
| **Escolha uma senha para manter a sua conta segura** | `Senactit@123` (repetir na confirmação) | Senha do usuário `senac`, também usada para elevação de privilégios via `sudo`. |
---

> ⚠️ **Atenção:** a formatação do disco (tela 07) **apaga todo o conteúdo existente**. Confirme que não há dados importantes no notebook antes de prosseguir. A opção **Sem Swap** é adequada para notebooks de laboratório com RAM suficiente (8 GB+); em máquinas com menos memória, avalie criar uma partição de swap.

> 💡 **Nota sobre a versão exibida no instalador:** a tela de boas-vindas do Calamares referencia **"Parrot 7.2 (Echo)"**, mesmo instalando a imagem **7.3** — isso é esperado, pois o Calamares herda o *codename* da base do sistema (Echo) e a numeração `7.3` refere-se ao *build* da ISO/lançamento incremental, não a uma nova *release* de codename. Registre a versão exata com o comando `cat /etc/os-release` após a instalação (ver item 04).

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | Notebook reinicia normalmente após a instalação, exibindo o menu do **GRUB** com a entrada `Parrot OS 7 GNU/Linux`. |
| 2 | Login gráfico realizado com sucesso com o usuário `senac`. |
| 3 | Comando `hostnamectl` confirma o hostname definido (`grupo-0x`). |
---

## 04 - Atualização do Sistema (APT)

O Parrot é baseado em **Debian testing**, com repositórios próprios (`deb.parrot.sh`). Assim como no Kali, a atualização completa é obrigatória logo após a instalação, antes de qualquer uso em laboratório.

```bash
# Conferir a versão instalada e o codename da base do Parrot
# opção do comando cat: -n (number all output lines)
sudo cat -n /etc/os-release

# Conferir os repositórios ativos do Parrot
# opção do comando cat: -n (number all output lines)
sudo cat -n /etc/apt/sources.list

# Atualizando as lista do Apt
# opção do comando apt: update (Used to re-synchronize the package index files from their sources. )
sudo apt update

# Atualizando todos os software do Parrot
# opções do comando apt: -y (Automatic yes to prompts), upgrade (Used to install the newest versions of all 
# packages currently installed on the system from the sources enumerated in /etc/apt/sources.list(5).)
sudo apt upgrade -y 

# Atualizando todos os softwares e o sistema operacional do Parrot
# opções do comando apt: -y (Automatic yes to prompts), full-upgrade (full-upgrade performs the function of 
# upgrade but will remove currently installed packages if this is needed to upgrade the system as a whole. )
sudo apt full-upgrade -y

# Atualizando a Distribuição e o Kernel do Parrot
# opções do comando apt: -y (Automatic yes to prompts), dist-upgrade (In addition to performing the function of upgrade)
sudo apt dist-upgrade -y

# Forçando a remoção dos softwares desnecessários do Parrot
# opções do comando apt: -y (Automatic yes to prompts), autoremove (autoremove is used to remove packages that 
# were automatically installed to satisfy dependencies for other packages)
sudo apt autoremove -y

# Limpando todo o repositório e cache do Parrot
# opções do comando apt: -y (Automatic yes to prompts), autoclean (Like clean, autoclean clears out the local repository
# of retrieved package files.)
sudo apt autoclean -y

# Reiniciando o sistema para aplicar as mudanças no Parrot
sudo reboot
```

| Comando | Descrição |
|---------|-----------|
| `apt update` | Atualiza a lista de pacotes disponíveis nos repositórios oficiais do Parrot (`deb.parrot.sh`). |
| `apt upgrade -y` | Atualiza todos os pacotes já instalados para a versão mais recente disponível — mesmo comando informado no material do procedimento original desta aula. |
| `reboot` | Recomendado após a atualização, especialmente se houver atualização de kernel. |
---

> 💡 **`apt upgrade` x `apt full-upgrade`:** o procedimento original desta aula utiliza `apt upgrade`, suficiente na maioria dos casos. Caso a atualização indique pacotes retidos (*kept back*) — comum quando há mudança de dependências entre versões do kernel/drivers —, utilize `sudo apt full-upgrade -y` para resolvê-los, mesma recomendação já aplicada ao Kali (Aula 02, item 04).

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | `apt update` conclui sem erros de conexão (`Could not resolve` ou `Connection timed out` indicam bloqueio de rede — comunicar à equipe de TI, mesmo procedimento de checagem já usado na Aula 02b, item 02). |
| 2 | `apt list --upgradable` não retorna mais pacotes pendentes após o `upgrade`/`full-upgrade`. |
| 3 | `uname -r` exibe a versão de kernel corrente, a ser registrada no relatório de homologação. |
---

## 05 - Configuração Básica Pós-Instalação do Parrot Linux

```bash
# Instalação de pacotes extras para facilitar o reconhecimento da rede sem-fio
# opções do comando apt: -y (Automatic yes to prompts), install (This option is followed by one or more 
# packages desired for installation.)
sudo apt install -y curl wget git vim net-tools wireless-tools inxi

# Configurando o Fuso Horário da America / São Paulo
# opção do comando timedatectl: set-timezone (Set the system time zone to the specified value)
sudo timedatectl set-timezone "America/Sao_Paulo"

# Verificando o Fuso Horário, Data e Hora
# opção do comando timedatectl: status (Show current settings of the system clock and RTC)
sudo timedatectl status

# Habilitar e iniciar o NetworkManager (gerenciamento de rede gráfico no Xfce)
# opções do comando systemctl: enable (Enable one or more units or unit instances), start (Start (activate) one or 
# more units specified on the command line), status (Show terse runtime status information about one or more units)
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
```

| Campo | Valor | Descrição |
|-------|-------|-----------|
| Hostname | `grupo-0x` | Já definido na instalação (tela 08) — confirmar com `hostnamectl`. |
| Fuso horário | `America/Sao_Paulo` | Sincroniza logs e timestamps com o horário de Brasília (a tela 05 do instalador já define a localização, mas vale confirmar via `timedatectl`). |
| Ambiente gráfico | `Xfce` | Ambiente padrão da edição Security do Parrot, leve e adequado para notebooks com recursos limitados. |
---

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | `timedatectl status` confirma fuso horário `America/Sao_Paulo` e hora sincronizada. |
| 2 | Ícone de rede do **NetworkManager** aparece na barra do Xfce. |
| 3 | `hostnamectl` confirma o hostname `grupo-0x` definido na instalação. |
---

## 06 - Verificação das Ferramentas de Pentest Wireless (Pré-instaladas no Parrot Security)

Diferente do Linux Mint (Aula 02b), onde as ferramentas precisam ser instaladas manualmente a partir dos repositórios `universe`/`multiverse` do Ubuntu, a edição **Parrot Security** já traz **pré-instalada** praticamente toda a suíte usada nas Aulas 06-10 (`04-aircrack-ng/*`) — o mesmo cenário do Kali (Aula 02), onde os metapacotes `top10`/`default` já cobrem essas ferramentas.

```bash
# Suíte aircrack-ng (airmon-ng, airodump-ng, aireplay-ng, aircrack-ng)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list aircrack-ng

# Reaver (wash) e pixiewps(ataques WPS — Aula 07)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list reaver
sudo dpkg --list pixiewps

# Hcxdumptool / hcxtools(ataque PMKID — Aula 08, item 09)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list hcxdumptool
sudo dpkg --list hcxtools

# Hashcat (aceleração por GPU/CPU — Aula 08, item 08)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list hashcat

# hydra (força bruta em WebGUI — Aula 10)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list hydra

# Wireshark e tcpdump (análise de tráfego)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list wireshark
sudo dpkg --list tcpdump

# mdk4, crunch e macchanger (ferramentas complementares)
# opção do comando dpkg: --list (List packages matching given pattern)
sudo dpkg --list mdk4
sudo dpkg --list crunch
sudo dpkg --list macchanger
```

## 07 - Instalação e Verificação das Antenas Wi-Fi Externas no Parrot 

### 📶 07.1 — Antena Qualcomm Atheros AR9271 (`ath9k_htc`) e Realtek RTL8188EUS (`rtl8188eus`)

Essas antenas é reconhecidas **nativamente** pelo kernel do Parrot, sem necessidade de driver adicional — mesmo comportamento do Kali e do Mint.

```bash
# Desativando o recurso de renomear os dispositivos de Rede com Base na Identificação da BIOS

# Alterar as configuração do arquivo do GRUB adicionando as opções de desativar o BIOS DevName
# opção do comando sed: -i (Edita o arquivo diretamente (in-place)),
# ^GRUB_CMDLINE_LINUX="\(.*\)" = Captura o conteúdo atual entre aspas da variável (ex.: quiet splash).
# \1 = Reinsere o conteúdo capturado (preserva quiet splash).
# net.ifnames=0 biosdevname=0 = Adiciona as novas opções ao final, mantendo o que já existia.
sudo sed -i 's/^GRUB_CMDLINE_LINUX="\(.*\)"/GRUB_CMDLINE_LINUX="\1 net.ifnames=0 biosdevname=0"/' /etc/default/grub

# Atualizando o GRUB com as novas opções
sudo update-grub

# Reiniciar a máquina para testar as configurações de Placa de Rede
sudo reboot
```

```bash
# Conectar a antena na porta USB e verificar reconhecimento dele com os comandos básicos

# Listando todos os dispositivos conectados na Porta USB no Parrot 
# opção do comando grep: -i (Ignore case distinctions in patterns and input data)
sudo lsusb | grep -i Atheros
sudo lsusb | grep -i Realtek

# Listando todas as interfaces de rede Sem-Fio no Parrot
sudo iwconfig

# Verificando as mensagens do Kernel referente ao Driver da Placa de Rede no Parrot
# opção do comando grep: -i (Ignore case distinctions in patterns and input data)
sudo dmesg | grep -i ath9k
sudo dmesg | grep -i rtl81
```

| Campo | Valor esperado | Descrição |
|-------|----------------|-----------|
| `lsusb` | Identifica fabricante/chipset `Qualcomm Atheros AR9271` - `Realtek RTL8188EUS` | Confirma que o hardware foi detectado na porta USB. |
| `iwconfig` | Lista uma interface `wlan1` (ou similar) | Confirma que o driver `ath9k_htc` - `rtl8188eus` carregou a interface de rede sem-fio. |
| `dmesg \| grep -i ath9k \| grep -i rtl81` | Mensagens de carregamento do módulo `ath9k_htc` - `rtl8188eus` e do firmware | Confirma que o firmware da antena foi carregado com sucesso pelo kernel. |
---

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | `iwconfig` exibe **duas** interfaces sem-fio externas distintas (Realtek e Atheros), além da placa interna do notebook, se houver. |
| 2 | Nenhuma mensagem de firmware ausente em `dmesg` para as duas antenas. |
| 3 | `airmon-ng` reconhece ambas as antenas antes de habilitar o modo monitor. |
---

> 💡 **Dica:** identifique e anote qual interface (`wlan0`, `wlan1`, etc.) corresponde a cada antena — essa informação será usada nas Aulas 06 a 10 (`airmon-ng start <interface>`), evitando confundir a placa Wi-Fi interna do notebook com as antenas externas de teste.
---

## 08 - Testes de Validação Final do Aircrack-NG no Parrot

Repita o fluxo completo de uma aula anterior (ex.: reconhecimento passivo da Aula `03-analytics/01-...`) usando somente o Parrot Security instalado neste procedimento, para confirmar que o ambiente está pronto:

```bash
# 1. Listando os processos conflitantes do gerenciador de Redes
# opção do comando airmon-ng: check (List all possible programs that could interfere with the wireless card)
sudo airmon-ng check

# 2. Finalizar processos conflitantes do gerenciador de Redes
# opções do comando airmon-ng: check (List all possible programs that could interfere with the wireless card), 
# kill (If 'kill' is specified, it will try to kill all of them)
sudo airmon-ng check kill

# 3. Habilitar modo monitor na antena externa USB
# opção do comando airmon-ng: start (Enable  monitor  mode on an interface (and specify a channel))
sudo airmon-ng start wlan1

# 4. Listando todas as interfaces de rede Sem-Fio no Parrot
# Verificar a opção: Mode:Monitor (Modo Monitor) da Interface WLAN1
sudo iwconfig

# 5. Reconhecimento passivo das redes sem-fio
sudo airodump-ng wlan1
```

### ✅ Checklist de Validação Final

| # | Teste |
|---|-------|
| 1 | Sistema inicia em **Xfce**, com login gráfico do usuário `senac` funcionando normalmente. |
| 2 | `apt update && apt upgrade` (ou `full-upgrade`) executado com sucesso e sem pacotes pendentes. |
| 3 | Fuso horário `America/Sao_Paulo` confirmado via `timedatectl`. |
| 4 | `aircrack-ng`, `airmon-ng`, `airodump-ng`, `aireplay-ng` executam sem erro. |
| 5 | `reaver`, `wash`, `pixiewps` executam sem erro. |
| 6 | `hcxdumptool`, `hcxpcapngtool` executam sem erro. |
| 7 | `hashcat -b -m 22000` roda um benchmark completo. |
| 8 | `hydra -h` executa sem erro. |
| 9 | `wireshark` abre um `.cap` de exemplo e reconhece o filtro `eapol`; `tcpdump -D` lista as interfaces disponíveis. |
| 10 | `mdk4`, `crunch`, `macchanger` executam sem erro. |
| 11 | Antena Atheros AR9271 reconhecida nativamente; antena Realtek RTL8188EUS reconhecida via DKMS compilado do GitHub. |
| 12 | Modo monitor habilitado com sucesso em ao menos uma das duas antenas (`wlan1mon` ou similar), com o `airodump-ng` listando os APs do laboratório. |
---

## ✅ Checklist Final da Aula

- [ ] Imagem ISO do Parrot Security 7.3 baixada e hash SHA256 validado
- [ ] Pendrive bootável gravado corretamente
- [ ] Secure Boot desabilitado na BIOS/UEFI do notebook Dell Inspiron (recomendado)
- [ ] Instalação via Calamares concluída (idioma PT-BR, localização São Paulo, teclado PT-BR)
- [ ] Particionamento com formatação do disco e sem swap realizado com sucesso
- [ ] Usuário `senac` criado, hostname `grupo-0x` definido e senha `Senactit@123` configurada
- [ ] Sistema reiniciado e login gráfico validado com o usuário `senac`
- [ ] `apt update && apt upgrade` (ou `full-upgrade`) executado com sucesso
- [ ] Fuso horário `America/Sao_Paulo` configurado e validado
- [ ] NetworkManager habilitado e funcionando no Xfce
- [ ] Ferramentas de pentest wireless (`aircrack-ng`, `reaver`, `pixiewps`, `hcxtools`, `hashcat`, `hydra`, `wireshark`, `tcpdump`, `mdk4`, `crunch`, `macchanger`) conferidas e, se necessário, instaladas via `apt`
- [ ] Antena Atheros AR9271 reconhecida nativamente (`ath9k_htc`)
- [ ] Driver DKMS da antena Realtek RTL8188EUS compilado e carregado (via `github.com/aircrack-ng/rtl8188eus`)
- [ ] Interfaces de cada antena identificadas e anotadas (`wlan0`, `wlan1`, etc.)
- [ ] `airmon-ng` reconhece ambas as antenas externas sem erros
- [ ] Fluxo completo de reconhecimento passivo (item 08) reproduzido com sucesso no Parrot
---

## 📊 Tabela-Resumo de Equivalência Kali → Parrot

| Item | Kali Linux (Aula 02) | Linux Mint 22.3 (Aula 02b) | Parrot Security 7.3 (esta aula) |
|---|---|---|---|
| Instalador | Debian Installer (gráfico) | Ubiquity/Calamares (conforme ISO) | **Calamares** (a partir do ambiente live Xfce) |
| Repositório principal | `http.kali.org` (kali-rolling) | `archive.ubuntu.com` (Ubuntu 24.04 `noble`) | `deb.parrot.sh` (base Debian testing) |
| Ferramentas de pentest wireless | Pré-instaladas (`top10`/`default`) | **Não** pré-instaladas — instalação manual via `universe`/`multiverse` | **Pré-instaladas** na edição Security (mesmo cenário do Kali) |
| Ambiente gráfico | Xfce | Cinnamon/Xfce/MATE | Xfce |
| Driver Realtek RTL8188EUS | `realtek-rtl8188eus-dkms` (pacote pronto) | Compilação manual via DKMS (GitHub) | Compilação manual via DKMS (GitHub) — mesmo procedimento do Mint |
| Driver Atheros AR9271 | nativo (`ath9k_htc`) | nativo (`ath9k_htc`) | nativo (`ath9k_htc`) |
| mdk3/mdk4 | pré-instalado (`mdk3` ou `mdk4`, conforme versão) | `mdk4` (mdk3 descontinuado) | `mdk4` (a conferir/instalar via `apt`) |
| Comando de atualização | `apt update && apt full-upgrade` | `apt update && apt full-upgrade` | `apt update && apt upgrade` (ou `full-upgrade` se houver pacotes retidos) |
---

> 📌 **Próxima aula:** com o notebook em Parrot Security 7.3 equipado com as mesmas ferramentas usadas nas Aulas 06-10 (WEP, WPS, WPA/WPA2, WPA3 e força bruta em WebGUI), os grupos podem reproduzir integralmente os roteiros de `04-aircrack-ng/*` substituindo apenas o sistema operacional da estação de ataque — os comandos, parâmetros e fluxos permanecem os mesmos documentados, com exceção da instalação do driver da antena Realtek (item 07), que segue o mesmo caminho manual já descrito para o Linux Mint (Aula 02b).
