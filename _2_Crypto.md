# Notas crypto

### 06-NOV-21

1. Cardano

   - Ha estado corrigiendo
   - Hay un soporte de 3100 SAT, abajo de ese hay uno de 2900
   - Hay una resistencia entre 3600 y 3900 SATS, si pasa esta resistencia probablemente podría llegar a la superior sino, el ciclo podría estar terminandose
   - En USD, hay una resistencia de 2.1$
   - El soporte está alrededor de los 1.7$, si se pierde este soporte el próximo es 1.5 - 1.4

2. Bitcoin

   - Está corrigiendo
   - Si testea 61k otra vez, puede agrrar momento hacia abajo
   - Reject en 62.5k

## Yield Farming

### Inversiones

1. ChronoSwap => 1200 usd en CNO-BUSD, 973 LP - (466% de APR)

   1. Las ganancias se reinvierten en DAI-USDC (54%), empezamos con 100usd

2. ApeSwap => 1000usd en USDC-ABR, 0.0002463873 LP - (165% de APR)

3. Beefy => 45$ - 0.00001455 LP - (50%)

Inversion total mas o menos 2350$

## Mirror Delta Neutral Strategy

Vamos a usar Long(apostamos que el valor va a subir) y Short(apostamos que el valor va bajar)

### 1. Short Farm + Long Hold (0% exposed to Impermanent Loss)

Para colateral en mirror podemos usar aUST que nos dan en Anchor

Suponiendo que tenemos 3K, 2/3 se usaran en Anchor y el otro 1/3 se usar para comprar el mAsset en mirror

1. Depositamos 2K en Anchor protrocol, esto nos da aUST
2. Con el aUST vamos a Mirror
3. Elegimos una asset y apostamos en corto

   - Ir en corto es "apostar" a que la accion va a bajar de precio
   - Por ejemplo si vamos en short a una accion que vale 100 lo que pasa es lo siguiente
     - Pedimos prestada la accion al valor actual 100
     - La accion se desvaloriza a 70 dolares
     - Ahora si pagamos la accion que prestamos
     - Ganamos 30 dolares

   Ahora que pasa si la accion sube? si la accion sube por ejemplo a 150 dolares, al pagarla tenemos que poner 50 extra, es decir, perdimos 50% de la inversion que teniamos presupuestada

   Por eso se dice que en short las perdidas son infinitas porque el precio puede subir infinitamente

4. Compramos la cantidad del mAsset contra la que apostamos y la holdeamos

   - Por ejemplo, con el aUST que tenemos compramos 1 stock de AMAZON
   - Compramos y holdeamos 1 stock de AMAZON
   - Asi, si el precio sube o baja no nos AFECTARA porque la "deuda" ya la tenemos
   - Estaremos ganando con los APY y con el ANCHOR que tenemos

### 2. Short Farm + Long LP (50% exposed to IL)

La estrategia numero 2, usara la anterior pero necesitara 1K extra

- Continuamos en el paso anterior, en vez de simplemente holdear las acciones las emparejamos con UST y las depositamos en un pool pero estaremos expuestos al impermanent loss

### 3. Long Borrow LP (67% exposed to IL)

## Terra Degen Stablefarm strategy

## Layer-1 blockchain staking: the base layer of a new global financial paradigm

No invertion is risk free, but in praactice, people take the debt issued by the US government as a benchmark, risk-free asset, because governments collect taxes

Cuando compran US Treasuries, estas directamente comprando una parte de GDP de US

Todas las blockchains cobran un fee, entre mas complicada la accion en la blockchain mas caro sera el fee, transferir fondos vale 21000 gas, hacer un swap en algun dex usa smart contracts y muchos pasos extra en el background por eso cuesta x10 mas

## Q1 2022 thesis

6 tokens that are mentioned in both accounts @blknoiz06 & @monetwithcarter

![9](images/9_1.png)

### @blknoiz06

Para resaltar el cambio constante en los lideres del mercado crypto, podemos mirar a 3 iteraciones de DeFi protocols

1. En 2019 con LINK & SNX

   Chainlink & Synthetix son 2 protocolos que fueron unas de las mayor innovaciones en DeFi, despues de subir mas de 50x contra ETH llegaron a su peak en Agosto del 2020 desde eso han estado en un bear market

2. Sushi, 40x junto a otras blue chips como Aave, Comp, Uni hasta el sellof en Mayo

3. La tercela oleada son OHM, DPX, SPELL, TOKE

   Actualmente estan abajo respecto a ETH hay que esperar si seguiran cayendo o si se recuperaran

4. SOL, AVAX, LUNA & COSMOS

   1. COSMOS

   La mayoria de la gente no sabe cuando estan usando una blockchain creada con el SDK(Software Development Kit) de Cosmos. El IBC (Inter Blockchain Communication) permite a varias blockchains hablar entre si

   Cosmos fue creado desde un principio para ser modular y descentralizado, Keplr ha estado bajo el radar, pero ya que algunas chains se han conectado y el gravity bridge esta casi listo en los proximos 12 meses puede habber un boom

#### Modular blockchains & ETH 2.0

Se separa la funcionalidad de la blockchain, creando chains especializadas para cada layer: executiion, data availability & consensus, and settlement

Actualmente las L1 hacen todo esto ellas mismas, por eso los cuellos de botella

ETH roadmap hacia modular, usar ETH as the settlement and data availability layer y otra layer 2 scaling solutions like zKRollups & optimistic rollups para encargarse de la mayoria de las transacciones

Una vez esto este completo, se podran usar L2 con la seguridaad de ETH

1. Celestia

   Blockchain dedicada to being the most efficient and decentralized data-availability layer. Se podran conectar a otras layer para lograr el full modular stack

   Se enfoca solo en data availability & ordering transaccions, esto hace que la verificada de bloques se pueda simplificar con `data availability proof`. Basicamente cada noto en la red debe muestrear un pedazo de codigo del bloque para confirmar su validez y a traves de este muestreo colectivo la red llega a un acuerdo. Entre mas nodos existan, mejor es el sistema

#### Play-to-Earn, The Gamification of Digital Assets

## 15 ENERO 2022

1. Narrativa en este momento es DeFi Play To Earn + NFT, gamification of DeFi
2. Institution to DeFi ??
   1. SOL -> Bank Of America recomendando comprar SOL (cheaper ETH)
   2. QRDO (QredoNetwork) -> Key management, sharting the key
      Metamask Institutional Wallet partner with QRDO
   3. Aave -> Aave Arc, institutional pool
   4. DON-KEY -> Low Cap play
   5. Spool -> Middleware entre Yield & Institution
   6. Luna -> Anchor + insurance. Atractivo para Institution

## 17-JAN-22

1. COTI en Cardano -> Djed stablecoin
2. Paribus PBX
3. Gero Wallet (GERO)
4. Algoritmic stablecoins
5. Investigar sobre Yearn finance, que hace
6. Olympus DAO -> Primer Protocol Owned Liquidity (PoL)

## 18-JAN-22

### SOLID - FTM new token Daniele & Andrej

TIME va pasar de OHM model a SPAC (special purpose acquisition company), es una compañia formada solamente para unir y adquirir companies

- OHM model incetivies liquidty, no es sostenible en el tiempo por el dilution
- ve(3,3) incentives fees. Ve - tocken locking. (3,3) rebasing, the longer you hold, more wMemo you earn

## 19-JAN

1. Mars protocol en terra, hay que estar atentas sera un AAVE -> Mucha ganancia si despega con el toque de gobernanza

   - Acumular MARS y bloquearlos en gobernanza cuando salga

2. Astroport -> Va ser el DEX de LUNA
   - Aportar a las pools para ganar tokens, si pega podemos sacar buena ganancia
