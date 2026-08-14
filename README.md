# RGB-Sync-UiPath

Automação em UiPath que sincroniza o Logitech G HUB e o SignalRGB no login/boot da máquina,
garantindo que o mouse (Logitech G903 LIGHTSPEED) fique com a iluminação sempre ativa e que os
dois apps de RGB não entrem em conflito.

## O que faz (`Main.xaml`)

1. Roda a tarefa agendada `Kill RGB Processes` (via `schtasks`) para encerrar instâncias antigas
   de `lghub.exe` e `SignalRgb.exe` antes de começar.
2. Abre/fecha a janela do Logitech G HUB e aguarda o app carregar.
3. Verifica o estado do dispositivo ("Dispositivos", "G903 LIGHTSPEED", tela de "ative ou ligue
   seu dispositivo") e, se necessário, reinvoca a si mesma (`Main.xaml`) para tentar de novo.
4. Abre as configurações do dispositivo e ativa a opção "SEMPRE ATIVADO" da iluminação,
   confirmando o valor "1 mW" antes de finalizar.

## Integração com o Agendador de Tarefas do Windows

A pasta `Agendador/` contém exports (`schtasks /query /xml`) das tarefas agendadas usadas por
essa automação — sirva como referência para recriá-las em outra máquina:

- **`RGB Sync.xml`** — dispara no logon e no boot, executa `wscript.exe K:\UIPath\RGB-Sync.vbs`,
  que por sua vez chama `UiRobot.exe execute --process "RGB.Sync"` (o processo publicado deste
  projeto) de forma oculta.
- **`Kill RGB Processes.xml`** — mata `lghub.exe` e `SignalRgb.exe` via `taskkill`; é chamada pelo
  próprio `Main.xaml` no início da execução.
- **`SignalRgb Admin.xml`** — inicia o `SignalRgbLauncher.exe` com privilégios elevados.

`RGB-Sync.vbs` (fora deste repositório, em `K:\UIPath\`) é o script que a tarefa `RGB Sync`
invoca para disparar o robô via UiRobot.

## Requisitos

- [UiPath Studio](https://www.uipath.com/product/studio) (schema versão 4.0 / Studio 26.0+).
- Dependências do projeto (restauradas automaticamente pelo Studio via `project.json`):
  - `UiPath.System.Activities` `25.10.3`
  - `UiPath.UIAutomation.Activities` `25.10.22`
- Logitech G HUB e SignalRGB instalados.
- As 3 tarefas agendadas acima criadas no Agendador de Tarefas do Windows (pasta `\Criados\`),
  apontando para o robô publicado (`UiRobot.exe execute --process "RGB.Sync"`).

## Como executar

- Manualmente: abra `Main.xaml` no UiPath Studio e rode (F5), ou publique o processo e rode
  `UiRobot.exe execute --process "RGB.Sync"`.
- Automaticamente: as tarefas agendadas cuidam disso no logon/boot; não é necessário rodar nada
  manualmente em uso normal.
