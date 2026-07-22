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

Avançado, Wireless, Configurações Wireless
#01_ Ocultar o SSID das redes 2.4GHz e 5.0GHz

Observação: verificar no Smartphone que as redes não aparecem mais
Observação: tentar conectar na rede digitando o nome do SSID e senha
---------------------------------------------

Avançado, Wireless, Agenda Wireless
#02_ Agenda Wireless: (ON) Habilitar
  <Adicionar>
    Horario de Desligamento Sem-Fio: de 18hs
    Para: 22hs
    Repetir: S (Dom), M (Seg), T (Ter), C (Qua), T (Qui), F (Sex) S (Sab)
  <Salvar>
<Salvar>
Observação: Verificar LED' s do Roteador quando a rede for desligada
Observação: Verificar a queda de conexão dos dispositivos conectados
------------------------------------------------

Avançado, Wireless, WPS
#03_ Desativar o WPS (OFF)

Observação: não é recomendado deixar o WPS ativado, somente se necessário para IoT
------------------------------------------------

Avançado, Wireless, Configurações Adicionais
#04_ Configurações Adicionar
  Frequência: 2.4GHz: 
    Isolamento de AP (ON), 
    Airtime Fairness (ON).
  Frequência: 5.0GHz: 
    Isolamento de AP (ON), 
    Airtime Fairness (ON).

Observação: testar os pings entre os dispositivos da rede
Observação: verificar no Wifiman no teste de rede que só aparece o seu dispositivo e o router
------------------------------------------------------------

Avançado, Wireless, Rede para Convidados
#05_ Habilitar redes para convidado
  2.4GHz (OFF) Desabilitado
  5.0GHz (ON) Habilitado
    Nome de Rede (SSID): guest-grupo-01-5.0
    Segurança: WPA2/WPA3-Pessoal
    Senha: Senactit@1123
  Permissões de Convidado
    (OFF) Permitir convidados ver uns aos outros
    (OFF) Permita que os convidados acessem sua rede local
<Salvar>

Observação: conectar na rede convidados, testar a conexão entre dispositivos
------------------------------------------------------------