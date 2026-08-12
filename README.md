# Painel de Custo de Folha — Pergamum Contabilidade

Painel gerencial de custo de pessoal por empresa, departamento e centro de custo.
Arquivo único, sem build e sem dependências: todo o HTML, CSS e JavaScript estão
em `index.html`, com a logo embutida em base64.

Abre no **Início**, com a folha da empresa selecionada no filtro Empresa. A aba
**Folha consolidada**, logo abaixo, soma o custo de todas as empresas num só painel.

**Competência atual:** jul/2026
**Empresas:** Gomes Comércio e Consultoria · Next Consultoria e Serviços · Q.P.A Distribuição · B2B Comércio

## Publicar no Vercel

1. Crie um repositório no GitHub e envie estes arquivos na raiz (`index.html`, `vercel.json`, `README.md`, `.gitignore`).
2. Em vercel.com, clique em **Add New → Project** e importe o repositório.
3. Em Framework Preset escolha **Other**. Deixe Build Command e Output Directory em branco — é um site estático.
4. Clique em **Deploy**. O painel fica disponível em `https://<nome-do-projeto>.vercel.app`.

A partir daí, todo `git push` na branch principal publica automaticamente.

### Se preferir sem GitHub

```bash
npm i -g vercel
cd <pasta-do-projeto>
vercel        # pré-visualização
vercel --prod # produção
```

## Restringir o acesso

O painel tem dados de folha de pagamento e, publicado assim, fica acessível a
quem tiver o link. Antes de divulgar o endereço, avalie no Vercel:

- **Deployment Protection → Vercel Authentication:** só quem estiver logado na
  equipe do Vercel abre o painel.
- **Password Protection:** senha única para o site (disponível nos planos pagos).

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
