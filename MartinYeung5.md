---
timezone: Asia/Shanghai
---

# MartinYeung5

1. 自我介绍
* Martin Yeung, 來自中國香港，計算機科學+電商專業，香港城市大學理學碩士畢業，專注於區塊鏈技術研究和應用。

2. 你认为你会完成本次残酷学习吗？
* 會的。

## Notes

<!-- Content_START -->

### 2024.09.07




Aptos是基於MOVE語言的一個公鏈，利用MOVE可以構建一個更安全的智能合約，特別是用於數字資產領域，會突顥出其價值，當然也需要看是在怎樣的應用場景。 
針對MOVE智能合約的開發，有不同的知識點需要學習，需要理解不同代碼的原理及應用方式。
要建立完整的Aptos項目，在完成MOVE智能合約的開發後，
需要設計frontend及與智能合約的進行交互，而我最近在學習與智能合約的交互，
透過web3電子錢包讓用戶可以與平台/智能合約互動。


今天想分享一下我對useWallet的學習和了解，
Frontend交互部分，我用了@aptos-labs/wallet-adapter-react，透過它使用到useWallet，然後從useWallet獲得需要用到的功能，如下:

```
const {
    account,
    network,
    signAndSubmitTransaction,
    signMessageAndVerify,
    signMessage,
    signTransaction,
} = useWallet();
```

使用useWallet，可以讓用戶進行一些簽名及提交交易的動作。
* 第一個例子(signMessage)是讓用戶對一項訊息進行簽名:

```
  const onSignMessage = async () => {
    const payload = {
      message: "Hello，我是Martin Yeung，對Aptos相當感興趣。",
      nonce: Math.random().toString(16),
    };

    //簽名後的回應訊息為response，可以留意到const response的類型是"SignMessageResponse"，
    //這個就是通過signMessage()所獲得到訊息類型
    
    const response = await signMessage(payload);
    
    //顯示response的訊息內容，以便做debug
    //而且可以讓用戶知道自己最終是否成功完成簽名
    //當中的await就是等待signMessage的回應，如果signMessage完成就能返會一些訊息。
    //如果沒有await的話，系統就不會一直等待signMessage的回應，所以會出現跳過signMessage回應的情況
    //有機會出現的情況就是signMessage失敗，但完成onSignMessage這功能。
    
    console.log(response);
  };
```

用戶執行以上功能的流程:
1. 在frontend頁面，當用戶觸發到onSignMessage功能，就會執行到以上的動作
2. 用戶的web3電子錢包會彈出
3. 在錢包上會顯示"Hello，我是Martin Yeung，對Aptos相當感興趣"
4. 及要求用戶對這訊息進行簽名，
因為這要確保用戶是能清楚知道自己是對什麼東西進行簽名確認，
5. 如果用戶確認沒問題，就可以在錢包上點擊"Sign"進行簽名的動作
6. 用戶等待簽名完成的回應 (這個可以在Frontend設計一個顯示回應的訊息，讓用戶知道最終結果)


* 第二個例子(signAndSubmitTransaction)是讓用戶對一項交易進行簽名及提出交易動作:
假設用戶需要轉帳APTOS到某個指定錢包，需要用戶進行簽名及交易。

```
  const APTOS_COIN = "0x1::aptos_coin::AptosCoin";
  const onSignAndSubmitTransaction = async () => {
    if (!account) return;
    const transaction: InputTransactionData = {
      data: {
        function: "0x1::coin::transfer",
        typeArguments: [APTOS_COIN],
        functionArguments: [account.address, 1], // 1 is in Octas
      },
    };
    try {

      //簽名後的回應訊息為response，可以留意到const response的類型是"any" 
      const response = await signAndSubmitTransaction(transaction);
      await aptosClient(network).waitForTransaction({
        transactionHash: response.hash,
      });
      console.log(response);

    } catch (error) {
      console.error(error);
    }
  };
```

在上面第二個的例子，可以留意到response的類型是any，跟第一個例子不同，因為它們所返回的訊息是有所不同，
在第二個例子返回的訊息是一個已完成簽名及已完成交易動作的紀錄。

第三個例子(signMessageAndVerify)是讓用戶對一項訊息進行簽名及進行確認:

```
  //
  const onSignMessageAndVerify = async () => {
    const payload = {
      message: "Hello，我是Martin Yeung，對Aptos相當感興趣。",
      nonce: Math.random().toString(16),
    };

    //簽名後的回應訊息為response，可以留意到const response的類型是"boolean"
    const response = await signMessageAndVerify(payload);
    console.log(response);
  };
```

在上面第三個的例子，可以留意到response的類型是boolean，跟第一、二個例子不同，
因為這個例子的目的是要用戶進行簽名及進行確認，所以返回的訊息只有true或false。
當然在第一個例子也可以通過做一些額外判斷(需要進一步編寫代碼)而獲得true或false，
但這例子就簡單直接，所以可以根據自己的需求而使用合適的功能。

### 2024.09.08


繼續昨天對useWallet的學習和了解，
第四個例子(signTransaction)是讓用戶進行一項交易動作:

```
  // Legacy typescript sdk support
  const onSignTransaction = async () => {
    try {
      const payload = {
        type: "entry_function_payload",
        function: "0x1::coin::transfer",
        type_arguments: ["0x1::aptos_coin::AptosCoin"],
        arguments: [account?.address, 1], // 1 is in Octas
      };

      // 交易完成後的回應為response，可以留意到const response的類型是"AccountAuthenticator"。
      const response = await signTransaction(payload);
      console.log(response);
    } catch (error) {
      console.error(error);
    }
  };
```

可以看到signTransaction中的參數是payload，它的type是"entry_function_payload"，
當動作完成後返回的數據類型是"AccountAuthenticator"。


第五個例子(signTransaction)也是讓用戶進行一項交易動作:

```
  const onSignTransactionV2 = async () => {
    if (!account) return;

    try {
      const transactionToSign = await aptosClient(
        network,
      ).transaction.build.simple({
        sender: account.address,
        data: {
          function: "0x1::coin::transfer",
          typeArguments: [APTOS_COIN],
          functionArguments: [account.address, 1], // 1 is in Octas
        },
      });
      
      // 交易完成後的回應為response，可以留意到const response的類型是"AccountAuthenticator"。
      const response = await signTransaction(transactionToSign);
      bobSenderAuthenticator = response;
      console.log(response);
    } catch (error) {
      console.error(error);
    }
  }
```

在這個例子中，signTransaction的參數是transactionToSign，而transactionToSign是通過"transaction.build.simple"獲得。跟第四個例子相比，主要是signTransaction的參數不同，不過最終的結果是相同的，所得出的類型也是一樣。

### 2024.09.09

* 創建智能合約
創建智能合約，在Aptos只需通過一句簡單命令即可完成。
```
aptos move init --name my_todo_list
```
當執行完成之後，就可會出現以下訊息:
```
{
  "Result": "Success"
}
```

在當下目錄會出現4個文件，分別是3個文件夾及一個Move.toml文件。

文件夾有scripts, sourcesm, tests，而我們在執行命令所輸入到的 -- name my-todo_list 
就會出現在Move.tome文件內。

* 安全性
針對安全性，Aptos有以下4個優勢:
1. Formal Verification	
* Aptos framework is fully specified and formally verified with the Move Prover. This includes the core contracts involving governance, NFTs, and Tokens.
2. Gas Coverage	
* Move VM has 100% gas coverage. Gas is charged based upon actual usage in the system (CPU, memory, storage, I/O). In other words, no gas exploits.
3. Security Redundancy	
* Security redundancy provided by runtime safety checks.
4. Permission Controls	
* Permission controls can flexibly be built at various levels. For example, token level permission controls exist by default to enable RWA tokenization.

### 2024.09.10
* Authenticator
在aptos, 未簽名的交易會被視為"RawTransaction"。
它們會包含所有有關在Aptos上執行的動作的訊息，但就沒有包括到合適的有效簽名或Authenticator。
在Aptos區塊鏈上，所有數據都會被加密為BCS(Binary Canonical Serialization)。
而Aptos也會支持多種簽名方式，會以Ed25519作為預設的選擇，大家可以根據自己的項目情況而作出更改。
Authenticator在執行交易的簽名過程中，會給Aptos區塊鏈櫂限來執行用戶的交易動作。
所以把Authenticator理解為收集用戶簽名權限的地方，然後再交給Aptos區塊鏈去執行最終的動作。

### 2024.09.11
* 怎樣使到RawTransaction 成為一個已簽名的交易
1. 首先用戶在頁面提供交易動作後，在經過區塊鏈最終確認前，會先被加密成為BCS，
2. 然後通過Serialization對訊息進行簽名，在這過程中，需要獲得用戶的private key，再利用用戶的private key
來生成一個有效的交易簽名。在這一步會產生一個RawTransaction的簽名，但仍未進行交易的。
3. 之後會經過Authenticator進行下一步動作，Authenticator會獲取用戶的public key和較早前獲到的RawTransaction的簽名。
4. 當Authenticator成功收集到兩項資料就會讓Aptos進行最後的交易簽名的動作。


### 2024.09.12
建立一個已簽名的文易
1. Raw Transaction
首先 raw transaction是由以下幾個部分組成的:
* 發送者的地址(Address)
* 一組序號(unit64): 這組數字是針對當下的交易，而它也是必須與儲存在發送者戶口中的序號相符。
* Payload：Aptos區塊鏈的指令，包括發佈模組、執行腳本函數或執行腳本有效負載。
* max_gas_amount (uint64): 此交易花費的最大總gas fee。帳戶必須擁有大過此gas fee的通證，
否則交易將在驗證過程中被丟棄。


### 2024.09.13
針對建立一個已簽名的交易，可以透過整個流程進作一步了解。
當中有幾個重點部分/術語可以深入認識。
* Raw Transaction
* BCS
* Signing message
* Signature
* Signed transaction
* Multisignature transactions

### 2024.09.14
建立一個已簽名的交易，以下是整個流程:
* 第一步: Creating a RawTransaction
這例子是假設交易具有腳本函數的負載。
```
interface AccountAddress {
  // 32-byte array
  address: Uint8Array;
}

interface ModuleId {
  address: AccountAddress;
  name: string;
}

interface ScriptFunction {
  module: ModuleId;
  function: string;
  ty_args: string[];
  args: Uint8Array[];
}

interface RawTransaction {
  sender: AccountAddress;
  sequence_number: number;
  payload: ScriptFunction;
  max_gas_amount: number;
  gas_unit_price: number;
  expiration_timestamp_secs: number;
  chain_id: number;
}

function createRawTransaction(): RawTransaction {
  const payload: ScriptFunction = {
    module: {
      address: hexToAccountAddress("0x01"),
      name: "AptosCoin",
    },
    function: "transfer",
    ty_args: [],
    args: [
      BCS.serialize(hexToAccountAddress("0x02")), // receipient of the transfer
      BCS.serialize_uint64(2), // amount to transfer
    ],
  };

  return {
    sender: hexToAccountAddress("0x01"),
    sequence_number: 1n,
    max_gas_amount: 2000n,
    gas_unit_price: 1n,
    // Unix timestamp, in seconds + 10 minutes
    expiration_timestamp_secs: Math.floor(Date.now() / 1000) + 600,
    payload: payload,
    chain_id: 3,
  };
}
```

* 第二步: 建立簽名訊息及進行簽名
1. 利用字串 APTOS::RawTransaction 的 SHA3_256 雜湊位元組產生前綴 (prefix_bytes)。
2. BCS 序列化 RawTransaction 的位元組。
3. 連接前綴和 BCS 位元組。
4. 使用用戶私鑰對位元組進行簽署。
```
import * as Nacl from "tweetnacl";

function hashPrefix(): Buffer {
  let hash = SHA3.sha3_256.create();
  hash.update(`APTOS::RawTransaction`);
  return Buffer.from(hash.arrayBuffer());
}

function bcsSerializeRawTransaction(txn: RawTransaction): Buffer {
  ...
}

// This will serialize a raw transaction into bytes
function serializeRawTransaction(txn: RawTransaction): Buffer {
  // Generate a hash prefix
  const prefix = hashPrefix();

  // Serialize txn with BCS
  const bcsSerializedTxn = bcsSerializeRawTransaction(txn);

  return Buffer.concat([prefix, bcsSerializedTxn]);
}

const rawTxn = createRawTransaction();
const signature = Nacl.sign(hashRawTransaction(rawTxn), ACCOUNT_PRIVATE_KEY);
```

* 第三步: 建立 Authenticator(驗證器)和 SignedTransaction
```
interface Authenticator {
  public_key: Uint8Array;
  signature: Uint8Array;
}

interface SignedTransaction {
  raw_txn: RawTransaction;
  authenticator: Authenticator;
}

const authenticator = {
  public_key: PUBLIC_KEY,
  signature: signature,
};

const signedTransaction: SignedTransaction = {
  raw_txn: rawTxn,
  authenticator: authenticator,
};
```

* 第四步: 序列化 SignedTransaction
使用 BCS 將 SignedTransaction 序列化為位元組。
```
const signedTransactionPayload = bcsSerializeSignedTransaction(signedTransaction);
```

提交已簽署的交易:
最後，使用所需的Header「Content-Type」提交相關交易。

如果以 BCS 格式提交簽章交易，用戶端必須傳入特定Header，
以下是例子：
```
curl -X POST -H "Content-Type: application/x.aptos.signed_transaction+bcs" --data-binary "@path/to/file_contains_bcs_bytes_of_signed_txn" https://some_domain/transactions
```

### 2024.09.15
* 交易的生命周期
為了更深入了解 Aptos 交易的生命週期（從操作角度），會追蹤交易的整個過程。
從提交到 Aptos fullnode，到提交到 Aptos 區塊鏈。
接下來會將重點放在 Aptos 節點的邏輯元件，看看交易如何與這些元件互動。

* 前設:
1. Alice 和 Bob 是兩個用戶，每個人在 Aptos 區塊鏈上都有一個帳戶。
2. Alice的帳戶有110個Aptos幣。
3. Alice 正在向 Bob 發送 10 個 Aptos 幣。
4. Alice帳戶目前的序號是5（這表示Alice帳戶已經發送了5筆交易）。
5. 網路上共有 100 個驗證節點 — 由 V1 到 V100。
6. Aptos 用戶端將 Alice 的交易提交到 Aptos fullnode上的 REST 服務。
fullnode將此交易轉送給驗證器fullnode，驗證器fullnode將其轉送給驗證器 V1。
7. 驗證者 V1 是本輪的提議者/領導者。

* 客戶提交交易:
Aptos 用戶端建立一個原始交易（稱為 T5），
將 10 個 Aptos 幣從 Alice 的帳戶轉移到 Bob 的帳戶。 
Aptos 用戶端使用 Alice 的私鑰對交易進行簽署。已簽署的交易T5包括以下：

* 原始交易
* Alice的公鑰
* Alice的簽名

原始交易包括以下欄位：
1. 帳戶地址: Alice的帳戶地址
2. Move模組: 包含 Alice 執行的操作的模組（或程式）。當中有：
* Move字節碼的點對點交易腳本
* 腳本輸入的清單（包含 Bob 的帳戶地址和 Aptos 幣的支付金額）。
3. 最大Gas的數量: 
Alice 願意為此交易支付的最大天然氣量。 Gas 是支付計算和儲存費用的一種方式。\
Gas的單位是計算的抽象測量。
4. Gas 價格: 
Alice 願意為執行交易每單位 Gas 支付的金額（以 Aptos 幣為單位）。
5. 到期時間: 
交易的到期時間。
6. 序號: 
帳戶的序號（例子為 5）表示已從該帳戶在鏈上提交和落實的交易數量。
在例子中，Alice 的帳戶已提交 5 筆交易，其中包括 T5。
注意：序號為5的交易只有在帳號序號為5的情況下才能在鏈上提交。
7. Chain ID: 
區分 Aptos 網路部署的識別碼（以防止跨網路攻擊）。

### 2024.09.16
#### 交易的生命周期
![alt text](https://github.com/MartinYeung5/intensive-colearning-aptos/blob/main/20240916_lifecycle_of_transaction.svg?raw=true)

交易的生命週期分為五個階段：
1. 接受：接受交易
2. 分享：與其他驗證節點分享交易數據
3. 提議：向區塊提出提議
4. 執行與共識：執行區塊汞達成共識機制
5. 提交：向區塊提出提交

### 2024.09.17
#### 驗證節點介紹
https://aptos.dev/en/network/blockchain/validator-nodes#consensus
Aptos 節點是 Aptos 生態系統中的一個實體，目的是用於追蹤 Aptos 區塊鏈的狀態。
客戶端可以透過 Aptos 節點與區塊鏈互動。
Aptos 有兩種類的節點: 分別是驗證者節點及完整節點。

當交易提交到Aptos區塊鏈時，驗證節點運行分希式的共識協議，執行交易，
並將交易和執行的交易結果儲存在區塊鏈上。驗證者節點會決定將哪些交易新增到區塊鏈及以順序形式執行。

Aptos 區塊鏈使用拜占庭容錯（Byzantine Fault Tolerance, BFT）共識協議讓驗證者節點就最終交易的帳本及執行結果達成一致共識。
驗證者節點會處理這些交易及將它們包含在本地的區塊鏈資料庫的副本。這表示最新的驗證者節點會在本地維持到一個最新狀態的區塊鏈副本。

驗證者節點透過私密網路直接與其他驗證者節點溝通。完整節點是外部驗證及(或)資源傳播的最終交易的歷史記錄。
他們能夠從其他具相同性質的節點接收交易，並且可以在本地重新執行相關數據（與驗證者節點執行交易的方式相同）。
完整節點會將重新執行交易的結果儲存到本地儲存位置。在整個過程中，他們可以隨時挑戰驗證者的任何犯規行為，及針對是否曾經試圖重寫或修改區塊鏈歷史紀錄的情況提供證據。這有助減輕驗證者節點的腐敗及(或)共謀作惡。

### 2024.09.18
#### 驗證節點的組成
* Mempool:
Mempool保存了已提交到區塊鏈但尚未達成一致或執行的交易在記憶體緩衝區的一個元件。
在這個緩衝區能夠在驗節點和完整節點之間複製。

在完整節點的JSON-RPC服務會將交易傳送到驗證節點的mempool。
然後Mempool對交易進行不同檢查，以確保交易的有效性及防止DOS攻擊。
當新交易通過初始驗證並添加到mempool時，它會即時被分發到網路中其他驗證節點的mempool。

當驗證節點暫時成為共識協議中的領導者時，共識會從mempool中提取一組交易，然後提出一個新的交易區塊。
之後該區塊會被傳送給其他驗證者，並包含該區塊中所有交易的總次序。
然後每個驗證者執行該區塊及對會否接受新的區塊所提出的提案進行投票。

* 共識(consensus):
共識是負責對交易區塊進行排序並透過與網路中的其他驗證器節點參與共識協議來就執行結果達成一致的元件。

* 執行 (execution)
執行是協調交易區塊的執行及維持短暫狀態的元件。共識投票會在短暫狀態進行，執行會在記憶體請求中維持一個執行結果，
直到達成共識及將該區塊提交到分布式資料庫為止。
執行使用虛擬機器來執行交易。執行的部分會充當系統輸入(以交易形式表示)、儲存(提供持久層)和虛擬機器(用於執行)之間的glue layer。

* 虛擬機器（Virtual machine）
用於執行每個交易內的Move程式及確定執行結果。節點的mempool使用虛擬機器對交易進行驗證檢查，
而執行會使用虛擬機器來執行交易。

* 儲存 (storage)
儲存元件用於將已確定的交易區塊及其執行結果持續地存在到本機數據庫。

* 狀態同步裝置(State synchronizer)
節點利用其狀態同步裝置元件來「趕上」區塊鏈的最新狀態和維持最新狀態。

### 2024.09.19
昨天參與了共學的線上活動，了解到Aptos的代幣標準，及合約相關的編寫。

```
module my_addr::fungible_asset_example {
    use aptos_framework::fungible_asset::{Self, MintRef, TransferRef, BurnRef, Metadata, FungibleAsset};
    use aptos_framework::object::{Self, Object};
    use aptos_framework::primary_fungible_store;
    use std::error;
    use std::signer;
    use std::string::utf8;
    use std::option;
  
  const ASSET_SYMBOL: vector<u8> = b"FA";
 
	// Make sure the `signer` you pass in is an address you own.
	// Otherwise you will lose access to the Fungible Asset after creation.
  entry fun init_module(admin: &signer) {
    // Creates a non-deletable object with a named address based on our ASSET_SYMBOL
    let constructor_ref = &object::create_named_object(admin, ASSET_SYMBOL);
    
    // Create the FA's Metadata with your name, symbol, icon, etc.
    primary_fungible_store::create_primary_store_enabled_fungible_asset(
        constructor_ref,
        option::none(),
        utf8(b"FA Coin"), /* name */
        utf8(ASSET_SYMBOL), /* symbol */
        8, /* decimals */
        utf8(b"http://example.com/favicon.ico"), /* icon */
        utf8(b"http://example.com"), /* project */
    );
    
    // Generate the MintRef for this object
    // Used by fungible_asset::mint() and fungible_asset::mint_to()
		let mint_ref = fungible_asset::generate_mint_ref(&constructor_ref)
		
    // Generate the TransferRef for this object
    // Used by fungible_asset::set_frozen_flag(), fungible_asset::withdraw_with_ref(),  
    // fungible_asset::deposit_with_ref(), and fungible_asset::transfer_with_ref().
		let transfer_ref = fungible_asset::generate_transfer_ref(&constructor_ref)
		
    // Generate the BurnRef for this object
    // Used by fungible_asset::burn() and fungible_asset::burn_from()
		let burn_ref = fungible_asset::generate_burn_ref(&constructor_ref)
    
    // Add any other logic required for your use case.
    // ...
  }
}
```

### 2024.09.20
實測了Aptos中的 "Connect" 功能，基本上是很容易使用，使用過程沒有遇到困難。

首先需要安裝Aptos Wallet Adapter
```
pnpm add @aptos-labs/wallet-adapter-react
```

AptosConnect is auto-added to the package, so no need to add it as a plugin. If you want to show other wallets you can include them in the plugins.

```
  import { AptosWalletAdapterProvider } from "@aptos-labs/wallet-adapter-react";
 
  const wallets = [new AnyOtherWalletYouWantToInclude()];
  <AptosWalletAdapterProvider
    plugins={wallets}
    autoConnect={true}
    optInWallets={["Petra"]}
    dappConfig={{ network: network.MAINNET, aptosConnectDappId: "dapp-id" }}>
      <App >
  </AptosWalletAdapterProvider>
```

Send a transaction
```
const { signAndSubmitTransaction } = useWallet();
 
const transaction: InputTransactionData = {
  data: {
    function: '0x1::coin::transfer',
    typeArguments: [APTOS_COIN],
    functionArguments: [account.address, 1],
  },
};
 
const txn = await signAndSubmitTransaction(transaction);
```
### 2024.09.21
The Digital Asset (DA) standard is a modern Non-Fungible Token (NFT) standard for Aptos. NFTs represent unique assets on-chain, and are stored in collections. These NFTs can be customized to later be transferred, soulbound, burned, mutated, or customized via your own smart contracts.

* smart contract testing:
```
use aptos_token_objects::collection;
use std::option::{Self, Option};
 
public entry fun create_collection(creator: &signer) {
    let max_supply = 1000;
    let royalty = option::none();
    
    // Maximum supply cannot be changed after collection creation
    collection::create_fixed_collection(
        creator,
        "My Collection Description",
        max_supply,
        "My Collection",
        royalty,
        "https://mycollection.com",
    );
}
```

### 2024.09.22
繼續學習Aptos Digital Asset (DA) Standard，
如果需要用到無限制的供應，可以用這個集合 - collection::create_unlimited_collection:

```
use std::option::{Self, Option};
 
public entry fun create_collection(creator: &signer) {
    let royalty = option::none();
 
    collection::create_unlimited_collection(
        creator,
        "My Collection Description",
        "My Collection",
        royalty,
        "https://mycollection.com",
    );
}
```

另外，也可以自定義一個集合，
Since each Collection is a Move Object, you can customize it by generating permissions called Refs. Each Ref allows you to modify an aspect of the Object later on. Beyond the normal Object Refs, Collections can also get a MutatorRef by calling get_mutator_ref like so:

```
use std::option::{Self, Option};
 
public entry fun create_collection(creator: &signer) {
    let royalty = option::none();
    let collection_constructor_ref = &collection::create_unlimited_collection(
        creator,
        "My Collection Description",
        "My Collection",
        royalty,
        "https://mycollection.com",
    );
    let mutator_ref = collection::get_mutator_ref(collection_constructor_ref);
    // Store the mutator ref somewhere safe
}
```
### 2024.09.23
閱讀Aptos Move v2、Move Objects
* Move Objects
https://aptos.dev/en/build/smart-contracts/objects
```
module my_addr::object_playground {
  use std::signer;
  use std::string::{Self, String};
  use aptos_framework::object::{Self, ObjectCore};
  
  struct MyStruct1 {
    message: String,
  }
  
  struct MyStruct2 {
    message: String,
  }
 
  entry fun create_and_transfer(caller: &signer, destination: address) {
    // Create object
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_object(caller_address);
    let object_signer = object::generate_signer(constructor_ref);
    
    // Set up the object by creating 2 resources in it
    move_to(&object_signer, MyStruct1 {
      message: string::utf8(b"hello")
    });
    move_to(&object_signer, MyStruct2 {
      message: string::utf8(b"world")
    });
 
    // Transfer to destination
    let object = object::object_from_constructor_ref<ObjectCore>(
      &constructor_ref
    );
    object::transfer(caller, object, destination);
  }
}
```

### 2024.09.24
學習 marketplace 的合約
```
address marketplace {
/// Defines a single listing or an item for sale or auction. This is an escrow service that
/// enables two parties to exchange one asset for another.
/// Each listing has the following properties:
/// * FeeSchedule specifying payment flows
/// * Owner or the person that can end the sale or auction
/// * Optional buy it now price
/// * Ending time at which point it can be claimed by the highest bidder or left in escrow.
/// * For auctions, the minimum bid rate and optional increase in duration of the auction if bids
///   are made toward the end of the auction.
module coin_listing {
    use std::error;
    use std::option::{Self, Option};
    use std::signer;
    use std::string::{Self, String};
    use aptos_std::math64;

    use aptos_framework::coin::{Self, Coin};
    use aptos_framework::object::{Self, ConstructorRef, Object, ObjectCore};
    use aptos_framework::timestamp;

    use marketplace::events;
    use marketplace::fee_schedule::{Self, FeeSchedule};
    use marketplace::listing::{Self, Listing};
    use aptos_framework::aptos_account;

    #[test_only]
    friend marketplace::listing_tests;

    /// There exists no listing.
    const ENO_LISTING: u64 = 1;
    /// This is an auction without buy it now.
    const ENO_BUY_IT_NOW: u64 = 2;
    /// The proposed bid is insufficient.
    const EBID_TOO_LOW: u64 = 3;
    /// The auction has not yet ended.
    const EAUCTION_NOT_ENDED: u64 = 4;
    /// The auction has already ended.
    const EAUCTION_ENDED: u64 = 5;
    /// The entity is not the seller.
    const ENOT_SELLER: u64 = 6;

    // Core data structures
    const FIXED_PRICE_TYPE: vector<u8> = b"fixed price";
    const AUCTION_TYPE: vector<u8> = b"auction";

    #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
    /// Fixed-price market place listing.
    struct FixedPriceListing<phantom CoinType> has key {
        /// The price to purchase the item up for listing.
        price: u64,
    }

    #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
    /// An auction-based listing with optional buy it now semantics.
    struct AuctionListing<phantom CoinType> has key {
        /// Starting bid price.
        starting_bid: u64,
        /// Price increment from the current bid.
        bid_increment: u64,
        /// Current bid, if one exists.
        current_bid: Option<Bid<CoinType>>,
        /// Auction end time in Unix time as seconds.
        auction_end_time: u64,
        /// If a bid time comes within this amount of time before the end bid, extend the end bid
        /// to the current time plus this amount.
        minimum_bid_time_before_end: u64,
        /// Buy it now price, ends auction immediately.
        buy_it_now_price: Option<u64>,
    }

    /// Represents a single bid within this auction house.
    struct Bid<phantom CoinType> has store {
        bidder: address,
        coins: Coin<CoinType>,
    }

    // Init functions

    public entry fun init_fixed_price<CoinType>(
        seller: &signer,
        object: Object<ObjectCore>,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        price: u64,
    ) {
        init_fixed_price_internal<CoinType>(seller, object, fee_schedule, start_time, price);
    }

    public(friend) fun init_fixed_price_internal<CoinType>(
        seller: &signer,
        object: Object<ObjectCore>,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        price: u64,
    ): Object<Listing> {
        let (listing_signer, constructor_ref) = init<CoinType>(
            seller,
            object,
            fee_schedule,
            start_time,
            price,
        );

        let fixed_price_listing = FixedPriceListing<CoinType> {
            price,
        };
        move_to(&listing_signer, fixed_price_listing);

        let listing = object::object_from_constructor_ref(&constructor_ref);

        events::emit_listing_placed(
            fee_schedule,
            string::utf8(FIXED_PRICE_TYPE),
            object::object_address(&listing),
            signer::address_of(seller),
            price,
            listing::token_metadata(listing),
        );

        listing
    }

    public entry fun init_fixed_price_for_tokenv1<CoinType>(
        seller: &signer,
        token_creator: address,
        token_collection: String,
        token_name: String,
        token_property_version: u64,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        price: u64,
    ) {
        init_fixed_price_for_tokenv1_internal<CoinType>(
            seller,
            token_creator,
            token_collection,
            token_name,
            token_property_version,
            fee_schedule,
            start_time,
            price,
        );
    }

    public(friend) fun init_fixed_price_for_tokenv1_internal<CoinType>(
        seller: &signer,
        token_creator: address,
        token_collection: String,
        token_name: String,
        token_property_version: u64,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        price: u64,
    ): Object<Listing> {
        let object = listing::create_tokenv1_container(
            seller,
            token_creator,
            token_collection,
            token_name,
            token_property_version,
        );
        init_fixed_price_internal<CoinType>(
            seller,
            object::convert(object),
            fee_schedule,
            start_time,
            price,
        )
    }

    public entry fun init_auction<CoinType>(
        seller: &signer,
        object: Object<ObjectCore>,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        starting_bid: u64,
        bid_increment: u64,
        auction_end_time: u64,
        minimum_bid_time_before_end: u64,
        buy_it_now_price: Option<u64>,
    ) {
        init_auction_internal<CoinType>(
            seller,
            object,
            fee_schedule,
            start_time,
            starting_bid,
            bid_increment,
            auction_end_time,
            minimum_bid_time_before_end,
            buy_it_now_price,
        );
    }

    public(friend) fun init_auction_internal<CoinType>(
        seller: &signer,
        object: Object<ObjectCore>,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        starting_bid: u64,
        bid_increment: u64,
        auction_end_time: u64,
        minimum_bid_time_before_end: u64,
        buy_it_now_price: Option<u64>,
    ): Object<Listing> {
        let (listing_signer, constructor_ref) = init<CoinType>(
            seller,
            object,
            fee_schedule,
            start_time,
            starting_bid,
        );

        let auction_listing = AuctionListing<CoinType> {
            starting_bid,
            bid_increment,
            current_bid: option::none(),
            auction_end_time,
            minimum_bid_time_before_end,
            buy_it_now_price,
        };
        move_to(&listing_signer, auction_listing);
        let listing = object::object_from_constructor_ref(&constructor_ref);

        events::emit_listing_placed(
            fee_schedule,
            string::utf8(AUCTION_TYPE),
            object::object_address(&listing),
            signer::address_of(seller),
            starting_bid,
            listing::token_metadata(listing),
        );

        listing
    }

    public entry fun init_auction_for_tokenv1<CoinType>(
        seller: &signer,
        token_creator: address,
        token_collection: String,
        token_name: String,
        token_property_version: u64,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        starting_bid: u64,
        bid_increment: u64,
        auction_end_time: u64,
        minimum_bid_time_before_end: u64,
        buy_it_now_price: Option<u64>,
    ) {
        init_auction_for_tokenv1_internal<CoinType>(
            seller,
            token_creator,
            token_collection,
            token_name,
            token_property_version,
            fee_schedule,
            start_time,
            starting_bid,
            bid_increment,
            auction_end_time,
            minimum_bid_time_before_end,
            buy_it_now_price,
        );
    }

    public(friend) fun init_auction_for_tokenv1_internal<CoinType>(
        seller: &signer,
        token_creator: address,
        token_collection: String,
        token_name: String,
        token_property_version: u64,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        starting_bid: u64,
        bid_increment: u64,
        auction_end_time: u64,
        minimum_bid_time_before_end: u64,
        buy_it_now_price: Option<u64>,
    ): Object<Listing> {
        let object = listing::create_tokenv1_container(
            seller,
            token_creator,
            token_collection,
            token_name,
            token_property_version,
        );
        init_auction_internal<CoinType>(
            seller,
            object::convert(object),
            fee_schedule,
            start_time,
            starting_bid,
            bid_increment,
            auction_end_time,
            minimum_bid_time_before_end,
            buy_it_now_price,
        )
    }

    inline fun init<CoinType>(
        seller: &signer,
        object: Object<ObjectCore>,
        fee_schedule: Object<FeeSchedule>,
        start_time: u64,
        initial_price: u64,
    ): (signer, ConstructorRef) {
        aptos_account::transfer_coins<CoinType>(
            seller,
            fee_schedule::fee_address(fee_schedule),
            fee_schedule::listing_fee(fee_schedule, initial_price),
        );

        listing::init(seller, object, fee_schedule, start_time)
    }

    // Mutators

    /// Purchase outright an item from an auction or a fixed price listing.
    public entry fun purchase<CoinType>(
        purchaser: &signer,
        object: Object<Listing>,
    ) acquires AuctionListing, FixedPriceListing {
        let listing_addr = listing::assert_started(&object);

        // Retrieve the purchase price if the auction has buy it now or this is a fixed listing.
        let (price, type) = if (exists<AuctionListing<CoinType>>(listing_addr)) {
            let AuctionListing {
                starting_bid: _,
                bid_increment: _,
                current_bid,
                auction_end_time,
                minimum_bid_time_before_end: _,
                buy_it_now_price,
            } = move_from<AuctionListing<CoinType>>(listing_addr);

            let now = timestamp::now_seconds();
            assert!(now < auction_end_time, error::invalid_state(EAUCTION_ENDED));

            assert!(option::is_some(&buy_it_now_price), error::invalid_argument(ENO_BUY_IT_NOW));
            if (option::is_some(&current_bid)) {
                let Bid { bidder, coins } = option::destroy_some(current_bid);
                aptos_account::deposit_coins(bidder, coins);
            } else {
                option::destroy_none(current_bid);
            };
            (option::destroy_some(buy_it_now_price), string::utf8(AUCTION_TYPE))
        } else if (exists<FixedPriceListing<CoinType>>(listing_addr)) {
            let FixedPriceListing {
                price,
            } = move_from<FixedPriceListing<CoinType>>(listing_addr);
            (price, string::utf8(FIXED_PRICE_TYPE))
        } else {
            // This should just be an abort but the compiler errors.
            abort (error::not_found(ENO_LISTING))
        };

        let coins = coin::withdraw<CoinType>(purchaser, price);

        complete_purchase(purchaser, signer::address_of(purchaser), object, coins, type)
    }

    /// End a fixed price listing early.
    public entry fun end_fixed_price<CoinType>(
        seller: &signer,
        object: Object<Listing>,
    ) acquires FixedPriceListing {
        let token_metadata = listing::token_metadata(object);

        let expected_seller_addr = signer::address_of(seller);
        let (actual_seller_addr, fee_schedule) = listing::close(seller, object, expected_seller_addr);
        assert!(expected_seller_addr == actual_seller_addr, error::permission_denied(ENOT_SELLER));

        let listing_addr = object::object_address(&object);
        assert!(exists<FixedPriceListing<CoinType>>(listing_addr), error::not_found(ENO_LISTING));
        let FixedPriceListing {
            price,
        } = move_from<FixedPriceListing<CoinType>>(listing_addr);

        events::emit_listing_canceled(
            fee_schedule,
            string::utf8(FIXED_PRICE_TYPE),
            listing_addr,
            actual_seller_addr,
            price,
            token_metadata,
        );
    }

    /// Make a bid on a listing. If the listing comes in near the end of an auction, the auction
    /// may be extended to give at least minimum_bid_time_before_end time remaining in the auction.
    public entry fun bid<CoinType>(
        bidder: &signer,
        object: Object<Listing>,
        bid_amount: u64,
    ) acquires AuctionListing {
        let listing_addr = listing::assert_started(&object);
        assert!(exists<AuctionListing<CoinType>>(listing_addr), error::not_found(ENO_LISTING));
        let auction_listing = borrow_global_mut<AuctionListing<CoinType>>(listing_addr);

        let now = timestamp::now_seconds();
        assert!(now < auction_listing.auction_end_time, error::invalid_state(EAUCTION_ENDED));

        let (previous_bidder, previous_bid, minimum_bid) = if (option::is_some(&auction_listing.current_bid)) {
            let Bid { bidder, coins } = option::extract(&mut auction_listing.current_bid);
            let current_bid = coin::value(&coins);
            aptos_account::deposit_coins(bidder, coins);
            (option::some(bidder), option::some(current_bid), current_bid + auction_listing.bid_increment)
        } else {
            (option::none(), option::none(), auction_listing.starting_bid)
        };

        assert!(bid_amount >= minimum_bid, error::invalid_argument(EBID_TOO_LOW));
        let coins = coin::withdraw<CoinType>(bidder, bid_amount);
        let bid = Bid {
            bidder: signer::address_of(bidder),
            coins,
        };
        option::fill(&mut auction_listing.current_bid, bid);

        let fee_schedule = listing::fee_schedule(object);
        aptos_account::transfer_coins<CoinType>(
            bidder,
            fee_schedule::fee_address(fee_schedule),
            fee_schedule::bidding_fee(fee_schedule, bid_amount),
        );

        let now = timestamp::now_seconds();
        let current_end_time = auction_listing.auction_end_time;
        let minimum_end_time = now + auction_listing.minimum_bid_time_before_end;

        if (current_end_time < minimum_end_time) {
            auction_listing.auction_end_time = minimum_end_time
        };

        events::emit_bid_event(
            fee_schedule,
            listing_addr,
            signer::address_of(bidder),
            bid_amount,
            auction_listing.auction_end_time,
            previous_bidder,
            previous_bid,
            current_end_time,
            listing::token_metadata(object),
        );
    }

    /// Once the current time has elapsed the auctions run time, allow the auction to be settled by
    /// distributing out the asset to the winner or the auction seller if no one bid as well as
    /// giving any fees to the marketplace that hosted the auction.
    public entry fun complete_auction<CoinType>(
        completer: &signer,
        object: Object<Listing>,
    ) acquires AuctionListing {
        let listing_addr = listing::assert_started(&object);
        assert!(exists<AuctionListing<CoinType>>(listing_addr), error::not_found(ENO_LISTING));

        let AuctionListing {
            starting_bid: _,
            bid_increment: _,
            current_bid,
            auction_end_time,
            minimum_bid_time_before_end: _,
            buy_it_now_price: _,
        } = move_from<AuctionListing<CoinType>>(listing_addr);

        let now = timestamp::now_seconds();
        assert!(auction_end_time <= now, error::invalid_state(EAUCTION_NOT_ENDED));

        let seller = listing::seller(object);

        let (purchaser, coins) = if (option::is_some(&current_bid)) {
            let Bid { bidder, coins } = option::destroy_some(current_bid);
            (bidder, coins)
        } else {
            option::destroy_none(current_bid);
            (seller, coin::zero<CoinType>())
        };

        complete_purchase(completer, purchaser, object, coins, string::utf8(AUCTION_TYPE));
    }

    inline fun complete_purchase<CoinType>(
        completer: &signer,
        purchaser_addr: address,
        object: Object<Listing>,
        coins: Coin<CoinType>,
        type: String,
    ) {
        let token_metadata = listing::token_metadata(object);

        let price = coin::value(&coins);
        let (royalty_addr, royalty_charge) = listing::compute_royalty(object, price);
        let (seller, fee_schedule) = listing::close(completer, object, purchaser_addr);

        // Take royalty first
        if (royalty_charge != 0) {
            let royalty = coin::extract(&mut coins, royalty_charge);
            aptos_account::deposit_coins(royalty_addr, royalty);
        };

        // Take commission of what's left, creators get paid first
        let commission_charge = fee_schedule::commission(fee_schedule, price);
        let actual_commission_charge = math64::min(coin::value(&coins), commission_charge);
        let commission = coin::extract(&mut coins, actual_commission_charge);
        aptos_account::deposit_coins(fee_schedule::fee_address(fee_schedule), commission);

        // Seller gets what is left
        aptos_account::deposit_coins(seller, coins);

        events::emit_listing_filled(
            fee_schedule,
            type,
            object::object_address(&object),
            seller,
            purchaser_addr,
            price,
            commission_charge,
            royalty_charge,
            token_metadata,
        );
    }

    // View

    #[view]
    public fun price<CoinType>(
        object: Object<Listing>,
    ): Option<u64> acquires AuctionListing, FixedPriceListing {
        let listing_addr = object::object_address(&object);
        if (exists<FixedPriceListing<CoinType>>(listing_addr)) {
            let fixed_price = borrow_global<FixedPriceListing<CoinType>>(listing_addr);
            option::some(fixed_price.price)
        } else if (exists<AuctionListing<CoinType>>(listing_addr)) {
            borrow_global<AuctionListing<CoinType>>(listing_addr).buy_it_now_price
        } else {
            // This should just be an abort but the compiler errors.
            assert!(false, error::not_found(ENO_LISTING));
            option::none()
        }
    }

    #[view]
    public fun is_auction<CoinType>(object: Object<Listing>): bool {
        let obj_addr = object::object_address(&object);
        exists<AuctionListing<CoinType>>(obj_addr)
    }

    #[view]
    public fun starting_bid<CoinType>(object: Object<Listing>): u64 acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        auction.starting_bid
    }

    #[view]
    public fun bid_increment<CoinType>(object: Object<Listing>): u64 acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        auction.bid_increment
    }

    #[view]
    public fun auction_end_time<CoinType>(object: Object<Listing>): u64 acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        auction.auction_end_time
    }

    #[view]
    public fun minimum_bid_time_before_end<CoinType>(
        object: Object<Listing>,
    ): u64 acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        auction.minimum_bid_time_before_end
    }

    #[view]
    public fun current_bidder<CoinType>(
        object: Object<Listing>,
    ): Option<address> acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        if (option::is_some(&auction.current_bid)) {
            option::some(option::borrow(&auction.current_bid).bidder)
        } else {
            option::none()
        }
    }

    #[view]
    public fun current_amount<CoinType>(
        object: Object<Listing>,
    ): Option<u64> acquires AuctionListing {
        let auction = borrow_auction<CoinType>(object);
        if (option::is_some(&auction.current_bid)) {
            let coins = &option::borrow(&auction.current_bid).coins;
            option::some(coin::value(coins))
        } else {
            option::none()
        }
    }

    inline fun borrow_auction<CoinType>(
        object: Object<Listing>,
    ): &AuctionListing<CoinType> acquires AuctionListing {
        let obj_addr = object::object_address(&object);
        assert!(exists<AuctionListing<CoinType>>(obj_addr), error::not_found(ENO_LISTING));
        borrow_global<AuctionListing<CoinType>>(obj_addr)
    }

    inline fun borrow_fixed_price<CoinType>(
        object: Object<Listing>,
    ): &FixedPriceListing<CoinType> acquires FixedPriceListing {
        let obj_addr = object::object_address(&object);
        assert!(exists<FixedPriceListing<CoinType>>(obj_addr), error::not_found(ENO_LISTING));
        borrow_global<FixedPriceListing<CoinType>>(obj_addr)
    }
}

// Tests

#[test_only]
module listing_tests {
    use std::option;

    use aptos_framework::aptos_coin::AptosCoin;
    use aptos_framework::coin;
    use aptos_framework::object::{Self, Object};
    use aptos_framework::timestamp;

    use aptos_token::token as tokenv1;

    use aptos_token_objects::token::Token;
    use marketplace::test_utils::{mint_tokenv2_with_collection_royalty, mint_tokenv1_additional_royalty, mint_tokenv1};

    use marketplace::coin_listing;
    use marketplace::fee_schedule::FeeSchedule;
    use marketplace::listing::{Self, Listing};
    use marketplace::test_utils;

    fun test_fixed_price(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, fee_schedule, listing) = fixed_price_listing(marketplace, seller);

        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
        assert!(listing::listed_object(listing) == object::convert(token), 0);
        assert!(listing::fee_schedule(listing) == fee_schedule, 0);
        assert!(coin_listing::price<AptosCoin>(listing) == option::some(500), 0);
        assert!(!coin_listing::is_auction<AptosCoin>(listing), 0);

        coin_listing::purchase<AptosCoin>(purchaser, listing);

        assert!(object::owner(token) == purchaser_addr, 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 6, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 10494, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9500, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_fixed_price_high_royalty(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);
        // TODO: add test that separates seller and creator
        let (_collection, additional_token) = mint_tokenv2_with_collection_royalty(seller, 100, 100);
        let (token, fee_schedule, listing) = fixed_price_listing_with_token(marketplace, seller, additional_token);

        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
        assert!(listing::listed_object(listing) == object::convert(token), 0);
        assert!(listing::fee_schedule(listing) == fee_schedule, 0);
        assert!(coin_listing::price<AptosCoin>(listing) == option::some(500), 0);
        assert!(!coin_listing::is_auction<AptosCoin>(listing), 0);

        coin_listing::purchase<AptosCoin>(purchaser, listing);

        assert!(object::owner(token) == purchaser_addr, 0);
        // Because royalty is 100, no commission is taken just the listing fee
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 10499, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9500, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_fixed_price_end(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, _purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, _fee_schedule, listing) = fixed_price_listing(marketplace, seller);

        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        coin_listing::end_fixed_price<AptosCoin>(seller, listing);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
        assert!(object::owner(token) == seller_addr, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_purchase(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, fee_schedule, listing) = auction_listing(marketplace, seller);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
        assert!(listing::listed_object(listing) == object::convert(token), 0);
        assert!(listing::fee_schedule(listing) == fee_schedule, 0);
        assert!(coin_listing::price<AptosCoin>(listing) == option::some(500), 0);
        assert!(coin_listing::is_auction<AptosCoin>(listing), 0);
        assert!(coin_listing::starting_bid<AptosCoin>(listing) == 100, 0);
        assert!(coin_listing::bid_increment<AptosCoin>(listing) == 50, 0);
        assert!(coin_listing::auction_end_time<AptosCoin>(listing) == timestamp::now_seconds() + 200, 0);
        assert!(coin_listing::minimum_bid_time_before_end<AptosCoin>(listing) == 150, 0);
        assert!(coin_listing::current_amount<AptosCoin>(listing) == option::none(), 0);
        assert!(coin_listing::current_bidder<AptosCoin>(listing) == option::none(), 0);

        coin_listing::purchase<AptosCoin>(purchaser, listing);

        assert!(object::owner(token) == purchaser_addr, 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 6, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 10494, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9500, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_bid_then_purchase(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);

        coin_listing::bid<AptosCoin>(seller, listing, 100);
        assert!(coin_listing::current_amount<AptosCoin>(listing) == option::some(100), 0);
        assert!(coin_listing::current_bidder<AptosCoin>(listing) == option::some(seller_addr), 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 3, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9897, 0);

        // Return the bid and insert a new bid
        coin_listing::bid<AptosCoin>(purchaser, listing, 150);
        assert!(coin_listing::current_amount<AptosCoin>(listing) == option::some(150), 0);
        assert!(coin_listing::current_bidder<AptosCoin>(listing) == option::some(purchaser_addr), 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 5, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9997, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9848, 0);

        // Return the bid and replace with a purchase
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(object::owner(token) == purchaser_addr, 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 10, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9498, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_bidding(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
        let end_time = timestamp::now_seconds() + 200;
        assert!(coin_listing::auction_end_time<AptosCoin>(listing) == end_time, 0);

        // Bid but do not affect end timing
        coin_listing::bid<AptosCoin>(seller, listing, 100);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 3, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9897, 0);
        assert!(coin_listing::auction_end_time<AptosCoin>(listing) == end_time, 0);

        // Return the bid and insert a new bid and affect end timing
        test_utils::increment_timestamp(150);
        coin_listing::bid<AptosCoin>(purchaser, listing, 150);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 5, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9997, 0);
        assert!(coin::balance<AptosCoin>(purchaser_addr) == 9848, 0);
        assert!(coin_listing::auction_end_time<AptosCoin>(listing) != end_time, 0);

        // End the auction as out of time
        test_utils::increment_timestamp(150);
        coin_listing::complete_auction<AptosCoin>(aptos_framework, listing);
        assert!(object::owner(token) == purchaser_addr, 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 6, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 10146, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_ended_auction_no_bid(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (marketplace_addr, seller_addr, _purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);

        test_utils::increment_timestamp(200);
        coin_listing::complete_auction<AptosCoin>(aptos_framework, listing);

        assert!(object::owner(token) == seller_addr, 0);
        assert!(coin::balance<AptosCoin>(marketplace_addr) == 1, 0);
        assert!(coin::balance<AptosCoin>(seller_addr) == 9999, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x30002, location = marketplace::listing)]
    fun test_not_started_fixed_price(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_fixed_price_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds() + 1,
            500,
        );

        coin_listing::purchase<AptosCoin>(purchaser, listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x30002, location = marketplace::listing)]
    fun test_not_started_auction(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_auction_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds() + 1,
            100,
            50,
            timestamp::now_seconds() + 200,
            150,
            option::some(500),
        );

        coin_listing::bid<AptosCoin>(purchaser, listing, 1000);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x30005, location = marketplace::coin_listing)]
    fun test_ended_auction_bid(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        test_utils::increment_timestamp(200);
        coin_listing::bid<AptosCoin>(purchaser, listing, 1000);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x30005, location = marketplace::coin_listing)]
    fun test_ended_auction_purchase(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        test_utils::increment_timestamp(200);
        coin_listing::purchase<AptosCoin>(purchaser, listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x10006, location = aptos_framework::coin)]
    fun test_not_enough_coin_fixed_price(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_fixed_price_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds(),
            100000,
        );

        coin_listing::purchase<AptosCoin>(purchaser, listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x10006, location = aptos_framework::coin)]
    fun test_not_enough_coin_auction_bid(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        coin_listing::bid<AptosCoin>(purchaser, listing, 100000);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x10003, location = marketplace::coin_listing)]
    fun test_bid_too_low(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = auction_listing(marketplace, seller);
        coin_listing::bid<AptosCoin>(purchaser, listing, 100);
        coin_listing::bid<AptosCoin>(purchaser, listing, 125);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x10006, location = aptos_framework::coin)]
    fun test_not_enough_coin_auction_purchase(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_auction_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds(),
            100,
            50,
            timestamp::now_seconds() + 200,
            150,
            option::some(50000),
        );

        coin_listing::purchase<AptosCoin>(purchaser, listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x60001, location = marketplace::coin_listing)]
    fun test_auction_view_on_fixed_price(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = fixed_price_listing(marketplace, seller);
        coin_listing::auction_end_time<AptosCoin>(listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x10002, location = marketplace::coin_listing)]
    fun test_purchase_on_auction_without_buy_it_now(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_auction_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds(),
            100,
            50,
            timestamp::now_seconds() + 200,
            150,
            option::none(),
        );

        coin_listing::purchase<AptosCoin>(purchaser, listing);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    #[expected_failure(abort_code = 0x50006, location = marketplace::coin_listing)]
    fun test_bad_fixed_price_end(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (_token, _fee_schedule, listing) = fixed_price_listing(marketplace, seller);
        coin_listing::end_fixed_price<AptosCoin>(purchaser, listing);
    }

    // Objects and TokenV2 stuff

    inline fun fixed_price_listing(
        marketplace: &signer,
        seller: &signer,
    ): (Object<Token>, Object<FeeSchedule>, Object<Listing>) {
        let token = test_utils::mint_tokenv2(seller);
        fixed_price_listing_with_token(marketplace, seller, token)
    }

    inline fun fixed_price_listing_with_token(
        marketplace: &signer,
        seller: &signer,
        token: Object<Token>
    ): (Object<Token>, Object<FeeSchedule>, Object<Listing>) {
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_fixed_price_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds(),
            500,
        );
        (token, fee_schedule, listing)
    }


    inline fun auction_listing(
        marketplace: &signer,
        seller: &signer,
    ): (Object<Token>, Object<FeeSchedule>, Object<Listing>) {
        let token = test_utils::mint_tokenv2(seller);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_auction_internal<AptosCoin>(
            seller,
            object::convert(token),
            fee_schedule,
            timestamp::now_seconds(),
            100,
            50,
            timestamp::now_seconds() + 200,
            150,
            option::some(500),
        );
        (token, fee_schedule, listing)
    }

    // TokenV1

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_fixed_price_for_token_v1(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);
        tokenv1::opt_in_direct_transfer(purchaser, true);

        let (token_id, _fee_schedule, listing) = fixed_price_listing_for_tokenv1(marketplace, seller);
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_fixed_price_for_token_v1_high_royalty(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);
        tokenv1::opt_in_direct_transfer(purchaser, true);
        let _token = mint_tokenv1(seller);
        let token_id = mint_tokenv1_additional_royalty(seller, 100, 100);

        let (_fee_schedule, listing) = fixed_price_listing_for_tokenv1_with_token(marketplace, seller, &token_id);
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
        // TODO balance checks
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_fixed_price_for_token_v1_bad_royalty(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);
        tokenv1::opt_in_direct_transfer(purchaser, true);
        let _token = mint_tokenv1(seller);
        let token_id = mint_tokenv1_additional_royalty(seller, 0, 0);

        let (_fee_schedule, listing) = fixed_price_listing_for_tokenv1_with_token(marketplace, seller, &token_id);
        // This should not fail, and no royalty is taken
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
        // TODO balance checks
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_purchase_for_tokenv1(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);
        tokenv1::opt_in_direct_transfer(purchaser, true);

        let (token_id, _fee_schedule, listing) = auction_listing_for_tokenv1(marketplace, seller);
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_purchase_for_tokenv1_without_direct_transfer(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token_id, _fee_schedule, listing) = auction_listing_for_tokenv1(marketplace, seller);
        coin_listing::purchase<AptosCoin>(purchaser, listing);
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
    }

    #[test(aptos_framework = @0x1, marketplace = @0x111, seller = @0x222, purchaser = @0x333)]
    fun test_auction_win_for_tokenv1_without_direct_transfer_and_non_winner_completer(
        aptos_framework: &signer,
        marketplace: &signer,
        seller: &signer,
        purchaser: &signer,
    ) {
        let (_marketplace_addr, _seller_addr, purchaser_addr) =
            test_utils::setup(aptos_framework, marketplace, seller, purchaser);

        let (token_id, _fee_schedule, listing) = auction_listing_for_tokenv1(marketplace, seller);
        coin_listing::bid<AptosCoin>(purchaser, listing, 100);
        test_utils::increment_timestamp(1000);
        let token_object = listing::listed_object(listing);
        coin_listing::complete_auction<AptosCoin>(aptos_framework, listing);
        listing::extract_tokenv1(purchaser, object::convert(token_object));
        assert!(tokenv1::balance_of(purchaser_addr, token_id) == 1, 0);
    }

    inline fun fixed_price_listing_for_tokenv1(
        marketplace: &signer,
        seller: &signer,
    ): (tokenv1::TokenId, Object<FeeSchedule>, Object<Listing>) {
        let token_id = test_utils::mint_tokenv1(seller);
        let (fee_schedule, listing) = fixed_price_listing_for_tokenv1_with_token(marketplace, seller, &token_id);
        (token_id, fee_schedule, listing)
    }

    inline fun fixed_price_listing_for_tokenv1_with_token(
        marketplace: &signer,
        seller: &signer,
        token_id: &tokenv1::TokenId,
    ): (Object<FeeSchedule>, Object<Listing>) {
        let (creator_addr, collection_name, token_name, property_version) =
            tokenv1::get_token_id_fields(token_id);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_fixed_price_for_tokenv1_internal<AptosCoin>(
            seller,
            creator_addr,
            collection_name,
            token_name,
            property_version,
            fee_schedule,
            timestamp::now_seconds(),
            500,
        );
        (fee_schedule, listing)
    }

    inline fun auction_listing_for_tokenv1(
        marketplace: &signer,
        seller: &signer,
    ): (tokenv1::TokenId, Object<FeeSchedule>, Object<Listing>) {
        let token_id = test_utils::mint_tokenv1(seller);
        let (creator_addr, collection_name, token_name, property_version) =
            tokenv1::get_token_id_fields(&token_id);
        let fee_schedule = test_utils::fee_schedule(marketplace);
        let listing = coin_listing::init_auction_for_tokenv1_internal<AptosCoin>(
            seller,
            creator_addr,
            collection_name,
            token_name,
            property_version,
            fee_schedule,
            timestamp::now_seconds(),
            100,
            50,
            timestamp::now_seconds() + 200,
            150,
            option::some(500),
        );
        (token_id, fee_schedule, listing)
    }
}
}
```

### 2024.09.25
最近看這個例子
https://github.com/aptos-labs/aptos-wallet-adapter/blob/ee95b8b3cd9eff0d7ebec1f3ee8017d222fe0e15/apps/nextjs-example/src/components/transactionFlows/MultiAgent.tsx#L18

在進行submit transaction:
```
const response = await submitTransaction({
        transaction: transactionToSubmit,
        senderAuthenticator: senderAuthenticator,
        additionalSignersAuthenticators: [secondarySignerAuthenticator],
      });
```
有error:
App.tsx:171 AptosApiError: Request to [Fullnode]: POST https://api.testnet.aptoslabs.com/v1/transactions failed with: {"message":"Invalid transaction: Type: Validation Code: INVALID_AUTH_KEY","error_code":"vm_error","vm_error_code":2}
    at y (core.ts:100:1)
    at async B (transactionSubmission.ts:262:1)
    at async submitTransaction (WalletProvider.tsx:145:1)
    at async onSubmitTransaction (App.tsx:163:1)

```
const response = await aptos.transaction.submit.multiAgent({
        transaction: transactionToSubmit,
        senderAuthenticator: senderAuthenticator,
        additionalSignersAuthenticators: [secondarySignerAuthenticator],
      });
```
也有Error:
App.tsx:173 AptosApiError: Request to [Fullnode]: POST https://api.testnet.aptoslabs.com/v1/transactions failed with: {"message":"Invalid transaction: Type: Validation Code: INVALID_AUTH_KEY","error_code":"vm_error","vm_error_code":2}
    at y (core.ts:100:1)
    at async B (transactionSubmission.ts:262:1)
    at async onSubmitTransaction (App.tsx:163:1)

### 2024.09.26
對於multisig的學習更深入的理解，在導師的指導和幫助下，順利完成一些官方的例子。
在學習過程中會遇到各種問題，特別是遇到舊的、錯誤的文檔，真的會讓人感到莫名其妙，要是沒有導師指導，
相信在短時間內是難以學習到。

### 2024.09.27
在使用Account.generate()時，如果要在testnet/devtest上進行單一測試時要大量使用，就要多留意。因為是有次數限制，如果到了某次數
就會被禁止使用。所以在testnet/devtest上，可以選擇使用自己創建的測試地址進行測試，盡量避免在短時間內使用多次Account.generate()。

### 2024.10.19
https://www.youtube.com/watch?v=_hZXS3Nc1d4
* NFT

* 在影片中，提到aptos move build --dev
怎樣用?

aptos 2.5.0
* aptos update

* aptos move compile --named-addresses contract=default --skip-fetch-latest-git-deps 
* aptos move deploy-object --address-name contract

Transaction submitted: https://explorer.aptoslabs.com/txn/0x7e38c78cd7ca227a50b24d80fa69b769d58361b8e12b05265551e2515b1a8742

<!-- Content_END -->
