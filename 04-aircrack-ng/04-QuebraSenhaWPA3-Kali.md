# 📡 Aula 09 — Testes de Invasão Wireless: WPA3-Personal (SAE) com Kali Linux

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 09 — Por que o Ataque de Dicionário Tradicional Não Funciona contra WPA3-SAE
> **Equipamento homologado para esta aula:** **TP-Link Archer AX12 (AX1500)** — únino modelo do laboratório com suporte a **WPA3-Personal**
> **Estação de ataque:** Kali Linux + placa de rede wireless compatível com modo monitor/injeção

> ⚠️ **Troca de equipamento:** de acordo com a documentação técnica oficial ([tp-link.com/br/.../ec220-g5/#specifications](https://www.tp-link.com/br/home-networking/wifi-router/ec220-g5/#specifications)), o **Archer EC220-G5** utilizado nas Aulas 01-08 **não possui suporte a WPA3**. Para esta aula, o equipamento homologado passa a ser o **TP-Link Archer AX12 (AX1500)**, que suporta oficialmente **WPA/WPA2/WPA3-Personal** e **WPA/WPA2-Enterprise**. A interface de configuração (Avançado → Wireless → Configurações Wireless) segue o **mesmo padrão** já utilizado nas Aulas 03 e 04, com um campo adicional na **Versão** de segurança: `WPA3-SAE`.
>
> ⚠️ **Numeração:** ajuste o prefixo (`09-...`) conforme a ordem final adotada no repositório.

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | _[preencher — Robson Vaamonde / Professor responsável]_ |
| **Baseado em** | Material do docente de Testes de Invasão em Rede Wireless (Prof. MSC Arlindo Batista) + documentação técnica oficial TP-Link |
| **Data de criação** | 31/07/2026 |
| **Versão** | 0.01 |
| **Status** | ⚠️ Roteiro **ainda não testado/validado** em ambiente real de laboratório |
| **Pré-requisito** | Aulas 01-08 concluídas (WEP, WPS e WPA/WPA2-PSK já executados) |
| **Escopo** | **Modo Pessoal (SAE)** apenas — sem RADIUS no laboratório, o modo Empresarial permanece fora do escopo |
| **Natureza da aula** | Predominantemente **conceitual/demonstrativa** — o objetivo não é "quebrar" o WPA3 puro (não há técnica prática equivalente às aulas anteriores), e sim demonstrar **por que** o ataque tradicional falha e discutir os vetores residuais conhecidos |

---

## ⚠️ Aviso Legal e Ético — Leia Antes de Iniciar

> Este material é destinado **exclusivamente a fins didáticos**, em **ambiente de rede controlada e isolada** (laboratório), com equipamentos de propriedade da instituição de ensino e consentimento expresso para os testes.
>
> A utilização das técnicas aqui descritas contra redes de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de responsabilidade civil (Código Civil, arts. 927 a 943).
>
> Cada grupo deve atacar **somente** o Access Point que lhe foi atribuído no laboratório.

---

## 📑 Sumário

1. [Contextualização — SAE/Dragonfly e Por Que o Ataque Offline Não Funciona](#01---contextualização--saedragonfly-e-por-que-o-ataque-offline-não-funciona)
2. [Configurando o WPA3 no Archer AX12](#02---configurando-o-wpa3-no-archer-ax12)
3. [Preparação do Ambiente e Modo Monitor](#03---preparação-do-ambiente-e-modo-monitor)
4. [Reconhecimento — Identificando SAE no Airodump-ng](#04---reconhecimento--identificando-sae-no-airodump-ng)
5. [Demonstração Prática — Tentativa de Captura e Ataque Tradicional](#05---demonstração-prática--tentativa-de-captura-e-ataque-tradicional)
6. [Vetor Residual 1 — Modo de Transição WPA2/WPA3 (Downgrade)](#06---vetor-residual-1--modo-de-transição-wpa2wpa3-downgrade)
7. [Vetor Residual 2 — Vulnerabilidades de Side-Channel (Dragonblood)](#07---vetor-residual-2--vulnerabilidades-de-side-channel-dragonblood)
8. [Validação e Preenchimento do Relatório](#08---validação-e-preenchimento-do-relatório)
9. [Encerramento do Módulo](#-encerramento-do-módulo)

---

## 01 - Contextualização — SAE/Dragonfly e Por Que o Ataque Offline Não Funciona

O **WPA3-Personal** substitui o PSK tradicional pelo **SAE (Simultaneous Authentication of Equals)**, também conhecido como protocolo **Dragonfly**. A diferença fundamental em relação ao WPA2 está em **como** a senha é validada:

| Característica | WPA2-PSK | WPA3-SAE |
|---|---|---|
| Troca de chave | 4-way handshake — a senha é usada para derivar a PMK, e o handshake capturado permite testar candidatos **offline** | SAE — cada tentativa de autenticação exige uma **troca interativa em tempo real** com o AP; não existe um handshake estático que possa ser levado para quebra offline |
| Ataque de dicionário | Possível offline, sem limite de tentativas, sem o AP saber que está sendo atacado (é exatamente o que fizemos na Aula 08) | **Só é possível online**, uma tentativa por vez, contra o AP real — o equipamento pode detectar e bloquear tentativas repetidas (rate limiting nativo do protocolo) |
| Forward secrecy | Não possui — se a senha vazar, capturas antigas podem ser decifradas | Possui — cada sessão gera chaves únicas; uma senha vazada não compromete capturas anteriores |
| Ferramenta de ataque | `aircrack-ng`, `hashcat` (offline, rápido) | Nenhuma ferramenta consolidada realiza o mesmo tipo de ataque offline puro contra SAE corretamente implementado |

> 💡 **Ponto central para a turma:** na Aula 08, o `aircrack-ng`/`hashcat` conseguiam testar **milhares de senhas por segundo** porque toda a computação acontecia **localmente**, sem interagir com o AP depois da captura do handshake. No SAE, **cada tentativa exige uma troca de mensagens com o roteador real**, tornando um ataque de força bruta impraticável (na ordem de poucas tentativas por segundo, sujeitas a bloqueio) — a mesma limitação, na prática, que tornaria o ataque ao WPS lento se não fosse a falha de design explorada na Aula 07.

---

## 02 - Configurando o WPA3 no Archer AX12

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** (mesmo caminho já utilizado nas Aulas 03 e 04) |

### 📶 Cenário A — WPA3-Personal Puro (sem retrocompatibilidade)

| Campo | Valor | Descrição |
|---|---|---|
| Segurança | `WPA3-Pessoal` | Exige que todos os clientes suportem SAE — dispositivos antigos (WPA2-only) não conseguem se conectar. |
| Versão | `WPA3-SAE` | Ativa exclusivamente o protocolo SAE, sem fallback para PSK tradicional. |
| Senha | `Senactit@123` | Chave de acesso (também chamada de *senha SAE* no contexto do WPA3). |

➡️ Utilizado nos passos 04 e 05 para demonstrar por que o ataque offline **falha**.

### 📶 Cenário B — Modo de Transição WPA2/WPA3-Personal (misto)

| Campo | Valor | Descrição |
|---|---|---|
| Segurança | `WPA2/WPA3-Pessoal` | Modo de compatibilidade — aceita tanto clientes WPA2 (PSK) quanto WPA3 (SAE) simultaneamente na mesma rede. |
| Versão | `Automático` | O AP negocia o protocolo de acordo com a capacidade de cada cliente que se conecta. |
| Senha | `Senactit@123` | Mesma senha compartilhada entre os dois protocolos no modo de transição. |

➡️ Utilizado no passo 06 para demonstrar o **vetor de downgrade**.

> 💡 **Dica didática:** configure o AP do grupo primeiro no **Cenário A** para a demonstração de "ataque que falha", e só depois reconfigure para o **Cenário B** para explorar o vetor de transição — isso reforça visualmente a diferença de comportamento para a turma.

---

## 03 - Preparação do Ambiente e Modo Monitor

### 🔧 Procedimento

```bash
# Verificar reconhecimento da placa/antena wireless
iwconfig
lsusb

# Finalizar processos conflitantes
airmon-ng check
airmon-ng check kill

# Habilitar modo monitor
airmon-ng start wlan0
```

> 💡 **Alternativa (drivers incompatíveis com airmon-ng):**
> ```bash
> ifconfig wlan0 down
> iwconfig wlan0 mode monitor
> ifconfig wlan0 up
> iwconfig   # confirmar o modo monitor
> ```

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe `wlan0mon` com **Mode:Monitor**. |
| 2 | Nenhum processo do NetworkManager reaparece interferindo na interface. |

---

## 04 - Reconhecimento — Identificando SAE no Airodump-ng

### 🔧 Procedimento

```bash
airodump-ng wlan0mon
```

| Campo | Descrição |
|---|---|
| **ENC** | Aparece como **`WPA3`** (Cenário A) ou **`WPA2 WPA3`** (Cenário B, modo de transição). |
| **AUTH** | Aparece como **`SAE`** (Cenário A) ou **`PSK SAE`** (Cenário B — indicando que ambos os métodos estão disponíveis simultaneamente). |
| **CIPHER** | Normalmente **`CCMP`** — o WPA3-Personal exige AES, não permitindo TKIP. |

➡️ Anotar `BSSID`, `Canal` e o valor de `AUTH` — é esse campo que revela se há (ou não) uma porta de entrada via WPA2 no mesmo AP.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | No Cenário A, o campo **AUTH** mostra apenas `SAE`. |
| 2 | No Cenário B, o campo **AUTH** mostra `PSK SAE`, confirmando o modo de transição ativo. |

---

## 05 - Demonstração Prática — Tentativa de Captura e Ataque Tradicional

Com o AP no **Cenário A (WPA3 puro)**, repita o fluxo da Aula 08 propositalmente, para que a turma observe o resultado.

### 🔧 Procedimento

```bash
# Captura direcionada (idêntico à Aula 08)
airodump-ng -c <CANAL> --bssid <BSSID_DO_AP> -w tentativawpa3 wlan0mon

# Em outro terminal: desautenticação do cliente
aireplay-ng -0 5 -a <BSSID_DO_AP> -c <MAC_DO_CLIENTE_STATION> wlan0mon
```

### ✅ Resultado Esperado (e ponto de discussão)

| Observação | Explicação |
|---|---|
| O `airodump-ng` **não exibe** a mensagem `WPA handshake: <BSSID>` da mesma forma que no WPA2 | O SAE não produz um handshake estático equivalente ao 4-way handshake do WPA2; o processo de autenticação SAE não gera o mesmo tipo de material criptográfico reutilizável offline. |
| Mesmo capturando pacotes de autenticação SAE, `aircrack-ng`/`hashcat -m 22000` **não conseguem** testar candidatos de senha offline com sucesso | O material capturado não permite validar uma senha candidata sem interação com o AP real — não há PMK derivável apenas da senha e dos pacotes capturados, como acontece no PSK. |

> 💡 **Conduza como discussão em sala:** peça para os grupos tentarem o mesmo fluxo da Aula 08 e documentarem no relatório **por que** o resultado é diferente, relacionando com a tabela da seção 01 (troca interativa vs. handshake estático).

---

## 06 - Vetor Residual 1 — Modo de Transição WPA2/WPA3 (Downgrade)

Reconfigure o AP para o **Cenário B** (seção 02). Nesse modo, um cliente mais antigo — ou um atacante que force esse comportamento — pode se conectar via **WPA2-PSK tradicional**, mesmo a rede anunciando suporte a WPA3.

### 🔧 Procedimento

```bash
# Reconhecimento confirma AUTH = PSK SAE
airodump-ng wlan0mon

# A partir daqui, o fluxo é IDÊNTICO ao da Aula 08 (captura de handshake WPA2 + dicionário)
airodump-ng -c <CANAL> --bssid <BSSID_DO_AP> -w downgradewpa3 wlan0mon
aireplay-ng -0 5 -a <BSSID_DO_AP> -c <MAC_DO_CLIENTE_STATION> wlan0mon
aircrack-ng -b <BSSID_DO_AP> -w 02-wordlist-lab-expandida.txt downgradewpa3-01.cap
```

> ⚠️ **Ponto-chave:** o ataque **não quebrou o WPA3** — ele explorou o fato de o AP **ainda aceitar WPA2** no mesmo SSID. A mitigação é simples e deve ser destacada aos alunos: **desativar o modo de transição** e operar em **WPA3-Personal puro** (Cenário A) sempre que todos os clientes da rede suportarem SAE.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | No Cenário B, é possível capturar um handshake WPA2 de um cliente que se conectou via PSK. |
| 2 | O ataque de dicionário da Aula 08 funciona normalmente contra esse handshake, mesmo com o AP anunciando WPA3. |
| 3 | No Cenário A (WPA3 puro), o mesmo ataque não produz um handshake WPA2 explorável, pois não há fallback disponível. |

---

## 07 - Vetor Residual 2 — Vulnerabilidades de Side-Channel (Dragonblood)

> 📚 **Conteúdo teórico — sem exploração prática nesta aula.**

Em 2019, pesquisadores publicaram o conjunto de vulnerabilidades conhecido como **Dragonblood**, afetando implementações específicas do protocolo SAE/Dragonfly (não o protocolo em si, mas erros de implementação em determinados firmwares/drivers da época).

| Categoria | Resumo conceitual |
|---|---|
| **Timing side-channel** | Diferenças no tempo de processamento de determinadas implementações do SAE podiam vazar informação suficiente para viabilizar um ataque de dicionário offline indireto, mesmo o protocolo sendo teoricamente resistente a isso. |
| **Cache side-channel** | Vulnerabilidade similar, explorando padrões de acesso à memória em vez de tempo de resposta. |
| **Ataques de downgrade** | Em implementações de modo de transição mal configuradas, era possível forçar o uso de WPA2 mesmo com WPA3 habilitado — o mesmo princípio demonstrado na seção 06 deste roteiro. |

> ⚠️ **Status atual:** a maior parte dos fabricantes, incluindo a TP-Link, lançou atualizações de firmware corrigindo as variantes conhecidas do Dragonblood nos anos seguintes à divulgação. Antes de qualquer discussão sobre esse tema em sala, **verifique a versão de firmware do Archer AX12** do laboratório e confirme junto à documentação oficial da TP-Link se há atualizações de segurança pendentes — isso deve ser tratado como **exercício de verificação de patch/CVE**, não como um roteiro de exploração ativa.

> 💡 **Atividade sugerida para a turma:** pesquisar o firmware atual do Archer AX12 no site de suporte da TP-Link e verificar no changelog se há menção a correções de segurança relacionadas a WPA3/SAE — reforçando a prática de gestão de patches como medida de segurança tão importante quanto a configuração correta do protocolo.

---

## 08 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Registrar no relatório o resultado da tentativa de ataque tradicional contra o Cenário A (WPA3 puro) — deve constar como **não bem-sucedido**, com a justificativa técnica (seção 01/05). |
| 2 | Registrar o resultado do ataque via modo de transição (Cenário B) — deve constar como **bem-sucedido**, evidenciando que o vetor explorado foi o fallback para WPA2, não uma falha do SAE. |
| 3 | Verificar e registrar a versão de firmware do Archer AX12 utilizado, relacionando com o tema Dragonblood (seção 07). |
| 4 | Discussão final em grupo: comparar os quatro protocolos estudados no módulo (WEP, WPS, WPA/WPA2, WPA3) quanto a **tempo de quebra**, **dependência de dicionário** e **recomendação de uso em produção**. |

## ✅ Checklist Final da Aula

- [ ] Archer AX12 configurado no Cenário A (WPA3-Personal puro) e testado
- [ ] Modo monitor habilitado e reconhecimento confirmando `AUTH = SAE`
- [ ] Tentativa de captura/ataque tradicional documentada como sem sucesso, com justificativa técnica
- [ ] Archer AX12 reconfigurado no Cenário B (WPA2/WPA3-Personal, modo de transição)
- [ ] Reconhecimento confirmando `AUTH = PSK SAE`
- [ ] Handshake WPA2 capturado via downgrade e quebrado com a wordlist do laboratório
- [ ] Versão de firmware do AX12 verificada e relacionada ao tema Dragonblood
- [ ] Relatório comparativo final preenchido (WEP x WPS x WPA/WPA2 x WPA3)

---

## 🏁 Encerramento do Módulo

Com a Aula 09, encerra-se o roteiro de **Testes de Invasão em Rede Wireless** iniciado na Aula 06 (WEP), passando por WPS (Aula 07), WPA/WPA2-Personal (Aula 08) e finalizando em WPA3-Personal (Aula 09).

### 📊 Quadro-Resumo do Módulo

| Aula | Protocolo | Método de ataque | Depende de dicionário? | Resultado esperado |
|---|---|---|---|---|
| 06 | WEP | Criptoanálise estatística (IVs) | Não | Quebra garantida com IVs suficientes |
| 07 | WPS | Força bruta de PIN (Pixie Dust) | Não | Quebra rápida, falha de design do protocolo |
| 08 | WPA/WPA2-PSK | Handshake + dicionário offline | **Sim** | Quebra depende inteiramente da qualidade da wordlist |
| 09 | WPA3-SAE | Sem ataque offline prático viável | Não aplicável | Resistente por design; único vetor viável é downgrade/implementação |

> 💡 **Mensagem final para a turma:** a evolução WEP → WPS → WPA/WPA2 → WPA3 é também uma evolução na **filosofia de defesa** — de uma cifra estatisticamente quebrável, passando por uma falha de design isolada (WPS), até um protocolo que elimina o ataque offline por completo (SAE). A conclusão prática para qualquer ambiente de produção: **desabilitar WPS, usar WPA3-Personal puro sempre que possível e, quando não for possível, usar WPA2 com senha longa e não presente em dicionários públicos.**

> 📌 **Sugestão de continuidade do curso:** com o módulo de quebra de senhas concluído, os próximos passos naturais seriam (a) ataques de Evil Twin / Rogue AP, (b) ataques a redes Enterprise (802.1X/EAP) — exigindo um servidor RADIUS no laboratório —, e (c) hardening avançado revisitando as Aulas 04 e 05 à luz de tudo o que foi explorado neste módulo.
