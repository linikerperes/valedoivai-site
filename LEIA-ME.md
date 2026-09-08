# Rumo ao Campori 2027 — Clube de Desbravadores Vale do Ivaí

Site da campanha de transporte do Clube Vale do Ivaí (Paranavaí — PR) para o
VI Campori Sul-Americano de Desbravadores, em Barretos (SP), em janeiro de 2027.

Site estático, sem dependências externas, sem framework e sem build.
São 3 arquivos + imagens.

---

## 1. Como atualizar a campanha (sem mexer em código)

Edite **somente** o arquivo `dados.json`. O site lê esse arquivo e recalcula
sozinho: porcentagem, quilômetros, posição do ônibus, marcos, mural,
transparência e os valores dos botões.

```json
{
  "arrecadado": 2450,
  "atualizadoEm": "2026-09-15",

  "pix": {
    "chave": "00000000000",
    "recebedor": "Nome exato de quem recebe",
    "instituicao": "Banco"
  },

  "whatsapp": "5544999999999",

  "totalApoiadores": 18,
  "apoiadores": [
    { "nome": "Família Silva", "km": 10 },
    { "nome": "Empresa XYZ",   "km": 50 },
    { "anonimo": true,          "km": 5 }
  ]
}
```

Regras dos campos:

| Campo | O que é |
|---|---|
| `arrecadado` | Total já recebido, em reais. É daqui que sai TUDO o resto. |
| `atualizadoEm` | Data no formato `AAAA-MM-DD`. Aparece na transparência. |
| `pix.chave` | Enquanto estiver vazio, o site mostra um aviso e **não inventa chave**. |
| `pix.tipo` | `telefone`, `cpf`, `cnpj`, `email` ou `aleatoria`. **Obrigatório acertar** — telefone e CPF têm 11 dígitos e o site não tem como adivinhar. |
| `pix.recebedor` | Nome completo, mostrado no site. |
| `pix.recebedorCurto` | Nome que vai **dentro** do código Pix. O padrão só aceita **25 caracteres** — nomes institucionais não cabem. Sem acento, em maiúsculas. |
| `pix.cidade` | Cidade do recebedor, sem acento, até 15 letras. |
| `whatsapp` | Só números, com 55 + DDD. Vazio = botão desativado. |
| `registroURL` | Opcional. URL da planilha que recebe os códigos gerados (seção 3.3). Vazio = o site não envia nada para lugar nenhum. |
| `totalApoiadores` | Número real de contribuições. Deixe `null` se ainda não quiser publicar. |
| `apoiadores` | Só entra quem autorizou. Use `"anonimo": true` para quem não quer o nome. |
| `despesas` | Prestação de contas. Cada saída do caixa vira uma linha no site. |

Dá para editar pelo celular, direto no painel do Cloudflare Pages ou do GitHub.

---

## 2. A matemática da campanha

- Meta: **R$ 15.500**
- Paranavaí → Barretos: **551 km** (aproximado, rodoviário)
- Ida e volta: **1.102 km**
- Valor por km: 15.500 ÷ 1.102 = **R$ 14,0653…**

Os botões são calculados pelo próprio site a partir da meta, então nunca ficam
dessincronizados:

| km | valor |
|---:|---:|
| 1 | R$ 14,07 |
| 5 | R$ 70,33 |
| 10 | R$ 140,65 |
| 25 | R$ 351,63 |
| 50 | R$ 703,27 |
| 100 | R$ 1.406,53 |

> **Atenção a um detalhe:** o briefing dizia "1 km ≈ R$ 14,06". O valor exato é
> R$ 14,0653, que arredondado para centavos dá **R$ 14,07**. Todos os outros
> valores do briefing (70,33 / 140,65 / 351,63 / 703,27 / 1.406,53) já vinham
> desse mesmo cálculo exato — ou seja, o "14,06" era um truncamento.
> O site usa **14,07** para não se contradizer.
> Se preferir manter 14,06 na comunicação, me avise que eu ajusto.

Para mudar a meta ou a distância, edite `CONFIG.meta` e `CONFIG.kmIda` no
`index.html`. Todo o resto se recalcula.

---

## 3. A estrada

A rota é um **circuito fechado**: a ida ocupa a faixa de cima, a volta a de baixo.

O ônibus é posicionado pela função `getPointAtLength()` do SVG e a direção dele
vem da **tangente da estrada** naquele ponto. Na prática isso significa que ele
é matematicamente incapaz de andar de ré ou de virar de cabeça para baixo, em
qualquer ponto da rota — foi verificado ponto a ponto ao longo dos 1.102 km.

Barretos cai exatamente em 50% do trajeto, porque o circuito é simétrico.

---

## 3.1. Escolher um trecho (cidades reais)

Duas formas de escolher, além dos botões de quantidade:

- **Por cidade** — "de Presidente Prudente até São José do Rio Preto"
- **Por quilômetro** — "do km 100 até o km 250"

O site calcula a distância e o valor sozinho. Se os extremos coincidirem com uma
cidade, o rótulo usa o nome dela em vez do número.

No topo da página há uma **estrada vertical interativa**: tocando em qualquer
cidade, ela mostra quanto falta para o ônibus chegar até lá, em km e em reais, e
leva esse trecho já preenchido para o formulário de doação.

### De onde vieram as cidades

Não foram inventadas. O trajeto foi calculado pelo roteador do OpenStreetMap
(OSRM) entre Paranavaí e o Parque do Peão, em Barretos. O resultado foi
**550,2 km** — o que confirma o número de 551 km usado pela campanha. Depois,
cada município foi localizado e conferido contra o traçado: **todos ficam a menos
de 6 km da estrada**, ou seja, a rota realmente passa por eles.

| km | cidade | | km | cidade |
|---:|---|---|---:|---|
| 0 | Paranavaí — PR | | 269 | Parapuã — SP |
| 18 | Alto Paraná — PR | | 280 | Rinópolis — SP |
| 58 | Cruzeiro do Sul — PR | | 356 | Penápolis — SP |
| 83 | Colorado — PR | | 415 | José Bonifácio — SP |
| 110 | Santo Inácio — PR | | 461 | São José do Rio Preto — SP |
| 188 | Presidente Prudente — SP | | 507 | Olímpia — SP |
| 198 | Regente Feijó — SP | | 551 | Barretos — SP |
| 213 | Martinópolis — SP | | | |

Para mexer nessa lista, edite `CIDADES` no `index.html`. As marcadas com
`hero: true` são as 10 que aparecem na estrada do topo — as outras ficariam
sobrepostas por estarem muito próximas.

> **Um limite a saber:** escolher um trecho é simbólico. Como não existe banco de
> dados ainda, duas pessoas podem escolher o mesmo trecho, e o site não "reserva"
> nada. Os textos foram escritos com esse cuidado: falam em *levar o ônibus por
> aquele trecho*, nunca em *o trecho é seu*. Se um dia quiser reserva de verdade,
> é preciso o painel administrativo com banco de dados (seção 6).

---

## 3.2. Pix Copia e Cola com o valor já preenchido

Quando a pessoa escolhe um trecho, o site **gera na hora um código Pix já com o
valor**. Ela copia, cola no app do banco em *Pix › Pix Copia e Cola*, e não
precisa digitar centavo nenhum. Isso reduz muito o erro de valor.

O código é o padrão do Banco Central (EMV MPM, o mesmo formato do QR Code Pix):
campos `<id><tamanho><valor>` encadeados, fechando com um CRC16-CCITT.

Como foi conferido:
- o CRC bate com o vetor de teste oficial — `CRC16("123456789") = 29B1`;
- o código gerado foi relido campo a campo por um verificador independente:
  chave `iasd.paranavai@anp.org.br`, moeda `986` (BRL), país `BR`, valor no campo
  `54`, e o CRC recalculado bate;
- trocando o trecho, o valor e o CRC mudam junto;
- e, acima de tudo, **um Pix real foi pago com sucesso** em 07/09/2026 (seção 3.3).

**Conta recebedora:** conta oficial da Igreja Adventista do Sétimo Dia, chave
`iasd.paranavai@anp.org.br`, em nome da **União Sul Brasileira da Igreja
Adventista do Sétimo Dia**.

O nome completo tem 55 caracteres e o campo do padrão Pix aceita **25**. Cortar
no meio da palavra deixaria `UNIAO SUL BRASILEIRA DA I` dentro do código. Por
isso existem dois campos:

- `recebedor` → `União Sul Brasileira da Igreja Adventista do Sétimo Dia` (aparece no site)
- `recebedorCurto` → `UNIAO SUL BRASILEIRA` (vai dentro do código Pix, 20 de 25)

Isso não muda o destino do dinheiro: quem manda é a **chave**. O nome no campo 59
é informativo, e o banco de quem paga mostra o titular real da conta.

> **Só o Copia e Cola.** A opção de copiar a chave avulsa e o campo de digitar um
> valor livre foram removidos a pedido: numa conta que recebe muitos Pix, valor
> arbitrário dificulta a conferência. Todo valor agora vem de um botão de km ou de
> um trecho entre cidades.

Ainda **não há QR Code** — só o Copia e Cola. Gerar QR exigiria uma biblioteca a
mais; se quiser, dá para acrescentar.

---

## 3.3. Conferir os Pix recebidos (código por doação)

Toda doação recebe um **código único**, no formato `VALEXXXXXX` — por exemplo
`VALEJQ6QDW`. Ele:

1. vai **dentro do código Pix**, no campo `txid` do padrão do Banco Central;
2. aparece **na tela**, em destaque, antes de a pessoa copiar;
3. entra **automaticamente na mensagem do WhatsApp** junto com o comprovante.

Assim, mesmo numa conta que recebe muitos outros Pix, dá para separar o que é da
campanha.

O alfabeto do código não tem `O`, `0`, `I` nem `1`, para ninguém errar ao ditar
ou digitar por telefone.

### Como conferir na prática

O caminho **garantido** é o WhatsApp: cada pessoa manda o comprovante e a
mensagem já vem com `🔖 Código: VALEJQ6QDW`, o valor e os quilômetros. Você casa
com o extrato pelo valor e pelo horário.

**Confirmado em teste real (07/09/2026).** Um Pix de R$ 14,07 gerado pelo site foi
pago e o comprovante do Bradesco trouxe:

```
Valor:         R$ 14,07
Identificador: VALEKX7WLW
Nome:          UNIAO SUL BRASILEIRA DA IGREJA ADVENTISTA
CNPJ:          79.080.602/0014-70
```

Três coisas ficaram provadas de uma vez:

1. **O código aparece como "Identificador"** no comprovante — é exatamente o
   mecanismo de conferência que a campanha precisa.
2. **O valor saiu certo**, R$ 14,07 = 1 km, sem ninguém digitar nada.
3. **O banco mostrou o nome completo** da conta, e não o `UNIAO SUL BRASILEIRA`
   abreviado que vai dentro do código. Confirma o que estava previsto: o campo 59
   é informativo, quem determina o destino é a chave.

> **O que ainda não foi verificado:** esse comprovante é do lado de **quem pagou**.
> Se o identificador também aparece no **extrato de quem recebe** (a conta da
> igreja), só dá para confirmar com acesso a esse extrato. Vale conferir com a
> tesouraria. Na prática isso pouco muda, porque o doador manda esse mesmo
> comprovante pelo WhatsApp — e ele já vem com o código.

> **Limite honesto:** o site é estático e não tem servidor. Os códigos nascem no
> celular de cada visitante, então **o site sozinho não consegue guardar uma lista
> central** do que foi gerado. Se você quiser essa lista automática, veja abaixo.

### Relatório automático em planilha (opcional, grátis)

Dá para fazer cada código gerado cair sozinho numa planilha do Google.
**Custo: R$ 0.** Leva uns 10 minutos para configurar.

1. Crie uma planilha nova no Google Sheets.
2. Menu **Extensões › Apps Script**. Apague o que estiver lá e cole:

```javascript
function doPost(e) {
  var ss  = SpreadsheetApp.getActiveSpreadsheet();
  var aba = ss.getSheetByName('Codigos') || ss.insertSheet('Codigos');
  if (aba.getLastRow() === 0) {
    aba.appendRow(['Quando', 'Codigo', 'Evento', 'Km', 'Valor', 'Trecho', 'Nome']);
  }
  var d = JSON.parse(e.postData.contents);
  aba.appendRow([new Date(), d.codigo, d.evento, d.km, d.valor, d.trecho, d.nome]);
  return ContentService.createTextOutput('ok');
}
```

3. **Implantar › Nova implantação › App da Web**
   · Executar como: **Eu**
   · Quem pode acessar: **Qualquer pessoa**
4. Copie a URL que aparece e cole no `dados.json`:

```json
"registroURL": "https://script.google.com/macros/s/AKfy.../exec"
```

Pronto. Cada vez que alguém copiar o código Pix ou abrir o WhatsApp, entra uma
linha na planilha com data, código, valor, trecho e nome.

**O que esperar dessa lista:** ela registra códigos *gerados*, não pagamentos
confirmados. Muita gente vai copiar e não pagar — é normal. A planilha serve para
você procurar um código que chegou, não para contar arrecadação. O total oficial
continua sendo o que você coloca em `arrecadado`.

Enquanto `registroURL` estiver vazio, **o site não envia nada para lugar nenhum**.

### A página de conferência (`relatorio.html`)

A planilha te dá a lista crua. Para **controlar a entrada do dinheiro** na conta
da igreja, o projeto inclui uma página só da diretoria:

**`valedoivai.org.br/relatorio.html`**

Ela não é linkada em lugar nenhum do site — só quem tem o endereço abre. O que dá
para fazer nela:

1. **Trazer os códigos** — escolher o CSV baixado da planilha (*Arquivo › Fazer
   download › CSV*) ou colar as linhas direto. Ela entende acento, aspas e valor
   tanto `140.65` quanto `R$ 1.406,53`.
2. **Lançar à mão** — quando alguém manda o comprovante pelo WhatsApp e o código
   não está na planilha.
3. **Conferir com o extrato** — chegou um Pix na conta? Busca o código (ou o nome,
   ou o valor) e marca. O painel mostra: quantos códigos, quantos conferidos,
   **quanto já entrou de verdade** e quanto falta conferir.
4. **Baixar CSV** com tudo, inclusive a coluna `Conferido`.

Ela ignora repetição: a mesma pessoa aparece na planilha duas vezes (uma quando
copia o código, outra quando abre o WhatsApp). Se o código **e** o valor forem
iguais, entra uma linha só. Se a pessoa mudou de valor no meio, as duas entram —
porque aí você precisa saber qual valor caiu.

> **Onde os dados ficam:** no navegador do aparelho onde você abrir, e só ali.
> Não sobe para servidor nenhum. Se abrir em outro celular, a lista começa vazia —
> use *Baixar CSV* para passar de um aparelho para outro. Isso é de propósito:
> evita expor nome de doador em página pública.

E o total que vale para o site continua sendo o `arrecadado` do `dados.json` —
a conferência é o seu controle interno, não muda o número que aparece na campanha.

---

## 3.4. Prestação de contas

A seção de transparência mostra **para onde o dinheiro foi**, não só quanto entrou.
Basta lançar cada saída no `dados.json`:

```json
"despesas": [
  { "descricao": "Sinal do ônibus (30%)",  "valor": 4650, "data": "2026-10-15" },
  { "descricao": "Seguro viagem do grupo", "valor": 820,  "data": "2026-11-02" }
]
```

O site calcula sozinho e publica:

```
Total recebido    R$ 8.200
Total gasto       R$ 5.470
Saldo em caixa    R$ 2.730
```

Enquanto a lista estiver vazia, aparece um aviso dizendo que nenhuma despesa foi
lançada ainda — em vez de um espaço em branco.

## 3.5. Por que a chave Pix não aparece na tela

A chave **não é mostrada em lugar nenhum** do site. A única forma de doar é pelo
botão *Copiar código*, que já leva o valor e o código da doação embutidos.

O motivo é de controle: com a chave à vista, qualquer pessoa mandaria um valor
arbitrário, sem código nenhum — e esse Pix cairia na conta da igreja, no meio de
muitos outros, sem nada que ligasse ele à campanha. Aí a conferência vira garimpo.

> **Até onde isso protege:** a chave continua existindo **dentro** do código Copia
> e Cola, porque é assim que o Pix funciona — não há como gerar um código válido
> sem ela. Alguém que cole o código num editor de texto consegue lê-la. O que
> mudou é que ninguém faz isso **sem querer**: o caminho fácil e óbvio passou a ser
> o fluxo com valor e código certos.

---

## 4. O que ainda falta antes de publicar

- [x] ~~**Chave PIX**~~ → `iasd.paranavai@anp.org.br` (e-mail, conta da igreja)
- [x] ~~**Número do WhatsApp**~~ → `44984033799` (Guilherme Mestriner, capelão)
- [x] ~~**Titular da conta**~~ → União Sul Brasileira da IASD
- [x] ~~**Contatos da diretoria**~~ → seção no site, com WhatsApp do diretor e do capelão
- [ ] **Fazer um Pix de teste de R$ 1,00** com o Copia e Cola antes de divulgar,
      e confirmar no extrato se o banco mostra o código `VALE...` no detalhe da transação
- [ ] **Depoimentos reais.** Os textos estão escritos na voz de cada Desbravador,
      mas **ainda não são as palavras deles**. Grave cada um respondendo
      *"Por que você quer ir ao Campori?"* e troque no `index.html`.
      Enquanto isso, há uma linha discreta no fim da seção avisando que os textos
      são preliminares — quando as falas reais entrarem, pode apagar essa linha.
      Considere autorização dos responsáveis antes de publicar nome/imagem de menores.
- [ ] **Fotos.** As três fotos em `assets/fotos/` são de edições anteriores,
      publicadas originalmente por Notícias Adventistas, Portal Bueno e Guaíra News,
      e estão creditadas no site. **Confirme a autorização de uso ou troque por
      fotos oficiais do Campori e do próprio clube antes de publicar.**
      Para trocar: substitua os arquivos mantendo os mesmos nomes.
- [ ] **Vídeo do Vale do Ivaí** (45–90s). Há um bloco reservado na seção de vídeos.
- [ ] Conferir os números do evento em <https://camporidsa.org/pt/>
      (hoje o site diz "+120 mil nas duas edições" e "8 países", conforme o site oficial).

---

## 4.1. Peso da página (otimização para celular)

O site é feito para abrir rápido em 4G, que é como a maioria vai receber o link
pelo WhatsApp.

| Item | Antes | Depois |
|---|---:|---:|
| Logo | 412 KB (PNG 554px) | **48 KB** (JPEG 320px) |
| Foto 1 | 239 KB | **72 KB** |
| Foto 2 | 217 KB | **124 KB** |
| Foto 3 | 161 KB | **148 KB** |
| **Imagens da página** | **1.029 KB** | **392 KB** |

O que foi feito:

- **Logo em JPEG a 320px.** O maior uso real dele é 230px, dentro do certificado.
  Não precisa de transparência porque é recortado em círculo pelo CSS.
- **Fotos a 820px, qualidade 55.** Os cards têm no máximo 420px, então 820px
  ainda cobre telas de alta densidade.
- **Vídeos só carregam no clique.** Antes, o player do YouTube era baixado assim
  que o vídeo entrava na tela, mesmo sem ninguém assistir. Agora aparece só a
  miniatura (16–39 KB) e o player entra quando a pessoa clica em ▶.
- Sem fontes externas, sem bibliotecas, sem rastreadores. O HTML é um arquivo só.

> **Ao trocar as fotos:** mantenha os mesmos nomes de arquivo e deixe cada uma com
> no máximo ~820px de largura e uns 150 KB. Foto direto do celular tem 3–5 MB e
> derruba o carregamento no 4G.

---

## 5. Publicação

**No ar em:** <https://valedoivai-site.linikerperes27.workers.dev> (Cloudflare)
**Domínio a ligar:** `valedoivai.site` (Hostinger, renova 07/09/2027)
**Repositório:** <https://github.com/linikerperes/valedoivai-site>

Testado em 07/09/2026: página carrega em 0,12 s, todos os arquivos respondem,
nenhum erro de console. O `_headers` **funciona** no Cloudflare — confirmado que o
`dados.json` sai com `max-age=60` e o `relatorio` com `noindex`.

O endereço da página de conferência ficou **`/relatorio`** (sem `.html`) — o
Cloudflare remove a extensão sozinho.

### Falta ligar o domínio

Enquanto isso não é feito, existe um efeito colateral concreto: as tags
`og:image` e `og:url` apontam para `valedoivai.site`, que hoje é a página de
parking do Hostinger. Ela responde `200` para qualquer endereço, mas devolve
**HTML no lugar da imagem** — ou seja, **o card de preview do WhatsApp fica
quebrado** até o domínio apontar para o Cloudflare.

**1. Adicionar o domínio à sua conta Cloudflare**

No dashboard → **Add a domain** → `valedoivai.site` → plano **Free**.
A Cloudflare vai varrer o DNS atual e mostrar **dois nameservers**
(algo como `xxx.ns.cloudflare.com`).

**2. Trocar os nameservers no Hostinger**

**hPanel** → **Domínios** → **Gerenciar** em `valedoivai.site` → **DNS / Nameservers**
→ trocar de "Nameservers do Hostinger" para **Personalizados** e colar os dois da
Cloudflare.

Hoje estão em `orbit.dns-parking.com` e `horizon.dns-parking.com` — são esses que saem.
Aqui é o campo **Nameservers**, não a tabela de registros DNS.

**3. Apontar o domínio para o site**

Quando a Cloudflare marcar o domínio como *Active*: abra o projeto
**valedoivai-site** → **Settings** → **Domains & Routes** → **Add** → **Custom domain**
→ `valedoivai.site`. Repita para `www.valedoivai.site`.

O certificado HTTPS é emitido automaticamente.

**4. Conferir depois**

- `https://valedoivai.site` abre o site
- `https://valedoivai.site/assets/og-campanha.jpg` devolve **uma imagem** (hoje devolve HTML)
- mandar o link no WhatsApp e ver o card de preview aparecer
- `https://valedoivai.site/relatorio` abre a página da diretoria

### Como atualizar a arrecadação

Depende de como o deploy foi feito no Cloudflare:

- **Se o projeto está conectado ao GitHub:** edite `dados.json` pelo site do GitHub
  (lápis → muda `arrecadado` e `atualizadoEm` → Commit) e o Cloudflare republica
  sozinho em ~1 minuto. Funciona pelo celular.
- **Se foi upload direto:** é preciso subir o arquivo de novo pelo painel do
  Cloudflare a cada atualização.

Vale confirmar qual dos dois é, porque o primeiro é bem mais prático no dia a dia.

O número novo aparece rápido: o site busca o `dados.json` com `cache: "no-store"`
e o `_headers` limita o cache a 60 segundos.

### GitHub Pages

Foi ativado antes, como alternativa, e **já está desligado**. O arquivo `CNAME`
foi removido do repositório para não disputar o domínio com o Cloudflare.

---

## 6. Painel administrativo — próximo passo

Hoje a atualização é feita editando `dados.json`. Isso é suficiente para começar,
custa R$ 0 e não tem risco de segurança.

Quando quiser um `/admin` de verdade (login, cadastrar doação, aprovar, mural
automático), as opções realistas são:

| Opção | Custo | Prós | Contras |
|---|---|---|---|
| Continuar no `dados.json` | R$ 0 | simples, sem login, sem risco | edição manual |
| Cloudflare Pages + D1 + Access | R$ 0 no início | mesmo provedor, login pronto | exige desenvolver o painel |
| Supabase | gratuito até certo limite | banco + autenticação prontos | mais um serviço para manter |

Sobre pagamento automático: para o PIX cair sozinho no site seria preciso um
gateway (Mercado Pago, Asaas, PagBank), que cobra taxa por transação e exige
CNPJ/conta PJ na maioria dos casos. Para uma campanha de clube, **PIX manual +
comprovante no WhatsApp custa zero e é mais simples de prestar contas.**

Em qualquer cenário: **nunca coloque senha, token ou chave de API no HTML público.**

---

## 7. Arquivos

```
index.html                     o site inteiro (HTML + CSS + JS)
relatorio.html                 página de conferência (só para a diretoria)
                               — inclui a lista CIDADES do trajeto real
dados.json                     ÚNICO arquivo que você precisa editar
LEIA-ME.md                     este arquivo
assets/
  logo-vale-do-ivai.png        logo oficial (usado no site e no certificado)
  logo-sm.png                  favicon
  og-campanha.jpg              imagem do preview no WhatsApp
  fotos/                       fotos do Campori (verificar autorização)
```

---

## 8. Acessibilidade e performance

- Mobile-first, testado em 390px e 430px, sem rolagem horizontal
- Nenhum botão ou campo abaixo de 44px de altura
- Respeita `prefers-reduced-motion`
- Sem fontes externas, sem bibliotecas, sem rastreadores
- Imagens com `loading="lazy"` e `alt`
- Se o JavaScript falhar, o conteúdo continua visível
