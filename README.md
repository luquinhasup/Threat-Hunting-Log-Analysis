# 🕵️‍♂️ Threat Hunting Lab: Análise de Processos, Conexões e Logs (macOS/Unix)

## 🎯 Objetivo do Projeto
Laboratório prático focado em metodologias de *Threat Hunting* e Engenharia de Detecção. O objetivo foi investigar processos suspeitos, rastrear conexões de rede não documentadas e analisar logs do sistema para identificar tentativas de comprometimento, como ataques de força bruta e elevação de privilégio.

## 🛠️ Metodologia e Comandos Utilizados
Para simular um ambiente de resposta a incidentes do mundo real, a investigação foi realizada nativamente em ambiente macOS (Unix-based), utilizando ferramentas de linha de comando para análise em profundidade:
* **Mapeamento de Rede:** Utilização de `netstat -an` e `lsof -iTCP -sTCP:LISTEN -n -P` para identificar portas abertas e estabelecer um *baseline* da rede.
* **Inspeção de Processos:** Rastreamento do *Process ID* (PID) de conexões ativas utilizando `ps -p <PID>` e `ps aux | grep <PID>` para validar a legitimidade dos binários em execução.

## ⚔️ Simulação de Ameaças (Purple Team)
Para validar a visibilidade dos logs do sistema, duas técnicas de ataque foram simuladas localmente:
1. **Comando e Controle (C2) / Backdoor:** Abertura de uma porta de escuta (listener) não autorizada utilizando o utilitário Netcat (`nc -l 4444`).
2. **Ataque de Força Bruta:** Múltiplas tentativas de autenticação SSH falhas executadas propositalmente contra a própria máquina (`ssh $(whoami)@localhost`).
   
<img width="1710" height="1107" alt="Captura de Tela 2026-05-15 às 10 14 14" src="https://github.com/user-attachments/assets/107d1aed-cb27-43e9-88a9-a61f0b0191c2" />

## 🔎 Análise de Logs e Visibilidade
Através do utilitário `log show`, os registros do sistema operacional foram filtrados para rastrear o rastro do atacante:
* Comando de busca de anomalias: `log show --last 10m | grep -Ei "failed|authentication|invalid sshd"`
* A análise confirmou o registro das tentativas falhas de logon através do processo `sshd`, validando a capacidade do sistema em detectar o ataque simulado.

<img width="1462" height="1091" alt="Captura de Tela 2026-05-16 às 11 08 03" src="https://github.com/user-attachments/assets/249af449-89a6-40f6-ae19-b7f56c72655a" />


## 🛡️ Engenharia de Detecção: Regra Proposta
Com base no comportamento anômalo mapeado durante o ataque de força bruta via SSH, propus a seguinte regra de detecção para ser implementada em um SIEM:

* **Condição de Disparo:** Mais de 5 tentativas de login SSH com falha no intervalo de 1 minuto originadas do mesmo endereço IP ou visando o mesmo usuário.
* **Fontes de Log (Data Sources):** `/var/log/auth.log` (Linux) ou logs do processo `sshd` (macOS).
* **Ação de Resposta:** 1. Geração de alerta de alta prioridade para o SOC.
  2. Notificação automática ao administrador.
  3. Bloqueio temporário do endereço IP de origem no Firewall (Active Response).<img width="1710" height="1107" 
