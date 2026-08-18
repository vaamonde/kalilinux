# 📡 Aula 10 — Testes de Invasão Wireless: Força Bruta (Brute Force) na WebGUI dos Access Points TP-Link com Hydra

> **Módulo:** Redes Sem-Fio — Testes de Invasão (Pentest Wireless)
> **Aula:** 10 — Ataque de Força Bruta contra a Interface Web de Gerenciamento (LAN e WAN) usando `hydra` e a Wordlist do Laboratório
> **Equipamentos homologados:** TP-Link Archer C50 (W) e TP-Link Archer EC220-G5 (login administrativo configurado pelos alunos na Aula 01)
> **Estação de ataque:** Kali Linux (ou Linux Mint 22.3, conforme Aula 02b) + `hydra`

---

## ℹ️ Informações do Documento

| Campo | Descrição |
|---|---|
| **Autor** | Robson Vaamonde |
| **Data de criação** | 04/08/2026 |
| **Data de atualização** | 04/08/2026 |
| **Versão** | 0.01 |
| **Status** | ⚠️ Roteiro **ainda não testado/validado** em ambiente real de laboratório — depende da estrutura exata do formulário de login de cada firmware (ver seção 04) |
| **Pré-requisito** | Aula 01 (senha de administrador definida), Aula 02 (LAN configurada) e wordlists da pasta `05-wordlist/` já disponíveis na estação de ataque |
| **Escopo** | Ataque de força bruta contra a **autenticação administrativa da WebGUI**, nos cenários **LAN (rede interna)** e **WAN (gerenciamento remoto habilitado)** — não confundir com os ataques às Aulas 06-09, que miram a **chave Wi-Fi** (WEP/WPS/WPA/WPA2/WPA3), e não a senha do painel administrativo |

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

## ⚠️ Aviso Legal e Ético — Leia Antes de Iniciar

> Este material é destinado **exclusivamente a fins didáticos**, em **ambiente de rede controlada e isolada** (laboratório), com equipamentos de propriedade da instituição de ensino e consentimento expresso para os testes.
>
> A utilização das técnicas aqui descritas contra painéis administrativos de terceiros, sem autorização formal, configura crime previsto no **Código Penal Brasileiro** (arts. 154-A e 154-B — invasão de dispositivo informático; art. 266 — interrupção de serviço telemático; arts. 155 e 157 — subtração de coisa alheia), além de responsabilidade civil (Código Civil, arts. 927 a 943).
>
> Cada grupo deve atacar **somente** o Access Point que lhe foi atribuído no laboratório, e o cenário de **gerenciamento remoto (WAN)** desta aula deve permanecer restrito ao endereço IP público/faixa liberada pela equipe de TI do laboratório — **nunca** exponha o gerenciamento remoto para a Internet aberta fora do horário e do escopo da aula.

---

## 📑 Sumário

1. [Contextualização — Força Bruta em WebGUI x Força Bruta em Wi-Fi](#01---contextualização--força-bruta-em-webgui-x-força-bruta-em-wi-fi)
2. [Habilitando o Gerenciamento Remoto (WAN) no TP-Link](#02---habilitando-o-gerenciamento-remoto-wan-no-tp-link)
3. [Preparação do Ambiente de Ataque](#03---preparação-do-ambiente-de-ataque)
4. [Identificando o Formulário de Login da WebGUI](#04---identificando-o-formulário-de-login-da-webgui)
5. [Ataque de Força Bruta na Rede Local (LAN)](#05---ataque-de-força-bruta-na-rede-local-lan)
6. [Ataque de Força Bruta na Rede Remota (WAN)](#06---ataque-de-força-bruta-na-rede-remota-wan)
7. [Limitações Conhecidas e Contramedidas do Firmware](#07---limitações-conhecidas-e-contramedidas-do-firmware)
8. [Validação e Preenchimento do Relatório](#08---validação-e-preenchimento-do-relatório)
9. [Encerramento do Módulo](#-encerramento-do-módulo)

---

## 01 - Contextualização — Força Bruta em WebGUI x Força Bruta em Wi-Fi

| Característica | Aulas 06-09 (WEP/WPS/WPA/WPA2/WPA3) | Aula 10 (esta aula) |
|---|---|---|
| Alvo do ataque | Chave de acesso à **rede Wi-Fi** (PSK/PIN) | Senha de **login do painel administrativo** da WebGUI |
| Protocolo atacado | 802.11 (camada de enlace sem-fio) | HTTP/HTTPS (camada de aplicação) |
| Ferramenta principal | `aircrack-ng`, `reaver`, `hashcat` | `hydra` (THC-Hydra) |
| Necessita modo monitor? | Sim | **Não** — basta conectividade IP normal (Ethernet ou Wi-Fi já associado) com o alvo |
| Consequência de um ataque bem-sucedido | Acesso à rede interna como um cliente comum | **Controle administrativo total do equipamento** (DNS, DHCP, WAN, Wi-Fi, firewall, etc.) |

> 💡 **Por que esta aula importa:** mesmo com uma senha Wi-Fi forte (Aula 08/09), um painel administrativo com senha fraca — ou exposto sem necessidade ao gerenciamento remoto (WAN) — é, na prática, uma porta de entrada equivalente ou pior, pois compromete o equipamento inteiro, não apenas o acesso à rede.

> 💡 **Ferramenta:** o **Hydra** (`thc-hydra`, já disponível nos metapacotes `top10`/`kali-linux-default` do Kali, ou instalável via `sudo apt install -y hydra` no Kali/Mint) é a ferramenta de força bruta de autenticação em rede mais usada, com suporte nativo ao módulo `http-post-form`/`http-get-form`, usado nesta aula.

---

## 02 - Habilitando o Gerenciamento Remoto (WAN) no TP-Link

Para reproduzir o cenário de ataque **externo (WAN)**, é necessário primeiro habilitar deliberadamente o gerenciamento remoto no AP — recurso que, em produção, **deve permanecer desabilitado** por padrão.

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Acessar: **Avançado → Sistema → Gerenciamento Remoto** (ou **Advanced → System Tools → Remote Management**, conforme idioma do firmware) |

| Campo | Valor | Descrição |
|---|---|---|
| Gerenciamento Remoto (WAN) | 🟢 Habilitado (ON) | Permite o acesso à interface de administração (`https://<IP_WAN>:<porta>`) a partir da Internet, e não apenas da rede local (LAN). |
| Faixa de IP Permitida | `IP público do notebook-atacante` ou `Todos` (apenas para fins desta aula) | Restringe (ou não) quais endereços de origem podem alcançar o painel remoto — em produção, **restrinja sempre** a um IP/faixa conhecida. |
| Porta de Gerenciamento Remoto | `8443` (ou porta customizada definida pelo instrutor) | Porta HTTPS usada para o acesso remoto, geralmente diferente da porta padrão da LAN. |

➡️ Clicar em **`Salvar`**

> ⚠️ **Atenção — uso restrito ao laboratório:** habilite este recurso **apenas durante o horário da aula**, com a faixa de IP de origem restrita ao(s) notebook(s) do próprio grupo (ou à faixa de IPs do laboratório liberada pela equipe de TI). Ao final da aula, **desabilite novamente** o Gerenciamento Remoto — é uma das más práticas de segurança mais exploradas em roteadores domésticos/comerciais reais.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | A partir de uma rede **externa** à LAN do AP (ex.: rede 4G do celular, ou outro segmento do laboratório), o navegador consegue abrir `https://<IP_WAN_DO_AP>:<PORTA>` e visualizar a tela de login. |
| 2 | Fora da faixa de IP liberada, o acesso remoto é recusado (teste negativo). |

---

## 03 - Preparação do Ambiente de Ataque

### 🔧 Procedimento

```bash
# Verificar se o hydra está instalado
hydra -h | head -n 5

# Caso não esteja instalado (Kali ou Mint/Ubuntu)
sudo apt update
sudo apt install -y hydra
```

### 📁 Wordlists Utilizadas (pasta `05-wordlist/`)

| Arquivo | Descrição | Uso recomendado |
|---|---|---|
| `01-wordlist-lab.txt` | Wordlist básica do laboratório (senhas didáticas curtas, ex.: `Senactit@123`, `grupo01`) | Primeira rodada — teste rápido, poucos candidatos |
| `02-wordlist-lab-expandida.txt` | Wordlist expandida com variações (maiúsculas, símbolos, anos, sufixos por grupo) | Segunda rodada — caso a wordlist básica não encontre a senha |

> 💡 **Reaproveitamento:** são as **mesmas wordlists já usadas na Aula 08** (WPA/WPA2) — o objetivo didático é reforçar que uma política de senha fraca compromete **múltiplas camadas** do equipamento (Wi-Fi e painel administrativo) com o mesmo dicionário.

### 📝 Identificando o Usuário Administrador

| Campo | Valor | Origem |
|---|---|---|
| Usuário | `admin` (padrão TP-Link) | Login padrão de fábrica, salvo alteração manual pelo grupo |
| Senha configurada na Aula 01 | `Senactit@123` | Usada como "senha correta" de referência, para validar que o ataque efetivamente a encontra na wordlist |

> 💡 **Dica didática:** para a aula funcionar como demonstração de sucesso, garanta que a senha definida na Aula 01 (`Senactit@123`) **esteja presente** em uma das duas wordlists — já é o caso das wordlists fornecidas em `05-wordlist/`.

---

## 04 - Identificando o Formulário de Login da WebGUI

Diferente do WPA2 (Aula 08), onde o alvo do ataque é sempre um handshake padronizado pelo protocolo 802.11, cada **firmware/versão** de WebGUI pode implementar o login de forma diferente (campos do formulário, método HTTP, uso de cookies/tokens, e até criptografia do campo de senha via JavaScript no navegador). É necessário **inspecionar o formulário real** antes de montar o comando do Hydra.

### 🔧 Procedimento

| Passo | Ação |
|---|---|
| 1 | Abrir a tela de login do AP no navegador (`http(s)://<IP_LAN_OU_WAN>[:<PORTA>]`) |
| 2 | Abrir as **Ferramentas do Desenvolvedor** do navegador (`F12`) → aba **Rede/Network** |
| 3 | Digitar um usuário/senha de teste (ex.: `admin` / `teste123`) e enviar o formulário |
| 4 | Localizar, na aba Network, a requisição enviada (geralmente `POST` para `/`, `/login`, `/cgi-bin/luci`, ou similar) |

| Campo a identificar | Onde encontrar | Exemplo típico |
|---|---|---|
| URL/endpoint de login | Coluna "Nome"/"Name" da requisição capturada | `/` ou `/login.cgi` |
| Nome do campo de usuário | Aba **Payload/Form Data** da requisição | `username` |
| Nome do campo de senha | Aba **Payload/Form Data** da requisição | `password` |
| Mensagem de falha de login | Resposta HTTP quando a senha está errada | Texto exibido na tela (ex.: "senha incorreta", "invalid") |
| Cookies/tokens exigidos | Cabeçalhos `Set-Cookie` / campos ocultos (`hidden`) no HTML | `stok`, `sysauth`, `csrf_token`, etc. |

> ⚠️ **Ponto crítico:** diversos firmwares TP-Link modernos **ofuscam ou criptografam a senha no navegador antes de enviá-la** (via JavaScript, usando MD5/RSA client-side), o que **impede** um ataque de força bruta HTTP simples e direto — ver seção 07 para como identificar esse comportamento e as alternativas.

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O endpoint, o método HTTP (`GET`/`POST`) e os nomes dos campos de usuário/senha foram anotados corretamente. |
| 2 | A mensagem/padrão que identifica uma tentativa de login **falha** foi identificada (necessária para o parâmetro `F=` do Hydra). |
| 3 | Foi verificado se a senha trafega em **texto plano** no `Form Data` ou se já chega **hasheada/cifrada** pelo JavaScript da página (ver seção 07). |

---

## 05 - Ataque de Força Bruta na Rede Local (LAN)

Com o formulário mapeado (seção 04) e assumindo que a senha trafega em texto plano no `POST`, monta-se o ataque via módulo `http-post-form` do Hydra.

### 🔧 Procedimento — Sintaxe Geral

```bash
hydra -l admin -P 02-wordlist-lab-expandida.txt \
  <IP_LAN_DO_AP> \
  http-post-form \
  "/<ENDPOINT_LOGIN>:username=^USER^&password=^PASS^:F=<TEXTO_DE_FALHA>"
```

| Parâmetro | Descrição |
|---|---|
| `-l admin` | Usuário fixo a ser testado (login administrativo padrão do TP-Link). |
| `-P 02-wordlist-lab-expandida.txt` | Caminho da wordlist de senhas a testar (arquivo `-P` = *password list*; use `-p` para uma única senha). |
| `<IP_LAN_DO_AP>` | Endereço IP da interface LAN do AP (ex.: `172.16.0x.254`, conforme configurado na Aula 02). |
| `http-post-form` | Módulo do Hydra especializado em formulários web via `POST`. |
| `/<ENDPOINT_LOGIN>` | Caminho identificado na seção 04 (ex.: `/` ou `/login.cgi`). |
| `username=^USER^&password=^PASS^` | Corpo do formulário — `^USER^` e `^PASS^` são substituídos pelo Hydra a cada tentativa; ajuste os nomes dos campos (`username`, `password`) conforme identificado na seção 04. |
| `F=<TEXTO_DE_FALHA>` | Trecho de texto (string) que aparece na resposta **somente quando o login falha** — é assim que o Hydra diferencia sucesso de fracasso. |

### 🔧 Exemplo com a Wordlist Expandida (segunda rodada)

```bash
hydra -l admin -P 02-wordlist-lab-expandida.txt \
  172.16.0X.254 \
  http-post-form \
  "/:username=^USER^&password=^PASS^:F=invalid" \
  -t 4 -f -V
```

| Parâmetro adicional | Descrição |
|---|---|
| `-t 4` | Número de threads paralelas (mantenha **baixo**, 1 a 4, para não sobrecarregar/travar o AP — muitos firmwares de baixo custo têm CPU limitada). |
| `-f` | Interrompe o ataque assim que a **primeira** combinação válida for encontrada. |
| `-V` | Modo *verbose* — exibe cada tentativa em tempo real, útil para fins didáticos (a turma acompanha o progresso). |

### ✅ Resultado Esperado

| Situação | Ação |
|---|---|
| ✅ `[<PORTA>][http-post-form] host: <IP> login: admin password: <senha>` | Senha do painel administrativo descoberta — anotar no relatório do grupo. |
| ❌ Nenhuma combinação encontrada | Confirmar se o texto de `F=` está correto (falso negativo mais comum) ou se a senha realmente não consta na wordlist. |
| ⚠️ AP trava, fica lento ou bloqueia IP de origem | Reduzir `-t` para `1`, aguardar alguns minutos e retomar — muitos firmwares aplicam *rate limiting* ou bloqueio temporário após tentativas repetidas (ver seção 07). |

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | Hydra retorna a senha correta, idêntica à definida na Aula 01. |
| 2 | Login manual no painel, usando a senha encontrada, confirma o acesso administrativo completo. |

---

## 06 - Ataque de Força Bruta na Rede Remota (WAN)

Com o Gerenciamento Remoto habilitado (seção 02), o mesmo ataque pode ser reproduzido **de fora da rede local**, simulando um atacante na Internet.

### 🔧 Procedimento

```bash
hydra -l admin -P 02-wordlist-lab-expandida.txt \
  -s <PORTA_REMOTA> \
  <IP_WAN_DO_AP> \
  https-post-form \
  "/<ENDPOINT_LOGIN>:username=^USER^&password=^PASS^:F=<TEXTO_DE_FALHA>" \
  -t 2 -f -V
```

| Parâmetro | Descrição |
|---|---|
| `-s <PORTA_REMOTA>` | Porta customizada do gerenciamento remoto definida na seção 02 (ex.: `8443`), obrigatória quando diferente da porta padrão. |
| `https-post-form` | Variante do módulo para conexões **HTTPS** — a maioria dos APs TP-Link força HTTPS no acesso remoto (diferente do acesso LAN, que às vezes ainda aceita HTTP simples). |
| `-t 2` | Threads reduzidas — conexões WAN têm mais latência e o teste deve ser ainda mais conservador para não gerar alarme falso de DDoS na rede do laboratório. |

> 💡 **Diferença chave em relação à LAN:** o Hydra precisa alcançar o **IP público (WAN)** do AP, e não o IP interno (`172.16.xxx.254`). Confirme esse endereço com a equipe de TI do laboratório antes do teste — normalmente é o IP fornecido pelo provedor de Internet do laboratório à interface WAN do equipamento.

> ⚠️ **Certificado autoassinado:** como o gerenciamento remoto do TP-Link normalmente usa um certificado HTTPS autoassinado, é comum a necessidade de aceitar o aviso de certificado inválido no navegador ao validar manualmente — isso **não impede** o Hydra de funcionar via linha de comando, mas é um bom ponto de discussão em sala sobre a diferença entre "conexão criptografada" e "conexão autenticada/confiável".

### ✅ Testes de Validação

| # | Teste |
|---|---|
| 1 | O ataque via WAN encontra a mesma senha já validada no teste LAN (seção 05). |
| 2 | Login manual, a partir de uma rede externa, confirma o acesso remoto completo ao painel. |
| 3 | Ao final da aula, o Gerenciamento Remoto é **desabilitado novamente** (seção 02), e o teste de acesso externo passa a falhar (teste negativo obrigatório). |

---

## 07 - Limitações Conhecidas e Contramedidas do Firmware

Ao contrário do WPA2 (Aula 08), onde "sem dicionário não há quebra" é a única barreira, a força bruta em WebGUI esbarra em proteções adicionais, comuns em firmwares modernos — e que devem ser discutidas com a turma, mesmo quando impedirem o ataque de funcionar de forma direta.

| Proteção | Efeito sobre o Hydra | Como identificar/contornar |
|---|---|---|
| **Criptografia client-side da senha (JavaScript)** | O `Form Data` capturado na seção 04 mostra um hash/valor cifrado, não a senha em texto plano — o Hydra enviaria o valor errado a cada tentativa. | Inspecionar o JavaScript da página de login (aba **Fontes/Sources** do navegador) para entender o algoritmo usado; em muitos casos, é necessário reproduzir esse cálculo (ex.: MD5, RSA com chave pública do próprio AP) antes de montar o payload — fora do escopo desta aula introdutória, tratar como discussão teórica. |
| **Token/CSRF dinâmico por sessão** | Cada tentativa de login exige um token diferente, obtido em uma requisição anterior — o Hydra sozinho, sem configuração adicional, reenvia sempre o mesmo valor estático. | Verificar se há um campo oculto (`hidden`) que muda a cada carregamento da página; nesses casos, um simples `http-post-form` não é suficiente, sendo necessário um script/wrapper (ex.: com `curl` ou `requests` em Python) que renove o token a cada tentativa. |
| **Bloqueio temporário (lockout) após N tentativas** | O AP passa a recusar **todas** as tentativas (mesmo a correta) por um período, até após o Hydra já ter enviado a senha certa. | Reduzir threads (`-t 1`), inserir atraso entre tentativas se o firmware suportar, ou reiniciar o AP para resetar o contador — mesmo princípio já discutido no `wash`/`reaver` da Aula 07. |
| **CAPTCHA na tela de login** | Impede totalmente a automação simples via Hydra. | Não há contorno automatizado dentro do escopo ético desta aula — tratar como controle de segurança eficaz e documentar como tal no relatório. |

> 💡 **Mensagem central para a turma:** diferente das Aulas 06-09 (onde o resultado prático de "quebrar ou não quebrar" é geralmente binário e replicável), o sucesso desta aula depende diretamente de **como o fabricante implementou** a autenticação web — reforçando que boas práticas de desenvolvimento (token CSRF, hashing client-side, rate limiting, CAPTCHA) são tão importantes quanto a força da própria senha.

---

## 08 - Validação e Preenchimento do Relatório

### ✅ Testes de Validação Final

| # | Teste |
|---|---|
| 1 | Registrar no relatório: IP LAN, IP/porta WAN, endpoint de login identificado, campos do formulário, wordlist utilizada e senha encontrada (LAN e WAN). |
| 2 | Registrar se alguma das proteções da seção 07 foi identificada (criptografia client-side, token CSRF, lockout, CAPTCHA) e como isso afetou o resultado do ataque. |
| 3 | Confirmar que o **Gerenciamento Remoto (WAN)** foi desabilitado ao final da aula, e que o teste de acesso externo passou a falhar. |
| 4 | Discutir em grupo: comparar o esforço/tempo desta aula com o das Aulas 06-09 — por que comprometer a **senha do painel** costuma ser mais crítico do que comprometer apenas a **senha Wi-Fi**. |

## ✅ Checklist Final da Aula

- [ ] Hydra instalado e validado (`hydra -h`)
- [ ] Gerenciamento Remoto (WAN) habilitado temporariamente, com faixa de IP restrita ao laboratório
- [ ] Formulário de login inspecionado (endpoint, campos, mensagem de falha) via DevTools do navegador
- [ ] Verificado se a senha trafega em texto plano ou cifrada via JavaScript (seção 07)
- [ ] Ataque de força bruta executado na **LAN** com `01-wordlist-lab.txt`
- [ ] (Se necessário) Ataque repetido na **LAN** com `02-wordlist-lab-expandida.txt`
- [ ] Ataque de força bruta executado na **WAN** (gerenciamento remoto), com threads reduzidas
- [ ] Senha do painel administrativo encontrada e validada com login manual (LAN e WAN)
- [ ] Proteções do firmware (lockout, CSRF, criptografia client-side, CAPTCHA) identificadas e documentadas
- [ ] Gerenciamento Remoto (WAN) **desabilitado novamente** ao final da aula, com teste negativo confirmado
- [ ] Relatório do grupo preenchido com todos os dados da tabela da seção 08

---

## 🏁 Encerramento do Módulo

Com a Aula 10, o módulo de **Testes de Invasão em Rede Wireless** passa a cobrir, além da camada 802.11 (WEP, WPS, WPA/WPA2, WPA3 — Aulas 06-09), a camada de **aplicação/gerenciamento** do próprio equipamento, evidenciando que a segurança de um Access Point depende de **múltiplas camadas independentes**: a chave Wi-Fi, a senha administrativa e a exposição (ou não) do gerenciamento remoto à Internet.

### 📊 Quadro-Resumo Ampliado do Módulo

| Aula | Alvo do ataque | Camada | Ferramenta principal | Depende de dicionário? |
|---|---|---|---|---|
| 06 | Chave WEP | 802.11 | `aircrack-ng` (criptoanálise) | Não |
| 07 | PIN WPS | 802.11 | `reaver`/`pixiewps` | Não |
| 08 | Chave WPA/WPA2-PSK | 802.11 | `aircrack-ng`/`hashcat` | **Sim** |
| 09 | Chave WPA3-SAE | 802.11 | N/A (resistente por design) | Não aplicável |
| **10** | **Senha do painel administrativo (LAN e WAN)** | **HTTP/HTTPS (aplicação)** | **`hydra`** | **Sim** |

> 💡 **Mensagem final para a turma:** uma rede wireless só está de fato protegida quando **todas** as camadas são endurecidas em conjunto — chave Wi-Fi forte (Aula 08/09), WPS desabilitado (Aula 04/07), **e** uma senha administrativa forte com Gerenciamento Remoto desabilitado por padrão (esta aula). Falhar em qualquer uma delas reabre a porta que as demais tentaram fechar.

> 📌 **Sugestão de continuidade do curso:** com o módulo de força bruta e quebra de senhas concluído, os próximos passos naturais seriam (a) hardening avançado do painel administrativo (autenticação em dois fatores, se suportado pelo firmware), (b) testes de força bruta em outros serviços expostos (SSH/Telnet, quando habilitados), e (c) consolidação de todos os resultados das Aulas 06-10 no documento de homologação (`07-checklist/01-HomologacaoRedesSemFioKali.md`).
