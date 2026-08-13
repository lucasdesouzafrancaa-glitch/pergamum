# Painel de Custo de Folha — Pergamum Contabilidade

Painel gerencial de custo de pessoal por empresa, centro de custo e filial.
Arquivo único, sem build e sem dependências: todo o HTML, CSS e JavaScript estão
em `index.html`, com a logo embutida em base64.

Abre no **Início**, com a folha da empresa selecionada no filtro Empresa. A aba
**Folha consolidada**, logo abaixo, soma o custo de todas as empresas num só painel.

**Competência atual:** jul/2026
**Empresas:** Gomes Comércio e Consultoria · Next Consultoria e Serviços · Q.P.A Distribuição · B2B Comércio

## Publicar no Vercel

1. Crie um repositório no GitHub e envie estes arquivos na raiz (`index.html`, `middleware.js`, `vercel.json`, `robots.txt`, `README.md`, `.gitignore`).
2. Em vercel.com, clique em **Add New → Project** e importe o repositório.
3. Em Framework Preset escolha **Other**. Deixe Build Command e Output Directory em branco — é um site estático.
4. Configure as variáveis de ambiente do login (seção abaixo) antes de publicar.
5. Clique em **Deploy**. O painel fica disponível em `https://<nome-do-projeto>.vercel.app`.

A partir daí, todo `git push` na branch principal publica automaticamente.

### Se preferir sem GitHub

```bash
npm i -g vercel
cd <pasta-do-projeto>
vercel        # pré-visualização
vercel --prod # produção
```

## Acesso protegido por login

O painel só é entregue depois do login. Quem chega sem sessão válida recebe apenas
a tela de login — o `index.html`, com todos os dados de folha, **não sai do servidor**.
Isso é feito pelo `middleware.js`, que roda na borda da Vercel antes de qualquer
arquivo ser servido.

### Configurar as credenciais

Em **Vercel → Settings → Environment Variables**, crie as três variáveis abaixo
(marque Production, Preview e Development) e faça um novo deploy:

| Variável | Conteúdo |
|---|---|
| `PAINEL_USUARIO` | usuário de acesso, ex.: `pergamum` |
| `PAINEL_SENHA` | senha longa — use um gerenciador de senhas |
| `SESSAO_SEGREDO` | texto aleatório longo, usado para assinar o cookie |

Para gerar o `SESSAO_SEGREDO`:

```bash
openssl rand -base64 48
```

Enquanto as três não estiverem definidas, o painel responde 503 e permanece fechado —
nunca fica aberto por engano.

### Como funciona a sessão

- O cookie de sessão é assinado (HMAC-SHA256), `HttpOnly`, `Secure` e `SameSite=Lax`.
- Dura 12 horas; depois disso é preciso entrar de novo.
- Cookie adulterado, assinatura inválida ou sessão expirada voltam para o login.
- O link **Sair**, no rodapé da barra lateral, encerra a sessão.

Para trocar a senha, basta editar `PAINEL_SENHA` e redeployar. Se quiser invalidar
todas as sessões abertas de uma vez, troque também o `SESSAO_SEGREDO`.

### Alternativa sem código

A Vercel também oferece **Deployment Protection** nas configurações do projeto:
*Vercel Authentication* (só quem está logado na sua equipe entra) ou
*Password Protection* (planos pagos). Pode ser usada junto com o login acima.

### O que este login não resolve

Depois de autenticado, o usuário recebe o HTML completo e pode ver os dados no
código-fonte da página — o que é esperado, já que ele precisa vê-los no painel.
A proteção impede o acesso de quem não tem a senha, não o vazamento por parte de
quem tem. Compartilhe as credenciais só com quem precisa e evite senha única
compartilhada por muita gente.

## Atualizar os dados na virada do mês

O painel começa na competência **jul/2026** e vai acumulando histórico a cada mês.
Tudo fica no bloco `<script>` de `index.html`.

### 1. Acrescente a nova competência

Logo no início do bloco de dados existem três listas que precisam ter sempre o
mesmo tamanho. Adicione o novo mês **no fim** das três:

```js
const MESES        = ['jul/26', 'ago/26'];
const FATOR_QUADRO = [ 1.00,     1.00   ];
const EXTRA_13     = [ 0,        0      ];
```

- `FATOR_QUADRO` — tamanho do quadro naquele mês em relação ao quadro cadastrado
  (`1.00` = igual). Se o quadro do mês anterior era menor, ajuste o valor daquela
  posição, ex.: `0.96`.
- `EXTRA_13` — parcelas do 13º pagas no mês, como fração da folha
  (`0` na maioria dos meses; algo como `0.45` e `0.62` em nov. e dez.).

### 2. Atualize o quadro de cada empresa

No array `EMPRESAS` (procure por `const EMPRESAS = [`), cada empresa tem:

| Campo | O que é |
|---|---|
| `nome`, `curto`, `cnpj` | identificação exibida no cabeçalho e na barra lateral |
| `filiais` | alimenta o filtro de Filial |
| `encargos` | alíquotas patronais sobre o salário (INSS, RAT + Terceiros, FGTS) |
| `beneficiosPct` | custo patronal de benefícios como fração do salário (hoje `0`) |
| `centros` | centro de custo com `cod`, `resp` e os departamentos que ele reúne |
| `colaboradores` | `nome`, `cargo`, `depto`, `filial`, `salario`, `admissao` |

O `colaboradores` deve refletir o quadro da competência mais recente: inclua
admissões, retire demitidos e atualize salários reajustados. Todo o resto do
painel — KPIs, participação por centro de custo, encargos, apropriação e a folha
consolidada — é calculado a partir daí.

### 3. Publique

`git commit` e `git push`: a Vercel republica sozinha.

## Pontos de atenção sobre os dados

- O histórico começa em **jul/2026**. Competências anteriores foram removidas de
  propósito: o painel passa a valer daqui para frente.
- Colaboradores **demitidos** na competência não entram no quadro.
- **Benefícios** estão zerados: os relatórios de origem trazem só o desconto do
  empregado, não o custo patronal.
- Os **encargos** usam alíquotas legais sobre o salário contratual, então não
  reproduzem o valor das guias, que incidem sobre os proventos totais.
- Enquanto houver uma única competência, os gráficos de evolução mostram um só
  ponto e o comparativo "vs. mês anterior" não aparece. Ambos se preenchem
  sozinhos a partir da segunda competência.

## Compatibilidade

Navegadores modernos (Chrome, Edge, Firefox, Safari), desktop e mobile.
Testado de 1920×1080 a 360×800, em retrato e paisagem.
