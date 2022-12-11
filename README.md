# BBB-work2

### 作成者

kii (E-Team)

## [M-7] 攻撃者は、ERC721 をサポートしていない NFT (クリプトパンクなど) でショート プット オプション注文を作成でき、ユーザーは注文を実行できますが、オプションを行使することはできません。

### ■ カテゴリー

ERC721

### ■ 条件

ERC721 をサポートしていないNFTでプットポジションを作成した場合

### ■ ハッキングの詳細

safeTransferFrom 関数呼び出しが失敗するため、ユーザーはオプションを実行できない
オプションの有効期限が切れた後、攻撃者はプレミアムを獲得し、baseAsset を取り戻すことができる

### ■ 修正方法
  
NFTはホワイトリスト制にする

## [M-8] ERC20やERC721のSafetransfer

### ■ カテゴリー

ERC20，ERC721

### ■ 条件

SafeTransferの呼び出し

### ■ ハッキングの詳細

SafeTransferを呼んでいる箇所があるが，規格に非対応のトークンは引き出せなくなりGOXする

### ■ 修正方法

トークンは全てホワイトリスト制にする
そして操作時にリステッドトークンかどうかrequireすればよい

## [M-9] 契約はフラッシュローンプールとして無料で機能します

### ■ カテゴリー

フラッシュローン攻撃

### ■ 条件

メーカーとテイカーが同じアドレスで、さらにコントラクトであること

### ■ ハッキングの詳細

悪意のあるユーザーは、PuttyV2 コントラクトを利用して、資産に料金を支払うことなくフラッシュローンを実行して利益を上げることができます。

メーカーとテイカーが同じで、契約であるというシナリオを考えてみましょう

コントラクトは PuttyV2 を呼び出します。標準 ERC20 以外のカスタム ロジックを持つコントラクトへの参照をfillOrder含むロング コール注文。order.baseAssets注文は、PuttyV2 コントラクトが未払いであることも指定します(erc20Assetsと同様)。tokentokenAmounterc721Assets
↓
実行がhttps://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L324である場合、カスタム ロジックはコントラクト アドレスで実行できますorder.baseAsset。
↓
その後、悪意のあるコントラクトはexerciseショート コール ポジションを行使するためにコールします。この呼び出しは、ロジックを実行することにより、注文で指定されたアセットを悪意のあるコントラクトに転送します。_transferERC20sOut, _transferERC721sOut
↓
契約は、その資産を使用して他のプラットフォームで利益を上げます。その後、実行はhttps://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L324で続行されます。
↓
の最後に、コントラクトはロジックを実行して実行を終了することfillOrderにより、十分なアセットを PuttyV2 に戻します。_transferERC20sIn, _transferERC721sIn

### ■ 修正方法

## [M-10] PUTTY ポジション トークンは非 ERC721 レシーバーに発行される可能性があります

### ■ カテゴリー

ERC721

### ■ 条件

ERC721Receiver対応していないコントラクトにミントされたとき

### ■ ハッキングの詳細

条件をみたすと，GOXする

### ■ 修正方法

mint関数のロジックにはSafemintのようなものを採用すること
ただしリエントランシー対策はするべき

```
     /*  ~~~ EFFECTS ~~~ */
        // create opposite long/short position for taker
        bytes32 oppositeOrderHash = hashOppositeOrder(order);
        positionId = uint256(oppositeOrderHash);
        // save floorAssetTokenIds if filling a long call order
        if (order.isLong && order.isCall) {
            positionFloorAssetTokenIds[uint256(orderHash)] = floorAssetTokenIds;
        }
        // save the long position expiration
        positionExpirations[order.isLong ? uint256(orderHash) : positionId] = block.timestamp + order.duration;
        emit FilledOrder(orderHash, floorAssetTokenIds, order);
        /* ~~~ INTERACTIONS ~~~ */
        _safeMint(order.maker, uint256(orderHash));
        _saf
```

## [M-11] FEE変更がユーザーの同意なしに変更可能

### ■ カテゴリー



### ■ 条件

ユーザーがポジションを所持している最中にFEEが変更された場合

### ■ ハッキングの詳細

条件を満たすとき，FEEはストレージからの絶対参照であるからユーザーは変更後のFEEでの取引を余儀なくされる
↑「そんなにFEE高いなら取引しなかったのに」問題の発生が予想される

### ■ 修正方法

- 手数料を格納しOrder、注文が約定されたときに手数料が正しいことを確認して、構造体にハードコードされるようにします。
- タイムスタンプを追加します。これは完全には軽減されませんが、現在の設定よりは改善されます
- 過去の手数料と手数料変更のタイムスタンプをメモリ (配列など) に保持して、出金時に作成時の手数料を取得できるようにする

採用されたのは一つ目のよう．
↓l384等
https://github.com/outdoteth/putty-v2/pull/4/commits/15599a0104dc21d1b20a807109dc1719b6eb7942

