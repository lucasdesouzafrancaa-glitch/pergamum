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

Os dados ficam no array `EMPRESAS`, no início do bloco `<script>` de `index.html`
(procure por `const EMPRESAS = [`). Cada empresa tem:

| Campo | O que é |
|---|---|
| `nome`, `curto`, `cnpj` | identificação exibida no cabeçalho e na barra lateral |
| `filiais` | alimenta o filtro de Filial |
| `encargos` | alíquotas patronais aplicadas sobre o salário (INSS, RAT + Terceiros, FGTS) |
| `beneficiosPct` | custo patronal de benefícios como fração do salário (hoje `0`) |
| `centros` | centro de custo com `cod`, `resp` e os departamentos que ele reúne |
| `colaboradores` | `nome`, `cargo`, `depto`, `filial`, `salario`, `admissao` |

Todo o resto do painel é calculado a partir daí: KPIs, participação por
departamento, encargos, apropriação e simulador. Não é preciso mexer em mais nada.

## Pontos de atenção sobre os dados

- Apenas **jul/2026** é competência real. Os outros meses dos gráficos são
  projeção sobre o quadro atual, pelos fatores `FATOR_QUADRO` e `EXTRA_13`.
- Colaboradores **demitidos** na competência não entram no quadro.
- **Benefícios** estão zerados: os relatórios de origem trazem só o desconto do
  empregado, não o custo patronal.
- Os **encargos** usam alíquotas legais sobre o salário contratual, então não
  reproduzem o valor das guias, que incidem sobre os proventos totais.

## Compatibilidade

Navegadores modernos (Chrome, Edge, Firefox, Safari), desktop e mobile.
Testado de 1920×1080 a 360×800, em retrato e paisagem.
