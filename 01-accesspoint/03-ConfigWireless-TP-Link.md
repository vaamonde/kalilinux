# 📡 Aula 03 — Configuração do Wireless dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 03 — Configuração da Rede Sem-Fio Wireless
> **Equipamentos homologados:** TP-Link Archer C50 e TP-Link Archer EC220-G5

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 20/07/2026 |
| **Data de atualização** | 20/07/2026 |
| **Versão** | 0.01 |
| **Equipamentos testados** | Archer C50 (W) e Archer EC220-G5 |

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

## 01 - Configuração da Rede Sem-Fio 2.4 GHz

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** |

### 📶 Parâmetros da Banda 2.4 GHz

| Campo | Valor |
|---|---|
| 2.4 GHz | 🟢 Habilitado (ON) |
| Nome de Rede (SSID) | `grupo-0x-2.4` |
| Segurança | WPA2/WPA3-Pessoal |
| Versão | Automático |
| Senha | `Senactit@123` |
| Poder de Transmissão | Baixa |
| Largura do Canal | 20 MHz |
| Canal | 1 |
| Modo | Somente 802.11n (Wi-Fi 4) |

➡️ Clicar em **`Salvar`**

> 💡 **Dica:** substitua `0x` no SSID pelo número real do grupo (ex.: `grupo-01-2.4`) para identificar cada equipamento no laboratório.

---

## 02 - Configuração da Rede Sem-Fio 5.0 GHz

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** |

### 📶 Parâmetros da Banda 5.0 GHz

| Campo | Valor |
|---|---|
| 5.0 GHz | 🟢 Habilitado (ON) |
| Nome de Rede (SSID) | `grupo-0x-5.0` |
| Segurança | WPA2/WPA3-Pessoal |
| Versão | Automático |
| Senha | `Senactit@123` |
| Poder de Transmissão | Média |
| Largura do Canal | 20 MHz |
| Canal | 36 |
| Modo | Somente 802.11a/n/ac (Wi-Fi 4/Wi-Fi 5) |
| MU-MIMO | 🟢 Habilitado (ON) |

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
