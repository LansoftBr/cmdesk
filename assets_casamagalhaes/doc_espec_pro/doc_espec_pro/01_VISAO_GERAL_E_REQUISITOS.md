# Visão Geral e Requisitos de Negócio (CMDesk White-label)

**Autor:** Gerência de Projetos
**Objetivo:** Definir o escopo, visão de negócio e requisitos funcionais para a reconstrução do CMDesk (solução proprietária de suporte remoto da Casa Magalhães) a partir de um fork limpo do [RustDesk Upstream](https://github.com/rustdesk/rustdesk).

---

## 1. Visão do Produto
O CMDesk é uma customização (White-label) do RustDesk, projetada para fornecer acesso remoto seguro, rápido e auditável entre o time técnico da Casa Magalhães e seus clientes. O sistema é dividido em dois builds distintos que não se misturam:
1. **CMCLIENTES:** App distribuído para o cliente final. O cliente apenas *recebe* conexões.
2. **CMTECH:** App utilizado internamente pelos técnicos para *originar* conexões e gerenciar os endpoints.

## 2. Requisitos de Negócio (Business Requirements)
- **Segurança Restritiva (Zero-Trust):** A identidade da rede (Chaves Públicas, Endereço do Relay e Servidor de Rendezvous) deve ser inalterável pelo usuário e não pode ser sobrescrita remotamente pela API, evitando sequestro de sessão ou falhas operacionais (o bug crítico "TCP deadline elapsed").
- **Identidade Visual:** Todo o software deve refletir a marca da Casa Magalhães. Cores corporativas (Azul Marinho e Verde Claro), logos horizontais, ícones físicos (`.exe`, `.apk`, titlebar) e restrições legais (Termos LGPD).
- **Usabilidade Direcionada:** Os recursos que não pertencem ao perfil devem ser completamente extirpados da UI (não apenas escondidos). O cliente não deve conseguir realizar login de técnico nem ter acesso a abas avançadas.

---

## 3. Requisitos Funcionais e Não-Funcionais

### 3.1. Requisitos do Build CMCLIENTES (Cliente)
- **RF-C01:** O aplicativo deve possuir dimensões de janela cravadas em `400x650` (modo compacto).
- **RF-C02:** A interface deve suprimir inteiramente as abas de configurações: `Segurança`, `Exibição`, `Conta` e `Impressora`.
- **RF-C03:** O fluxo de OIDC/Webauth e campos de login não devem estar disponíveis para o cliente.
- **RF-C04:** Apresentar termos da LGPD e aviso do UAC (User Account Control) utilizando botões na cor corporativa base.

### 3.2. Requisitos do Build CMTECH (Técnico)
- **RF-T01:** O aplicativo deve ter a janela configurada em `420x650`.
- **RF-T02:** Tema global (ThemeData) injetado com cores `0xFF002244` (Azul Marinho) e `0xFF00E600` (Verde Claro).
- **RF-T03:** O componente de Login não deve exibir o botão de OIDC/Webauth (mantendo apenas usuário e senha locais/proprietários).
- **RF-T04:** A aba de 'Sobre/About' deve ser purgada de referências a terceiros e conter os Copyrights exclusivos da Casa Magalhães.

### 3.3. Requisitos Não-Funcionais (RNF)
- **RNF-01 (Infraestrutura):** O ecossistema depende de uma API de controle de usuários e um conjunto HBBS/HBBR. A injeção de parâmetros na compilação deve ocorrer via `--dart-define` de modo seguro, lidando estritamente com strings Base64.
- **RNF-02 (Atualização de Código):** Qualquer reconstrução futura advinda de um novo *fork* do RustDesk deve aplicar as travas de rede e UI por meio de *patches* controlados, para garantir escalabilidade da manutenção sem comprometer o core de segurança do fork original.
