# 📡 Homologação de Documentação em Prática — Redes Sem-Fio / Kali Linux

> **Objetivo:** acompanhar a aplicação prática dos procedimentos documentados (Aulas 01-09, módulos `01-accesspoint`, `02-installkali`, `03-analytics` e `04-aircrack-ng`) com os equipamentos reais, e registrar qualquer divergência para alimentar a próxima revisão dos arquivos `.md`.

| Campo | Valor |
|---|---|
| Curso / Turma | _____________________ |
| Instrutor responsável | _____________________ |
| Data de início da homologação | _____________________ |
| Versão da documentação avaliada | _____________________ |

---

## 📑 Como usar este documento

1. **Inventário** — cadastre os equipamentos (APs, antenas, notebooks) atribuídos a cada grupo antes da aula.
2. **Checklist de Configuração (Aulas 01-05)** — marque o status de cada item de configuração dos Access Points, por grupo.
3. **Baseline Wireless** — preencha com os dados coletados na aula de reconhecimento passivo (BSSID, canal, PWR etc.).
4. **Ataques — Resultados (Aulas 06-09)** — registre o resultado de cada ataque prático (WEP, WPS, WPA/WPA2, WPA3).
5. **Não Conformidades** — registre divergências entre o que está documentado e o que aconteceu na prática.

**Legenda de status:** `OK` = concluído · `Pendente` = ainda não testado/atenção · `Falhou` = não conformidade · `N/A` = não se aplica

---

## 1) Inventário de Equipamentos por Grupo

| Grupo | Modelo do AP | Nº patrimônio / etiqueta | Notebook (hostname) | Antena externa (modelo/chipset) | Interface (wlanX) | Responsável | Observações |
|---|---|---|---|---|---|---|---|
| Grupo 01 | | | | | | | |
| Grupo 02 | | | | | | | |
| Grupo 03 | | | | | | | |
| Grupo 04 | | | | | | | |
| Grupo 05 | | | | | | | |
| Grupo 06 | | | | | | | |
| Grupo 07 | | | | | | | |
| Grupo 08 | | | | | | | |
| Grupo 09 | | | | | | | |
| Grupo 10 | | | | | | | |

---

## 2) Checklist de Configuração — Aulas 01 a 05

> Repita o bloco abaixo para cada grupo (copie a tabela e troque o número do grupo).

### Grupo __

| Aula / Módulo | Item do checklist | Status | Responsável | Data | Observações |
|---|---|---|---|---|---|
| Aula 01 — Reset e Config. Básica | Reset de fábrica realizado com sucesso | Pendente | | | |
| Aula 01 — Reset e Config. Básica | Conexão de rede testada via ping (192.168.0.1) | Pendente | | | |
| Aula 01 — Reset e Config. Básica | Acesso à interface web realizado e senha de admin definida | Pendente | | | |
| Aula 01 — Reset e Config. Básica | Fuso horário configurado (Brasília — UTC-03:00) | Pendente | | | |
| Aula 01 — Reset e Config. Básica | WAN em IP Dinâmico configurada | Pendente | | | |
| Aula 01 — Reset e Config. Básica | WLAN 2.4/5.0 GHz desabilitadas propositalmente no wizard | Pendente | | | |
| Aula 02 — WAN e LAN | DNS da WAN configurado com servidor de proteção familiar | Pendente | | | |
| Aula 02 — WAN e LAN | Hostname (`ap-grupo-0X`) definido corretamente | Pendente | | | |
| Aula 02 — WAN e LAN | IP da LAN alterado para `172.16.xxx.254/24` | Pendente | | | |
| Aula 02 — WAN e LAN | Placa de rede do computador reconfigurada para a nova faixa | Pendente | | | |
| Aula 02 — WAN e LAN | DHCP habilitado com pool `172.16.xxx.100–150` | Pendente | | | |
| Aula 02 — WAN e LAN | Gateway e DNS primário do DHCP apontando para `172.16.xxx.254` | Pendente | | | |
| Aula 02 — WAN e LAN | Servidores NTP (`a.st1.ntp.br` / `c.st1.ntp.br`) configurados | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Rede 2.4 GHz habilitada com SSID `grupo-0x-2.4` | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Segurança WPA2/WPA3-Pessoal configurada em 2.4 GHz | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Canal 1 e largura 20 MHz definidos em 2.4 GHz | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Rede 5.0 GHz habilitada com SSID `grupo-0x-5.0` | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Segurança WPA2/WPA3-Pessoal configurada em 5.0 GHz | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Canal 36, largura 20 MHz e MU-MIMO habilitados em 5.0 GHz | Pendente | | | |
| Aula 03 — Wireless 2.4/5.0 GHz | Conexão testada em ambas as bandas com o Wifiman | Pendente | | | |
| Aula 04 — Segurança Wireless | SSID 2.4/5.0 GHz ocultado e testado | Pendente | | | |
| Aula 04 — Segurança Wireless | Agenda Wireless configurada (desligamento programado) | Pendente | | | |
| Aula 04 — Segurança Wireless | WPS desativado em ambas as bandas | Pendente | | | |
| Aula 04 — Segurança Wireless | Isolamento de AP e Airtime Fairness habilitados (2.4/5.0 GHz) | Pendente | | | |
| Aula 04 — Segurança Wireless | Rede de convidados habilitada apenas em 5.0 GHz, com SSID/senha próprios | Pendente | | | |
| Aula 04 — Segurança Wireless | Permissões de convidado restritas (sem LAN, sem visibilidade entre convidados) | Pendente | | | |
| Aula 05 — Monitoramento e Acesso | Lista de clientes DHCP consultada (nomes, IPv4/IPv6, MAC) | Pendente | | | |
| Aula 05 — Monitoramento e Acesso | Controle de Acesso habilitado (Lista Negra ou Branca) e testado | Pendente | | | |
| Aula 05 — Monitoramento e Acesso | Vínculo IP e MAC habilitado para o(s) dispositivo(s) selecionado(s) | Pendente | | | |
| Aula 05 — Monitoramento e Acesso | Perfil de Controle dos Pais criado e testado | Pendente | | | |

---

## 3) Baseline do Ambiente Sem-Fio (Reconhecimento Passivo)

| Grupo | Hostname notebook | Interface(s) sem-fio | Antena (modelo/chipset) | BSSID do AP | ESSID (2.4/5.0 GHz) | Canal(is) | Largura de canal | ENC/CIPHER/AUTH | PWR médio (dBm) | APs vizinhos no mesmo canal | Cliente teste (MAC) | Observações |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Grupo 01 | | | | | | | | | | | | |
| Grupo 02 | | | | | | | | | | | | |
| Grupo 03 | | | | | | | | | | | | |
| Grupo 04 | | | | | | | | | | | | |
| Grupo 05 | | | | | | | | | | | | |
| Grupo 06 | | | | | | | | | | | | |
| Grupo 07 | | | | | | | | | | | | |
| Grupo 08 | | | | | | | | | | | | |
| Grupo 09 | | | | | | | | | | | | |
| Grupo 10 | | | | | | | | | | | | |

---

## 4) Resultados dos Ataques Práticos — Aulas 06 a 09

### Grupo __

| Aula | Protocolo | BSSID alvo | Canal | Ferramenta principal | Senha/PIN encontrado | IVs / tentativas | Tempo total (min) | Data do teste | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| Aula 06 | WEP | | | aircrack-ng (IVs) | | | | | Pendente | |
| Aula 07 | WPS | | | reaver / pixiewps | | | | | Pendente | |
| Aula 08 | WPA/WPA2-PSK (AES) | | | aircrack-ng / hashcat | | | | | Pendente | |
| Aula 08 | WPA/WPA2-PSK (AES+TKIP) | | | aircrack-ng / hashcat | | | | | Pendente | |
| Aula 09 | WPA3-SAE puro (Cenário A) | | | N/A — demonstrativo | | | | | Pendente | |
| Aula 09 | WPA2/WPA3 downgrade (Cenário B) | | | aircrack-ng / hashcat | | | | | Pendente | |

---

## 5) Registro de Não Conformidades e Sugestões de Atualização

> Toda divergência entre o `.md` e a prática deve virar uma linha aqui — alimenta a próxima revisão da documentação.

| Nº | Arquivo .md / Aula | Seção / Passo | O que a documentação diz | O que aconteceu na prática | Ação sugerida | Responsável | Status |
|---|---|---|---|---|---|---|---|
| 1 | `04-aircrack-ng/09` (WPA3) | Cabeçalho / equipamento | Aula usa Archer AX12, diferente das Aulas 01-08 (C50/EC220-G5) | | Confirmar disponibilidade de AX12 suficiente para todos os grupos | | Pendente |
| 2 | `01-accesspoint/05` (Monitoramento) | Item 09 - Controle dos Pais | Procedimento descrito como incompleto na documentação original | | Completar passo a passo do cadastro de perfil antes da aula | | Pendente |
| 3 | `05-wordlist/02-expandida.txt` | Sufixos de ano | Algumas entradas parecem concatenadas incorretamente (ex.: `Senactit@1232024`) | | Revisar script gerador da wordlist expandida | | Pendente |
| 4 | | | | | | | Pendente |
| 5 | | | | | | | Pendente |
| 6 | | | | | | | Pendente |
| 7 | | | | | | | Pendente |
| 8 | | | | | | | Pendente |

---

## 6) Dashboard Resumido (preencher manualmente ao final da homologação)

| Métrica | Total | OK | Pendente | Falhou | % Concluído |
|---|---|---|---|---|---|
| Itens de configuração (todos os grupos) | | | | | |
| Testes de ataque (todos os grupos) | | | | | |

| Protocolo | Total testes | OK | Pendente | Falhou | % Sucesso |
|---|---|---|---|---|---|
| WEP | | | | | |
| WPS | | | | | |
| WPA/WPA2-PSK (AES) | | | | | |
| WPA/WPA2-PSK (AES+TKIP) | | | | | |
| WPA3-SAE puro (Cenário A) | | | | | |
| WPA2/WPA3 downgrade (Cenário B) | | | | | |

> 💡 **Na versão Excel (.xlsx)** este dashboard é calculado automaticamente por fórmulas (`COUNTIF`/`SUMPRODUCT`) a partir das abas de checklist e ataques — recomendado para uso contínuo durante a homologação. Esta versão em Markdown/PDF é indicada para impressão e preenchimento manual em campo.
