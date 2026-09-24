# Arquitetura de Software e Diretrizes de Segurança

**Autor:** Arquitetura de Software
**Objetivo:** Estabelecer as fundações técnicas e os vetores de mitigação de risco do sistema CMDesk, com foco na injeção de ambiente, gerenciamento de estado e arquitetura Zero-Trust de rede.

---

## 1. Arquitetura Geral do Ecossistema RustDesk Modificado

O RustDesk é construído sob uma arquitetura de cliente leve em Dart (Flutter) e um *core* robusto e de baixo nível em Rust (tokio/ffi). As comunicações e orquestração de vídeo, input, e rede P2P (Rendezvous/Relay) ocorrem 100% no core Rust.
A arquitetura do CMDesk divide a esteira de build (via CI/CD) para gerar dois *flavorings* separados.

- **Componente Nativo (Rust):** Mantém a conectividade. Nenhuma interface é desenhada aqui.
- **Componente UI (Flutter):** Manipula configurações de usuário, renderização visual, menus e a troca de mensagens de controle com a API externa.
- **API Externa:** Fornece o catálogo de contatos e validação de tokens JWT (Login e OIDC).

## 2. Paradigma Zero-Trust e O Bug Crítico de Rede

**O Problema (Vulnerabilidade Padrão):** 
No upstream do RustDesk, o login na API faz com que o cliente receba um *payload* de configurações (`LoginResponse.user`). O aplicativo Flutter, visando facilitar instâncias On-Premise, aciona métodos como `setServerConfig` e injeta esses valores no `bind.mainSetOption` do Rust. Quando a API de terceiros ou um OIDC envia propriedades de rede nulas ou diferentes, as chaves hardcoded e parâmetros nativos de rede (injetados na compilação) são reescritos, causando a desconexão fatal: `"Failed to secure tcp: deadline has elapsed"`.

**A Solução de Arquitetura (Blindagem):**
A arquitetura do CMDesk estabelece uma regra fundamental: **A rede e a criptografia nascem e morrem com o binário compilado. Nenhuma API pode alterar a topologia P2P/Rendezvous de um cliente atrelado.**

### 2.1. Trava Arquitetural na Camada Flutter
Em `flutter/lib/common.dart` (método `setServerConfig`), a arquitetura impõe a total **amputação** de qualquer chamada que altere as configurações sensíveis via API ou Clipboard. O código Dart foi modificado para ignorar e nunca propagar comandos `bind.mainSetOption` para os campos:
- `custom-rendezvous-server`
- `relay-server`
- `api-server`
- `key`

### 2.2. Trava Arquitetural na Camada Rust (Opcional, porém Fortemente Recomendado)
Em `src/hbbs_http/sync.rs`, o parseador HTTP que desserializa configurações avançadas do servidor deve ter os blocos que populam a rede da estrutura de options permanentemente inativados ou envoltos em macros de compilação condicionais de segurança.

## 3. Gestão de Build, CI/CD e Secrets

Devido a problemas de quebra de *strings* Base64 em pipelines GitHub Actions, a arquitetura exige injeção determinística de parâmetros:
- A pipeline deve resgatar os parâmetros de rede no estágio de compilação.
- Variáveis estáticas devem ser encapsuladas perfeitamente para evitar corrupção por caracteres como `+` e `=`.
- Padrão arquitetural de injeção no Build: `--dart-define="KEY=${VALUE}"`.

## 4. Gerenciamento Visual (Global Theming)

Ao invés de aplicar *hardcodes* espalhados pela UI do Flutter, a arquitetura do CMDesk baseia sua UI na sobrescrita do componente nativo global em `flutter/lib/common.dart` (`MyTheme`). 
- As cores base e tipografia são amarradas no raiz do `MaterialApp`.
- Assets isolados da organização são empacotados em um diretório não poluidor: `assets_casamagalhaes/`. O arquivo `pubspec.yaml` expõe o diretório inteiro, permitindo atualizações de ícones sem intervenções longas no código.
