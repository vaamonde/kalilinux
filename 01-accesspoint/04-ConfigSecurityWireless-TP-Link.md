# 📡 Aula 04 — Configuração de Segurança Básica do Wireless dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 04 — Configuração da Segurança Básica da Rede Sem-Fio Wireless
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 22/07/2026 |
| **Data de atualização** | 22/07/2026 |
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

1. [Ocultação do SSID](#01---ocultação-do-ssid-das-redes-24-ghz-e-50-ghz)
2. [Agenda Wireless](#02---agenda-wireless-horário-de-funcionamento)
3. [Desativação do WPS](#03---desativação-do-wps)
4. [Isolamento de AP e Airtime Fairness](#04---configurações-adicionais-isolamento-de-ap-e-airtime-fairness)
5. [Rede para Convidados](#05---rede-para-convidados-guest-network)

---

## 01 - Ocultação do SSID das Redes 2.4 GHz e 5.0 GHz

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Wireless** |
| 2 | Desmarcar/ocultar o SSID das bandas **2.4 GHz** e **5.0 GHz** |

| Campo | Valor | Descrição |
|---|---|---|
| Ocultar SSID (2.4 GHz e 5.0 GHz) | 🔴 Não Divulgar | O nome da rede deixa de ser transmitido em *broadcast*, não aparecendo automaticamente na lista de redes Wi-Fi dos dispositivos. A conexão só é possível informando manualmente o SSID e a senha. |

➡️ Clicar em **`Salvar`**

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Verificar no **smartphone** que as redes `grupo-0x-2.4` e `grupo-0x-5.0` não aparecem mais na lista de redes disponíveis. |
| 2 | Tentar conectar-se **digitando manualmente** o nome do SSID e a senha, confirmando que a rede continua acessível mesmo oculta. |

> ⚠️ **Atenção:** ocultar o SSID **não é uma medida de segurança forte** — a rede continua detectável por ferramentas de análise de tráfego sem-fio (*sniffers*). Trata-se apenas de uma camada extra que reduz a visibilidade casual da rede.

---

## 02 - Agenda Wireless (Horário de Funcionamento)

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Agenda Wireless** |
| 2 | Habilitar a **Agenda Wireless** e clicar em **`Adicionar`** |

| Campo | Valor | Descrição |
|---|---|---|
| Agenda Wireless | 🟢 Habilitado (ON) | Ativa a função de desligamento programado da rede sem-fio, controlando automaticamente os horários em que o Wi-Fi fica disponível. |
| Horário de Desligamento | Das `18h` às `22h` | Período em que o rádio Wi-Fi (2.4 GHz e/ou 5.0 GHz) permanece desligado, cortando o sinal sem-fio automaticamente. |
| Repetir | Todos os dias (Dom, Seg, Ter, Qua, Qui, Sex, Sáb) | Define os dias da semana em que a regra de agendamento será aplicada. |

➡️ Clicar em **`Salvar`** (na regra) e depois em **`Salvar`** (na tela principal)

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Verificar os **LEDs do roteador** durante o horário programado, confirmando que o indicador de Wi-Fi se apaga. |
| 2 | Verificar a **queda de conexão** dos dispositivos já conectados no início do período de desligamento. |

> 💡 **Dica:** esse recurso é útil em ambientes escolares, corporativos ou residenciais para reduzir consumo de energia e limitar o acesso à rede fora do horário desejado (ex.: durante a noite).

---

## 03 - Desativação do WPS

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → WPS** |

| Campo | Valor | Descrição |
|---|---|---|
| WPS (Wi-Fi Protected Setup) | 🔴 Desabilitado (OFF) | Desativa o método simplificado de conexão via PIN ou botão físico, que é considerado um ponto vulnerável de segurança em redes Wi-Fi. |

> ⚠️ **Atenção:** o WPS possui vulnerabilidades conhecidas (como ataques de força bruta ao PIN), permitindo que invasores obtenham a senha da rede. **Não é recomendado manter o WPS ativado**, exceto quando estritamente necessário para pareamento rápido de dispositivos IoT em ambientes controlados.

---

## 04 - Configurações Adicionais: Isolamento de AP e Airtime Fairness

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Configurações Adicionais** |

| Campo | Valor | Descrição |
|---|---|---|
| Isolamento de AP — 2.4 GHz | 🟢 Habilitado (ON) | Impede que os dispositivos conectados à mesma rede Wi-Fi (2.4 GHz) se enxerguem e se comuniquem diretamente entre si, isolando cada cliente — comum em redes de convidados e ambientes públicos. |
| Airtime Fairness — 2.4 GHz | 🟢 Habilitado (ON) | Distribui o tempo de transmissão do rádio de forma equilibrada entre os dispositivos conectados, evitando que um único cliente mais lento (ex.: 802.11n) prejudique o desempenho dos demais. |
| Isolamento de AP — 5.0 GHz | 🟢 Habilitado (ON) | Mesma função do isolamento de AP, aplicada à banda de 5.0 GHz. |
| Airtime Fairness — 5.0 GHz | 🟢 Habilitado (ON) | Mesma função do Airtime Fairness, aplicada à banda de 5.0 GHz. |

➡️ Clicar em **`Salvar`**

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Testar o **ping** entre dois dispositivos conectados na mesma rede, confirmando que a comunicação direta entre eles não é possível (isolamento de AP ativo). |
| 2 | Verificar no **Wifiman**, na varredura de rede, que apenas o **próprio dispositivo** e o **roteador** aparecem — os demais clientes ficam ocultos entre si. |

---

## 05 - Rede para Convidados (Guest Network)

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Wireless → Rede para Convidados** |

| Campo | Valor | Descrição |
|---|---|---|
| Rede de Convidados — 2.4 GHz | 🔴 Desabilitado (OFF) | A rede de convidados não será disponibilizada na banda 2.4 GHz, restringindo o acesso de visitantes apenas à banda 5.0 GHz. |
| Rede de Convidados — 5.0 GHz | 🟢 Habilitado (ON) | Cria uma rede sem-fio separada e isolada da rede principal, destinada a visitantes, sem expor a rede interna/administrativa. |
| Nome de Rede (SSID) | `guest-grupo-01-5.0` | Nome público da rede de convidados, exibido separadamente da rede principal na lista de redes disponíveis. |
| Segurança | WPA2/WPA3-Pessoal | Protocolo de criptografia e autenticação aplicado também à rede de convidados, mantendo o acesso protegido por senha. |
| Senha | `Senactit@1123` | Chave de acesso (*PSK*) específica da rede de convidados, diferente da senha da rede principal. |

### 🔐 Permissões de Convidado

| Campo | Valor | Descrição |
|---|---|---|
| Permitir convidados verem uns aos outros | 🔴 Desabilitado (OFF) | Impede que dispositivos conectados à rede de convidados se enxerguem e se comuniquem entre si, isolando cada visitante (semelhante ao isolamento de AP). |
| Permitir que os convidados acessem a rede local | 🔴 Desabilitado (OFF) | Bloqueia o acesso dos dispositivos convidados aos recursos da rede interna (LAN), como compartilhamentos de arquivos, impressoras e outros equipamentos administrativos. |

➡️ Clicar em **`Salvar`**

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Conectar um dispositivo à rede `guest-grupo-01-5.0` e confirmar o acesso à Internet. |
| 2 | Testar a **comunicação entre dispositivos** conectados na rede de convidados, confirmando que o isolamento impede o tráfego entre eles. |

> 💡 **Dica:** a rede de convidados é uma boa prática de segurança para separar visitantes e dispositivos não confiáveis da rede administrativa/interna, reduzindo a superfície de ataque em caso de dispositivos comprometidos.

---

## ✅ Checklist Final da Aula

- [ ] SSID das bandas 2.4 GHz e 5.0 GHz ocultado e testado
- [ ] Agenda Wireless configurada (desligamento das 18h às 22h, todos os dias)
- [ ] Queda de conexão e LEDs verificados durante o horário programado
- [ ] WPS desativado em ambas as bandas
- [ ] Isolamento de AP e Airtime Fairness habilitados em 2.4 GHz e 5.0 GHz
- [ ] Isolamento testado via ping entre dispositivos e validado no Wifiman
- [ ] Rede de convidados habilitada apenas em 5.0 GHz, com SSID e senha próprios
- [ ] Permissões de convidado restritas (sem visibilidade entre convidados e sem acesso à LAN)
- [ ] Conexão e isolamento da rede de convidados testados

> 📌 **Próxima aula:** testes avançados de desempenho e segurança da rede sem-fio configurada.
