# Como trabalhar com Issues e Preview

Duas mudanças de rotina que resolvem os dois problemas que já
apareceram: perder pendência de vista, e publicar algo quebrado.

---

## Parte 1 — Issues: parar de perder pendência

Hoje as pendências ficam no meio da conversa e somem. Uma Issue é um
bilhete que não some.

### Criar

No GitHub, aba **Issues** → **New issue**. Escreva assim:

```
Título: Cards do dashboard sem o delta de variação

O que acontece: os cards mostram só o valor do mês.
O que deveria: mostrar ↑12,1% comparado ao mês anterior.
Onde: Dashboard, 4 cards do topo.
Precisa de: consulta do mês anterior dentro de loadDashboard().
```

Três linhas bastam. O objetivo é você entender daqui a duas semanas.

### Etiquetas

Crie estas quatro em **Issues → Labels**:

| Etiqueta | Quando usar |
|---|---|
| `correção` | Está quebrado |
| `melhoria` | Funciona, mas pode ficar melhor |
| `nova função` | Não existe ainda |
| `urgente` | Trava o uso do sistema |

### Pendências de hoje

Vale abrir uma Issue para cada item da lista de pendências do
`PROJETO.md`. São seis.

---

## Parte 2 — Preview: testar antes de publicar

Já aconteceu duas vezes de algo quebrado chegar em você. Com preview,
isso para.

### Como funciona

A Vercel gera uma URL de teste para cada branch, separada da URL de
produção. Você abre, testa, e só depois manda pro ar.

### Passo a passo

**1. Criar a branch.** No GitHub, no seletor que mostra `main`, digite
um nome novo e clique em **Create branch**. Exemplo: `delta-dashboard`.

**2. Subir o arquivo nela.** Com a branch selecionada, faça o upload do
`index.html` normalmente.

**3. Pegar o link de teste.** A Vercel cria a URL sozinha em cerca de um
minuto. Está no painel da Vercel, em **Deployments**, ou no comentário
automático dentro do Pull Request.

**4. Testar.** Login, dashboard, lançamentos, agenda, e o que você
alterou.

**5. Publicar.** Se estiver bom: **Pull Request** → **Merge**. A Vercel
publica em produção sozinha.

Se estiver ruim, apaga a branch. Produção nunca foi tocada.

### Ligando a Issue ao PR

Na descrição do Pull Request, escreva:

```
Resolve #12
```

O GitHub fecha a Issue 12 automaticamente quando o PR for publicado.
Use `Resolve #número`, com o número que aparece na Issue.

---

## Roteiro de teste antes de publicar

Cinco minutos. Faça sempre, mesmo em mudança pequena — as duas quebras
que aconteceram vieram de alterações "inofensivas".

- [ ] Entrar no sistema (tela de login carrega e aceita)
- [ ] Dashboard abre com os números preenchidos
- [ ] Lançamentos: tabela carrega, filtro de mês funciona
- [ ] Trocar de empresa na barra superior
- [ ] Abrir um relatório (Produtos ou Serviços)
- [ ] Controladoria: abrir o painel, ver as tarefas
- [ ] Agenda: abrir, conectar, criar e apagar um evento de teste
- [ ] Console do navegador (F12) sem erro em vermelho

O último item é o que mais pega problema. Erro no console quase sempre
significa que alguma parte da página não carregou.

---

## Quando algo quebrar em produção

1. Vercel → **Deployments** → localize a versão anterior que funcionava
2. Menu `...` → **Promote to Production**

Volta ao ar em segundos. Depois se investiga com calma.
