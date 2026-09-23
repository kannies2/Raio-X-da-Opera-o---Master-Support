# Raio-X da Operação · Master Support

Página de portfólio e diagnóstico de maturidade operacional da Master Support.
Feita para ser acessada por QR code em eventos: o visitante conhece o portfólio,
responde seis perguntas, recebe um índice de 0 a 100, uma estimativa do custo da
indisponibilidade no ambiente dele e é levado ao próximo passo adequado ao que
respondeu.

Primeira campanha: **BMC Helix Roadshow · São Paulo**. O evento é um parâmetro
trocável, e a página serve para qualquer ação.

---

## Como publicar

É um site estático. Não tem build, não tem dependência de servidor.

### GitHub Pages

1. Suba estes arquivos na raiz do repositório (ou em `/docs`).
2. **Settings → Pages → Source:** `Deploy from a branch`, escolha a branch e a
   pasta correspondente.
3. A página fica em `https://<organização>.github.io/<repositório>/`.

O arquivo `.nojekyll` já está incluído: sem ele o GitHub Pages ignora pastas que
começam com underscore e pode interferir no processamento.

### Domínio próprio

Para servir em `diagnostico.mastersupport.com.br` ou similar, crie um arquivo
`CNAME` na raiz com o domínio, e aponte um registro DNS `CNAME` para
`<organização>.github.io`.

### Qualquer outra hospedagem

Copiar a pasta inteira para qualquer servidor de arquivos estáticos funciona:
Netlify, Vercel, S3, Nginx, IIS. Os caminhos são todos relativos.

---

## Configuração antes de ir ao ar

No `index.html`, procure por `var CFG = {` (perto do início do script principal).

```js
var CFG = {
  CAMPANHA: "BMC Helix Roadshow · São Paulo",
  SITE_CONTATO: "https://www.mastersupport.com.br/#contato",
  EMAIL: "comercial@mastersupport.com.br",
  TELEFONE: "(91) 3222-1678",
  AGENDA_MONITORAMENTO: "https://w.app/07eykk",
  AGENDA_ESPECIALISTA: "https://w.app/xgagoh",
  AGENDA_COMERCIAL: "https://w.app/6igqwk",
  HUBSPOT_PORTAL_ID: "49225425",
  HUBSPOT_FORM_GUID: ""
};
```

| Campo | O que acontece se ficar vazio |
|---|---|
| `CAMPANHA` | A tarja de evento no topo some. Troque a cada ação. |
| `AGENDA_*` | O botão daquele caminho cai para `SITE_CONTATO`. Cada um aponta para o WhatsApp de quem atende esse caminho. |
| **`HUBSPOT_FORM_GUID`** | **Os leads não chegam ao CRM.** Ficam apenas no `localStorage` do navegador do visitante e se perdem. |

### O GUID do HubSpot

É o item que falta. Sem ele nada do que o visitante preenche sai do aparelho dele.

Para obter: HubSpot → Marketing → Formulários → crie ou abra um formulário →
o GUID está na URL do editor, ou em *Compartilhar → Incorporar*, no campo
`formId`.

O formulário precisa aceitar estes campos:

`nome`, `email`, `empresa`, `cargo`, `telefone`, `origem`, `origem_detalhe`,
`origem_completa`, `campanha`, `sessao`, `status`, `etapa_alcancada`,
`total_etapas`, `percentual_concluido`, `pergunta_parada`, `pergunta_parada_id`,
`desafio`, `resolvedores`, `usuarios`, `deteccao`, `paradas`, `mttr`, `indice`,
`faixa`, `rota`, `rota_escolhida`, `horas_indisponiveis_ano`,
`custo_estimado_ano`.

---

## O que a página faz

**Portfólio** — serviços, parcerias auditadas (BMC Premier Partner, Zabbix
Premium Delivery Partner, serviços especializados em Oracle), o selo BMC Partner
of the Year 2026 em tamanho cheio ao clicar, ferramentas operadas em produção,
setores atendidos e números da operação.

**Diagnóstico** — identificação do lead com validação de domínio corporativo
(e-mails pessoais são recusados) e seis perguntas com leitura de especialista a
cada resposta. O visitante só avança depois de responder a pergunta que está na
tela, e pode voltar para trocar qualquer resposta. Se passar de 45 segundos
parado na mesma pergunta, aparece um aviso com o primeiro nome dele perguntando
se continua. Quem sai no meio e volta depois retoma exatamente na pergunta em que
parou, com as respostas preservadas (guardadas no navegador dele por sete dias).

**Resultado** — índice de 0 a 100 dividido em visibilidade, resiliência e
medição, cada um com a cor e o nome do seu nível, o elo mais fraco marcado e um
limitador: o índice não ultrapassa a faixa que esse elo permite. Em seguida, o custo da indisponibilidade em quatro
blocos (paradas × horas × custo por hora = custo no ano), a composição do valor
em uma barra, e dois blocos recolhidos que o visitante abre se quiser: a memória
de cálculo e as premissas ajustáveis.

**Caminhos** — três destinos em acordeão, na mesma página: avaliação de
monitoramento, sessão técnica com especialista e avaliação do resultado com o
head comercial. Abrir um recolhe o anterior. Nenhum passa de 30 minutos de
conversa, e cada botão leva ao WhatsApp de quem atende aquele caminho.

**PDF** — o visitante baixa o diagnóstico em um documento A4 de duas páginas
gerado no navegador, na identidade da marca, com o link de agendamento do
caminho recomendado.

**Rastreio de abandono** — quem desiste no meio é registrado com os dados de
contato, as respostas até ali e a pergunta exata em que parou
(`pergunta_parada`), para follow-up por e-mail.

---

## Estrutura

```
index.html                 a página inteira: marcação, estilo e script
vendor/jspdf.umd.min.js    gerador de PDF, carregado só quando alguém pede o PDF
assets/                    logos, marca, selo e logotipos de parceiros
.nojekyll                  desliga o processamento Jekyll do GitHub Pages
```

A fonte Inter está embutida no `index.html` como subconjunto woff2, então a
página não depende de CDN nem do Google Fonts e renderiza igual offline.

O `vendor/jspdf.umd.min.js` tem 410 KB e **não** entra na primeira carga: só é
buscado quando o visitante clica em baixar o PDF. Isso mantém a abertura da
página leve no celular, que é como ela será usada no evento.

---

## Manutenção

**Trocar o evento:** altere `CFG.CAMPANHA`.

**Trocar as perguntas:** o array `Q` define as seis perguntas, suas opções, os
pesos de cada resposta e a leitura do especialista que aparece após a escolha.

**Trocar os caminhos:** o objeto `ROUTES` define os três destinos (título,
chamada, três passos e texto do botão) e a função `route()` decide qual sai a
partir das respostas. O link de cada um vem de `CFG.AGENDA_*`.

**Trocar o cálculo de custo:** a função `cost()` e o objeto `costState` com os
valores iniciais das premissas.

**Trocar parceiros e ferramentas:** os arrays `PARTNERS` e `TOOLS`.

**Trocar o tempo do aviso de inatividade:** a constante `IDLE_MS`, em
milissegundos (hoje 45 s). A retomada fica em `salvarSessao()`, `lerSessao()` e
`retomar()`, sob a chave `ms_diag_sessao` do navegador.

**Trocar a escala de maturidade:** o array `NIVEIS` define a cor e o nome de cada
nível (inicial, em evolução, consistente, avançado) e alimenta as cápsulas do
pré-resultado, os cards de dimensão e o anel do resultado.

**Trocar as curvas entre as seções:** a função `edgePath()` desenha a mesma
forma em todas as passagens entre o branco e o navy.

---

Material interno da Master Support. Contém marcas de terceiros (BMC, Zabbix,
Oracle, Prometheus) usadas com base nas parcerias vigentes.
