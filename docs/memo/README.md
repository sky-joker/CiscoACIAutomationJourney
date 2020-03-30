# Tips

## RHEL8でACI接続時に証明書エラーが発生する

RHEL8(CentOS8)にAnsibleをインストールしてACIに接続すると以下のエラーが発生する。

```
"msg": "Connection failed for https://xxx.xxx.xxx.xxx/api/aaaLogin.json. Request failed: <urlopen error [SSL: DH_KEY_TOO_SMALL] dh key too small (_ssl.c:897)>"
```

これは、RHEL8の暗号化ポリシーによって拒否されてる事象です。

[https://www.redhat.com/en/blog/consistent-security-crypto-policies-red-hat-enterprise-linux-8](https://www.redhat.com/en/blog/consistent-security-crypto-policies-red-hat-enterprise-linux-8)

例えば、DevNetのRESERVEのACI Simulatorで使われている証明書のキーの長さは256bitなのでポリシーにマッチせず拒否されます。  
一時的に回避する方法は、ポリシーを `LEGACY` に変更します。

```
# update-crypto-policies --set LEGACY
```

ただし、これは推奨されないのでポリシーに則った証明書を用意する必要があります。
