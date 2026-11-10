# Mineração de Processos: Análise de Logs de Autenticação

Projeto de análise de logs de eventos para identificação de padrões de uso e conformidade de fluxo de autenticação (Senha + MFA) utilizando Python.

---

## Evidência de Execução

![Resultado da Execução no Terminal](assets/execucao_terminal.png)

---

## Resultados da Análise (Entrega da Atividade)

### Bloco 1. Event Log (`descoberta-0001`)

| caso | atividade | instante |
| :--- | :--- | :--- |
| `descoberta-0001` | `falha_senha` | `2026-10-01T12:01:01+00:00` |
| `descoberta-0001` | `falha_senha` | `2026-10-01T12:01:02+00:00` |
| `descoberta-0001` | `senha_aceita` | `2026-10-01T12:01:03+00:00` |
| `descoberta-0001` | `desafio_mfa` | `2026-10-01T12:01:04+00:00` |
| `descoberta-0001` | `login_sucesso` | `2026-10-01T12:01:05+00:00` |

---

### Bloco 2. Traço versus Cesta (`descoberta-0001`)

* **Traço:** `falha_senha → falha_senha → senha_aceita → desafio_mfa → login_sucesso`
* **Conjunto:** `{desafio_mfa, falha_senha, login_sucesso, senha_aceita}`

---

### Bloco 3. Caminho Feliz — Suporte e Confiança

Regra avaliada: `(senha_aceita → desafio_mfa) → login_sucesso`

* **Sessões em que o prefixo cabe (`senha_aceita → desafio_mfa`):** 900
* **Sessões em que a regra completa cabe (`senha_aceita → desafio_mfa → login_sucesso`):** 700
* **Suporte da regra:** `700/2000`
* **Confiança da regra:** `700/900`

**Interpretação no produto:**
* **Suporte:** Entre todas as 2.000 tentativas de login analisadas, é um comportamento observado em 700 casos a pessoa aceitar a senha, receber o desafio MFA e concluir a autenticação com sucesso.
* **Confiança:** Para os usuários que já aceitaram a senha e foram apresentados ao desafio MFA, é frequente que 700 em 900 deles concluam o login com sucesso em vez de abandonar a sessão ou falhar no segundo fator.

---

### Bloco 4. Ordem no Fluxo (Senha → MFA)

* **Padrão A — Conjunto `{senha_aceita, desafio_mfa}`**
  * **Significado:** Tentativas em que a senha foi aceita e o MFA foi pedido, olhando apenas co-ocorrência.
  * **Suporte:** `900/2000`

* **Padrão B — Sequência `desafio_mfa → senha_aceita`**
  * **Significado:** Tentativas em que o sistema registrou o pedido de MFA antes da validação da senha aceita (inversão do fluxo previsto).
  * **Contagem:** `0`

---

## Como Executar o Script

1. Clone este repositório:
   ```bash
   git clone [https://github.com/seu-usuario/nome-do-repositorio.git](https://github.com/seu-usuario/nome-do-repositorio.git)
   ```
2. Navegue até a pasta do projeto:
   ```bash
   cd nome-do-repositorio
   ```
3. Execute o script em Python:
   ```bash
   python3 analise_logs.py
   ```
