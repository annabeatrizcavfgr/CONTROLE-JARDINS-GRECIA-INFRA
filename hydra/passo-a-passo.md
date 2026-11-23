# Publicar o Controle Jardins Grécia no Hydra (Coolify) — passo a passo

Este app é só HTML (um arquivo, sem banco e sem servidor próprio). No Hydra isso é o **Caminho 1A**, "HTML estático": Dockerfile com nginx, porta **80**, **nenhuma variável de ambiente** e o Studio não é necessário.

Estrutura do repositório:

```
public/index.html   ← o app (só esta pasta vai para o ar)
Dockerfile          ← nginx, porta 80
nginx.conf          ← /healthz e "sempre conferir versão nova"
.dockerignore
```

## 1. Colocar no GitHub

1. Abra o **GitHub Desktop** → **File → Add local repository** → escolha a pasta `CONTROLE-JARDINS-GRECIA`. (Já está com o primeiro commit feito, na branch `hydra`.)
2. **Publish repository** → o GitHub Desktop pode sugerir outro nome (aqui ficou `CONTROLE-JARDINS-GRECIA-INFRA`); use o nome real nos próximos passos.
3. Se for usar **Public Repository** no Coolify (como no app de metas), **desmarque** "Keep this code private". O código ficará visível para qualquer pessoa, mas não tem nenhum dado real: o app abre vazio e os dados ficam no navegador de quem usa.

## 2. Criar a aplicação no Coolify

Precisa estar no escritório da FGR ou na VPN.

1. `https://hydra.fgr.com.br` → login Microsoft → escolha o **Team do seu departamento** (o mesmo do app de metas, se for o do seu departamento).
2. **Projects → +** → nome `controle-obras` (sugestão) → ambiente **production**.
3. **+ New Resource → Application → Public Repository** (ou o GitHub App do Team, se preferir deixar o repositório privado).
4. Repository URL: `https://github.com/annabeatrizcavfgr/CONTROLE-JARDINS-GRECIA-INFRA` · Branch: **`hydra`**.
5. Server (rodar): **FGRLNX07** · Build server: **FGRLNX09**. Não troque.
6. **Build Pack: Dockerfile** (não Nixpacks).
7. **Ports Exposes: `80`** (não use 3000).
8. **Domains:** `https://controle-jardins-grecia-infra.hydra.fgr.com.br` — **só o endereço, sem nada depois de `.br`** (sem `/healthz`; senão o app não abre na raiz e dá 503). Remova a linha do `www`. Se o nome já estiver em uso por outro app do Lab, escolha outro.
9. **Environment Variables:** nenhuma.
10. **Deploy** e acompanhe o *Deployment log* até ficar **Running** (verde).

## 3. Conferir que subiu

- `https://controle-jardins-grecia-infra.hydra.fgr.com.br/healthz` → mostra `ok`
- `https://controle-jardins-grecia-infra.hydra.fgr.com.br/` → abre o Dashboard vazio
- Cadastre um pedido e uma restrição de teste e confirme que aparecem no Dashboard.

## 4. Atualizar depois

Mudou o `public/index.html`? No GitHub Desktop: **Commit** → **Push origin**. No Coolify: **Redeploy** (ou deixe o deploy automático ligado, se o Team oferecer).

## 5. O que esta versão faz e o que não faz

- Os dados são **compartilhados** entre todos que entram (seção 8). Enquanto o banco não existir, ou se ele estiver fora do ar, o painel funciona em **modo local**: cada navegador com a sua cópia, e um aviso amarelo no topo avisa isso.
- Só abre na **rede FGR (escritório ou VPN)**. De casa, sem VPN, o endereço `*.hydra` não abre.
- Tem **login com a conta FGR** (Microsoft/Entra), veja a seção 7. A tela de login só aparece no endereço `*.hydra.fgr.com.br`.
- As fontes vêm do Google Fonts. Se a rede da FGR bloquear, o app usa a fonte padrão do Windows e funciona do mesmo jeito.

## 6. Próximo passo: dados compartilhados

Para a equipe enxergar os mesmos dados, o caminho no Hydra é o mesmo do app de metas:

1. Schema próprio no Studio, no formato `app_<departamento>_<nome>` e único (por exemplo, no padrão do app de metas, que usa `app_planejamento_sprint`), com RLS ligada e acesso direto fechado.
2. Login Microsoft (Entra) no Auth do app, com Site URL e Redirect URLs do endereço novo.
3. Trocar o `localStorage` por leitura e gravação no banco, por uma API Node pequena no Coolify (com a chave `service_role` só no servidor, nunca no navegador).
4. Depois: ler o FlowMaster (tabela do Fabric) e o UAU automaticamente. Isso depende do TI.

## Se algo der errado

| Sintoma | O que olhar |
| --- | --- |
| Build falha | Confira se o Build Pack é **Dockerfile** e se existe `public/index.html` |
| **Running**, mas 502 ou página vazia | A porta no Coolify tem que ser **80** |
| Endereço não abre em casa | Conecte na VPN da FGR |
| Página antiga depois de atualizar | Faça **Redeploy** e recarregue com Ctrl+F5 |
| Nome de domínio já usado | Escolha outro nome em **Domains** |

## 7. Login com a conta FGR (Microsoft/Entra)

O painel mostra a tela "Entrar com conta Microsoft" no endereço `*.hydra.fgr.com.br` e só deixa entrar contas `@fgr.com.br` (ou `@fgr.com`). Ele usa o Auth do Hydra (`dados.hydra.fgr.com.br`), o mesmo do app de metas. O provedor Microsoft já está ligado no Studio do Lab. Falta só **liberar o endereço novo**:

1. `https://studio.hydra.fgr.com.br` → **Authentication → URL Configuration**.
2. Em **Redirect URLs**, **adicione** (sem apagar as que já existem):
   - `https://controle-jardins-grecia-infra.hydra.fgr.com.br`
   - `https://controle-jardins-grecia-infra.hydra.fgr.com.br/**`
3. **Não mude o Site URL.** O Auth é compartilhado no Lab e o Site URL atual pertence a outro app (o de metas).
4. Depois do `git push`, faça **Redeploy** no Coolify.
5. Teste: abra o endereço → tela de login → conta Microsoft → volta para o painel, com o seu nome no menu lateral ("Entrou como..." e **Sair**).

Se, depois de entrar, você cair em **outro app** (o de metas, por exemplo), é porque o endereço novo não está nas Redirect URLs.

Importante: nesta versão o login controla o **uso da tela**. O arquivo `index.html` continua sendo entregue pelo servidor a quem alcançar o endereço, e ele não contém dados. A proteção de verdade dos dados vem quando eles passarem para o banco, com o servidor conferindo quem está logado.

### Nome de quem usa
No primeiro login, o painel pergunta **"Como você quer ser chamado(a) no painel?"** (já vem preenchido com o nome da conta Microsoft). O nome fica guardado por e-mail, neste navegador, e é usado ao **assumir** um pedido. Dá para mudar depois em **Trocar nome**, no menu lateral. Não existe mais o campo "Meu nome" nem o cadastro de pessoas: as sugestões de responsável vêm dos nomes já usados no painel.

### Ligar e desligar o login
No começo do trecho de login de `public/index.html` há `var LOGIN_LIGADO=true;`. Mudando para `false` e fazendo push e Redeploy, o painel abre direto, sem a tela de login.

## 8. Dados compartilhados (banco do Hydra)

Todo mundo que entra no painel passa a ver e editar os mesmos pedidos, flows, contratações, projetos e restrições. Cada item é uma linha numa tabela do Supabase interno (`dados.hydra.fgr.com.br`); o painel envia só o que mudou e, a cada ~8 segundos, busca o que os outros mudaram. Não precisa de variável de ambiente nem de servidor: o painel fala com o banco usando o login Microsoft de quem está usando.

### Criar o banco (uma vez)
1. `https://studio.hydra.fgr.com.br` → **SQL Editor → New query**.
2. Cole o conteúdo inteiro de [`01-schema.sql`](01-schema.sql) e clique em **Run**. Deve terminar sem erro. (Pode rodar de novo: ele só cria o que ainda não existe.)
3. **Espere uns 5 minutos.** A plataforma libera o schema na API sozinha depois de criado.
4. Confira a liberação (não precisa de login; deve responder `200` e `[]`):

   ```bash
   curl -s -D - -H "apikey: <chave anon do guia>" -H "Authorization: Bearer <chave anon do guia>" \
     -H "Accept-Profile: app_planejamento_controle_obras" \
     "https://dados.hydra.fgr.com.br/rest/v1/docs?select=*&limit=1"
   ```
   `406` significa que ainda não liberou: espere mais um pouco. Se passar de 15 minutos, é caso para o TI.
5. Só depois: `git push` e **Redeploy** no Coolify.

### Como o painel se comporta
- **Primeiro acesso, banco vazio, com dados no navegador:** o painel pergunta se envia os dados que já estavam neste navegador. Responda **OK** uma única vez (na primeira pessoa que entrar); depois os outros já entram vendo tudo.
- **Edição ao mesmo tempo:** se duas pessoas editarem o **mesmo item** quase juntas, vale a última gravação (por item, não por campo). Itens diferentes não se atrapalham.
- **O painel não redesenha a tela enquanto você digita** (formulário aberto, campo em edição, menu aberto). Quando você termina, as mudanças dos outros aparecem.
- **Sem conexão:** aparece um aviso amarelo, as alterações ficam guardadas e são enviadas quando a conexão volta.
- **Banco fora do ar na abertura:** o painel abre em **modo local**, com aviso e botão "Tentar de novo". Nesse modo o que você muda fica só no seu navegador.
- **Excluir** marca a linha como apagada (`deleted = true`) para os outros navegadores ficarem sabendo.

### Se aparecer "Não foi possível abrir os dados compartilhados"
O aviso amarelo traz um **detalhe** com o motivo:

| Detalhe | O que significa | O que fazer |
| --- | --- | --- |
| `HTTP 404` ou `HTTP 406` | O schema ainda não existe ou ainda não foi liberado na API | Rodar o `01-schema.sql` no Studio e esperar uns 5 minutos (até 15) |
| `HTTP 401` | O banco não aceitou o login | Sair e entrar de novo; se persistir, falar com o TI |
| `HTTP 403` | Faltam permissões (a liberação automática ainda não rodou, ou o e-mail não é `@fgr.com.br`/`@fgr.com`) | Esperar uns minutos; conferir o e-mail da conta |
| `sem resposta do servidor` | O painel não alcançou `dados.hydra.fgr.com.br` | Conferir a rede/VPN da FGR |

Depois de resolver, clique em **Tentar de novo** (ou recarregue a página).
### Segurança, em uma frase
A tabela só aceita contas com e-mail `@fgr.com.br`/`@fgr.com` (regras no `01-schema.sql`). Qualquer pessoa da FGR que consiga entrar no Hydra conseguiria, tecnicamente, ler e gravar nessa tabela pela API, mesmo sem usar o painel. Se um dia for preciso restringir por obra ou por papel (como no app de metas, com planejador/coordenador/visualizador), o caminho é colocar um servidor no meio, o mesmo desenho do app de metas.

## 9. Duas obras (INFRA e CASAS) e acessos por obra

O painel tem as obras INFRA e CASAS, cada uma com os próprios dados. Quem pode ver ou editar cada uma é definido na tela **Cadastros**, no quadro "Quem pode acessar" (só administradora).

**Ordem para publicar (importante):**
1. No Studio (SQL Editor, New query), rode o arquivo `hydra/02-acessos.sql` inteiro. Confira antes o e-mail da administradora no fim do arquivo. Tudo o que já existe vira da obra INFRA; nada é apagado.
2. Espere uns 5 a 15 minutos para a API enxergar a tabela nova (`acessos`), como aconteceu com a `docs`.
3. Push origin e Redeploy no Coolify; Ctrl+F5 nas páginas abertas.

**Se algo falhar:** o aviso "modo local" com `HTTP 404` ou `HTTP 403` em acessos/docs, depois de 15 minutos, quer dizer que a API ainda não liberou a tabela nova. Aí vale chamar o TI.

**Níveis:** "Só visualiza" vê tudo da obra e não altera nada (o banco também recusa a gravação). "Pode editar" altera a obra. "Administradora" vê e altera as duas obras e mexe na tela Acessos. Quem entra com e-mail FGR e não está na lista vê "Sem acesso".

**Trocar o endereço do painel:** o login só volta para endereços que estão na lista de Redirect URLs do Hydra. Se o endereço mudar, alguém precisa incluir o novo na lista (o TI fez isso para o endereço atual).

**Observação:** o Hydra não deixa o usuário do Studio rodar comandos que apagam (DROP). Por isso o `02-acessos.sql` não usa nenhum: as regras por obra são somadas às que já existem. Se o Studio acusar `cannot run destructive DDL`, você está com uma versão antiga do arquivo: pegue a mais recente do repositório.
