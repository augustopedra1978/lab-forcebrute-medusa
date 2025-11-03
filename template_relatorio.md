# Relatório de Testes — Laboratório Força-Bruta & Password-Spray (Exemplo Preenchido)

**Projeto:** Laboratório Kali + Metasploitable2 / DVWA  
**Data do Teste:** 2025-11-03 (exemplo fictício)  
**Autor:** Augusto C. P. Pedra (exemplo)

---

## 1. Objetivo
Avaliar a resistência de serviços típicos (FTP, Web login via DVWA, e SMB) a ataques de força-bruta e password-spraying em um ambiente de laboratório isolado, documentando métodos, evidências e propondo medidas de mitigação.

## 2. Escopo e Regras
- VMs: Kali (192.168.56.10), Metasploitable2/DVWA (192.168.56.20)  
- Serviços testados: FTP (vsftpd), Web (DVWA, formulário de login), SMB (Samba shares)  
- Regras: testes realizados exclusivamente em ambiente isolado Host-Only; snapshots criados antes e após; logs e pcap coletados; listas de senha pequenas usadas para evitar impacto no serviço.

## 3. Configuração do Ambiente
- VirtualBox (versão de teste), rede Host-Only configurada para isolar as VMs.  
- Snapshot "clean-install" criado antes dos testes.  
- Ferramentas na VM Kali: Medusa (usado em modo controlado), tcpdump, Wireshark (apenas para captura), e scripts de geração/parse de wordlists.

## 4. Ferramentas e Artefatos Produzidos
- Medusa (uso conceitual, pseudocmds no relatório)  
- scripts/generate_wordlist.py (geração de variações)  
- scripts/parse_test_logs.py (parser de logs)  
- wordlists/small_passlist.txt (10 itens)  
- Capturas de tela (anexadas na pasta images/ — fictícias)  
- PCAPs gerados localmente e **anonymizados** antes de publicação (não incluídos neste repositório público)

## 5. Wordlists (resumo)
- users.txt (10 usuários de teste)
- small_passlist.txt (10 senhas de baixa entropia, usadas apenas para demonstração no lab)
- Método: variações geradas via script simples (capitalize, append years, add common suffixes)

## 6. Procedimentos Executados (resumo, pseudocmds)
> **Nota:** Os comandos abaixo são **ilustrativos** e contém placeholders. Só execute comandos reais em ambiente isolado e com autorização.

- **FTP (força-bruta controlada):**  
  Pseudocmd: `medusa --service ftp --target 192.168.56.20 --userlist wordlists/users.txt --passlist wordlists/small_passlist.txt --rate-limit 5`  
  Observação: taxas e limites foram aplicados para evitar desgaste do serviço.

- **Web (DVWA) — formulário:**  
  Automação de POSTs com delay (pseudocódigo):  
  ```py
  for user in users:
      for pwd in passlist:
          POST http://192.168.56.20/dvwa/login.php {username:user, password:pwd}
          sleep(2)
  ```

- **SMB (password-spray):**  
  Pseudocmd: `medusa --service smb --target 192.168.56.20 --userlist wordlists/users.txt --passlist wordlists/small_passlist.txt --rate-limit 2`  
  Abordagem: poucas senhas contra muitos usuários, ritmo lento para evitar detecção imediata por bloqueios simples.

## 7. Resultados (exemplo fictício)

| # | Timestamp (UTC)       | Alvo (IP)      | Serviço | User      | Password       | Resultado | Evidência (arquivo) |
|---|----------------------:|---------------:|--------:|----------:|---------------:|----------:|---------------------:|
| 1 | 2025-11-03T14:05:00Z  | 192.168.56.20  | FTP     | test      | Password123    | SUCCESS   | images/ftp_success_01.png |
| 2 | 2025-11-03T14:12:00Z  | 192.168.56.20  | Web     | dvwa_user | Welcome1       | FAILED    | images/dvwa_attempt_01.png |
| 3 | 2025-11-03T14:35:00Z  | 192.168.56.20  | SMB     | dev       | Admin2025      | FAILED    | images/smb_attempt_01.png |

**Observações sobre os resultados:**  
- O teste de FTP retornou sucesso para a combinação `test/Password123` em ambiente fictício; evidência é um screenshot simulado.  
- O formulário web bloqueou algumas tentativas por comportamento de sessão; observou-se alteração no tamanho da resposta HTTP em casos de sucesso vs. falha.  
- Para SMB, nenhuma credencial válida foi descoberta com a pequena lista utilizada; o teste demonstrou a eficácia relativa de policy-based lockouts e conta de monitoramento.

## 8. Análise e Lições Aprendidas
- Senhas fracas continuam sendo um vetor crítico; mesmo listas pequenas resultaram em acesso FTP no lab fictício.  
- Aplicações web podem sinalizar sucesso por diferenças sutis nas respostas; detecção por WAF/IDS depende de regras configuradas.  
- Password-spraying é um risco real quando senhas fracas são usadas por muitos usuários — mitigação com MFA e detecção por padrão de tentativas é essencial.

## 9. Recomendações de Mitigação (acionáveis)
1. **Política de senhas fortes:** exigir 12+ caracteres e checks contra listas de senhas vazadas.  
2. **MFA:** aplicar 2FA em acessos administrativos e serviços críticos.  
3. **Rate limiting & account lockout:** bloquear ou desacelerar tentativas após limiar configurado (por conta e por IP), com cuidados para evitar DoS.  
4. **Monitoramento / SIEM:** criar correlações para detectar password-spraying (muitas contas alvejadas com poucas senhas).  
5. **Honeypots/Canaries:** criar contas canário para detecção precoce.  
6. **Atualização e hardening:** desabilitar serviços desnecessários, aplicar patches e revisar configurações padrão (ex.: vsftpd, Samba).

## 10. Plano de Revalidação
- Implementar lockout e MFA no ambiente de teste.  
- Re-executar os mesmos testes com as proteções ativas e comparar métricas: tempo até bloqueio, taxa de sucesso, alertas gerados.

## 11. Conclusão
O experimento em ambiente controlado demonstrou que senhas previsíveis e políticas fracas facilitam acessos indevidos. A aplicação imediata de MFA, políticas de senha e monitoramento reduzem significativamente o risco. Este relatório serve como exemplo preenchido para demonstrar formato e tipo de evidência a ser coletada; os testes reais devem ser executados com autorização e em ambientes isolados.

---
*Este arquivo é um exemplo fictício e foi gerado para fins educativos.*
