# Manual Definitivo de Reconstrução (Disaster Recovery) - Casa Magalhães

Este documento é o manual oficial de "Disaster Recovery" para recriar o ecossistema White-label do RustDesk para a Casa Magalhães a partir do zero (um clone limpo do upstream original). Siga os passos cirurgicamente para não perder nenhuma das regras de segurança e customização.

---

## Seção 1: Infraestrutura (API e Zero-Trust)

A API customizada e o servidor de sinalização (hbbs/hbbr) devem ser configurados para injetar e forçar a autenticação segura do técnico.

1. **Subida do Docker da API:**
   Ao rodar a API, é estritamente necessário usar `--network host` (ou redes devidamente roteadas) e injetar as variáveis de identificação do servidor para que as chaves coincidam:
   ```bash
   docker run --network host -e KEY="<sua-chave>" -e ID_SERVER="<seu-servidor>" -e RELAY_SERVER="<seu-relay>" ...
   ```
2. **Blindagem do App na Camada Rust:**
   Para prevenir o bug crasso onde o login via Oauth retorna um *payload* de rede vazio e sobrescreve as chaves fixas do cliente (causando o erro `deadline has elapsed` no TCP), você deve neutralizar a sobrescrita de rede localizando e removendo/comentando as linhas em `src/hbbs_http/sync.rs` que fazem o *parse* e sobrescrevem as `options` locais de rede durante a sincronização de configurações de perfil.

---

## Seção 2: CI/CD (GitHub Actions)

A injeção de parâmetros nativos deve ser blindada contra falhas de escape de *strings* Base64 (que frequentemente contêm `+` e `=`).

1. **Sintaxe Correta no Build:**
   Em `build.py` e nos YAMLs de Actions (ex: `.github/workflows/flutter-build.yml`), a injeção do `--dart-define` deve proteger o valor com aspas e evitar espaços errados na flag.
   - **Formato correto:** `--dart-define="KEY=${VALUE}"`
   - O Python ou Shell Script deve recuperar os `secrets` e repassá-los dessa maneira no array de argumentos do comando `flutter build`.

---

## Seção 3: Injeção de Assets

Para que o Flutter reconheça o diretório centralizado de customizações, declare no manifesto e propague para os ícones físicos:

1. **Manifesto (`flutter/pubspec.yaml`):**
   ```yaml
   flutter:
     assets:
       - assets_casamagalhaes/
   ```
2. **Ícones Nativos do OS:**
   Os arquivos base devem ser sobrescritos nos seus respectivos ambientes nativos antes da compilação:
   - **Windows:** Copie o `.ico` para `flutter/windows/runner/resources/app_icon.ico`
   - **Android:** Copie o `.png` e distribua nas pastas `flutter/android/app/src/main/res/mipmap-*/ic_launcher.png`
   - **Linux:** Copie o `.png` para a raiz como `res/rustdesk.png`

---

## Seção 4: CMCLIENTES (Token)

A versão do cliente deve ser uma via de mão única (ele recebe suporte, mas não presta suporte).

1. **Janela e Aspecto:** 
   Restringir tamanho da janela principal (`isMainWindow`) para `400x650` em `flutter/lib/main.dart`.
2. **Restrição de Acesso:**
   Em `flutter/lib/desktop/pages/desktop_setting_page.dart`, remover inteiramente as abas sensíveis: `Segurança`, `Exibição`, `Conta` e `Impressora`.
3. **Customização LGPD e UAC:**
   - Adicionar os textos e aceites de termos da LGPD na inicialização se aplicável.
   - No `flutter/lib/desktop/widgets/button.dart` (classe `FixedWidthButton`), incluir suporte ao parâmetro `bgColor` e instanciá-lo na UI do `desktop_home_page.dart` (no aviso do UAC) utilizando a cor base corporativa.

---

## Seção 5: CMTECH (Técnico)

A versão do técnico deve respirar a identidade visual da empresa e focar apenas no painel de controle, barrando também usos indesejados.

1. **Injeção do ThemeData Global:**
   Em `flutter/lib/common.dart`, altere as instâncias de cor do `ThemeData`:
   - **Primary Color:** Azul Marinho (`Color(0xFF002244)`)
   - **Accent/Secondary:** Verde Claro (`Color(0xFF00E600)`)
2. **Tamanho Focado:**
   Janela configurada em `flutter/lib/main.dart` para `420x650`.
3. **Limpeza do Webauth:**
   Em `flutter/lib/common/widgets/login.dart`, neutralizar o botão de `Continuar com Webauth` (OIDC) e o separador `"ou"`, deixando aparente apenas a caixa de Usuário e Senha normais.
4. **Assinatura Corporativa:**
   Em `flutter/lib/desktop/pages/desktop_setting_page.dart` (Aba Sobre), remover os textos da Purslane e instituir o Copyright customizado da Casa Magalhães.
