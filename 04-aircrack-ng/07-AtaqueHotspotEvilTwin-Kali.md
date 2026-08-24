# 📡 Aula 11 — Testes de Invasão Wireless: Ataques a Hotspot / Rede de Convidados (Evil Twin, Karma e Captive Portal Falso) com Kali Linux

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 11 — Rogue AP, Evil Twin, Karma Attack e Captive Portal Falso contra a Rede de Convidados
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (Rede de Convidados configurada pelos alunos na Aula 04, item 05)
> **Estação de ataque:** Kali Linux (ou Linux Mint 22.3, conforme Aula 02b) + placa de rede wireless compatível com modo monitor/injeção + segunda interface (opcional, para o AP falso)

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 21/08/2026 |
| **Data de atualização** | 21/08/2026 |
| **Versão** | 0.01 |
| **Status** | ⚠️ Roteiro **ainda não testado/validado** em ambiente real de laboratório |
| **Pré-requisito** | Aulas 01-10 concluídas (configuração do AP, WEP, WPS, WPA/WPA2/WPA3 e força bruta na WebGUI) — em especial a Aula 04, item 05 (Rede de Convidados) e a Aula 08, item 05 (`aireplay-ng -0`, reaproveitado aqui) |
| **Escopo** | Ataques que **não** miram a chave Wi-Fi diretamente (diferente das Aulas 06-09), e sim a **camada de confiança do usuário** e o **modelo de autenticação do hotspot/rede de convidados** (SSID aberto, captive portal, cadastro por MAC) |
| **Natureza da aula** | Mista — prática (Evil Twin, deauth, MAC spoofing) e conceitual/demonstrativa (Captive Portal falso e Karma Attack, tratados com foco em reconhecimento e defesa, não em coleta real de credenciais de terceiros) |

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
> A utilização das técnicas aqui descritas contra redes, portais ou dispositivos de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de crimes específicos de **interceptação de comunicações** (Lei nº 9.296/1996) e de **proteção de dados pessoais** (LGPD — Lei nº 13.709/2018, especialmente quanto à captura não autorizada de credenciais).
>
> Cada grupo deve atacar **somente** o Access Point/Hotspot que lhe foi atribuído no laboratório, com dispositivos-cliente de teste também pertencentes ao próprio grupo — **nunca** contra dispositivos de terceiros presentes no mesmo ambiente físico.
>
> ⚠️ **Atenção redobrada nesta aula:** diferente das Aulas 06-10, os ataques aqui descritos (Evil Twin, Karma, Captive Portal falso) têm como alvo **pessoas**, não apenas protocolos — reforce com a turma que qualquer captura de credenciais deve ocorrer **apenas** com dispositivos e contas de teste criadas para a aula, nunca com contas reais dos próprios alunos ou de terceiros.

---

## 📑 Sumário

1. [Contextualização — Hotspot como Vetor de Ataque](#01---contextualização--hotspot-como-vetor-de-ataque)
2. [Preparação do Ambiente e Modo Monitor](#02---preparação-do-ambiente-e-modo-monitor)
3. [Reconhecimento do Hotspot/Rede de Convidados Alvo](#03---reconhecimento-do-hotspotrede-de-convidados-alvo)
4. [Ataque Evil Twin — Clonagem do AP](#04---ataque-evil-twin--clonagem-do-ap)
5. [Forçando a Migração de Clientes — Deauth Combinado](#05---forçando-a-migração-de-clientes--deauth-combinado)
6. [Captive Portal Falso — Reconhecimento e Defesa](#06---captive-portal-falso--reconhecimento-e-defesa)
7. [Karma Attack — Resposta a Probe Requests](#07---karma-attack--resposta-a-probe-requests)
8. [Bypass de Autenticação por MAC (MAC Spoofing)](#08---bypass-de-autenticação-por-mac-mac-spoofing)
9. [Vetor Complementar — Interceptação Pós-Associação (Conceitual)](#09---vetor-complementar--interceptação-pós-associação-conceitual)
10. [Validação e Preenchimento do Relatório](#10---validação-e-preenchimento-do-relatório)
11. [Encerramento do Módulo](#-encerramento-do-módulo)

---

## 01 - Contextualização — Hotspot como Vetor de Ataque

Diferente das Aulas 06-09 (que atacam a **chave** de uma rede protegida por WEP/WPS/WPA/WPA2/WPA3), o hotspot e a rede de convidados (Aula 04, item 05) costumam ter um modelo de ameaça diferente: em muitos casos, **não há chave para quebrar** — a rede é aberta, protegida por um portal cativo, ou libera acesso mediante simples cadastro. O ataque, portanto, desloca-se de "quebrar a criptografia" para **se passar pelo próprio hotspot** ou **explorar a confiança do usuário** nele.

| Conceito | Descrição |
|---|---|
| **Rogue AP** | Qualquer Access Point não autorizado inserido na rede ou nas proximidades dela, com o objetivo de interceptar tráfego ou capturar credenciais. |
| **Evil Twin** | Tipo específico de Rogue AP que **clona o SSID (e opcionalmente o BSSID)** de uma rede legítima, induzindo dispositivos a se conectarem a ele por engano — geralmente com sinal mais forte que o AP original. |
| **Karma Attack** | Técnica em que o AP falso responde a **todas** as *Probe Requests* recebidas, fingindo ser qualquer rede que o dispositivo-cliente já tenha conhecido antes (dispositivos frequentemente "perguntam" por SSIDs salvos). |
| **Captive Portal** | Página de autenticação exibida ao cliente antes de liberar acesso à Internet — legítima em hotspots comerciais (hotéis, aeroportos, eventos); pode ser **clonada** por um atacante para capturar credenciais ou dados pessoais. |

> 💡 **Ponto-chave desta aula:** nas Aulas 06-09, o sucesso do ataque é binário e mensurável (a chave foi ou não recuperada). Aqui, o "sucesso" depende de **engenharia social** — o dispositivo (ou a pessoa) precisa ser convencido a se associar ao AP falso. Isso torna a defesa mais comportamental (educação do usuário, certificados, VPN) do que puramente técnica.

> ⚠️ **Relação com a Aula 04:** a Rede de Convidados configurada pelo grupo (`guest-grupo-0x-5.0`, WPA2/WPA3-Pessoal) já é mais segura que um hotspot comercial típico (que costuma ser aberto). Para esta aula, o instrutor pode orientar o grupo a **reconfigurar temporariamente** a rede de convidados como **aberta (sem senha)** — cenário mais realista de hotspot público — documentando a mudança e revertendo ao final.

---

## 02 - Preparação do Ambiente e Modo Monitor

### 🔧 Procedimento

```bash
# Verificar reconhecimento da(s) placa(s)/antena(s) wireless
iwconfig
lsusb

# Finalizar processos conflitantes
airmon-ng check
airmon-ng check kill

# Habilitar modo monitor na interface de ataque/captura
airmon-ng start wlan0
```

> 💡 **Duas interfaces facilitam esta aula:** uma em **modo monitor** (reconhecimento/deauth) e outra dedicada a **subir o AP falso** (`airbase-ng`/`hostapd`). Com apenas uma antena disponível, os passos podem ser executados sequencialmente, alternando o modo da interface conforme a etapa.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe `wlan0mon` com **Mode:Monitor**. |
| 2 | Nenhum processo do NetworkManager reaparece interferindo na interface. |

---

## 03 - Reconhecimento do Hotspot/Rede de Convidados Alvo

### 🔧 Procedimento

```bash
airodump-ng wlan0mon
```

| Campo | Descrição |
|---|---|
| **BSSID** | Endereço MAC do AP legítimo da rede de convidados — necessário para clonar corretamente na seção 04. |
| **CH** | Canal em que o AP opera — o AP falso deve ser criado no **mesmo canal** ou em canal próximo, para reforçar a proximidade percebida pelo cliente. |
| **ENC/CIPHER/AUTH** | Tipo de segurança da rede de convidados (`OPN` se aberta, ou `WPA2`/`PSK` conforme configurado na Aula 04). |
| **ESSID** | Nome público da rede (ex.: `guest-grupo-0x-5.0`) — será clonado integralmente no Evil Twin. |
| **STATION** | Clientes já conectados ao AP legítimo — usados na seção 05 para o ataque de desautenticação direcionado. |

➡️ Anotar `BSSID`, `Canal`, `ESSID` e ao menos um `STATION` associado (dispositivo de teste do próprio grupo).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O AP de convidados do grupo aparece na varredura, com `ESSID` e `BSSID` corretamente identificados. |
| 2 | Ao menos um cliente de teste aparece associado ao AP legítimo. |

---

## 04 - Ataque Evil Twin — Clonagem do AP

Com o ESSID e o canal do hotspot legítimo identificados, sobe-se um AP falso com o **mesmo nome de rede**, usando o `airbase-ng` (suíte aircrack-ng, já instalada nas Aulas 02/02b) ou o `hostapd` (mais próximo de um AP real, recomendado quando o cenário exige portal cativo — seção 06).

### 🔧 Procedimento — Método 1: `airbase-ng` (rápido, didático)

```bash
airbase-ng -e "guest-grupo-0x-5.0" -c <CANAL> wlan0mon
```

| Parâmetro | Descrição |
|---|---|
| `-e` | Define o ESSID do AP falso — deve ser **idêntico** ao da rede de convidados legítima para induzir a reconexão automática dos clientes. |
| `-c` | Canal em que o AP falso será criado — usar o mesmo canal do AP legítimo (identificado na seção 03) reduz a distância percebida entre os dois sinais. |
| `wlan0mon` | Interface em modo monitor usada para criar o AP virtual. |

➡️ O `airbase-ng` cria a interface `at0`, que deve ser configurada com IP, DHCP e roteamento para que os clientes conectados recebam endereço de rede.

```bash
# Configurar a interface virtual criada pelo airbase-ng
ifconfig at0 up 10.0.0.1 netmask 255.255.255.0

# Habilitar o encaminhamento de pacotes (opcional — permite acesso à Internet via at0)
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 🔧 Procedimento — Método 2: `hostapd` (mais próximo de um AP real)

```bash
# Exemplo de arquivo de configuração hostapd-eviltwin.conf
cat <<EOF > hostapd-eviltwin.conf
interface=wlan0
driver=nl80211
ssid=guest-grupo-0x-5.0
hw_mode=g
channel=<CANAL>
EOF

hostapd hostapd-eviltwin.conf
```

| Parâmetro | Descrição |
|---|---|
| `interface` | Interface Wi-Fi usada para criar o AP (não precisa estar em modo monitor para o `hostapd`). |
| `ssid` | Nome da rede clonada — idêntico ao AP legítimo. |
| `channel` | Canal do AP falso, alinhado ao AP legítimo. |

> 💡 **Quando usar cada um:** `airbase-ng` é mais simples para demonstrações rápidas de Evil Twin puro. `hostapd` é recomendado quando a aula avança para o Captive Portal (seção 06), pois se integra melhor com `dnsmasq` para DHCP/DNS e com servidores web para a página de login falsa.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O AP falso aparece na lista de redes Wi-Fi de um dispositivo de teste com o **mesmo nome** (`guest-grupo-0x-5.0`) do AP legítimo. |
| 2 | `airodump-ng wlan0mon` (em outra interface/terminal) mostra **dois BSSIDs distintos** anunciando o mesmo ESSID — evidência visual do Evil Twin para a turma. |

---

## 05 - Forçando a Migração de Clientes — Deauth Combinado

Um Evil Twin sozinho depende do cliente **escolher manualmente** a rede falsa (ou de o dispositivo already ter reconexão automática habilitada). Para acelerar a demonstração, reutiliza-se o ataque de desautenticação já aplicado na Aula 08, item 05 — desta vez contra o **AP legítimo**, empurrando o cliente para o único sinal ainda disponível com aquele nome: o AP falso.

### 🔧 Procedimento

```bash
aireplay-ng -0 10 -a <BSSID_DO_AP_LEGITIMO> -c <MAC_DO_CLIENTE_STATION> wlan1mon
```

| Parâmetro | Descrição |
|---|---|
| `-0 10` | Envia 10 pacotes de desautenticação ao cliente-alvo, forçando-o a procurar novamente por redes com o SSID conhecido. |
| `-a` | BSSID do AP **legítimo** da rede de convidados (obtido na seção 03). |
| `-c` | MAC do cliente de teste a ser desconectado. |
| `wlan1mon` | Interface em modo monitor **diferente** da usada para o AP falso (seção 04), caso duas antenas estejam disponíveis. |

> ⚠️ **Cuidado com auto-reconexão vs. escolha manual:** muitos sistemas operacionais modernos (Android, iOS, Windows) tratam redes com o mesmo SSID mas BSSID diferente como **redes distintas** na primeira conexão, exigindo nova autenticação manual — a auto-reconexão automática ao Evil Twin não é garantida em todo dispositivo. Trate esse comportamento como ponto de discussão em sala (mecanismo de defesa nativo do sistema operacional).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O cliente de teste perde a conexão com o AP legítimo durante o deauth. |
| 2 | O cliente reconecta-se (automaticamente ou por seleção manual) ao AP falso, confirmado pelo aparecimento do seu MAC nos logs do `airbase-ng`/`hostapd`. |

---

## 06 - Captive Portal Falso — Reconhecimento e Defesa

> 📚 **Seção majoritariamente demonstrativa/teórica** — o objetivo é que a turma **reconheça** a técnica e suas defesas, não colete credenciais reais. Use apenas contas e dispositivos de teste criados exclusivamente para esta aula.

Com o Evil Twin (seção 04) já ativo e clientes se associando (seção 05), é possível ir além e apresentar uma página de login falsa antes de liberar o acesso à Internet — reproduzindo o comportamento de portais de hotéis, aeroportos e eventos.

### 🔧 Procedimento — Estrutura Geral

```bash
# Servidor DHCP + DNS falso apontando todo tráfego para o servidor web local
cat <<EOF > dnsmasq-portal.conf
interface=at0
dhcp-range=10.0.0.10,10.0.0.100,12h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
address=/#/10.0.0.1
EOF

dnsmasq -C dnsmasq-portal.conf -d
```

| Parâmetro | Descrição |
|---|---|
| `dhcp-range` | Faixa de IPs concedida aos clientes que se conectam ao AP falso. |
| `address=/#/10.0.0.1` | Redireciona **toda** resolução DNS para o servidor local — qualquer site que o cliente tente acessar retorna a página do portal falso. |

```bash
# Servidor web simples servindo a página de login clonada
cd /caminho/da/pagina-clonada
python3 -m http.server 80
```

| Componente | Descrição |
|---|---|
| **Página clonada** | Cópia estática (HTML/CSS) do portal real, com um formulário de login que, em vez de autenticar, apenas registra os dados digitados em um arquivo/log local. |
| **Ferramentas especializadas** | Suítes como `wifiphisher` automatizam todo esse fluxo (AP falso + DHCP/DNS + portal + captura), sendo a opção recomendada para reproduzir o ataque completo de forma mais estável. |

```bash
# Alternativa: wifiphisher automatiza AP falso, DHCP, DNS e portal cativo
sudo apt install -y wifiphisher
sudo wifiphisher -e "guest-grupo-0x-5.0" --forceHostapd
```

### 🛡️ Foco em Defesa — O Que Discutir com a Turma

| Sinal de alerta para o usuário | Por que protege |
|---|---|
| Certificado HTTPS ausente/inválido no portal | Portais cativos legítimos, quando bem implementados, avisam antes de qualquer captura de credencial sensível; certificados quebrados são um forte indicador de portal falso. |
| Solicitação de senhas de e-mail/redes sociais no portal | Hotspots legítimos **nunca** deveriam pedir credenciais de serviços de terceiros — apenas dados próprios do provedor do hotspot (quarto, código do evento, etc.). |
| Uso de VPN corporativa/pessoal | Neutraliza a interceptação de tráfego mesmo que o dispositivo esteja associado a um Evil Twin. |
| Comportamento do SO ao reconectar a um SSID conhecido | Sistemas modernos alertam ou exigem nova autenticação quando o certificado/identidade do AP muda — reforçar que ignorar esse aviso é o principal erro do usuário. |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O dispositivo de teste, ao se conectar ao AP falso, é redirecionado automaticamente para a página do portal clonado ao tentar navegar. |
| 2 | Dados de teste digitados (conta fictícia, criada exclusivamente para a aula) aparecem no log do servidor local, demonstrando a captura. |
| 3 | A turma identifica ao menos dois dos sinais de alerta da tabela acima ao inspecionar o portal falso. |

---

## 07 - Karma Attack — Resposta a Probe Requests

### 🔧 Procedimento

```bash
# airbase-ng em modo "responder a qualquer probe request" (Karma)
airbase-ng -P -C 30 -v wlan0mon
```

| Parâmetro | Descrição |
|---|---|
| `-P` | Modo *responder a todas as Probe Requests* — o AP falso passa a "confirmar" ser qualquer rede que o dispositivo pergunte, e não apenas o SSID clonado da seção 04. |
| `-C 30` | Intervalo (segundos) para reenviar beacons e reforçar a presença do AP falso. |

> 💡 **Por que funciona:** muitos dispositivos, ao ligar o Wi-Fi, enviam *Probe Requests* perguntando "a rede X (salva anteriormente) está por perto?" — um AP em modo Karma responde "sim" a todas, mesmo sem conhecer o SSID de antemão, capturando dispositivos que nunca estiveram perto da rede de convidados do laboratório.

> ⚠️ **Ferramentas mais completas:** suítes modernas como `wifiphisher` (modo `--kB` ou plugins de Karma) e `hostapd-mana` implementam o ataque de forma mais robusta que o `airbase-ng -P` isolado, incluindo suporte a WPA2 Enterprise falso — citar como aprofundamento opcional.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Um dispositivo de teste, com uma rede fictícia previamente salva (nunca vista no laboratório), exibe essa rede como disponível assim que o AP em modo Karma é ligado. |
| 2 | A turma discute por que esse comportamento é uma falha de design dos dispositivos-cliente, não do protocolo Wi-Fi em si. |

---

## 08 - Bypass de Autenticação por MAC (MAC Spoofing)

Alguns hotspots liberam acesso simplesmente **cadastrando o MAC Address** do dispositivo (comum em portais de hotéis/eventos sem senha real). Reconhecido o padrão, é possível assumir o MAC de um cliente já autenticado para herdar seu acesso.

### 🔧 Procedimento

```bash
# Identificar MACs de clientes já conectados/autorizados no hotspot
airodump-ng --bssid <BSSID_DO_AP> -c <CANAL> wlan0mon

# Alterar o MAC da própria interface para o de um cliente já autorizado
ifconfig wlan1 down
macchanger -m <MAC_DO_CLIENTE_AUTORIZADO> wlan1
ifconfig wlan1 up
```

| Parâmetro | Descrição |
|---|---|
| `--bssid` / `-c` | Filtra a captura no AP-alvo, para listar os MACs de clientes associados (seção `STATION`, já usada na Aula 06/08). |
| `macchanger -m` | Define manualmente o MAC da interface atacante para um valor específico — nesse caso, o de um cliente já reconhecido/autorizado pelo hotspot. |

> ⚠️ **Colisão de MAC:** ao clonar o MAC de um cliente **ainda ativo** na rede, ocorre conflito de endereço (ambos os dispositivos com o mesmo MAC simultaneamente), o que pode gerar instabilidade para os dois lados — em laboratório, prefira clonar o MAC de um dispositivo de teste do próprio grupo que esteja **momentaneamente desconectado**.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `macchanger -s wlan1` confirma que o MAC da interface atacante foi alterado com sucesso para o valor desejado. |
| 2 | O acesso à Internet via hotspot é obtido sem novo cadastro, apenas herdando a autorização do MAC clonado. |

---

## 09 - Vetor Complementar — Interceptação Pós-Associação (Conceitual)

> 📚 **Conteúdo teórico — sem exploração prática nesta aula**, servindo de gancho para módulos futuros de análise de tráfego (Man-in-the-Middle).

Uma vez que um cliente está associado ao AP falso (Evil Twin, seções 04-05), todo o seu tráfego passa pela estação atacante antes de seguir para a Internet, abrindo espaço para técnicas adicionais:

| Técnica | Resumo conceitual | Ferramenta típica |
|---|---|---|
| **DNS Spoofing** | Responde a consultas DNS do cliente com endereços IP controlados pelo atacante, redirecionando sites legítimos para páginas falsas. | `bettercap`, `dnsspoof` |
| **SSL Strip** | Tenta rebaixar conexões HTTPS para HTTP, removendo a criptografia antes que o navegador do cliente perceba. | `bettercap`, `sslstrip` |
| **Captura de tráfego não criptografado** | Qualquer dado trafegado sem TLS (ex.: HTTP puro, alguns apps mal configurados) fica visível ao atacante na posição de MITM. | `Wireshark`, `tcpdump` (já instalados na Aula 02b) |

> 💡 **Mensagem central:** a melhor defesa contra esse vetor **não está na rede**, e sim na aplicação — HTTPS bem implementado (com HSTS) e VPN neutralizam a maior parte desses ataques mesmo com o dispositivo associado a um AP malicioso. Recomenda-se tratar este item como discussão teórica nesta aula, reservando a exploração prática de MITM para um módulo dedicado.

---

## 10 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Registrar no relatório: ESSID clonado, BSSID do AP legítimo, BSSID do AP falso, canal utilizado e ferramenta principal (`airbase-ng`/`hostapd`/`wifiphisher`). |
| 2 | Registrar se a migração do cliente ocorreu por auto-reconexão ou seleção manual, e discutir o porquê. |
| 3 | Registrar os resultados dos testes de Captive Portal falso (seção 06) e Karma Attack (seção 07), incluindo os sinais de alerta identificados pela turma. |
| 4 | Registrar o resultado do bypass por MAC Spoofing (seção 08), se aplicável ao cenário do hotspot testado. |
| 5 | Discutir em grupo: diferença entre "quebrar uma senha" (Aulas 06-09) e "enganar um usuário" (esta aula) — qual exige mais tempo/técnica, e qual é mais eficaz na prática contra hotspots reais. |
| 6 | Confirmar que a Rede de Convidados foi **restaurada à configuração original** (Aula 04) ao final da aula, caso tenha sido temporariamente alterada para rede aberta. |

## ✅ Checklist Final da Aula

- [ ] Modo monitor habilitado (`wlan0mon`)
- [ ] AP legítimo da rede de convidados identificado (BSSID, canal, ESSID)
- [ ] Evil Twin criado com o mesmo ESSID (`airbase-ng` ou `hostapd`)
- [ ] Dois BSSIDs distintos anunciando o mesmo SSID confirmados no `airodump-ng`
- [ ] Deauth aplicado contra o AP legítimo (`aireplay-ng -0`) para forçar migração do cliente de teste
- [ ] Cliente de teste associado ao AP falso confirmado
- [ ] Captive Portal falso demonstrado (DHCP/DNS via `dnsmasq` + página clonada ou `wifiphisher`)
- [ ] Sinais de alerta do portal falso identificados e discutidos em grupo
- [ ] Karma Attack testado com dispositivo de teste e rede fictícia previamente salva
- [ ] Bypass de autenticação por MAC testado (quando aplicável ao cenário do hotspot)
- [ ] Vetor de interceptação pós-associação (DNS spoofing/SSL strip) discutido em nível teórico
- [ ] Rede de Convidados restaurada à configuração original da Aula 04
- [ ] Relatório do grupo preenchido com todos os dados da seção 10

---

## 🏁 Encerramento do Módulo

Com a Aula 11, o módulo de **Testes de Invasão em Rede Wireless** passa a cobrir, além da chave Wi-Fi (Aulas 06-09) e da senha administrativa (Aula 10), a camada de **confiança do usuário** explorada por hotspots e redes de convidados — evidenciando que nem todo ataque relevante em Wi-Fi depende de quebrar uma criptografia.

### 📊 Quadro-Resumo Ampliado do Módulo

| Aula | Alvo do ataque | Camada | Ferramenta principal | Depende de dicionário? |
|---|---|---|---|---|
| 06 | Chave WEP | 802.11 | `aircrack-ng` (criptoanálise) | Não |
| 07 | PIN WPS | 802.11 | `reaver`/`pixiewps` | Não |
| 08 | Chave WPA/WPA2-PSK | 802.11 | `aircrack-ng`/`hashcat` | **Sim** |
| 09 | Chave WPA3-SAE | 802.11 | N/A (resistente por design) | Não aplicável |
| 10 | Senha do painel administrativo (LAN e WAN) | HTTP/HTTPS (aplicação) | `hydra` | **Sim** |
| **11** | **Confiança do usuário / modelo de acesso do hotspot** | **802.11 + engenharia social** | **`airbase-ng`/`hostapd`/`wifiphisher`** | **Não aplicável** |

> 💡 **Mensagem final para a turma:** proteger uma rede de convidados/hotspot vai além de configurar WPA2/WPA3 (Aula 04) — exige também **educar o usuário final** (reconhecer portais suspeitos, desconfiar de certificados inválidos, usar VPN) e, do lado do provedor da rede, **monitorar a presença de APs não autorizados** (Rogue AP detection), algo que ferramentas comerciais de WIDS/WIPS endereçam e que foge do escopo de hardware doméstico como o TP-Link Archer usado neste laboratório.

> 📌 **Sugestão de continuidade do curso:** com o módulo de invasão wireless (Aulas 06-11) concluído, os próximos passos naturais seriam (a) ataques a redes Enterprise (802.1X/EAP) — exigindo um servidor RADIUS no laboratório —, (b) um módulo dedicado de Man-in-the-Middle pós-associação (aprofundando a seção 09 desta aula), e (c) consolidação de todos os resultados das Aulas 06-11 no documento de homologação (`07-checklist/01-HomologacaoRedesSemFioKali.md`), que deve ser atualizado com uma nova seção específica para os ataques de Hotspot/Evil Twin.
