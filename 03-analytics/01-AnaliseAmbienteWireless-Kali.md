# 📡 Aula — Análise do Ambiente Sem-Fio e Verificação de Hardware (Baseline Pré-Ataque)

> **Módulo:** Redes Sem-Fio — Reconhecimento Passivo (Pré-Pentest)
> **Aula:** Verificação de Hardware, Conceitos de Análise do Ambiente e Reconhecimento de APs com Kali Linux
> **Equipamentos homologados:** Notebook Dell Inspiron + Antenas USB Realtek RTL8188EUS e Qualcomm Atheros AR9271
> **Pré-requisito:** Aula de Instalação/Configuração Básica do Kali Linux concluída (drivers das antenas já reconhecidos)

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 01/08/2026 |
| **Data de atualização** | 01/08/2026 |
| **Versão** | 0.01 |
| **Equipamentos testados** | Notebook Dell Inspiron, Antena Realtek RTL8188EUS, Antena Qualcomm Atheros AR9271 |

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

## 🎯 Objetivo da Aula

Antes de iniciar qualquer teste de invasão contra os protocolos de segurança wireless (WEP, WPS, WPA/WPA2, WPA3 — Aulas seguintes), o aluno precisa saber **avaliar o ambiente** em que está atuando. Esta aula cobre três frentes:

1. **Verificação de hardware** — confirmar que o notebook e as antenas estão aptos para captura/monitoramento (antes de cogitar injeção de pacotes).
2. **Conceitos básicos de análise do ambiente sem-fio** — bandas, canais, sobreposição, intensidade de sinal.
3. **Reconhecimento passivo** — identificar APs, canais, frequências e clientes presentes no laboratório, documentando um cenário de referência (*baseline*) **antes** de qualquer ataque.

> 💡 **Por que isso importa:** um bom pentest wireless começa pela leitura do cenário — atacar o AP errado, no canal errado, ou sem entender a topologia do laboratório, é a causa mais comum de falha nas Aulas seguintes (06 a 09).

---

## 📑 Sumário

1. [Verificação de Hardware — Notebook e Antenas](#01---verificação-de-hardware--notebook-e-antenas)
2. [Conceitos Básicos de Análise do Ambiente Sem-Fio](#02---conceitos-básicos-de-análise-do-ambiente-sem-fio)
3. [Reconhecimento de APs em Modo Gerenciado (sem modo monitor)](#03---reconhecimento-de-aps-em-modo-gerenciado-sem-modo-monitor)
4. [Habilitando o Modo Monitor para Reconhecimento Passivo](#04---habilitando-o-modo-monitor-para-reconhecimento-passivo)
5. [Reconhecimento com Airodump-ng — Leitura de Campos](#05---reconhecimento-com-airodump-ng--leitura-de-campos)
6. [Identificação de Clientes Conectados (STATION)](#06---identificação-de-clientes-conectados-station)
7. [Intensidade de Sinal e Qualidade da Captura (PWR)](#07---intensidade-de-sinal-e-qualidade-da-captura-pwr)
8. [Ferramentas Complementares de Apoio](#08---ferramentas-complementares-de-apoio)
9. [Documentando o Cenário — Relatório de Baseline](#09---documentando-o-cenário--relatório-de-baseline)

---

## 01 - Verificação de Hardware — Notebook e Antenas

### 🔧 Procedimento

```bash
# Informações gerais do notebook
inxi -Fxz          # visão geral (CPU, memória, rede, disco)
lscpu               # detalhes do processador
free -h              # memória RAM disponível
df -h                # espaço em disco

# Reconhecimento das interfaces de rede sem-fio
iwconfig
iw dev
lsusb
```

| Campo | Descrição |
|---|---|
| `inxi -Fxz` | Fornece uma visão geral rápida do hardware do notebook (CPU, RAM, rede), útil para registrar no relatório qual máquina foi usada no teste. |
| `iwconfig` / `iw dev` | Lista todas as interfaces sem-fio reconhecidas: a placa Wi-Fi interna do notebook e as antenas USB externas conectadas. |
| `lsusb` | Confirma que as antenas externas (Realtek RTL8188EUS e/ou Atheros AR9271) foram detectadas na porta USB, exibindo fabricante e chipset. |

### 📶 Checklist de Hardware por Interface

| Interface | Origem | Suporta Modo Monitor? | Observação |
|---|---|---|---|
| `wlan0` (ou similar) | Placa Wi-Fi interna do notebook | ⚠️ Depende do chipset — normalmente **não recomendada** para pentest | Usar apenas para conexão administrativa (ex.: acessar a interface web do AP), nunca como interface de ataque. |
| Antena USB Realtek `RTL8188EUS` | Externa | ✅ Sim, com driver `realtek-rtl8188eus-dkms` | Confirmar que o driver DKMS (e não o `r8188eu` genérico) está carregado — ver Aula de instalação do Kali, item 06.2. |
| Antena USB Atheros `AR9271` | Externa | ✅ Sim, nativamente (`ath9k_htc`) | Não exige driver adicional. |

> ⚠️ **Atenção:** antes de qualquer captura, identifique **qual interface (`wlan0`, `wlan1`, `wlan2`...) corresponde a qual antena**. Usar a interface errada é a causa mais comum de comandos que "não funcionam" nas próximas aulas.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig`/`iw dev` lista a placa interna **e** ao menos uma antena externa, com nomes de interface distintos. |
| 2 | `lsusb` reconhece o chipset da(s) antena(s) externa(s) conectada(s). |
| 3 | Cada interface identificada e anotada no relatório do grupo (ex.: `wlan1 = Antena Atheros AR9271`). |

---

## 02 - Conceitos Básicos de Análise do Ambiente Sem-Fio

> 💡 Estes conceitos já foram introduzidos na configuração dos APs (bandas, canais e largura de canal). Aqui o foco é **aplicá-los na leitura do cenário real do laboratório**, e não apenas na configuração de um único equipamento.

| Conceito | Descrição | Por que importa no reconhecimento |
|---|---|---|
| **Banda / Frequência** | 2.4 GHz, 5.0 GHz (e 6.0 GHz em equipamentos Wi-Fi 6E/7) | Um mesmo SSID pode existir em duas bandas diferentes (ex.: `grupo-01-2.4` e `grupo-01-5.0`) — é preciso identificar corretamente qual banda está sendo analisada. |
| **Canal** | Subdivisão da banda usada pelo AP para transmitir | Em 2.4 GHz, apenas os canais **1, 6 e 11** não se sobrepõem; em 5.0 GHz há muito mais canais não sobrepostos. Vários grupos no mesmo laboratório podem gerar interferência mútua. |
| **Largura de Canal** | 20/40/80/160 MHz | Canais mais largos aumentam a velocidade, mas também aumentam a chance de sobreposição com AP's vizinhos — relevante ao comparar sinais no reconhecimento. |
| **BSSID** | Endereço MAC único do rádio do AP | Identificador confiável do equipamento — diferente do SSID, não muda mesmo que o nome da rede seja alterado ou oculto. |
| **SSID / ESSID** | Nome público da rede | Pode estar oculto (ver Aula de segurança wireless, item 01) — nesse caso, o reconhecimento passivo ainda revela o BSSID mesmo sem o nome aparecer diretamente. |
| **PWR (dBm)** | Intensidade do sinal recebido | Quanto mais próximo de `0`, mais forte o sinal (valores negativos, ex.: `-40` é mais forte que `-80`). Ajuda a estimar distância/posicionamento do AP. |
| **Beacons** | Quadros de anúncio periódicos do AP | Confirmam que o AP está ativo e transmitindo, mesmo sem clientes conectados no momento. |

> 💡 **Dica didática:** peça para os alunos desenharem um "mapa mental" simples do laboratório antes de rodar qualquer ferramenta — quantos APs esperam encontrar, em quais canais, e quais SSIDs. Depois, comparem esse mapa com o resultado real do reconhecimento.

---

## 03 - Reconhecimento de APs em Modo Gerenciado (sem modo monitor)

Antes de habilitar o modo monitor, é possível (e recomendado) fazer uma primeira leitura do ambiente usando a própria interface em **modo gerenciado** (managed) — sem qualquer captura de pacotes de terceiros, apenas consultando os APs que anunciam sinal.

### 🔧 Procedimento

```bash
# Escaneamento simples via NetworkManager (linha de comando)
nmcli device wifi list

# Escaneamento via ferramentas nativas do driver
sudo iw dev wlan0 scan | less
sudo iwlist wlan0 scan | grep -E "ESSID|Address|Channel|Signal"
```

| Comando | Descrição |
|---|---|
| `nmcli device wifi list` | Lista de forma simples e legível os APs visíveis, com SSID, canal, sinal e tipo de segurança — ótimo primeiro comando para ter uma visão geral rápida. |
| `iw dev wlan0 scan` | Escaneamento mais detalhado via `iw`, retornando BSSID, capacidades e frequência de cada AP encontrado. |
| `iwlist wlan0 scan` | Comando legado, ainda amplamente usado, filtrando os campos mais relevantes (`ESSID`, `Address`, `Channel`, `Signal`). |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O AP do grupo aparece na listagem do `nmcli` com o SSID esperado (ex.: `grupo-0x-2.4`). |
| 2 | Canal e nível de sinal exibidos são compatíveis com o que foi configurado no AP (ver Aula de configuração wireless). |

> 💡 **Vantagem desta etapa:** não exige modo monitor nem antena especial — pode ser feita até com a placa Wi-Fi interna do notebook, servindo como um primeiro "raio-x" do ambiente antes de partir para as ferramentas do `aircrack-ng`.

---

## 04 - Habilitando o Modo Monitor para Reconhecimento Passivo

Para um reconhecimento mais completo (incluindo AP's com SSID oculto, clientes conectados e detalhes de criptografia), é necessário colocar a antena externa em **modo monitor** — o mesmo procedimento que será reutilizado em todas as aulas de exploração.

### 🔧 Procedimento

```bash
# Verificar e finalizar processos conflitantes
airmon-ng check
airmon-ng check kill

# Habilitar o modo monitor na antena externa
airmon-ng start wlan1
```

> 💡 **Alternativa (drivers incompatíveis com airmon-ng):**
> ```bash
> ifconfig wlan1 down
> iwconfig wlan1 mode monitor
> ifconfig wlan1 up
> iwconfig   # confirmar o modo monitor
> ```

| Campo | Descrição |
|---|---|
| `wlan1mon` (ou similar) | Interface virtual criada em modo monitor, usada em todos os comandos de reconhecimento e, posteriormente, nas aulas de exploração. |

> ⚠️ **Importante:** nesta aula, o modo monitor é usado **apenas para observação** (reconhecimento passivo) — nenhum pacote de desautenticação, injeção ou ataque é enviado. O objetivo é exclusivamente mapear o ambiente.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | `iwconfig` exibe a interface em modo monitor (`Mode:Monitor`). |
| 2 | Nenhum processo do NetworkManager interfere na interface (`airmon-ng check` limpo). |

---

## 05 - Reconhecimento com Airodump-ng — Leitura de Campos

### 🔧 Procedimento

```bash
airodump-ng wlan1mon
```

Deixe o comando rodando por alguns minutos, observando a lista de APs (parte superior) e de clientes (parte inferior).

### 📋 Interpretando a Saída — Lista de APs

| Campo | Descrição |
|---|---|
| **BSSID** | MAC do AP — identificador único, mesmo se o SSID estiver oculto. |
| **PWR** | Potência do sinal recebido (dBm) — quanto mais próximo de `0`, mais forte. |
| **Beacons** | Quantidade de quadros de anúncio capturados — confirma que o AP está ativo. |
| **#Data, #/s** | Quantidade de pacotes de dados capturados e taxa por segundo — indica se há tráfego real acontecendo na rede (útil para saber se há atividade de clientes). |
| **CH** | Canal em que o AP está transmitindo. |
| **MB** | Taxa máxima suportada pelo AP (ex.: `54`, `130`, `270` Mbps), relacionada ao padrão Wi-Fi em uso. |
| **ENC** | Tipo de criptografia (`OPN` = aberta, `WEP`, `WPA`, `WPA2`, `WPA3`). |
| **CIPHER** | Cifra utilizada (`WEP`, `TKIP`, `CCMP`). |
| **AUTH** | Tipo de autenticação (`PSK`, `SAE`, `MGT`). |
| **ESSID** | Nome da rede — pode aparecer em branco/`<length: X>` quando o SSID está oculto (ver Aula de segurança wireless, item 01). |

> 💡 **SSID oculto na prática:** mesmo com o nome oculto, o `airodump-ng` continua exibindo o `BSSID`, o `CH` e o `ENC` do AP — a ocultação de SSID **não impede** o reconhecimento passivo, apenas o esconde de uma varredura casual (celular, por exemplo).

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O AP do grupo é localizado corretamente pelo `BSSID` e `CH` esperados (mesmo que o SSID esteja oculto). |
| 2 | O tipo de criptografia (`ENC`/`CIPHER`/`AUTH`) exibido corresponde ao que foi configurado no AP. |
| 3 | Os APs de outros grupos do laboratório também aparecem na varredura, permitindo avaliar o nível de congestionamento do ambiente. |

---

## 06 - Identificação de Clientes Conectados (STATION)

### 🔧 Procedimento

Na mesma execução do `airodump-ng`, observe a seção inferior da tela (**Station**):

```bash
airodump-ng wlan1mon
```

| Campo | Descrição |
|---|---|
| **BSSID** | AP ao qual o cliente está associado (ou `(not associated)` se ainda não estiver conectado a nenhum AP). |
| **STATION** | Endereço MAC do dispositivo cliente. |
| **PWR** | Intensidade do sinal do cliente em relação à antena de captura. |
| **Rate** | Taxa de transmissão/recepção do cliente naquele instante. |
| **Probes** | Nomes de redes que o dispositivo já "procurou" anteriormente (útil para identificar redes conhecidas pelo dispositivo, mesmo fora de alcance). |

> 💡 **Uso nas próximas aulas:** o MAC de um `STATION` associado ao AP-alvo é exatamente o dado necessário para os ataques de desautenticação e ARP replay das Aulas seguintes — por isso vale já se acostumar a localizá-lo aqui, em um contexto puramente de observação.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Ao menos um dispositivo cliente (ex.: smartphone de teste) aparece associado ao BSSID do AP do grupo. |
| 2 | O campo `Probes` é identificado e discutido em grupo (o que ele revela sobre o histórico de conexões de um dispositivo). |

---

## 07 - Intensidade de Sinal e Qualidade da Captura (PWR)

### 🔧 Procedimento

```bash
# Focar a captura em um único AP/canal para observar o PWR com mais estabilidade
airodump-ng -c <CANAL> --bssid <BSSID_DO_AP> wlan1mon
```

| Faixa de PWR (dBm) | Qualidade aproximada | Descrição |
|---|---|---|
| `-30` a `-50` | Excelente | Antena muito próxima do AP — sinal forte e estável. |
| `-50` a `-67` | Boa | Sinal adequado para a maioria dos usos, incluindo captura de handshake. |
| `-67` a `-75` | Razoável | Pode haver perda de pacotes; considerar aproximar a antena. |
| Abaixo de `-75` | Fraca | Captura pouco confiável — recomenda-se reposicionar a antena antes de prosseguir para as aulas de exploração. |

> 💡 **Dica prática:** peça para os alunos moverem fisicamente a antena (ou o notebook) enquanto observam o campo `PWR` variar em tempo real — isso cria uma percepção intuitiva de como a distância e obstáculos (paredes, metal) afetam o sinal, antes de depender dessa mesma antena para capturar um handshake nas próximas aulas.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O grupo identifica a posição no laboratório onde o `PWR` do AP-alvo é mais forte (mais próximo de `0`). |
| 2 | A posição de melhor sinal é registrada no relatório, para ser usada como referência nas próximas aulas. |

---

## 08 - Ferramentas Complementares de Apoio

| Ferramenta | Uso | Observação |
|---|---|---|
| **Wifiman** (Android/iOS/Desktop/Web) | Validação de cobertura, velocidade e qualidade de sinal do ponto de vista de um cliente comum | Já utilizado na Aula de configuração wireless — útil aqui para comparar a leitura "do lado do cliente" com a leitura "do lado do atacante" (`airodump-ng`). |
| `iwlist wlan0 scan` | Alternativa simples de escaneamento, sem modo monitor | Bom para conferência rápida quando não há antena externa disponível no momento. |
| `kismet` *(opcional, se instalado)* | Monitoramento passivo contínuo com interface web, incluindo detecção de dispositivos e alertas | Ferramenta mais avançada — pode ser citada como próximo passo para quem quiser aprofundar o reconhecimento passivo fora do escopo desta aula. |

> 💡 **Dica:** utilize o Wifiman **antes** e **depois** do reconhecimento com `airodump-ng` para comparar a experiência "visível ao usuário comum" com o que efetivamente é capturado com a antena em modo monitor — essa comparação é um ótimo gancho de discussão em sala.

---

## 09 - Documentando o Cenário — Relatório de Baseline

Antes de avançar para as aulas de exploração de protocolos, cada grupo deve preencher um **relatório de baseline** do próprio AP e do ambiente ao redor.

### 📋 Template de Baseline

| Campo | Valor Registrado |
|---|---|
| Hostname do notebook | |
| Interface(s) sem-fio identificada(s) | |
| Antena utilizada (modelo/chipset) | |
| BSSID do AP do grupo | |
| ESSID do AP do grupo (2.4 GHz / 5.0 GHz) | |
| Canal(is) em uso | |
| Largura de canal | |
| ENC / CIPHER / AUTH detectados | |
| PWR médio na posição de teste | |
| Quantidade de APs vizinhos detectados no mesmo canal | |
| Cliente(s) de teste associado(s) (MAC) | |
| Observações sobre congestionamento/interferência | |

> 💡 **Por que documentar:** esse baseline será comparado, na Aula de WEP (próxima), com o comportamento do ambiente durante um ataque ativo — permitindo discutir em grupo o que muda (ex.: aparecimento de pacotes de desautenticação, aumento de `#Data`, etc.) entre um cenário de observação passiva e um cenário de exploração ativa.

---

## ✅ Checklist Final da Aula

- [ ] Hardware do notebook verificado (`inxi`, `lscpu`, `free`, `df`)
- [ ] Todas as interfaces sem-fio (interna + antenas externas) identificadas e anotadas
- [ ] Reconhecimento inicial feito em modo gerenciado (`nmcli`, `iw scan`, `iwlist scan`)
- [ ] Modo monitor habilitado com sucesso (`wlan1mon` ou similar)
- [ ] AP do grupo localizado via `airodump-ng` (BSSID, canal, ENC/CIPHER/AUTH)
- [ ] Clientes conectados (STATION) identificados e MAC anotado
- [ ] Melhor posição de sinal (`PWR`) identificada e registrada
- [ ] Comparação feita entre leitura do Wifiman (lado cliente) e `airodump-ng` (lado atacante)
- [ ] Relatório de baseline preenchido com todos os campos do template

> 📌 **Próxima aula:** com o ambiente mapeado e o hardware validado, o grupo está pronto para iniciar a exploração de vulnerabilidades — começando pela quebra de senha WEP via criptoanálise de IVs.
