# 📡 Aula 01 — Reset e Configuração Básica de Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 01 — Reset e Configuração Básica
> **Equipamentos homologados:** TP-Link Archer EC220-G5
---

## ℹ️ Informações do Documento

| Campo | Descrição |
|-------|-----------|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 20/07/2026 |
| **Data de atualização** | 18/08/2026 |
| **Versão** | 0.04 |
| **Equipamentos testados** | TP-Link Archer EC220-G5 - Versão 3.0 PT-BR |
---

### 🔗 Links do Autor

| Canal | Link |
|-------|------|
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

1. [Reset dos Access Points TP-Link Archer](#01---reset-dos-access-points-tp-link-archer)
2. [Testando a Conexão de Rede](#02---testando-a-conexão-de-rede-dos-access-points-tp-link-archer)
3. [Acessando via Navegador Web](#03---acessando-via-web-browser-os-access-points-tp-link-archer)
---

## 01 - Reset dos Access Points TP-Link Archer

### 📋 Especificações dos Equipamentos

| Modelo | Padrão Wi-Fi | Velocidade | Datasheet | Manual |
|--------|--------------|------------|-----------|--------|
| **TP-Link Archer EC220-G5** | Wi-Fi 5 — 802.11ac | AC1200 | [📄 Datasheet](https://static.tp-link.com/upload/product-overview/2025/202505/20250527/Datasheet_EC220-G5.pdf) | [📘 Manual (BBA Mesh)](https://static.tp-link.com/upload/manual/2026/202607/20260714/BBA%20Mesh_UG_REV1.0.1.pdf) |
---

### 🔧 Procedimento de Reset de Fábrica

> ⚠️ **Atenção:** o reset de fábrica apaga **todas as configurações do equipamento** (senhas, rede Wi-Fi, IPs configurados, etc.), retornando-o ao estado original de fábrica. Use somente quando necessário.
---

> ⚠️ **Atenção:** o modelo **TP-Link EC220-G5** não tem o Botão __`Liga e Desligada`__ e utiliza Fonte de **12V**
---

1. Localize o botão **Reset** (geralmente um orifício pequeno na parte traseira ou inferior do equipamento).
2. Com o Access Point **ligado**, utilize um clipe ou objeto pontiagudo para pressionar o botão.
3. Mantenha pressionado por **aproximadamente 5 a 10 segundos**, até que os LEDs do equipamento pisquem simultaneamente.
4. Solte o botão e aguarde o reinício completo do equipamento (cerca de 1 a 2 minutos).

> 💡 **Dica:** após o reset, o Access Point volta a utilizar o **IP padrão de fábrica** (geralmente `192.168.0.1` ou `192.168.1.1`, dependendo do modelo) e a rede Wi-Fi padrão impressa na etiqueta do equipamento.
---

## 02 - Testando a Conexão de Rede dos Access Points TP-Link Archer

> 💡 **Dica:** Antes de acessar a interface web do Access Point, é importante confirmar se o computador está corretamente conectado à rede.
---

### 🪟 01. Microsoft Windows (10 ou 11)

| Passo | Ação |
|-------|------|
| 1 | Abrir o **PowerShell** (Pesquisa do Windows → `Powershell`) |
| 2 | Executar o comando: `ipconfig /all` |
| 3 | Verificar o campo: **`Gateway Padrão`** |
---

### 🐧 02. GNU/Linux (Linux Mint ou Kali Linux)

| Passo | Ação |
|-------|------|
| 1 | Abrir o **Terminal** (atalho `Ctrl + Alt + T`) |
| 2 | Executar o comando: `ip route show` |
| 3 | Verificar o campo: **`default via`** |
---

### 🌐 03. Testando a Comunicação com o Access Point

| Passo | Ação |
|-------|------|
| 1 | Executar o comando: `ping 192.168.0.1` |
---

> ✅ **Resultado esperado:** respostas de ping bem-sucedidas (sem *timeout* ou *destination unreachable*) confirmam que o computador está se comunicando corretamente com o Access Point.
---

## 03 - Acessando via Web Browser os Access Points TP-Link Archer

### 🖥️ 01. Acesso à Interface Web

| Item | Valor |
|------|-------|
| **Navegadores recomendados** | `Google Chrome ou Mozilla Firefox` |
| **Endereço de acesso** | `192.168.0.1` |
---

### 🔐 02. Criando a Senha do Usuário Administrador

| Campo | Valor |
|-------|-------|
| **Nova Senha** | `Senactit@123` |
| **Confirmar Senha** | `Senactit@123` |
---

> ⚠️ **Importante:** utilize sempre senhas fortes em ambientes de produção. A senha acima é padronizada apenas para fins didáticos desta aula.
---

### 🌎 03. Selecionando o Fuso Horário

| Campo | Valor |
|-------|-------|
| **Fuso Horário** | `(UTC-03:00) Brasília` |
---

➡️ Clicar em **`Próximo`**

### 🌐 04. Tipo de Conexão (WAN — ISP)

| Campo | Valor |
|-------|-------|
| **Tipo de Conexão** | `IP Dinâmico` |
---

| **Valor** | **Descrição** |
| --------- | ------------- |
| **IP Dinâmico (DHCP)** | Obtém automaticamente o endereço IP, gateway, DNS e demais parâmetros de rede do provedor de Internet (ISP). |
| **IP Estático** | Requer a configuração manual do endereço IP, máscara de rede, gateway e servidores DNS fornecidos pelo provedor. |
| **PPPoE** | Estabelece a conexão utilizando o protocolo PPPoE (Point-to-Point Protocol over Ethernet), utiliza autenticação por nome de usuário e senha para estabelecer a conexão com o provedor de Internet. |
| **L2TP** | Estabelece a conexão utilizando o protocolo L2TP (Layer 2 Tunneling Protocol), normalmente exigindo servidor, usuário e senha fornecidos pelo ISP. |
| **PPTP** | Estabelece a conexão utilizando o protocolo PPTP (Point-to-Point Tunneling Protocol), exigindo endereço do servidor, usuário e senha fornecidos pelo ISP. |
---

➡️ Clicar em **`Próximo`**

### 🔧 06. Configuração do IP Dinâmico (WAN — ISP)

| Campo | Valor |
|-------|-------|
| **Endereço MAC do Roteador** | `Utilizar Endereço MAC Padrão` |
---

| **Valor** | **Descrição** |
| --------- | ------------- |
| **Nenhum** | Utiliza a configuração padrão da conexão com a Internet, sem aplicar perfis de IPTV, VLAN ou modo de ponte. |
| **Personalizado** | Permite configurar manualmente os parâmetros de IPTV e VLAN de acordo com os requisitos do provedor de Internet (ISP). |
| **Ponte (Bridge)** | Encaminha o tráfego diretamente para outro equipamento, como um roteador ou firewall, sem realizar roteamento ou NAT para a conexão configurada. |
---

➡️ Clicar em **`Próximo`**

### 📶 06. Personalizando as Configurações Sem Fio (WLAN)

| Configuração | Status |
|--------------|--------|
| **Direção de Banda (Band Steering)** | 🔴 `Desabilitado (OFF)` |
| **Rede 2.4 GHz** | 🔴 `Desabilitado (OFF)` |
| **Rede 5.0 GHz** | 🔴 `Desabilitado (OFF)` |
---

> 💡 **Observação:** as redes sem fio são desabilitadas propositalmente nesta etapa para que a configuração da WLAN seja realizada manualmente, reforçando o entendimento de cada parâmetro.
---

➡️ Clicar em **`Próximo`**

### ⏳ 07. Personalizando as Configurações Sem Fio (WLAN)

| Status |
|--------|
| Escolha sua conexão atual para continuar:|
| ✅ (ON) Estou usando uma conexão cabeada. |
---

➡️ Clicar em **`Próximo`**

### ⏳ 08. Conectando à Internet

| Status |
|--------|
| Aguardando conexão... |
| ✅ Concluído com sucesso |
---

➡️ Clicar em **`Próximo`**

### ☁️ 08. Serviço TP-Link Cloud

| Ação |
|------|
| **Não será utilizado** nesta aula |
---

➡️ Clicar em **`Pular`**

---

## ✅ Checklist Final da Aula

- [ ] Reset de fábrica realizado com sucesso
- [ ] Conexão de rede testada via `ping`
- [ ] Acesso à interface web (`192.168.0.1`) realizado
- [ ] Senha de administrador definida
- [ ] Fuso horário configurado (Brasília — UTC-03:00)
- [ ] Tipo de conexão WAN definido (IP Dinâmico)
- [ ] WLAN 2.4 GHz e 5.0 GHz desabilitadas propositalmente
- [ ] Etapa do TP-Link Cloud pulada