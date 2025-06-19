# Meteora DLMM CLI - Fixed Flowchart

```mermaid
flowchart TD
    %% Start and Initialization
    A([Start]) --> B[Load package.json dependencies]
    B --> C[Import Solana Web3.js modules]
    C --> D[Import Meteora DLMM SDK]
    D --> E[Import LocalWalletProvider]
    E --> F[Import Jupiter API client]
    F --> G[Initialize global variables]
    G --> H[Create MeteoraClient instance]
    
    %% MeteoraClient Constructor
    H --> I[Initialize Solana Connection to Helius RPC]
    I --> J[Create LocalWalletProvider with keypair path]
    J --> K[Load keypair from file system]
    K --> L{Keypair file exists?}
    L -->|No| M[Throw Error: Keypair file not found]
    L -->|Yes| N[Parse keypair data from JSON]
    N --> O[Create Keypair from secret key]
    O --> P[Log wallet public key]
    P --> Q[Create Jupiter API client]
    Q --> R[Connect to DLMM pool]
    
    %% Pool Connection
    R --> S[Create PublicKey from pool address]
    S --> T[Call DLMM.create with connection and pool key]
    T --> U{DLMM pool created successfully?}
    U -->|No| V[Log Error: Pool connection failed]
    V --> W[Return false]
    U -->|Yes| X[Log: Successfully connected to pool]
    X --> Y[Display main menu]
    
    %% Main Menu Loop
    Y --> Z[Show 6 menu options]
    Z --> AA[Wait for user input]
    AA --> BB{User selection}
    
    %% Option 1: Check Positions
    BB -->|1| CC[Get current active bin]
    CC --> DD[Call dlmmPool.getActiveBin]
    DD --> EE[Format price: price * 1000]
    EE --> FF[Fetch wallet positions]
    FF --> GG[Call dlmmPool.getPositionsByUserAndLbPair]
    GG --> HH[Get bins around active bin]
    HH --> II[Call dlmmPool.getBinsAroundActiveBin]
    II --> JJ[Process each position]
    JJ --> KK[Calculate total fees and liquidity]
    KK --> LL[Format token amounts with decimals]
    LL --> MM[Calculate USD values]
    MM --> NN[Display position details]
    NN --> Y
    
    %% Option 2: Open New Position
    BB -->|2| OO[Prompt: Enter amount of SOL]
    OO --> PP[Parse user input as float]
    PP --> QQ{Valid SOL amount?}
    QQ -->|No| RR[Display: Invalid SOL amount]
    RR --> Y
    QQ -->|Yes| SS[Get current active bin]
    SS --> TT[Calculate bin range: activeBin ± binRange]
    TT --> UU[Convert SOL to lamports: amount * 10^9]
    UU --> VV[Create BN instance for totalXAmount]
    VV --> WW[Call autoFillYByStrategy for balanced Y amount]
    WW --> XX[Create new position Keypair]
    XX --> YY[Call dlmmPool.initializePositionAndAddLiquidityByStrategy]
    YY --> ZZ[Set slippage to 10]
    ZZ --> AAA[Send transaction with sendAndConfirmTransaction]
    AAA --> BBB{Transaction successful?}
    BBB -->|No| CCC[Check if slippage error]
    CCC -->|Yes| DDD[Double bin range and retry]
    DDD --> SS
    CCC -->|No| EEE[Log error and throw]
    EEE --> Y
    BBB -->|Yes| FFF[Display position result]
    FFF --> Y
    
    %% Option 3: Close Position
    BB -->|3| GGG[Fetch wallet positions]
    GGG --> HHH{Has positions?}
    HHH -->|No| III[Display: No positions available]
    III --> Y
    HHH -->|Yes| JJJ[List available positions with values]
    JJJ --> KKK[Prompt: Select position number]
    KKK --> LLL{Valid selection?}
    LLL -->|No| MMM[Display: Invalid selection]
    MMM --> Y
    LLL -->|Yes| NNN[Get position data]
    NNN --> OOO[Check for unclaimed fees]
    OOO --> PPP{Has unclaimed fees?}
    PPP -->|Yes| QQQ[Call dlmmPool.claimAllSwapFee]
    QQQ --> RRR[Send claim fee transaction]
    PPP -->|No| SSS[Check for liquidity]
    RRR --> SSS
    SSS --> TTT{Has liquidity?}
    TTT -->|Yes| UUU[Call dlmmPool.removeLiquidity]
    UUU --> VVV[Set bps to 10000 100%]
    VVV --> WWW[Set shouldClaimAndClose to true]
    WWW --> XXX[Send remove liquidity transaction]
    TTT -->|No| YYY[Call dlmmPool.closePosition]
    XXX --> ZZZ[Wait for transaction confirmation]
    YYY --> ZZZ
    ZZZ --> AAAA{Close successful?}
    AAAA -->|No| BBBB[Log error and throw]
    BBBB --> Y
    AAAA -->|Yes| CCCC[Display close result]
    CCCC --> Y
    
    %% Option 4: Automatic Rebalance
    BB -->|4| DDDD[Prompt: Enter SOL amount for rebalance]
    DDDD --> EEEE{Valid SOL amount?}
    EEEE -->|No| FFFF[Display: Invalid SOL amount]
    FFFF --> Y
    EEEE -->|Yes| GGGG[Set wiggle room: 0.02]
    GGGG --> HHHH[Set poll interval: 5000ms]
    HHHH --> IIII[Set max poll attempts: 15]
    IIII --> JJJJ[Set max errors: 3]
    JJJJ --> KKKK[Initialize error counter: 0]
    KKKK --> LLLL[Set running flag: true]
    LLLL --> MMMM[Get initial balance]
    MMMM --> NNNN[Start rebalance loop]
    
    %% Rebalance Loop
    NNNN --> OOOO[Get current active bin]
    OOOO --> PPPP[Fetch wallet positions]
    PPPP --> QQQQ[Check each position]
    QQQQ --> RRRR{Position out of range?}
    RRRR -->|Yes| SSSS[Store bin range before closing]
    SSSS --> TTTT[Get balance before closing]
    TTTT --> UUUU[Close out-of-range position]
    UUUU --> VVVV[Wait for funds to return]
    VVVV --> WWWW[Poll balance every 5 seconds]
    WWWW --> XXXX{Balance returned?}
    XXXX -->|No| YYYY[Check timeout: 15 attempts]
    YYYY -->|Yes| ZZZZ[Log timeout warning]
    ZZZZ --> AAAAA[Proceed anyway]
    XXXX -->|Yes| BBBBB[Update initial balance]
    YYYY -->|No| BBBBB
    AAAAA --> BBBBB
    BBBBB --> CCCCC[Execute auto Jupiter swap]
    CCCCC --> DDDDD[Calculate target balances]
    DDDDD --> EEEEE[Set SOL rent reserve: $10]
    EEEEE --> FFFFF[Calculate total value USD]
    FFFFF --> GGGGG[Calculate target per side]
    GGGGG --> HHHHH{Excess in SOL or USDC?}
    HHHHH -->|SOL Excess| IIIII[Calculate SOL to USDC swap]
    HHHHH -->|USDC Excess| JJJJJ[Calculate USDC to SOL swap]
    HHHHH -->|Balanced| KKKKK[No swap needed]
    IIIII --> LLLLL{Amount >= $1.00?}
    JJJJJ --> LLLLL
    LLLLL -->|No| MMMMM[Skip swap: too small]
    LLLLL -->|Yes| NNNNN[Get Jupiter quote]
    MMMMM --> OOOOO[Return success]
    NNNNN --> PPPPP{Quote valid?}
    PPPPP -->|No| QQQQQ[Log quote error]
    PPPPP -->|Yes| RRRRR[Execute swap with 50000 priority fee]
    QQQQQ --> OOOOO
    RRRRR --> SSSSS{Swap successful?}
    SSSSS -->|No| TTTTT[Log swap error]
    SSSSS -->|Yes| UUUUU[Update balance display]
    TTTTT --> VVVVV[Check in-range positions]
    UUUUU --> VVVVV
    KKKKK --> VVVVV
    VVVVV --> WWWWW{Has in-range positions?}
    WWWWW -->|No| XXXXX[Open new position with stored bin range]
    WWWWW -->|Yes| YYYYY[Log: position in range]
    XXXXX --> ZZZZZ{Position opened?}
    ZZZZZ -->|No| AAAAAA[Log position error]
    ZZZZZ -->|Yes| BBBBBB[Log position success]
    YYYYY --> CCCCCC[Wait 30 seconds]
    AAAAAA --> CCCCCC
    BBBBBB --> CCCCCC
    CCCCCC --> DDDDDD{User interrupted?}
    DDDDDD -->|Yes| EEEEEE[Graceful shutdown]
    DDDDDD -->|No| FFFFFF{Too many errors?}
    EEEEEE --> Y
    FFFFFF -->|Yes| GGGGGG[Emergency shutdown: close all positions]
    FFFFFF -->|No| NNNN
    GGGGGG --> Y
    RRRR -->|No| HHHHHH[Log: position in range]
    HHHHHH --> VVVVV
    
    %% Option 5: Jupiter Swap
    BB -->|5| IIIIII[Get current wallet balance]
    IIIIII --> JJJJJJ[Display SOL and USDC balance]
    JJJJJJ --> KKKKKK[Display total USD value]
    KKKKKK --> LLLLLL[Show swap direction options]
    LLLLLL --> MMMMMM{User choice}
    MMMMMM -->|1| NNNNNN[Prompt: Enter SOL amount]
    MMMMMM -->|2| OOOOOO[Prompt: Enter USDC amount]
    NNNNNN --> PPPPPP{Valid SOL amount?}
    PPPPPP -->|No| QQQQQQ[Display: Invalid amount]
    QQQQQQ --> Y
    PPPPPP -->|Yes| RRRRRR[Convert to lamports: amount * 10^9]
    OOOOOO --> SSSSSS{Valid USDC amount?}
    SSSSSS -->|No| TTTTTT[Display: Invalid amount]
    TTTTTT --> Y
    SSSSSS -->|Yes| UUUUUU[Convert to USDC decimals: amount * 10^6]
    RRRRRR --> VVVVVV[Show priority fee options]
    UUUUUU --> VVVVVV
    VVVVVV --> WWWWWW{Priority level}
    WWWWWW -->|1| XXXXXX[Set priority fee: 50000 default]
    WWWWWW -->|2| YYYYYY[Set priority fee: 10000 low]
    WWWWWW -->|3| ZZZZZZ[Set priority fee: 50000 medium]
    WWWWWW -->|4| AAAAAAA[Set priority fee: 100000 high]
    WWWWWW -->|5| BBBBBBB[Set priority fee: 500000 very high]
    XXXXXX --> CCCCCCC[Get Jupiter quote]
    YYYYYY --> CCCCCCC
    ZZZZZZ --> CCCCCCC
    AAAAAAA --> CCCCCCC
    BBBBBBB --> CCCCCCC
    CCCCCCC --> DDDDDDD{Quote received?}
    DDDDDDD -->|No| EEEEEEE[Display: No quote found]
    EEEEEEE --> Y
    DDDDDDD -->|Yes| FFFFFFF[Get swap instructions]
    FFFFFFF --> GGGGGGG[Set slippage to 50 bps 0.5%]
    GGGGGGG --> HHHHHHH[Set wrapAndUnwrapSol to true]
    HHHHHHH --> IIIIIII[Set dynamic compute unit limit]
    IIIIIII --> JJJJJJJ[Deserialize transaction]
    JJJJJJJ --> KKKKKKK[Sign transaction with wallet]
    KKKKKKK --> LLLLLLL[Send raw transaction]
    LLLLLLL --> MMMMMMM[Wait for confirmation]
    MMMMMMM --> NNNNNNN{Swap successful?}
    NNNNNNN -->|No| OOOOOOO[Log swap error]
    OOOOOOO --> Y
    NNNNNNN -->|Yes| PPPPPPP[Calculate price impact]
    PPPPPPP --> QQQQQQQ[Update total fees]
    QQQQQQQ --> RRRRRRR[Display swap results]
    RRRRRRR --> Y
    
    %% Option 6: Exit
    BB -->|6| SSSSSSS[Close readline interface]
    SSSSSSS --> TTTTTTT[Log: Goodbye!]
    TTTTTTT --> UUUUUUU([End])
    
    %% Error Handling
    M --> UUUUUUU
    V --> UUUUUUU
    EEE --> UUUUUUU
    BBBB --> Y
    FFFF --> Y
    TTTTT --> Y
    QQQQQQ --> Y
    TTTTTT --> Y
    EEEEEEE --> Y
    OOOOOOO --> Y
    
    %% Styling
    classDef startEnd fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#f3e5f5,stroke:#4a148c,stroke-width:1px
    classDef decision fill:#fff3e0,stroke:#e65100,stroke-width:1px
    classDef error fill:#ffebee,stroke:#c62828,stroke-width:1px
    classDef success fill:#e8f5e8,stroke:#2e7d32,stroke-width:1px
    classDef external fill:#fce4ec,stroke:#ad1457,stroke-width:1px
    
    class A,UUUUUUU startEnd
    class B,C,D,E,F,G,H,I,J,K,N,O,P,Q,R,S,T,X,Y,Z,AA,CC,DD,EE,FF,GG,HH,II,JJ,KK,LL,MM,NN,OO,PP,SS,TT,UU,VV,WW,XX,YY,ZZ,AAA,FFF,GGG,JJJ,KKK,NNN,OOO,QQQ,RRR,SSS,UUU,VVV,WWW,XXX,YYY,ZZZ,CCCC,DDDD,EEEE,GGGG,HHHH,IIII,JJJJ,KKKK,LLLL,MMMM,OOOO,PPPP,QQQQ,SSSS,TTTT,UUUU,VVVV,WWWW,BBBBB,CCCCC,DDDDD,EEEEE,FFFFF,GGGGG,HHHHH,IIIII,JJJJJ,KKKKK,LLLLL,MMMMM,NNNNN,OOOOO,PPPPPP,RRRRRR,SSSSSS,TTTTTT,UUUUUU,VVVVVV,WWWWWW,XXXXXX,YYYYYY,ZZZZZZ,AAAAAAA,BBBBBBB,CCCCCCC,FFFFFF,GGGGGGG,HHHHHHH,IIIIIII,JJJJJJJ,KKKKKKK,LLLLLLL,MMMMMMMM,PPPPPPP,QQQQQQQ,RRRRRRR,SSSSSSS,TTTTTTT process
    class L,QQ,BB,HHH,LLL,BBB,CCC,DDD,EEE,AAAA,DDDDD,FFFFF,HHHHH,LLLLL,MMMMM,NNNNN,PPPPPP,SSSSSS,TTTTTT,UUUUUU,WWWWWW,DDDDDDD,NNNNNNN decision
    class M,V,EEE,BBB,FFFF,BBBB,TTTTT,QQQQQQ,TTTTTT,EEEEEE error
    class X,NN,FFF,CCCC,OOOOO,UUUUU,RRRRRR success
    class T,GG,AAA,ZZZ,RRRRR,CCCCCC external
```

## Where to View This Fixed Flowchart:

### **1. GitHub/GitLab (Recommended)**
- Upload this markdown file to your repository
- GitHub/GitLab will automatically render the Mermaid diagram

### **2. Mermaid Live Editor**
- Go to https://mermaid.live/
- Copy the code between the ```mermaid tags
- Paste it into the editor for instant preview

### **3. VS Code with Extension**
- Install "Markdown Preview Mermaid Support" extension
- Open the markdown file and use Ctrl+Shift+V for preview

### **4. Online Documentation Platforms**
- GitBook, Notion, Confluence (with plugins)

The main fixes I made:
- Removed parentheses from function calls in node names
- Removed special characters like `()` and `[]` from node text
- Simplified node names to avoid parsing conflicts
- Kept all the logic and flow intact

This should now render properly in any Mermaid-compatible viewer!
