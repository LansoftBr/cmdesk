# Documentação de Customizações - CMDESK (Casa Magalhães)

Este arquivo documenta as alterações realizadas no código-fonte original do RustDesk para implementar as compilações personalizadas das versões `CMCLIENTES` e `CMTECH`.

As alterações estão divididas em três escopos principais: **Base (CI/CD)**, **Cliente (`build/cmclientes`)** e **Técnico (`build/cmtech`)**.

## 1. Alterações Base (Branch `master` e propagadas)

### 1.1 Injeção de Variáveis de Ambiente Seguras (Zero-Trust)
O objetivo principal foi evitar que chaves de API, senhas ou URLs de servidores de retransmissão estivessem diretamente no código-fonte.

- **`build.py`**: A função `get_dart_defines()` foi criada para recuperar as variáveis de ambiente `RENDEZVOUS_SERVER`, `API_SERVER` e `KEY` do CI/CD. O script agora anexa `--dart-define` aos comandos `flutter build` (Linux, MacOS e Windows).
- **`.github/workflows/flutter-build.yml`**: Adicionado o gatilho `workflow_dispatch` para permitir a execução manual por branch. Modificado os scripts shell (especialmente o de compilação do Android - APK) para passar as mesmas variáveis via `--dart-define`.
- **`flutter/lib/main.dart`**: Na função `initEnv()`, adicionado código para recuperar as variáveis injetadas via `String.fromEnvironment()` e gravá-las no `bind.mainSetOption()`, aplicando o servidor de retransmissão e a chave simétrica no início do aplicativo em vez de usar hardcode.

### 1.2 Ocultação de Configurações de Rede
Para impedir que o usuário modifique a conexão e desvincule o aplicativo do servidor seguro:
- **`flutter/lib/desktop/pages/desktop_setting_page.dart`**: Removido o item `SettingsTabKey.network` da lista de guias.
- **`flutter/lib/mobile/pages/settings_page.dart`**: O valor de `_hideNetwork` foi forçado (hardcoded) como `true` na rotina de inicialização.

---

## 2. Versão do Cliente (Branch `build/cmclientes`)

Esta branch isola as configurações feitas especificamente para o cliente final.
- **Objetivo**: O cliente não deve conseguir controlar outras máquinas, apenas fornecer seu ID e Senha para suporte.

### Alterações:
- **Desktop (`flutter/lib/desktop/pages/desktop_home_page.dart`)**: A tela inicial (`build()`) foi alterada para renderizar apenas o `buildLeftPane(context)` usando a largura total (`Expanded`). A função de conexão remota (Right Pane) foi escondida.
- **Mobile (`flutter/lib/mobile/pages/home_page.dart`)**: A guia de conexão para outras máquinas (`ConnectionPage`) foi removida das abas.
- **Branding**: O aplicativo teve seu `app_name` alterado para `CMCLIENTES` no manifesto do Android (`AndroidManifest.xml`), no Windows (`Runner.rc` e `main.cpp`) e no Linux (`rustdesk.desktop`). Os ícones oficiais foram substituídos por `icon_cliente.ico` e `icon_cliente.png`.

---

## 3. Versão do Técnico (Branch `build/cmtech`)

Esta branch é voltada para a equipe de suporte.
- **Objetivo**: O técnico usará o aplicativo para controlar outras máquinas. O painel de recebimento de suporte (ID local e Senha) foi desativado e uma validação Zero-Trust foi adicionada à comunicação.

### Alterações:
- **Desktop (`flutter/lib/desktop/pages/desktop_home_page.dart`)**: O lado esquerdo (`buildLeftPane()`) foi removido, forçando a interface a renderizar somente o `buildRightPane()` (painel de conexão remota).
- **Mobile (`flutter/lib/mobile/pages/home_page.dart`)**: A guia de identificação do servidor (`ServerPage`) foi desativada e ocultada.
- **Branding**: O aplicativo teve seu `app_name` alterado para `CMTECH` (Android, Windows, Linux) e os ícones foram substituídos por `icon_tech.ico` e `icon_tech.png`.
- **Validação Zero-Trust (`flutter/lib/common.dart`)**: Adicionada uma verificação no escopo inicial da função `connect()`. A função agora checa `!gFFI.userModel.isLogin`. Se o técnico não estiver autenticado/logado corretamente na instância (servidor Oauth/API da Casa Magalhães), a conexão aborta e exibe uma mensagem nativa exigindo autenticação.

## Dicas para Próximas Atualizações (IAs ou Humanos)

- **Mudança de Versão do RustDesk**: Ao dar "merge" no upstream do repositório oficial do RustDesk, sempre atente-se às mudanças de UI (`home_page.dart` e `desktop_home_page.dart`). O Flutter sofre refatorações constantes pelos criadores originais do RustDesk, o que pode quebrar a injeção do `--dart-define` ou as remoções da aba.
- **Variáveis CI/CD**: Para rodar localmente no seu computador simulando a CI/CD, chame o comando flutter build passando o `dart-define`:
  `flutter build windows --dart-define=RENDEZVOUS_SERVER="seu-servidor" --dart-define=KEY="sua-chave"`
- **Controle de Branches**: Nunca desenvolva novas configurações diretamente em `build/cmtech` ou `build/cmclientes`. Crie as lógicas baseadas em configurações híbridas na `master`, e só realize modificações de UI e bloqueios hardcoded em suas respectivas branches derivadas, garantindo facilidade de merge futuro.
