# FANTOM

## Curve Wars

1. Verano de 2020, explosion de DeFi 1.0
2. Curve fue creado, se conviertio en un stablecoin Automated Market Maker (AMM)

   - En julio de 2020, TVL 46M, actualmente esta en 23.5B
   - La idea es simple, permitir swap de stablecoins con bajo fee y slippage
   - Pioneros en usar su token ($CRV) a los que depositavan liquidez en sus LP

### $veCRV

veCRV (vote-escrowed CRV) es el token que dan cuando uno BLOQUEA su CRV por un tiempo, este token es el de governanza

1.  $veCRV = $inSPIRIT
2.  $CRV lockers -> se volvieron $veCRV holders -> Ganan 50% de todos los feeds de curve
3.  Igualmente, $inSPIRIT holders ganan $SPIRIT
4.  $veCRV con sus votos influyen en las pools, y redirigen los gauges, los votos son cada 10 dias
5.  Gauges deciden a donde va la inflacion de $CRV, controlan la liquidez

### Bribes

Como los protocolos y DAOs intentan influenciar la liquidez hacia algunos pools ? con BRIBES, bribes a veCRV holders para que voten por algun gauges en especifico que beneficie el protocolo

Como curve sigue entregando CRV, la cantidad de CRV `RECIBIDO` (inflacion) por una pool (`GAUGE`) se basa en el `GAUGE WEIGHT` que depende de cuanto `veCRV` ha votado por ese `GAUGE`

### ConvexFinance

Convex ofrece recompensas buenas a quienes cambian `CRV` a `cvxCRV` y lo stakean en convex

1. Esto le da poder a **convex** sobre el poder de voto del `CRV` que le estan delegando (veCRV)
2. `cvxCRV` -> `linSPIRIT`
3. Convex posee el **51%** de todos los veCRV
4. LiquidDriver tiene el **20%** por ahora

Con el 51% de `veCRV` convex puede ahora usar los bribes para favorecer su token `CVX`

El exito que ha tenido CONVEX empezo la fase 2 de las curve wars, ahora los protocolos se estan peleando para adquirir `CVX`

El que maneje el CVX, maneja la liquidez y gobernanza de CURVE. CVX ha hecho un x20 en 5 meses

## Spirit Wars

- SPIRIT = CURVE
  - CRV = SPIRIT
  - veCRV = inSPIRIT
- LIQUID = CONVEX
  - LQDR = CVX
  - xLQDR = vlCVX (vote lock CVX)
  - linSPIRIT = cvxCRV

El 1ro de diciembre spirit anuncio cambios importantes, basicamente se volveran un curve

1. Transicion a DAO
2. Migracion a Boosted Farms
3. Generar demanda del inSPIRIT con smart tokenomics
   - Smart Tokenomics: Acquire $SPIRIT, lock $SPIRIT, receive $inSPIRIT, use $inSPIRIT to vote for a higher weight of emissions (APR) on a protocols LP pools (the gauge)

El que controle veCRV controla la liquidez
El que controle inSPIRIT controla la liquidez

CONVEX ofrece recompensas al que cambia CRV x `cvxCRV`
LIQUID ofrece recompensas al que cambia SPIRIT x `linSPIRIT` y con el voto aumentan la ganancia de los que stakean `xLQDR`

Teniendo esto en cuenta y pensando en precios

1. CURVE esta en 3.4x desde Julio (90% del CURVE esta bloquead)
2. CONVEX esta en 16.5x desde Julio

LIQUID y su linSPIRIT retiran el SPIRIT del mercado para siempre, lo que genera escases

## Estrategia LQDR

Use MIM/FTM to build your $LQDR position.

1. Stake MIM/FTM for 153% APR
2. Receive LQDR
3. Lockup LQDR rewards as xLQDR
4. Receive 110% on those xLQDR
5. Funnel rewards back into MIM/FTM or xLQDR

Probablemente agregar una pool en TOMB para generar mas FTM
