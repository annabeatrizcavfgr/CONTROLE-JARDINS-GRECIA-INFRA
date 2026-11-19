# Controle Jardins Grécia

Painel de controle da obra, da FGR: Dashboard, Matriz GUT, controle de pedidos, flows (aditivos do FlowMaster), contratações e aditivos, e validação de projetos.

É um arquivo único (`public/index.html`), sem build e sem dependências.

- **Publicar no Hydra:** veja [hydra/passo-a-passo.md](hydra/passo-a-passo.md).
- **Testar localmente:** abra `public/index.html` no navegador.
- **Login:** conta FGR (Microsoft/Entra) pelo Auth do Hydra, só no endereço `*.hydra.fgr.com.br`. Passos em [hydra/passo-a-passo.md](hydra/passo-a-passo.md), seção 7.
- **Dados:** ficam no navegador de cada pessoa (`localStorage`). Ainda não são compartilhados; o próximo passo é o banco no Hydra.
- **Cadastros:** obras e tipos de pedido se editam na tela **Cadastros**. O nome de quem usa é pedido no primeiro login.
