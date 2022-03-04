# 星火链SDK

为了方便开发者接入, 星火链提供了基于各种语言的SDK. 本章以Java SDK为例, 描述星火链SDK提供的主要功能和使用示例

## SDK 离线API

离线API主要是账户和密码学相关API, 不需要连接星火链网RPC接口也能工作. 主要接口如下:

### 账户生成

1. 接口 ` KeyPairEntity.getBidAndKeyPair`

1. 用途:

    用来生成一个星火链bid地址和对应私钥

1. 示例

    ```java
    import cn.bif.model.crypto.KeyPairEntity;

    KeyPairEntity keypair = KeyPairEntity.getBidAndKeyPair();
    String encAddress = keypair.getEncAddress();
    String encPublicKey = keypair.getEncPublicKey();
    String encPrivateKey = keypair.getEncPrivateKey();
    byte[] rawPublicKey = keypair.getRawPublicKey();
    byte[] rawPrivateKey = keypair.getRawPrivateKey();
    System.out.println(JsonUtils.toJSONString(keypair))
    ```


### 加密私钥生成keystore

1. 接口 `KeyStore.generateKeyStore`

1. 用途:

    用一个密码来加密保护私钥, 得到一个json表示的keystore, 对应密码不泄露的情况下, 可以公开保存.

1. 示例

    ```java
    package cn.bif.sdkSamples.encryption.example;

    import cn.bif.common.JsonUtils;
    import cn.bif.module.encryption.crypto.keystore.KeyStore;
    import cn.bif.module.encryption.crypto.keystore.entity.KeyStoreEty;

    import java.util.HashMap;
    import java.util.Map;


    public class TestCrypto {
        public static void main(String argv[]) {
            String encPrivateKey = "priSPKqru2zMzeb14XWxPNM1sassFeqyyUZotCAYcvCjhNof7t";
            String password = "bif8888";
            TestKeyStoreWithPrivateKey(encPrivateKey, password);

        }

        public static void TestKeyStoreWithPrivateKey(String encPrivateKey, String password) {
            try {
                int n = (int) Math.pow(2, 16);
                //生成keystore-1
                KeyStoreEty returEencPrivateKey = KeyStore.generateKeyStore(password,encPrivateKey, 2, 1, 1, n);
                System.out.println(JsonUtils.toJSONString(returEencPrivateKey));

                //生成keystore-2
                KeyStoreEty keyStore1 = KeyStore.generateKeyStore(password, encPrivateKey, n);
                System.out.println(JsonUtils.toJSONString(keyStore1));

                //从keystore反解出私钥
                String keyStoreStr="{\"address\":\"did:bid:efEScJgGPf54vfU8ciEjjugkJLB4xYzp\",\"aesctr_iv\":\"EEDDD37CEB6864030124142CEB081BCD\",\"cypher_text\":\"7274705F65388E30338A2D69AE2241DBABCF66550C0453BEE30CFA45F02E04D08FAC551B46171531CA067B6E85BC342F43C8\",\"scrypt_params\":{\"n\":16384,\"p\":1,\"r\":8,\"salt\":\"82D37133C13525EDE4EF19DCD692592FC1685B5EDAABA8C943EA2C1AD4596FB3\"},\"version\":2}";
                String privateKey = KeyStore.decipherKeyStore(password, JsonUtils.toJavaObject(keyStoreStr,KeyStoreEty.class));
                System.out.println(privateKey);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
    ```

## SDK 在线API

在线API主要用于向星火链上发出交易和查询合约, 需要初始化SDK连接后使用.

### 初始化SDK

1. 示例

    ```java
    import cn.bif.api.BIFSDK;

    BIFSDK sdk = BIFSDK.getInstance(SDK_INSTANCE_URL);   //SDK_INSTANCE_URL为星火链RPC地址
    ```

### 账户处理接口

1. 查询账户信息

    1. 接口 `BIFAccountGetInfoRequest`

    1. 用途:

        用来获取一个账户当前信息

    1. 示例
        ```java
        String accountAddress = "did:bid:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2";
        BIFAccountGetInfoRequest request = new BIFAccountGetInfoRequest();
        request.setAddress(accountAddress);
        // 调用getAccount接口
        BIFAccountGetInfoResponse response = sdk.getBIFAccountService().getAccount(request);

        if (response.getErrorCode() == 0) {
            System.out.println(JsonUtils.toJSONString(response.getResult()));
        } else {
            System.out.println(JsonUtils.toJSONString(response));
        }
        ```

1. 获取账户nonce

    1. 接口 `BIFAccountGetNonceRequest`

    1. 用途:

        用来获取一个账户当前nonce值, 有关nonce含义, 请参照星火链开发基础章节.

    1. 示例:
        ```java
        String accountAddress = "did:bid:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2";
        BIFAccountGetNonceRequest request = new BIFAccountGetNonceRequest();
        request.setAddress(accountAddress);
        BIFAccountGetNonceResponse response = sdk.getBIFAccountService().getNonce(request);
        if (0 == response.getErrorCode()) {
            System.out.println("Account nonce:" + response.getResult().getNonce());
        }else {
            System.out.println(JsonUtils.toJSONString(response));
        }
        ```

1. 获取账户余额

    1. 接口 `BIFAccountGetBalanceRequest`

    1. 用途:

        用来获取一个账户当前的XHT余额.

    1. 示例:

        ```java
        String accountAddress = "did:bid:efzE8AcDgWUeNbgujA5hK3oUeuG9k19b";
        BIFAccountGetBalanceRequest request = new BIFAccountGetBalanceRequest();
        request.setAddress(accountAddress);

        BIFAccountGetBalanceResponse response = sdk.getBIFAccountService().getAccountBalance(request);
        if (0 == response.getErrorCode()) {
            System.out.println("Gas balance：" + ToBaseUnit.ToGas(response.getResult().getBalance().toString()) + "Gas");
        }else {
            System.out.println(JsonUtils.toJSONString(response));
        }
        ```

### Block接口

1. 获取当前块高度

    1. 接口 `getBlockNumber`

    1. 用途:

        获取当前链上最新的Block号

    1. 示例:

        ```java
        BIFBlockGetNumberResponse response = sdk.getBIFBlockService().getBlockNumber();
        System.out.println(JsonUtils.toJSONString(response));
        ```

1. 获取指定块内的交易列表

    1. 接口 `getTransactions(request)`

    1. 用途:

        给定block号,获取该block内的交易列表信息

    1. 示例:

        ```java
        Long blockNumber = 1L;
        BIFBlockGetTransactionsRequest request = new BIFBlockGetTransactionsRequest();
        request.setBlockNumber(blockNumber);
        BIFBlockGetTransactionsResponse response = sdk.getBIFBlockService().getTransactions(request);
        if (0 == response.getErrorCode()) {
            System.out.println(JsonUtils.toJSONString(response.getResult()));
        } else {
            System.out.println(JsonUtils.toJSONString(response));
        }
        ```

1. 获取指定块的统计信息

    1. 接口 `getBlockInfo(request)`

    1. 用途:

        给定block号, 查询指定block的信息.

    1. 示例:

        ```java
         BIFBlockGetInfoRequest blockGetInfoRequest = new BIFBlockGetInfoRequest();
        blockGetInfoRequest.setBlockNumber(10L);
        BIFBlockGetInfoResponse lockGetInfoResponse = sdk.getBIFBlockService().getBlockInfo(blockGetInfoRequest);
        if (lockGetInfoResponse.getErrorCode() == 0) {
            BIFBlockGetInfoResult lockGetInfoResult = lockGetInfoResponse.getResult();
            System.out.println(JsonUtils.toJSONString(lockGetInfoResult));
        } else {
            System.out.println(JsonUtils.toJSONString(lockGetInfoResponse));
        }
        ```

1. 查询最新块的信息

    1. 接口 `getBlockLatestInfo`

    1. 用途:

        获取当前最新块的信息.

    1. 示例:

        ```java
        BIFBlockGetLatestInfoResponse lockGetLatestInfoResponse = sdk.getBIFBlockService().getBlockLatestInfo();
        if (lockGetLatestInfoResponse.getErrorCode() == 0) {
            BIFBlockGetLatestInfoResult lockGetLatestInfoResult = lockGetLatestInfoResponse.getResult();
            System.out.println(JsonUtils.toJSONString(lockGetLatestInfoResult));
        } else {
            System.out.println(JsonUtils.toJSONString(lockGetLatestInfoResponse));
        }
        ```