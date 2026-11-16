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

1. Abra o **GitHub Desktop** → **File → Add local repository** → escolha a pasta `CONTROLE-JARDINS-GRECIA`. (Já está com o primeiro commit feito, na branch `main`.)
2. **Publish repository** → nome sugerido: `controle-jardins-grecia`.
3. Se for usar **Public Repository** no Coolify (como no app de metas), **desmarque** "Keep this code private". O código ficará visível para qualquer pessoa, mas não tem nenhum dado real: o app abre vazio e os dados ficam no navegador de quem usa.

## 2. Criar a aplicação no Coolify

Precisa estar no escritório da FGR ou na VPN.

1. `https://hydra.fgr.com.br` → login Microsoft → escolha o **Team do seu departamento** (o mesmo do app de metas, se for o do seu departamento).
2. **Projects → +** → nome `controle-obras` (sugestão) → ambiente **production**.
3. **+ New Resource → Application → Public Repository** (ou o GitHub App do Team, se preferir deixar o repositório privado).
4. Repository URL: `https://github.com/annabeatrizcavfgr/controle-jardins-grecia` · Branch: **`main`**.
5. Server (rodar): **FGRLNX07** · Build server: **FGRLNX09**. Não troque.
6. **Build Pack: Dockerfile** (não Nixpacks).
7. **Ports Exposes: `80`** (não use 3000).
8. **Domains:** `https://controle-jardins-grecia.hydra.fgr.com.br` (sugestão). Se o nome já estiver em uso por outro app do Lab, escolha outro.
9. **Environment Variables:** nenhuma.
10. **Deploy** e acompanhe o *Deployment log* até ficar **Running** (verde).

## 3. Conferir que subiu

- `https://controle-jardins-grecia.hydra.fgr.com.br/healthz` → mostra `ok`
- `https://controle-jardins-grecia.hydra.fgr.com.br/` → abre o Dashboard vazio
- Cadastre um pedido e uma restrição de teste e confirme que aparecem no Dashboard.

## 4. Atualizar depois

Mudou o `public/index.html`? No GitHub Desktop: **Commit** → **Push origin**. No Coolify: **Redeploy** (ou deixe o deploy automático ligado, se o Team oferecer).

## 5. O que esta versão faz e o que não faz

- Os dados ficam no **navegador de cada pessoa** (`localStorage`), por endereço. Cada pessoa vê só a própria cópia; limpar o cache apaga tudo; nada é compartilhado entre coordenador, engenheiros e diretoria.
- Só abre na **rede FGR (escritório ou VPN)**. De casa, sem VPN, o endereço `*.hydra` não abre.
- Não tem login: quem alcança o endereço abre o app (hoje sem dados compartilhados).
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
