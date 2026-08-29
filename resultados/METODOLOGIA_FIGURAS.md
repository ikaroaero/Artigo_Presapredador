# Metodologia das figuras

Este documento descreve como foram produzidas as quatro figuras principais do
projeto. Os dados estão organizados por taxa de mutação da presa (`W2` a `W5`),
rugosidade do ambiente (`H`) e probabilidade de reprodução do predador
(`rP`). Nas figuras, a notação das pastas `W` é apresentada como taxa de
mutação:

| Pasta | Notação nas figuras |
|---|---|
| `W2` | $\nu=10^{-2}$ |
| `W3` | $\nu=10^{-3}$ |
| `W4` | $\nu=10^{-4}$ |
| `W5` | $\nu=10^{-5}$ |

Cada configuração possui cinco repetições independentes, chamadas de ciclos.
O tempo é registrado de 500 em 500 passos até, idealmente, 500000. As figuras
foram salvas em PNG com 300 dpi e sem título principal, para facilitar seu uso
em artigos e apresentações.

## 1. Diagrama de fases ecológicas

**Arquivo principal:**
`diagramas_fase_rp_2_casas/diagrama_fases_unificado.png`

**Dados de entrada:** arquivos
`Presa_predador_ciclo-*_com_predação_*.csv`. A primeira coluna contém o tempo,
a segunda a população de predadores e a terceira a população de presas.

### Estado final de cada ciclo

Para reduzir a influência de uma flutuação isolada no último instante, são
selecionados os dez últimos registros de cada ciclo. Calculam-se separadamente
as médias finais das populações de presas e predadores:

$$
\bar N_{\mathrm{presa}}=\frac{1}{10}\sum_{t=T-9}^{T}N_{\mathrm{presa}}(t),
\qquad
\bar N_{\mathrm{predador}}=\frac{1}{10}\sum_{t=T-9}^{T}N_{\mathrm{predador}}(t).
$$

Quando as duas populações estão presentes, calcula-se a fração de presas:

$$
f_{\mathrm{presa}}=
\frac{\bar N_{\mathrm{presa}}}
{\bar N_{\mathrm{presa}}+\bar N_{\mathrm{predador}}}.
$$

O limiar de presença é zero. Cada ciclo é classificado pelas seguintes regras:

1. **Prey dominance:** presas presentes e predadores ausentes.
2. **Partial prey dominance:** ambas as populações presentes e
   $f_{\mathrm{presa}}>0{,}55$.
3. **Equal coexistence:** ambas presentes e
   $0{,}40\leq f_{\mathrm{presa}}\leq0{,}55$.
4. **Partial predator dominance:** predadores presentes e
   $f_{\mathrm{presa}}<0{,}40$, incluindo o caso de ausência das presas.
5. **Mutual extinction:** presas e predadores ausentes.

### Combinação dos cinco ciclos

Primeiro, os cinco ciclos são classificados individualmente. Se uma mesma
configuração contém simultaneamente ciclos em **Partial predator dominance** e
ciclos em **Mutual extinction**, o resultado final é **Indeterminate**. Essa é
a única condição especial de indeterminação.

Nos demais casos, prevalece o estado com maior número de ciclos. Se duas ou
mais classes empatarem, as populações finais são médias entre os cinco ciclos e
essa média conjunta é classificada pelas mesmas regras. Se o resultado médio
não pertencer às classes empatadas, aplica-se uma ordem determinística apenas
para garantir reprodutibilidade computacional.

### Filtro de $r_P$

A versão principal conserva somente valores com até duas casas decimais e
remove explicitamente:

`0.14`, `0.16`, `0.17`, `0.46`, `0.47`, `0.60` e `0.70`.

Assim, o eixo contém:

`0.01`, `0.03`, `0.04`, `0.05`, `0.06`, `0.07`, `0.08`, `0.10`, `0.11`,
`0.12`, `0.18`, `0.20`, `0.22`, `0.25`, `0.30`, `0.35`, `0.38`, `0.40`,
`0.41`, `0.42`, `0.43`, `0.44`, `0.45`, `0.48`, `0.49`, `0.50`, `0.55`,
`0.65` e `0.75`.

### Organização visual

Os três painéis empilhados representam `H=0.01`, `0.50` e `0.99`. As linhas
de cada painel representam $\nu=10^{-5},10^{-4},10^{-3},10^{-2}$ e as colunas
representam $r_P$. A cor de cada célula é o estado final da configuração. Uma
única legenda em inglês é compartilhada pelos painéis.

O arquivo `classificacao_ciclo_a_ciclo.csv` conserva as médias finais e as
classes de cada ciclo, permitindo auditar cada célula do diagrama.

## 2. Espécies × tempo — visão global

**Arquivo principal:**
`especiesXtempo_global/especies_tempo_global_WxH.png`

**Dados de entrada:** arquivos `N_Presa_ciclo-*.csv`. A primeira coluna contém
o tempo e a segunda o número de espécies de presas, $S_i(t)$, no ciclo $i$.

Para cada configuração $(\nu,H,r_P)$, a curva apresentada é a média ponto a
ponto dos cinco ciclos:

$$
\langle S(t)\rangle=\frac{1}{5}\sum_{i=1}^{5}S_i(t).
$$

Não se calcula uma média entre diferentes valores de $r_P$: cada $r_P$ gera
uma curva própria. Também não são usados apenas os instantes finais; toda a
série temporal é representada. Os tempos dos ciclos precisam coincidir
exatamente, caso contrário o script interrompe a configuração para evitar uma
média temporal incorreta.

São utilizados os mesmos valores de $r_P$ e as mesmas exclusões do diagrama de
fases, tornando as duas figuras diretamente comparáveis.

### Organização visual e cores

As colunas representam $\nu=10^{-2},10^{-3},10^{-4},10^{-5}$ e as linhas
representam `H=0.01`, `0.50` e `0.99`. Cada painel possui eixo Y próprio, pois a
riqueza pode diferir por ordens de grandeza entre taxas de mutação.

As curvas usam um gradiente azul quase preto–quase branco. A associação entre
cor e $r_P$ não é linear: utiliza-se uma normalização por potência
`PowerNorm(gamma=0.45)`. Essa transformação aumenta o contraste visual entre
os muitos valores concentrados na região baixa e intermediária de $r_P$. A
barra de cores continua rotulada nos valores físicos de $r_P$; a transformação
afeta somente a distribuição das cores.

O eixo temporal é exibido em notação científica e o eixo vertical usa
$\langle S(i)\rangle$.

## 3. Populações médias × $r_P$

**Arquivo principal:** `popXrp/populacao_x_rP_WxH.png`

**Dados de entrada:** arquivos
`Presa_predador_ciclo-*_com_predação_*.csv`.

Para cada ciclo, calcula-se a média das populações nos dez últimos registros.
Depois, para cada configuração $(\nu,H,r_P)$, calcula-se a média entre ciclos e
o desvio-padrão amostral:

$$
\langle N\rangle=\frac{1}{n}\sum_{i=1}^{n}\bar N_i,
\qquad
s=\sqrt{\frac{1}{n-1}\sum_{i=1}^{n}(\bar N_i-\langle N\rangle)^2}.
$$

Com cinco arquivos completos, $n=5$. As barras de erro mostram $\pm1$ desvio-
padrão entre ciclos, e não o erro-padrão da média.

As colunas representam $\nu=10^{-2},10^{-3},10^{-4},10^{-5}$ e as linhas
representam os três valores de H. Cada painel contém apenas duas séries:

- **Prey:** azul, linha contínua e marcadores circulares preenchidos;
- **Predators:** vermelho, linha tracejada e losangos brancos.

Os números do eixo Y aparecem somente na coluna esquerda para evitar repetição.
O eixo X possui marcações de 0.10 em 0.10 e é identificado por $r_P$. O símbolo
$\langle S\rangle$ foi mantido no eixo Y conforme a convenção visual adotada
no conjunto de figuras; numericamente, as curvas representam abundâncias
populacionais médias de presas e predadores.

Diferentemente da figura global de espécies, este gráfico utiliza todos os
valores de $r_P$ disponíveis nas pastas, inclusive os pontos intermediários de
três casas decimais.

## 4. Curvas de Whittaker

**Arquivo principal:** `whittaker/curvas_whittaker.png`

**Dados de entrada:** arquivos `Rede_final_ciclo_com_predação_*.dat`, que
representam a rede no último instante. Na codificação usada, `0` representa
sítio vazio, `1` representa predador e valores maiores que `1` identificam
espécies de presas.

Para cada ciclo, contam-se os indivíduos de cada espécie de presa sobrevivente.
A abundância relativa da espécie $j$ é

$$
p_j=\frac{n_j}{\sum_k n_k}.
$$

As abundâncias são ordenadas da maior para a menor. A posição na sequência é o
índice de rank $i$, produzindo a curva de rank-abundance $(i,p_i)$.

Como ciclos diferentes podem terminar com riquezas diferentes, as curvas são
alinhadas pelo rank. Posições inexistentes em um ciclo são tratadas como
ausentes (`NaN`), não como abundância zero. Para cada rank, calcula-se a média,
o primeiro quartil (Q25) e o terceiro quartil (Q75) entre os ciclos que possuem
aquele rank. A linha mostra a média e a faixa transparente mostra Q25–Q75.

São comparados $r_P=0.01$, `0.20`, `0.45` e `0.50`. Quando nenhuma presa
sobrevive em uma configuração, a curva de Whittaker é matematicamente
indefinida e, portanto, não é desenhada; o script emite um aviso.

As colunas representam $\nu=10^{-2},10^{-3},10^{-4},10^{-5}$ e as linhas os
três valores de H. Ambos os eixos são logarítmicos. A nomenclatura segue o uso
comum na literatura de rank-abundance:

- eixo X: **Species rank**, $i$;
- eixo Y: **Relative abundance**, $p_i$.

O arquivo `curvas_whittaker.csv` armazena, para cada rank, a média, Q25, Q75 e
o número de ciclos que contribuíram para aquele ponto.

## Reprodução

Na raiz do projeto, as figuras podem ser reproduzidas com:

```bash
python3 scripts/analise.py --modo fases
python3 scripts/analise.py --modo especies_global
python3 scripts/pop_vs_rp.py
python3 scripts/analise.py --modo whittaker
python3 scripts/analise.py --modo index
```

Os parâmetros metodológicos centrais ficam em `scripts/analise.py`. A lógica
interna do diagrama está em `scripts/_interno/gerar_diagrama_fases.py`; a visão
global de espécies em `scripts/_interno/gerar_especies_x_tempo_global.py`; as
populações em `scripts/pop_vs_rp.py`; e a Whittaker em
`scripts/_interno/gerar_whittaker.py`.
