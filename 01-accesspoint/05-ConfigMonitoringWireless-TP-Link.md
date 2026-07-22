# 📡 Aula 05 — Monitoramento e Controle de Acesso à Rede dos Access Points TP-Link

> **Módulo:** Redes Sem-Fio
> **Aula:** 05 — Monitoramento de Rede e Segurança Básica de Acesso
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

1. [Monitoramento dos Leases (Aluguéis) do DHCP Server](#06---monitoramento-dos-leases-aluguéis-do-dhcp-server)
2. [Controle de Acesso — Listas Negras e Brancas](#07---controle-de-acesso-listas-negras-e-brancas)
3. [Vínculo de Endereço IP e MAC](#08---vínculo-de-endereço-ip-e-mac)
4. [Controle dos Pais (Parental Control)](#09---controle-dos-pais-parental-control)

---

## 06 - Monitoramento dos Leases (Aluguéis) do DHCP Server

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Rede → Servidor DHCP** |
| 2 | Consultar a **Lista de Clientes DHCP** |

| Campo | Descrição |
|---|---|
| Lista de Clientes DHCP | Exibe todos os dispositivos que receberam um endereço IP automaticamente do servidor DHCP, permitindo auditar em tempo real quem está conectado à rede. |
| Nome do Dispositivo | Identifica o *hostname* de cada cliente (ex.: nome do smartphone, notebook), facilitando o reconhecimento visual do equipamento na rede. |
| Endereço IPv4 / IPv6 | Endereço IP atribuído a cada dispositivo no momento da concessão (*lease*), usado para localizá-lo e gerenciá-lo dentro da rede. |
| MAC Address | Endereço físico (*hardware*) único da interface de rede do dispositivo, utilizado como identificador para filtros de acesso, vínculo IP/MAC e controle dos pais. |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Verificar na lista os **nomes**, **endereços IPv4/IPv6** e **MAC Address** de cada dispositivo conectado à rede. |

> 💡 **Dica:** o monitoramento dos leases é o primeiro passo para qualquer política de segurança de acesso — é a partir dessa lista que se obtém o MAC Address necessário para as próximas configurações (controle de acesso, vínculo IP/MAC e controle dos pais).

---

## 07 - Controle de Acesso — Listas Negras e Brancas

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Segurança → Controle de Acesso** |
| 2 | Clicar em **`Adicionar`** e selecionar o dispositivo desejado |

| Campo | Valor | Descrição |
|---|---|---|
| Controle de Acesso | 🟢 Habilitado (ON) | Ativa o filtro de acesso à rede sem-fio com base no MAC Address dos dispositivos. |
| Método de Acesso | Lista de Bloqueio (Negra) | Bloqueia especificamente os dispositivos adicionados à lista, **permitindo o acesso de todos os demais**. O modo alternativo — Lista de Permissão (Branca) — faz o inverso: só libera os dispositivos cadastrados, bloqueando todo o restante. |

➡️ Clicar em **`Adicionar`**

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Verificar a **queda de conexão** do dispositivo adicionado à lista de bloqueio, confirmando que ele perde o acesso à rede. |

> 💡 **Dica para sala de aula:** explique a diferença entre os dois modos — **Lista Negra (Blacklist)** é indicada quando poucos dispositivos precisam ser bloqueados; **Lista Branca (Whitelist)** é indicada quando se quer restringir a rede a um conjunto conhecido e limitado de dispositivos (mais segura, porém mais restritiva).

---

## 08 - Vínculo de Endereço IP e MAC

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Segurança → Vínculo IP e MAC** |
| 2 | Selecionar o dispositivo na **Lista ARP** e vincular |

| Campo | Valor | Descrição |
|---|---|---|
| Vínculo IP e MAC | 🟢 Habilitado (ON) | Associa de forma fixa um endereço IP a um MAC Address específico, impedindo que o dispositivo receba um IP diferente ou que outro equipamento "falsifique" (spoofing) esse endereço na rede. |
| Lista ARP | Selecionar dispositivo → Vincular (ON) | Exibe a tabela ARP (*Address Resolution Protocol*) com a relação atual entre IPs e MACs ativos na rede, servindo de base para criar os vínculos manualmente. |

> ⚠️ **Atenção:** tenha cuidado com dispositivos que possuem **múltiplos endereços MAC** (ex.: interfaces Wi-Fi e Ethernet distintas no mesmo aparelho, ou recursos de *MAC randomization* em smartphones modernos). Vincular o IP ao MAC errado pode causar conflitos de endereço ou perda de conectividade do dispositivo.

> 💡 **Dica:** essa técnica ajuda a mitigar ataques de **ARP Spoofing/Poisoning**, garantindo que um mesmo IP sempre corresponda ao mesmo dispositivo físico dentro da rede.

---

## 09 - Controle dos Pais (Parental Control)

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Controle dos Pais** |
| 2 | Clicar em **`Adicionar`** e criar o perfil do dispositivo/usuário |

| Campo | Descrição |
|---|---|
| Controle dos Pais | Recurso que permite gerenciar e restringir o uso da Internet de dispositivos específicos (ex.: de crianças e adolescentes), associando regras a um MAC Address. |
| Criar Perfil | Cadastro de um perfil vinculado a um dispositivo, permitindo definir horários de acesso permitido, limite de tempo de uso e, em alguns modelos, bloqueio de categorias de conteúdo. |

> ⚠️ **Atenção:** este procedimento estava **incompleto na documentação original** (interrompido em "Criar o perfil:"). Recomenda-se, ao aplicar em sala de aula, completar o cadastro com: nome do perfil, seleção do dispositivo (MAC Address), horário de acesso permitido e, se disponível no firmware, categorias de conteúdo a bloquear.

> 💡 **Dica:** o Controle dos Pais utiliza a mesma base de identificação por **MAC Address** obtida na lista de clientes DHCP (item 06), reforçando a importância de monitorar a rede antes de aplicar regras de restrição.

---

## ✅ Checklist Final da Aula

- [ ] Lista de clientes DHCP consultada (nomes, IPv4/IPv6 e MAC Address)
- [ ] Controle de Acesso habilitado no modo Lista Negra (ou Branca)
- [ ] Dispositivo de teste adicionado à lista e queda de conexão confirmada
- [ ] Vínculo IP e MAC habilitado para o(s) dispositivo(s) selecionado(s)
- [ ] Atenção redobrada para dispositivos com múltiplos MAC Address
- [ ] Perfil de Controle dos Pais criado e associado a um dispositivo
- [ ] Regras de horário/conteúdo do Controle dos Pais definidas e testadas

> 📌 **Fim do módulo:** com o reset, configuração de WAN/LAN, wireless, segurança básica e monitoramento concluídos, o Access Point está pronto para uso em ambiente de produção ou para os próximos módulos avançados do curso.
