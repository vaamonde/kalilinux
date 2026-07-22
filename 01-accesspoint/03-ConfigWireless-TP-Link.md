# 📡 Aula 03 — Configuração do Wireless dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 03 — Configuração da Rede Sem-Fio Wireless
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 20/07/2026 |
| **Data de atualização** | 21/07/2026 |
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

## 📑 Sumário

1. [Configuração da Rede Sem-Fio 2.4 GHz](#01---configuração-da-rede-sem-fio-24-ghz)
2. [Configuração da Rede Sem-Fio 5.0 GHz](#02---configuração-da-rede-sem-fio-50-ghz)
3. [Ferramenta de Teste — Wifiman](#-ferramenta-de-teste-unifi-wifiman)

---

| Normas/Padrão | Frequência | Descrição |
|---|---|---|
| **Wi-Fi 4** (802.11n) | `2.4 GHz e 5.0 GHz` | Primeiro padrão a suportar MIMO (múltiplas antenas), aumentando velocidade e estabilidade. Velocidade teórica de até 600 Mbps. Ainda muito usado em dispositivos mais antigos e IoT. |
| **Wi-Fi 5** (802.11ac) | `5.0 GHz` | Padrão focado em alta velocidade, com canais mais largos (até 160 MHz) e MU-MIMO. Velocidade teórica de até 3.5 Gbps. Não opera em 2.4 GHz. |
| **Wi-Fi 6** (802.11ax) | `2.4 GHz e 5.0 GHz` | Introduz o OFDMA (divide o canal entre vários dispositivos simultaneamente) e o TWT (economia de energia), melhorando desempenho em redes com muitos dispositivos conectados. Velocidade teórica de até 9.6 Gbps. |
| **Wi-Fi 6E** (802.11ax) | `2.4 GHz, 5.0 GHz e 6.0 GHz` | Mesma tecnologia do Wi-Fi 6, porém estendida para a nova faixa de 6 GHz, com canais mais largos e livres de interferência de redes antigas (2.4/5 GHz), exigindo dispositivos compatíveis com essa faixa. |
| **Wi-Fi 7** (802.11be) | `2.4 GHz, 5.0 GHz e 6.0 GHz` | Padrão mais recente, com canais de até 320 MHz, MLO (*Multi-Link Operation* — uso simultâneo de várias bandas por um mesmo dispositivo) e latência ultrabaixa. Velocidade teórica acima de 30 Gbps. |
---

> 💡 **Dica:** quanto mais recente o padrão (Wi-Fi 4 → 7), maior a velocidade e a eficiência, mas a compatibilidade com a faixa de 6 GHz depende tanto do roteador quanto do dispositivo cliente ser Wi-Fi 6E ou Wi-Fi 7.

| Frequência | Largura de Canal / Quantidade | Descrição |
|---|---|---|
| **2.4 GHz** | `20 MHz — 11 a 14 canais (varia por região; Brasil: 1 a 13)` | Faixa mais estreita e congestionada. Os canais se sobrepõem entre si, restando apenas **3 canais sem sobreposição: 1, 6 e 11**. Recomendado usar somente esses três em ambientes com muitas redes próximas. |
| **5.0 GHz** | `20/40/80/160 MHz — até 25 canais (dependendo da largura escolhida)` | Faixa mais ampla, com muito mais canais disponíveis e menos congestionamento. Com largura de 20 MHz, há até **25 canais sem sobreposição** (ex.: 36, 40, 44, 48, 149, 153, 157, 161...). Alguns canais (52–144) exigem DFS (detecção de radar), podendo causar mudança automática de canal. |
| **6.0 GHz** | `20/40/80/160/320 MHz — até 59 canais (largura de 20 MHz)` | Faixa mais recente e limpa, sem interferência de redes legadas (2.4/5 GHz). Oferece a maior quantidade de **canais sem sobreposição** de todas (até 59 com 20 MHz, ou 7 canais com 160 MHz), ideal para ambientes densos. Requer dispositivos Wi-Fi 6E ou Wi-Fi 7. |
---

> 💡 **Dica:** quanto mais larga a faixa de frequência (2.4 → 5 → 6 GHz), maior a quantidade de canais disponíveis e menor a chance de sobreposição/interferência entre redes vizinhas — por isso a banda 5 GHz e, principalmente, a 6 GHz são preferíveis em ambientes com muitos Access Points próximos (como salas de aula e escritórios).
---

## 01 - Configuração da Rede Sem-Fio 2.4 GHz

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** |
---

### 📶 Parâmetros da Banda 2.4 GHz

| Campo | Valor | Descrição |
|---|---|---|
| 2.4 GHz | 🟢 `Habilitado (ON)` | Ativa a transmissão da rede sem-fio na banda de 2.4 GHz, faixa com maior alcance e melhor penetração em paredes, porém mais sujeita a interferências (outros roteadores, Bluetooth, micro-ondas). |
| Nome de Rede (SSID) | `grupo-0x-2.4` | Nome público da rede sem-fio, exibido na lista de redes disponíveis dos dispositivos clientes ao procurar por Wi-Fi. |
| Segurança | `WPA2/WPA3-Pessoal` | Define o protocolo de criptografia e autenticação da rede. O modo misto permite que dispositivos mais antigos (WPA2) e mais novos (WPA3) se conectem com segurança. |
| Versão | `Automático` | O Access Point escolhe automaticamente qual versão do protocolo de segurança (WPA2 ou WPA3) utilizar, de acordo com a capacidade de cada dispositivo cliente. |
| Senha | `Senactit@123` | Chave de acesso (*PSK — Pre-Shared Key*) exigida dos dispositivos para se conectarem e autenticarem na rede sem-fio. |
| Poder de Transmissão | `Baixa` | Reduz a potência do sinal irradiado pela antena, diminuindo o alcance da rede — útil em laboratórios para evitar interferência entre equipamentos próximos. |
| Largura do Canal | `20 MHz` | Define a "largura" do canal de rádio utilizado. Canais mais estreitos (20 MHz) sofrem menos interferência, mas oferecem menor taxa de transmissão em comparação a canais mais largos (40/80 MHz). |
| Canal | `1` | Frequência específica dentro da banda 2.4 GHz utilizada para transmissão. Os canais 1, 6 e 11 são os únicos que não se sobrepõem entre si nessa banda, sendo os mais recomendados. |
| Modo | `Somente 802.11n (Wi-Fi 4)` | Restringe a conexão apenas a dispositivos compatíveis com o padrão 802.11n (Wi-Fi 4), impedindo conexões de equipamentos muito antigos (802.11b/g) que reduziriam o desempenho geral da rede. |
---

➡️ Clicar em **`Salvar`**

> 💡 **Dica:** substitua `0x` no SSID pelo número real do grupo (ex.: `grupo-01-2.4`) para identificar cada equipamento no laboratório.
---

## 02 - Configuração da Rede Sem-Fio 5.0 GHz

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** |
---

### 📶 Parâmetros da Banda 5.0 GHz

| Campo | Valor | Descrição |
|---|---|---|
| 5.0 GHz | 🟢 `Habilitado (ON)` | Ativa a transmissão da rede sem-fio na banda de 5.0 GHz, faixa com maior velocidade de transmissão e menos interferência, porém com menor alcance e penetração em paredes se comparada à banda 2.4 GHz. |
| Nome de Rede (SSID) | `grupo-0x-5.0` | Nome público da rede sem-fio, exibido na lista de redes disponíveis dos dispositivos clientes ao procurar por Wi-Fi. O sufixo `5.0` diferencia essa rede da banda 2.4 GHz. |
| Segurança | `WPA2/WPA3-Pessoal` | Define o protocolo de criptografia e autenticação da rede. O modo misto permite que dispositivos mais antigos (WPA2) e mais novos (WPA3) se conectem com segurança. |
| Versão | `Automático` | O Access Point escolhe automaticamente qual versão do protocolo de segurança (WPA2 ou WPA3) utilizar, de acordo com a capacidade de cada dispositivo cliente. |
| Senha | `Senactit@123` | Chave de acesso (*PSK — Pre-Shared Key*) exigida dos dispositivos para se conectarem e autenticarem na rede sem-fio. |
| Poder de Transmissão | `Média` | Define a potência do sinal irradiado pela antena nessa banda. Um nível médio equilibra alcance e economia de energia, adequado ao ambiente de laboratório. |
| Largura do Canal | `20 MHz` | Define a "largura" do canal de rádio utilizado. Na banda 5 GHz é possível usar canais mais largos (40/80/160 MHz) para maior taxa de transmissão, mas 20 MHz reduz interferência entre equipamentos próximos. |
| Canal | `36` | Frequência específica dentro da banda 5.0 GHz utilizada para transmissão. O canal 36 pertence à faixa UNII-1, de uso livre e sem restrições de radar (DFS), sendo uma escolha estável para ambientes internos. |
| Modo | `Somente 802.11a/n/ac (Wi-Fi 4/Wi-Fi 5)` | Restringe a conexão apenas a dispositivos compatíveis com os padrões 802.11a/n/ac, impedindo conexões de equipamentos muito antigos e mantendo maior desempenho na banda 5 GHz. |
| MU-MIMO | 🟢 `Habilitado (ON)` | *Multi-User MIMO* — tecnologia que permite ao Access Point transmitir dados simultaneamente para múltiplos dispositivos clientes ao mesmo tempo, em vez de atendê-los sequencialmente, melhorando o desempenho em redes com vários usuários conectados. |
---

➡️ Clicar em **`Salvar`**

---

## 📱 Ferramenta de Teste — Unifi Wifiman

Após salvar as configurações de cada banda (2.4 GHz e 5.0 GHz), utilize o aplicativo **Wifiman** para validar cobertura, velocidade e qualidade do sinal sem-fio.

| Plataforma | Link |
|---|---|
| 🤖 Android | https://play.google.com/store/apps/details?id=com.ubnt.usurvey&hl=pt_BR |
| 🍎 Apple iPhone | https://apps.apple.com/br/app/ubiquiti-wifiman/id1385561119#information |
| 💻 Desktop (Windows, Mac e Linux) | https://ui.com/download/app/wifiman-desktop |
| 🌐 Teste on-line | https://wifiman.com/ |
---

> 💡 **Dica:** repita o teste de conexão para **cada banda separadamente** (2.4 GHz e 5.0 GHz), comparando velocidade, intensidade de sinal e alcance entre elas.
---

## ✅ Checklist Final da Aula

- [ ] Rede 2.4 GHz habilitada com SSID `grupo-0x-2.4`
- [ ] Segurança WPA2/WPA3-Pessoal configurada na banda 2.4 GHz
- [ ] Canal 1 e largura 20 MHz definidos na banda 2.4 GHz
- [ ] Rede 5.0 GHz habilitada com SSID `grupo-0x-5.0` (nome diferente da rede 2.4 GHz)
- [ ] Segurança WPA2/WPA3-Pessoal configurada na banda 5.0 GHz
- [ ] Canal 36, largura 20 MHz e MU-MIMO habilitados na banda 5.0 GHz
- [ ] Conexão testada em ambas as bandas com o aplicativo Wifiman

> 📌 **Próxima aula:** testes avançados de desempenho e segurança da rede sem-fio configurada.
