# 📡 Aula 04 — Configuração de Segurança Básica do Wireless dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 03 — Configuração da segurança básico da Rede Sem-Fio Wireless
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

Monitoramento de rede:

Avançado, Rede, Servidor DHCP
#06_ Monitoramento do Leases (ALugueis) do DHCP Server
  Lista de Clientes DHCP

Observação: verificar os nomes, endereços IPv4/IPv6 e MAC Address dos dispositivos
------------------------------------------------------------

Segurança Básica de Acesso a Rede

Avançado, Segurança, Controle de Acesso
#07_ Trabalhando com Listas Negras e Brancas
  Controle de Acesso: (ON) Habilitado
  Método de Acesso: Lista de Bloqueio (Negras)
  <Adicionar>
    Selecionar o dispositivo
  <Adicionar>

Observação: Verificar queda de conexão no dispositivo da rede
--------------------------------------------

Avançado, Segurança, Vinculo IP e MAC
#08_ Vincular o Endereço IP ao MAC
  Vínculo IP e MAC: (ON) Habilitado
  Lista ARP
    Selecionar o dispositivo: (ON) Vincular

Observação: cuidado com dispositivos que tenha vários MAC Address
---------------------------------------------

Avançado, Controle dos Pais
#09_ Configuração do Controle dos Pais
	Controle dos Pais
	<Adicionar>
    Criar o perfil: 
-------------------------------------------------------------