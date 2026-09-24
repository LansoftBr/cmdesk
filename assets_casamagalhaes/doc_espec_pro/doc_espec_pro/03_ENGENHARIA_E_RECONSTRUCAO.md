# Guia de Engenharia e Reconstrução

**Autor:** Engenharia de Software
**Objetivo:** Instruções "mão na massa" (step-by-step) de como executar um hard fork do repositório original do RustDesk e replicar toda a engenharia CMDesk (Zero-Trust, CI/CD e Assets).

---

## 1. Preparação do Repositório (Fork Original)

A partir de um Fork ou cópia limpa do repositório `rustdesk/rustdesk` no GitHub:

1. Inicie e faça o checkout em duas novas branches de trabalho, protegendo a master:
   ```bash
   git checkout -b build/cmclientes-homolog
   # e separadamente para o tecnico:
   git checkout -b build/cmtech-homolog
   ```
2. Crie na raiz do repositório a pasta de customizações visuais exclusivas da empresa:
   `mkdir assets_casamagalhaes`

3. Mova/inclua dentro desta pasta os logos `logo_cliente.png`, `icon_cliente.ico`, `icon_cliente.png` e as variantes respectivas do técnico (`icon_tech.ico`, `icon_tech.png`).

---

## 2. Passo a Passo Técnico de Reconstrução

### 2.1. Inclusão dos Assets
Edite o arquivo `flutter/pubspec.yaml` e mapeie a pasta de assets na seção `flutter`:
```yaml
flutter:
  assets:
    - assets/
    - assets_casamagalhaes/
```
Após o mapeamento, copie os ícones físicos nos níveis nativos:
- **Windows:** Substitua o `flutter/windows/runner/resources/app_icon.ico` pelo `.ico` desejado.
- **Android:** Distribua as resoluções necessárias em `flutter/android/app/src/main/res/mipmap-*/ic_launcher.png`.
- **Linux:** Adicione em `res/rustdesk.png`.

### 2.2. Intervenção Cirúrgica (Zero-Trust Flutter)
Esta alteração é idêntica para o build de Cliente e do Técnico e é a mais importante de todo o processo de engenharia.

1. Abra o arquivo `flutter/lib/common.dart`.
2. Busque pela assinatura da função `Future<bool> setServerConfig(`.
3. Navegue para o final da função, identificando o seguinte bloco:
   ```dart
   // should set one by one
   await bind.mainSetOption(
       key: 'custom-rendezvous-server', value: config.idServer);
   await bind.mainSetOption(key: 'relay-server', value: config.relayServer);
   await bind.mainSetOption(key: 'api-server', value: config.apiServer);
   await bind.mainSetOption(key: 'key', value: config.key);
   ```
4. **Comente rigorosamente todo este bloco**. Nenhuma propriedade vital de rede pode ser executada ou reescrita após a compilação do Flutter.

### 2.3. Execução CMCLIENTES (Cliente)

Estando na branch de cliente (`build/cmclientes-homolog`):
1. **Configuração de Janela:** Em `flutter/lib/main.dart`, logo no método principal ou na parametrização do `DesktopWindow`, crave o tamanho (`minimumSize` e initial size) em `400x650`.
2. **Logo na Titlebar:** Em `flutter/lib/desktop/widgets/tabbar_widget.dart` (ou na respectiva versão do RustDesk atualizada), modifique o método `loadIcon(16)` ou equivalente no top header para `Image.asset('assets_casamagalhaes/icon_cliente.png', width: 16)`.
3. **Logo Principal:** Adicione `Image.asset('assets_casamagalhaes/logo_cliente.png', width: 150)` no início do método `buildLeftPane()` ou do `buildRightPane()` em `desktop_home_page.dart`.
4. **Restrição de Acesso:** Em `flutter/lib/desktop/pages/desktop_setting_page.dart`, vá até o switch case ou a lista de construção das *tabs* (`_tabs`) e apague sem hesitação as abas: **Segurança**, **Exibição**, **Conta** e **Impressora**.
5. **UAC / Botões:** Se o componente `FixedWidthButton` (em `flutter/lib/desktop/widgets/button.dart`) não possuir suporte para cor de fundo, estenda sua classe adicionando o parâmetro `final Color? bgColor;` para que as telas de termos da LGPD e aviso do UAC sigam as cores da corporação.
6. Commit das alterações.

### 2.4. Execução CMTECH (Técnico)

Estando na branch do técnico (`build/cmtech-homolog`):
1. **Configuração de Janela:** Semelhante ao cliente, restrinja o tamanho para `420x650` em `flutter/lib/main.dart`.
2. **Tema Global:** Em `flutter/lib/common.dart`, localize a variável `MyTheme` (ou a instância de `ThemeData`) e sobrescreva os hexadecimais principais para `0xFF002244` (Azul Marinho) e a tipografia de destaque para `0xFF00E600` (Verde Claro).
3. **Restrição de Login Webauth:** Em `flutter/lib/common/widgets/login.dart`, procure o componente visual e exclua a renderização do botão "Logar com OIDC / Webauth", forçando o técnico a usar apenas a caixa de texto direta com a API autenticada local.
4. **Copyright (Sobre):** Em `flutter/lib/desktop/pages/desktop_setting_page.dart`, procure pela aba 'Sobre'. Modifique o copyright, remover a versão de checagem online e os textos open source padronizados que conflitem com a apresentação do software da Casa Magalhães.
5. Commit das alterações.

## 3. Diretrizes de Compilação CI/CD
Tanto para gerar o artefato do CMCLIENTES quanto do CMTECH, certifique-se de invocar o runner passando o comando Dart correto com as devidas aspas:

```bash
flutter build windows --dart-define="KEY=${KEY_VALUE}" --dart-define="API_SERVER=${API_VALUE}" --release
```
