# VN Controladoria — regras do projeto

Leia este arquivo antes de mexer em qualquer coisa. Ele reúne o que já
custou caro descobrir. Ignorar um item daqui costuma reintroduzir um bug
que já foi corrigido.

---

## O que é o sistema

Painel financeiro multiempresa para salões de beleza.

| | |
|---|---|
| Arquivo | `index.html` — **arquivo único**, ~1,8 MB, ~31 mil linhas |
| Estrutura | HTML + CSS + JS tudo junto. Sem build, sem npm, sem framework |
| Banco | Supabase (PostgreSQL) |
| Deploy | Vercel — `landing-page-vn.vercel.app` |
| Fonte | Outfit (Google Fonts) |
| Gráficos | Chart.js 4 via CDN |

Não existe etapa de compilação. O que está no arquivo é o que roda.

---

## Regras que não podem ser quebradas

### 1. Booleano do banco nunca se compara com `=== false`

O Supabase pode devolver `false`, `0`, `"false"`, `"f"` ou `null` para a
mesma coluna. `e.ativo !== false` deixa passar quase tudo.

```js
// ERRADO — já causou bug de empresa desativada reaparecendo
empresas.filter(e => e.ativo !== false)

// CERTO
empresas.filter(empresaEstaAtiva)
```

Funções existentes: `empresaEstaAtiva(e)`, `empresasAtivas()`,
`empresasAtivasIds()`.

`empresasVisiveis()` abre exceção para admin e **só** deve ser usada na
tela de gestão de empresas, onde é preciso ver as desativadas para
reativá-las.

### 2. Login não é o do Supabase

O sistema tem autenticação própria: tabela `vn_usuarios`, sessão em
`sessionStorage('vn_user_id')`, objeto global `_vnUser`.

```js
// ERRADO — derruba o estado de sessão do cliente Supabase
const { data } = await sb.auth.getUser();

// CERTO
const meuId = _vnUser?.id || sessionStorage.getItem('vn_user_id');
```

`sb.auth` só aparece em recuperação de senha e cadastro. Não use em
mais nada.

### 3. O corpo da página tem `zoom: 1.3`

Isso afeta cálculos de posição. O dropdown da Conciliação divide o
`scrollY` por esse fator. **Mudar o zoom quebra o dropdown** (já quebrou
quatro vezes). Se precisar mexer, ajuste o divisor junto.

Também afeta o `canvas`: os gráficos são desenhados com
`devicePixelRatio` multiplicado pelo zoom, senão ficam borrados.

### 4. `position: fixed` e `backdrop-filter`

O `#btn-chat` é filho da `.topbar`. Qualquer `filter`, `backdrop-filter`,
`transform` ou `will-change` num ancestral faz o `fixed` ancorar nesse
ancestral em vez da tela. O botão vai parar no canto errado.

### 5. Estilo inline vence CSS

Há mais de 3.700 atributos `style=` dentro das tags. Escrever uma regra
CSS para um elemento que tem estilo inline não funciona. Ou se converte o
elemento para classe, ou se edita o inline.

### 6. Verificar sintaxe não é verificar funcionamento

`node --check` pega erro de sintaxe, não pega:

- reatribuição de `const` (`TypeError` só em execução)
- campo que vem `undefined` do banco
- id de elemento que não existe no DOM

Depois de mexer, confira também com um parser de HTML se os `id` que o
JS usa continuam alcançáveis.

---

## Agenda / Google Calendar

- **O Google Agenda é a fonte única.** Nada de evento é gravado no
  Supabase. Evita conflito de versão entre dois lugares.
- Tipo, empresa e status ficam em `extendedProperties.private`
  (`vnTipo`, `vnEmpresa`, `vnStatus`, `vnOrigemId`).
- **A regra de repetição mora só no evento-pai.** Mandar `recurrence`
  numa ocorrência é ignorado em silêncio. Para alterar a série, use
  `PATCH` no `recurringEventId`.
- **`PATCH`, nunca `PUT`.** O `PUT` exige o evento completo e o Google
  recusa os campos só-leitura (`etag`, `kind`, `created`, `organizer`).
- Evento de dia inteiro: o `end` é **exclusivo** — dia 15 tem
  `start: 15`, `end: 16`.
- Sempre enviar `timeZone`, senão o Google grava em UTC.
- Token dura 1 hora, fica em `localStorage` e é renovado sozinho 5
  minutos antes de vencer. Só o botão Desconectar apaga a autorização.
- Só a credencial **ID do cliente** entra no código. A **chave secreta**
  jamais — ficaria visível no código-fonte da página.

---

## Padrões visuais

- **Paleta:** prata, grafite, branco e verde. Vermelho só para valor
  negativo, onde a cor carrega informação.
- **Cards de número** (`.hcard`, `.fcard`, `.metric-card`, `.mini`,
  `.kpi-hero`) usam malha metálica de 6 paradas de luz, uma cor por
  card, ciclando por posição (`nth-child(6n+N)`).
- **Ordem importa no CSS:** a cor padrão vem antes das variantes. Se
  inverter, o padrão engole todas as variantes.
- **Fileiras seguintes entram deslocadas** para não repetir a mesma cor
  empilhada.
- **Ícones:** SVG de linha, `viewBox="0 0 24 24"`, `stroke="currentColor"`.
  Nunca emoji em menu ou card — emoji não aceita tamanho nem cor e
  desalinha a coluna.
- **Gráficos:** barras com raio só no topo (objeto, não número — número
  arredonda os 4 cantos e vira cápsula). Rosca com `borderRadius: 5` e
  `spacing: 1`; valores maiores quebram fatias pequenas em pastilhas.
- **Hover:** quanto mais clicável, mais forte a reação. Linha de tabela
  destaca; botão sobe; card de número salta.

---

## Números financeiros — regra de ouro

**Card de banco = saldo ACUMULADO até o fim do período filtrado.**
Não é o movimento do período.

Isso já foi trocado uma vez por engano (passou a somar só o mês) e a
tela ficou dias mostrando CAIXA e STONE zerados e ITAÚ com o valor do
"Resultado". O card ao lado, "Saldo Final", continuava usando a base
acumulada — dois números da mesma tela seguindo regras diferentes.

Regra geral: **card de saldo usa `todos` até `ultimaDataFiltroGlobal`.
Card de movimento usa `lancs`.** Nunca misture.

### O conferidor

Existe uma verificação automática (`vnConferir`) que roda a cada
carregamento dos Lançamentos e checa:

- soma dos bancos = saldo final
- entradas − saídas = resultado
- contagem de linhas na tela = contagem calculada

Se algo não fechar, aparece uma tarja vermelha no topo com o valor da
diferença, e o erro vai para o console e para o Sentry.

**Ao criar tela nova com número financeiro, adicione as regras dela ao
conferidor.** É o que transforma um erro silencioso em erro visível.

Tolerância padrão: 2 centavos (arredondamento). Valor nulo ou inválido
é ignorado, para não gerar alarme falso.

---

## Desempenho

- `loadDashboard()` buscava `pdv_caixas` e `pdv_itens` **quatro vezes**.
  Existe um cache por período (`pdvPeriodo(ini, fim)`) — use ele.
- Lotes de `in()` devem ser disparados com `Promise.all`, não num laço
  com `await` (cada um esperando o anterior).
- Ainda há dois `select('*')` em `lancamentos` que poderiam listar só as
  colunas usadas.

---

## Como pedir alteração

Descrever o efeito, não a solução. "Os cards do dashboard estão
repetindo verde" leva a um diagnóstico melhor que "troca a cor do
segundo card".

Se mandar print, apontar o que está errado nele. Três tentativas de
adivinhar custam mais que uma frase de contexto.

---

## Pendências conhecidas

- [ ] Delta de variação (↑12,1% vs mês anterior) nos cards do dashboard
- [ ] ~890 emojis no resto do sistema (botões, títulos de seção)
- [ ] Importação de fatura de cartão em XLS com senha
- [ ] Expansão dos módulos RH e Precificação
- [ ] `select('*')` em `lancamentos` — listar só as colunas usadas
- [ ] Integrações: Open Finance, módulo fiscal
