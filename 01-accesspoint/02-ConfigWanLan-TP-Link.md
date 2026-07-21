# 📡 Aula 02 — Configuração da WAN e LAN dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 02 — Configuração Básica da Rede WAN e LAN
> **Equipamentos homologados:** TP-Link Archer C50 e TP-Link Archer EC220-G5

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

1. [Configuração do Servidor DNS da Interface WAN](#01---configuração-do-servidor-dns-da-interface-de-rede-wan)
2. [Configuração da Interface de Rede LAN](#02---configuração-da-interface-de-rede-lan)
3. [Configuração do Servidor DHCP](#03---configuração-do-serviço-de-dhcp-server)
4. [Configuração do Servidor NTP](#04---configuração-do-serviço-de-ntp-server)
---

## 01 - Configuração do Servidor DNS da Interface de Rede WAN

### 🌐 Servidores DNS com Proteção Familiar/Conteúdo

> 💡 **Dica:** esses servidores DNS filtram automaticamente conteúdo adulto/malicioso na rede, sendo úteis para demonstrar filtragem de DNS em laboratório.

| Provedor | DNS Primário | DNS Secundário |
|---|---|---|
| **Cloudflare for Families** | `1.1.1.3` | `1.0.0.3` |
| **OpenDNS FamilyShield** | `208.67.222.123` | `208.67.220.123` |
| **AdGuard DNS (Proteção Familiar)** | `94.140.14.15` | `94.140.15.15` |
---

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Rede → Internet** |
| 2 | Clicar em: **Configurações avançadas** |
---

| Campo | Valor | Descrição |
|---|---|---|
| Endereço DNS | Utilize os Endereços DNS a seguir | Define que o equipamento não usará o DNS fornecido automaticamente pelo provedor (ISP), e sim os servidores DNS informados manualmente abaixo. |
| DNS Primário | `1.1.1.3` | Servidor DNS principal, consultado primeiro para resolver nomes de domínio (traduzir `www.site.com` em um endereço IP). Neste caso, é um DNS Cloudflare com filtro de conteúdo adulto (Family). |
| DNS Secundário | `1.0.0.3` | Servidor DNS de contingência, utilizado automaticamente caso o DNS Primário fique indisponível ou não responda, garantindo continuidade na resolução de nomes. |
| Nome Host | `ap-grupo-0X` | Nome (hostname) atribuído ao Access Point na rede, usado para identificá-lo facilmente entre outros dispositivos — por exemplo, em listas de clientes conectados, logs do roteador e ferramentas de gerenciamento de rede. |
---

➡️ Clicar em **`Salvar`**

> ⚠️ **Atenção:** substitua `ap-grupo-0X` pelo número real do grupo/equipamento em laboratório, para facilitar a identificação na rede.

---

## 02 - Configuração da Interface de Rede LAN

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Rede → LAN** |
---

| Campo | Valor | Descrição |
|---|---|---|
| Endereço IP | `172.16.xxx.254` | Endereço IP atribuído à interface LAN do Access Point. É por meio dele que os dispositivos da rede local o acessam (via navegador) e, geralmente, também atua como gateway padrão da rede interna. |
| Máscara de Sub-rede | `255.255.255.0` | Define o tamanho da rede local, delimitando quais endereços IP pertencem à mesma sub-rede (`/24`, ou seja, de `172.16.xxx.1` a `172.16.xxx.254`), permitindo até 254 hosts válidos nessa faixa. |
---

➡️ Clicar em **`Salvar`**

Ao salvar, será exibida a mensagem:

> *"O novo endereço IP será usado para acessar a página de gerenciamento da web a partir de agora. Salvar e fazer login novamente?"*

➡️ Clicar em **`OK`**

### ⚠️ Observações Importantes

| # | Observação |
|---|---|
| 1 | A partir deste momento, a **placa de rede do computador local** deve ser reconfigurada para a nova faixa de rede `172.16.xxx.0/24`, sendo necessário atualizar manualmente as configurações de IP local. |
| 2 | Fique atento ao **cache do navegador** — muitas vezes o acesso à página de gerenciamento falha justamente por conta de dados em cache da configuração anterior. Utilize a navegação anônima/privada ou limpe o cache se necessário. |
---

## 03 - Configuração do Serviço de DHCP Server

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Rede → Servidor DHCP** |
---

| Campo | Valor | Descrição |
|---|---|---|
| DHCP | 🟢 Habilitado (ON) | Ativa o serviço de atribuição automática de endereços IP na rede local, dispensando a configuração manual de IP em cada dispositivo cliente. |
| Pool Address IP | `172.16.xxx.100` – `172.16.xxx.150` | Faixa de endereços IP reservada para distribuição automática aos dispositivos que se conectam à rede. Neste caso, permite até 51 hosts simultâneos (`.100` a `.150`). |
| Tempo de Concessão de Endereço | `7200` segundos (120 minutos / 2 horas) | Período (*lease time*) durante o qual um IP fica reservado a um dispositivo antes de precisar ser renovado. Após esse tempo, o cliente solicita a renovação ou recebe um novo endereço do pool. |
| Gateway Padrão | `172.16.xxx.254` | Endereço IP repassado aos clientes DHCP como porta de saída da rede local — geralmente o próprio IP do Access Point/roteador, usado para rotear tráfego para fora da sub-rede (ex.: Internet). |
| DNS Primário | `172.16.xxx.254` | Servidor DNS informado aos clientes DHCP para resolução de nomes. Ao apontar para o próprio Access Point, ele pode atuar como *proxy DNS*, repassando as consultas ao DNS configurado na WAN. |
---

➡️ Clicar em **`Salvar`**

> 💡 **Dica:** o pool de endereços (`.100` a `.150`) define quantos dispositivos podem receber IP automaticamente. Ajuste a faixa conforme a quantidade de hosts esperada na rede do laboratório.

> Testar a conexão com o site: [My-DNS](https://www.top10vpn.com/tools/what-is-my-dns-server/)

---

## 04 - Configuração do Serviço de NTP Server

> 🔗 Referência oficial: [NTP.br](https://ntp.br/)

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Sistema → Hora e Idioma → Tempo de Sistema** |
---

| Campo | Valor | Descrição |
|---|---|---|
| Servidor NTP I | `a.st1.ntp.br` | Servidor NTP (*Network Time Protocol*) principal, do projeto NTP.br, utilizado para sincronizar automaticamente o relógio interno do Access Point com um horário oficial e preciso. |
| Servidor NTP II | `c.st1.ntp.br` | Servidor NTP secundário, usado como contingência caso o servidor primário fique indisponível, garantindo que o equipamento continue sincronizando seu horário corretamente. |
---

➡️ Clicar em **`Salvar`**

> 💡 **Dica:** a sincronização de horário via NTP é essencial para logs, certificados e autenticação corretos na rede — sempre configure esse serviço após ajustar WAN, LAN e DHCP.

---

## ✅ Checklist Final da Aula

- [ ] DNS da WAN configurado com servidor de proteção familiar
- [ ] Nome do host (`ap-grupo-0X`) definido corretamente
- [ ] Endereço IP da LAN alterado para `172.16.xxx.254/24`
- [ ] Placa de rede do computador reconfigurada para a nova faixa `172.16.xxx.0/24`
- [ ] Cache do navegador limpo (ou navegação anônima utilizada), se necessário
- [ ] Servidor DHCP habilitado com pool `172.16.xxx.100–150`
- [ ] Gateway padrão e DNS primário do DHCP apontando para `172.16.xxx.254`
- [ ] Servidores NTP (`a.st1.ntp.br` e `c.st1.ntp.br`) configurados

> 📌 **Próxima aula:** configuração manual da rede Wi-Fi (SSID, senha, canais 2.4 GHz e 5.0 GHz).
