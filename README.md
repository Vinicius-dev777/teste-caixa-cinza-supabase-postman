# Teste Caixa Cinza com Supabase e Postman

**Aluno:** Vinicius Teixeira Melo — RA 251136
**Curso:** Análise e Desenvolvimento de Sistemas — Centro Universitário FACENS
**Componente:** UX/UI e Testes de Software (AS406TSN1)
**GitHub:** [Vinicius-dev777](https://github.com/Vinicius-dev777)

> AVISO (apague antes de entregar): trechos entre `[COLCHETES]` são para você escrever com suas palavras. Tudo que está fora dos colchetes vem dos dados reais que você me passou.

---

# Introdução

Este repositório documenta a atividade de **testes funcionais de caixa cinza** aplicados a uma API de autenticação. Foi criado um ambiente de autenticação no **Supabase**, e as requisições HTTP foram executadas e validadas no **Postman Desktop**. Não há interface gráfica: toda a validação é feita diretamente sobre o endpoint de login.

Estrutura do repositório:

```
/README.md
/postman/    -> collection e modelo de environment exportados
/prints/     -> evidências (capturas de tela reais)
/planilha/   -> planilha de registro dos testes
```

# Objetivo da Atividade

Aplicar testes de caixa cinza em uma API de autenticação, utilizando ferramentas de mercado para:

- configurar um ambiente de autenticação com Supabase;
- criar usuários para testes;
- compreender o funcionamento de requisições HTTP;
- executar testes no Postman;
- validar respostas de autenticação (status HTTP, JSON, mensagens de erro e *access token*);
- documentar tecnicamente todo o processo.

No teste de caixa cinza o testador conhece parcialmente o sistema (endpoint, headers, corpo JSON e respostas HTTP), sem acesso ao código-fonte interno.

# Configuração do Supabase

**Como o projeto foi criado:** [descreva: criação da conta, "New project", nome do projeto, região, senha do banco].

**Tipo de autenticação utilizada:** autenticação por **e-mail e senha** (provider *Email* habilitado em *Authentication → Sign In / Providers*). [Informe se *Confirm email* ficou ligado ou desligado.]

**Como o usuário foi criado:** [descreva: *Authentication → Users → Add user*, e se *Auto Confirm User* foi marcado: CONFIRMAR]. O e-mail real do usuário de teste não é exibido neste documento.

**Objetivo da configuração:** disponibilizar um serviço real de autenticação (com banco Postgres e API pronta) para que as requisições de login pudessem ser testadas.

**Chave de API:** no painel atual do Supabase a chave aparece como *publishable*, e a chave *anon* fica na aba *Legacy*. Esse ponto gerou dificuldade (ver "Erros encontrados").

**Evidências:**

| Evidência | Print |
|---|---|
| Projeto criado | ![projeto](prints/01-projeto-criado.png) |
| Painel do Supabase | ![painel](prints/02-painel-supabase.png) |
| Autenticação habilitada | ![auth](prints/03-auth-habilitada.png) |
| Usuário criado | ![usuario](prints/04-usuario-criado.png) |
| Credenciais da API (chave mascarada) | ![api](prints/05-credenciais-api.png) |
| Identificação do aluno no ambiente | ![aluno](prints/06-identificacao-aluno.png) |

# Configuração do Postman

**Workspace:** "Caixa Cinza - Vinicius 251136" [como foi criado: menu *Workspaces → Create Workspace*].

**Environment:** "Supabase Caixa Cinza" [como foi criado: aba *Environments*].

Variáveis configuradas:

| Variável | Descrição | Valor no README |
|---|---|---|
| `base_url` | URL da API do projeto Supabase (não é o link do painel) | `https://<ID-DO-PROJETO>.supabase.co` |
| `api_key` | Chave de API do projeto | `XXXXX` |
| `email` | E-mail do usuário criado para testes | `usuario@email.com` |
| `password` | Senha do usuário de teste | `******` |
| `access_token` | Preenchida automaticamente pelo script do login válido | — |

**Finalidade das variáveis:** evitar repetir dados em cada requisição e permitir trocar de ambiente/credenciais sem editar cada teste.

**Como o Postman é utilizado:** as requisições ficam organizadas em uma *Collection* ("Supabase Auth - Caixa Cinza"); cada cenário é uma requisição própria, com *script de teste* que valida automaticamente status e corpo da resposta.

**Evidências:**

| Evidência | Print |
|---|---|
| Workspace criado | ![workspace](prints/07-workspace.png) |
| Environment criado e variáveis configuradas | ![env](prints/08-environment.png) |
| Organização da collection | ![collection](prints/09-collection.png) |

> Observação sobre o print 08: [explique com suas palavras se a `base_url` aparece ali com o link do painel, que foi o erro que causou o 405, e que depois foi corrigida para a URL da API].

# Configuração das Requisições

- **Endpoint:** `{{base_url}}/auth/v1/token?grant_type=password`
- **Método HTTP:** `POST`
- **Finalidade:** autenticar um usuário por e-mail e senha. Em caso de sucesso, a API retorna `access_token`, `refresh_token` e dados do usuário.

**Headers:**

| Header | Valor |
|---|---|
| `apikey` | `{{api_key}}` |
| `Authorization` | `Bearer {{api_key}}` |
| `Content-Type` | `application/json` |

**Body (raw / JSON):**

```json
{
  "email": "{{email}}",
  "password": "{{password}}"
}
```

No CT04 (campos vazios), o body é `{"email": "", "password": ""}`.

**Evidências:**

| Evidência | Print |
|---|---|
| Endpoint configurado | ![endpoint](prints/10-endpoint.png) |
| Headers | ![headers](prints/11-headers.png) |
| Body JSON | ![body](prints/12-body.png) |
| Requisição criada no Postman | ![requisicao](prints/13-requisicao.png) |

A collection exportada está em [`/postman`](postman/).

# Execução dos Testes

Foram executados os 5 cenários obrigatórios. Cada um é uma requisição da collection **Supabase Auth - Caixa Cinza**.

| Cenário | Entrada Utilizada | Resultado Esperado | Resultado Obtido | Status |
|---|---|---|---|---|
| CT01 - Login válido | E-mail e senha corretos | Status 200 + `access_token` | 200 OK com `access_token` | OK |
| CT02 - Senha incorreta | E-mail correto, senha errada | Erro de autenticação (400) | 400, `invalid_credentials`, "Invalid login credentials" | OK |
| CT03 - Usuário inexistente | E-mail não cadastrado | Acesso negado (400) | 400, `invalid_credentials`, "Invalid login credentials" | OK |
| CT04 - Campos vazios | `email` e `password` vazios | Erro de validação | 400, `validation_failed`, "missing email or phone" | OK |
| CT05 - Credenciais inválidas | E-mail e senha inválidos | Mensagem de erro | 400, `invalid_credentials`, "Invalid login credentials" | OK |

**Cenários executados e entradas:** [descreva cada um brevemente com suas palavras].

**Resultados obtidos:** os resultados estão na tabela acima. O CT02 e o CT03 retornaram exatamente a mesma resposta (`invalid_credentials`), o que impede descobrir se um e-mail existe ou não (evita enumeração de usuários). [comente com suas palavras]

**Erros encontrados (dificuldades reais):**

- Achar a API Key no painel novo do Supabase, que mostra a chave *publishable* e deixa a *anon* na aba *Legacy*.
- Erro **405 Method Not Allowed**, causado por usar na `base_url` o link do painel do Supabase em vez da URL da API.
- O body da requisição sumiu ao trocar o tipo para JSON.

[Complete ou reescreva com suas palavras.]

**Evidências:**

| Evidência | Print |
|---|---|
| CT01 - Login válido (status 200 + token mascarado) | ![ct01](prints/14-ct01-login-valido.png) |
| CT02 - Senha incorreta | ![ct02](prints/15-ct02-senha-incorreta.png) |
| CT03 - Usuário inexistente | ![ct03](prints/16-ct03-usuario-inexistente.png) |
| CT04 - Campos vazios | ![ct04](prints/17-ct04-campos-vazios.png) |
| CT05 - Credenciais inválidas | ![ct05](prints/18-ct05-credenciais-invalidas.png) |
| Resultado da execução completa (Collection Runner) | ![runner](prints/19-runner.png) |

# Registro dos Testes

Os testes foram registrados na planilha [`planilha/PLANO_DE_TESTE_CAIXA_CINZA.xlsx`](planilha/PLANO_DE_TESTE_CAIXA_CINZA.xlsx), com as abas **INFORMATIVO** e **REGISTRO DE TESTES**, contendo: ID do teste, cenário, entrada, resultado esperado, resultado obtido, status e observações.

**Como os testes foram registrados:** [descreva com suas palavras].

**Importância da documentação dos testes:** [escreva com suas palavras. Pontos para pensar: rastreabilidade, repetição dos testes, evidência para o cliente/equipe, base para regressão].

**Falhas identificadas:** [liste. Se nenhum cenário falhou, diga isso e cite as dificuldades de configuração acima].

| Evidência | Print |
|---|---|
| Planilha preenchida | ![planilha](prints/20-planilha.png) |
| Exemplos de registros | ![registros](prints/21-registros.png) |

# Resultados Obtidos

No **Collection Runner**, a execução completa teve **10 testes, 10 aprovados e 0 falhas**. [Complete com suas palavras: o que isso mostra sobre a autenticação.]

| Total de cenários | Aprovados | Reprovados |
|---|---|---|
| 5 | 5 | 0 |

# Conclusão

<!-- Escreva com suas palavras. O enunciado exige comentar TODOS os pontos abaixo: -->

- **Os testes foram executados corretamente?** [ ]
- **A autenticação funcionou?** [ ]
- **Quais dificuldades encontrou?** [ ]
- **Quais falhas foram identificadas?** [ ]
- **Quais melhorias poderiam ser implementadas?** [ ]
- **Qual a importância dos testes caixa cinza em APIs?** [ ]
