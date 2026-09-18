# Manual Definitivo de Reconstrução (Disaster Recovery) - Casa Magalhães

Este documento é o manual oficial de "Disaster Recovery" para recriar o ecossistema White-label do RustDesk para a Casa Magalhães a partir do zero (um clone limpo do upstream original). Siga os passos cirurgicamente para não perder nenhuma das regras de segurança e customização.

---

## Seção 1: Infraestrutura (API)

A API customizada e o servidor de sinalização (hbbs/hbbr) devem ser configurados para injetar e forçar a autenticação segura do técnico. Ao rodar o Docker Compose (ou o contêiner isolado da API), é estritamente necessário usar `--network host` (ou redes devidamente roteadas) e injetar as variáveis de identificação do servidor para que as chaves coincidam:

```bash
docker run --network host -e KEY="<sua-chave>" -e ID_SERVER="<seu-servidor>" -e RELAY_SERVER="<seu-relay>" ...
```

---

## Seção 2: Blindagem Zero-Trust (Rust e Flutter)

**[CRÍTICO]** A dupla trava a seguir é vital para impedir o erro fatal `"TCP deadline elapsed"` que ocorre ao fazer login. A API, sob nenhuma hipótese, deve ditar ou sobrescrever a configuração de rede injetada nativamente durante a compilação.

1. **Bloqueio no Rust:**
   Localize o arquivo `src/hbbs_http/sync.rs`. Deve-se neutralizar a extração e salvamento das propriedades de rede vinda do servidor na camada Rust. O Rust não deve aceitar propriedades de rede empurradas durante a sincronização de configurações de perfil.
2. **Bloqueio no Flutter (Dart):**
   No arquivo `flutter/lib/common.dart`, localize a função `setServerConfig(...)`. É obrigatório comentar ou remover absolutamente todas as chamadas de `bind.mainSetOption` para `custom-rendezvous-server`, `relay-server`, `api-server` e `key` nesta função (e qualquer outra que dispare após o login). Isso amputa qualquer sobreposição por parte do backend.

---

## Seção 3: CI/CD

A injeção de parâmetros nativos deve ser blindada contra falhas de escape de *strings* Base64 (que frequentemente contêm `+` e `=`).

- Em `build.py` e nos fluxos do GitHub Actions, a injeção do `--dart-define` deve proteger o valor com aspas e evitar espaços errados.
- **Formato correto:** `--dart-define="KEY=${VALUE}"`
- A pipeline deve recuperar os *secrets* de forma literal para o *build*.

---

## Seção 4: Injeção de Assets

Para que o Flutter reconheça o diretório centralizado de customizações, declare no manifesto e propague para os ícones físicos:

1. **Manifesto (`flutter/pubspec.yaml`):**
   ```yaml
   flutter:
     assets:
       - assets_casamagalhaes/
   ```
2. **Substituição Física Obrigatória:**
   - **Windows:** Copie o ícone corporativo (`.ico`) para `flutter/windows/runner/resources/app_icon.ico`.
   - **Android:** Copie a imagem (`.png`) para as pastas `flutter/android/app/src/main/res/mipmap-*/ic_launcher.png`.
   - **Linux/Global:** Copie a imagem (`.png`) para a raiz do repositório como `res/rustdesk.png`.

---

## Seção 5: Customizações CMCLIENTES (Token)

O binário de cliente (CMCLIENTES) opera de forma unilateral: ele serve para que a Casa Magalhães preste suporte, sem viabilizar uso interno indevido do cliente.

- **Janela e Aspecto:** Janela principal `isMainWindow` cravada em `400x650` (`flutter/lib/main.dart`).
- **Abas Removidas:** `Segurança`, `Exibição`, `Conta` e `Impressora` foram eliminadas das configurações (`desktop_setting_page.dart`).
- **Cores UAC e LGPD:** Instanciação do construtor de botões para aceitar o parâmetro customizado `bgColor`, formatado com a cor primária corporativa, e os textos padronizados da LGPD.

---

## Seção 6: Customizações CMTECH (Suporte)

A versão do técnico (CMTECH) contém a identidade visual robusta da empresa, restringindo distrações.

- **Tema Global:** Em `flutter/lib/common.dart`, `ThemeData` sobrescrito com a cor primária `0xFF002244` (Azul Marinho) e a secundária/accent `0xFF00E600` (Verde Claro).
- **Tamanho Focado:** Janela configurada rigidamente para `420x650`.
- **Limpeza do Webauth:** Desabilitação total do botão "Continuar com Webauth" (OIDC) e do separador visual "ou" no componente de `login.dart`.
- **Assinatura Corporativa:** A aba "Sobre" carrega o *Copyright* e os avisos legais integralmente customizados com as credenciais da Casa Magalhães.
