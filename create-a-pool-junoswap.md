# Junoswap上でPoolを作る

JunoswapとはJuno上にあるUniswap型（x*y=k)のAMMのDEX.
このDocの目的としては、最小限の説明でJunoswapに特定のトークンペアを持つPoolを作成する手順を書き記すことにある。


## 事前準備

### `必要なもの`

- junod - 主にTxを送信するために使うSoftware
- $JUNO - Txの手数料として支払うためのトークン
- $DENGAKU, $NATTO - Poolに入れるための資産

### `必要な工程`


1. 上場させるTokenのJuno上でのDenom(cw20にとってはContract address)を正確に把握する。

```markdown
DENGAKU = "juno176mpx9m30667vrkcrzxq8t3cpaws6gya7tgjq07xnvy79q9zwldss7fwv6"
NATTO = "juno1vn7z90efc3xju59ahrawchd6t70xsmmewuyzhms3vftfwcy928dqck2qnk"
```

2. 空のPoolを作成する
  
 JunoswapのContract codeをインスタンス化して、専用のcontractを作成する。
使うJunoswapのcontractのCode IDは16。

e.g.
```shell
junod tx wasm instantiate 16 \
'{"token1_denom": {"native": "ujuno"},"token2_denom": {"native": "ibc/CD78EE5B20682E5A61B4D96C9F4DC39361269B88A6B3462C26A18652F7A90A9A"},"lp_token_code_id": 1}' \
--label "JUNO SCRT Pool" \
--gas 10000000 \
--fees 250000ujuno \
--admin juno1pv88fxyhvyv2edwed72leupsuympnwvmsny4jtzspv9zw2dcrhkseh58kn \
--from <wallet-name>
```

上から順にコマンドの中身を説明すると、`16`と言うのはさっき述べた通り、JunoswapのContracnのcode IDで、一番上の行ではJunoswapのcontractをインスタンス化して新しいプールを作ることを宣言している。　　　
二行目では、プール内の資産に関する設定を記述している。`lp_token_code_id`とは、LP tokenを発行するためのContract Code Idを指定するためのもので、今現在だと`1`を指定したら良い。(使われるLP＿token用の[コントラクトのCode IDが1](https://www.mintscan.io/juno/wasm/contract/juno1pe2umwzqea0hvstzug2mm2ntrcxm336374954p5f6e52s5gf2zusklhqqr))   
`label`とはPoolを識別するための文字列で、`admin`は、コントラクトのStateを管理するアドレス。個人のアドレスの他コントラクトアドレスも指定することができるし、なにも指定しないこともできる。つまり、DENGAKU DAOのコントラクトアドレスも指定することが可能。

For this time:

```shell
junod tx wasm instantiate 16 \
'{"token1_denom": {"cw20": "juno176mpx9m30667vrkcrzxq8t3cpaws6gya7tgjq07xnvy79q9zwldss7fwv6"},"token2_denom": {"cw20": "juno1vn7z90efc3xju59ahrawchd6t70xsmmewuyzhms3vftfwcy928dqck2qnk"},"lp_token_code_id": 1}' \
--label <TODO> \
--gas 10000000 \
--fees 250000ujuno \
--admin <TODO> \
--from <wallet-name> \
--node https://rpc.juno.omniflix.co:443
```

1. コントラクトによる資産への干渉を許可する

2で作ったContractに対して＄DENGAKUと＄NATTOをApproveしてやる必要があります。
各コマンドは、   
DENGAKU:

```shell
junod tx wasm execute juno176mpx9m30667vrkcrzxq8t3cpaws6gya7tgjq07xnvy79q9zwldss7fwv6 '{"increase_allowance": {"amount": <approving_amount>, "spender": <junoswap_contract_address>}}' \
--from <wallet-name> \
--gas 900000 \
--gas-prices 0.0025ujuno \
--node https://rpc.juno.omniflix.co:443
```

NATTO:

```shell
junod tx wasm execute juno1vn7z90efc3xju59ahrawchd6t70xsmmewuyzhms3vftfwcy928dqck2qnk '{"increase_allowance": {"amount": <approving_amount>, "spender": <junoswap_contract_address>}}' \
--from <wallet-name> \
--gas 900000 \
--gas-prices 0.0025ujuno \
--node https://rpc.juno.omniflix.co:443
```

`approving_amount`は、LPへ入れたい量を指定したらよく、`junoswap_contract_address`には、2で得られたTx result内の情報を参照する必要があります。 　　
NOTE:コマンド内でのtoken量は、Decimalの桁数かけられた数字となります。

4. 資産を投入する

後は、実際に資産を投入するだけです。
Command：

```shell
junod tx wasm execute <junoswap_contract_address> \
'{"add_liquidity": {"token1_amount": <DENGAKU_amoount>, "min_liquidity": "0", "max_token2": <NATTO_amount>}}' \
--from <wallet-name> \
--gas 1500000 \
--amount <DENGAKU_amount>juno176mpx9m30667vrkcrzxq8t3cpaws6gya7tgjq07xnvy79q9zwldss7fwv6,<NATTO_amount>juno1vn7z90efc3xju59ahrawchd6t70xsmmewuyzhms3vftfwcy928dqck2qnk \
--fees 37500ujuno \
--node https://rpc.juno.omniflix.co:443
```

Liquidity(LP token）が0の場合は、投入されるToken2の量は Maxで指定した量になります。()


Ref:
https://github.com/CosmosContracts/junoswap-docs/blob/main/developers/create-an-amm-pool.md

## JunoSwap UIへ追加

In progress...
