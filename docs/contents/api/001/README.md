# ACI REST API基礎

# 初めに読んで理解すると良いドキュメント

* [Cisco APIC REST API ユーザ ガイド](https://www.cisco.com/c/ja_jp/td/docs/csm/policyautomationcntrllers/apppolicyinfracntrller-ap/pug/001/b_APIC_RESTful_API_User_Guide/b_APIC_RESTful_API_User_Guide_preface_00.html)
* [Cisco APIC REST API Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/2-x/rest_cfg/2_1_x/b_Cisco_APIC_REST_API_Configuration_Guide/b_Cisco_APIC_REST_API_Configuration_Guide_chapter_01.html)
* [Programmability and Automation with Cisco ACI](https://www.ciscolive.com/c/dam/r/ciscolive/apjc/docs/2016/pdf/DEVNET-1627.pdf)
* [APICのREST API入門](https://www.slideshare.net/ssuser442f5f/apicrest-api)

# APIリファレンス

[https://developer.cisco.com/site/apic-mim-ref-api/](https://developer.cisco.com/site/apic-mim-ref-api/)

上記ページはフレーム分割されていますが、フレームの高さの変更に限界があるため対象のClassを見たい時は右クリックして新しいタブで開くことをお勧めします。

# REST APIについて

## URL構造

URL構造については、上記ドキュメントを読めば大体イメージがつくと思いますが、簡単に説明します。  
ACIのREST APIのURL構造は以下のようになっています。

`http(s)://` `host:port` `/api` `/(mo|class)` `/(dn|classname)` `.(xml|json)` `?[options]`

|        項目        |                                    説明                                    |
|--------------------|----------------------------------------------------------------------------|
| `http(s)://`       | 接続するプロトコルです                                                     |
| `host:port`        | ACIのホスト名とポート番号です                                              |
| `/api`             | REST APIへアクセスするためのオペレーターです                               |
| `/(mo｜class)`     | アクセス先が `Managed Object(管理オブジェクト)` か `クラス` かを指定します |
| `/(dn｜classname)` | アクセ先が `dn` か `classname` かを指定します                              |
| `.(xml｜json)`     | XMLまたはJSONフォーマットのどちらかを選択します                            |
| `?[options]`       | 検索のクエリーなどを指定します                                             |

## moとclassについて

REST APIでアクセスする時にmo(Managed Object)またはclassを指定する必要があります。  
moとclassのざっくりとした違いは次のようなイメージです。(間違ってる可能性があります)

### moについて

オブジェクト単体を操作する時に使用するイメージです。  
例えば、単一のテナントを直接指定して情報を取得したり変更したりします。  
moを使う場合はdnを指定する必要があります。

### classについて

moの全体情報を取得したりする時に使用するイメージです。  
例えば、テナントを操作する場合、moは単体のオブジェクトを操作する時に使用しますが、classはテナント一覧を取得することができます。  
classを使う時はclassnameを指定する必要があります。

# Ansibleを使ったREST APIの例

## classで情報を取得する

ここではテナント情報一覧を取得してみます。  
テナント情報一覧を取得するには [fvTenant](https://pubhub.devnetcloud.com/media/apic-mim-ref-401/docs/MO-fvTenant.html) クラスを使用します。

```yaml
---
- name: cisco aci aci_rest module test
  hosts: localhost
  gather_facts: no
  vars:
    aci_hostname: sandboxapicdc.cisco.com
    aci_username: admin
    aci_password: ciscopsdt
    validate_certs: no
  tasks:
    - name: rest api demo
      aci_rest:
        host: "{{ aci_hostname }}"
        username: "{{ aci_username }}"
        password: "{{ aci_password }}"
        validate_certs: "{{ validate_certs }}"
        method: get
        path: /api/class/fvTenant.json
      register: rest_api_result

    - debug: var=rest_api_result
```

実行結果です。

```bash
# ansible-playbook sample-rest.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


PLAY [cisco aci aci_rest module test] **************************************************************************************************************************************

TASK [rest api demo] *******************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ***************************************************************************************************************************************************************
ok: [localhost] => {
    "rest_api_result": {
        "changed": false,
        "failed": false,
        "imdata": [
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-infra",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T03:16:07.270+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "infra",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "0"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-mgmt",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T03:17:26.805+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "mgmt",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "0"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-common",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T03:16:04.618+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "common",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "0"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-Heroes",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T03:21:05.187+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "Heroes",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "15374"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "TenantA",
                        "dn": "uni/tn-TenantA",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T07:16:59.528+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "TenantA",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "15374"
                    }
                }
            }
        ],
        "status": -1,
        "totalCount": 5
    }
}

PLAY RECAP *****************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

このような感じで、対象classの一覧が取得できます。  
フィルターをかける時は次のようにクエリを指定します。  
ここでは `query-target-filter` を使っています。  
他のフィルターについてはドキュメントを参照ください。

```yaml
---
- name: cisco aci aci_rest module test
  hosts: localhost
  gather_facts: no
  vars:
    aci_hostname: sandboxapicdc.cisco.com
    aci_username: admin
    aci_password: ciscopsdt
    validate_certs: no
  tasks:
    - name: rest api demo
      aci_rest:
        host: "{{ aci_hostname }}"
        username: "{{ aci_username }}"
        password: "{{ aci_password }}"
        validate_certs: "{{ validate_certs }}"
        method: get
        path: /api/class/fvTenant.json?query-target-filter=eq(fvTenant.name,"infra")
      register: rest_api_result

    - debug: var=rest_api_result
```

実行結果です。

```bash
# ansible-playbook sample-rest.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


PLAY [cisco aci aci_rest module test] **************************************************************************************************************************************

TASK [rest api demo] *******************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ***************************************************************************************************************************************************************
ok: [localhost] => {
    "rest_api_result": {
        "changed": false,
        "failed": false,
        "imdata": [
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-infra",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T03:16:07.270+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "infra",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "0"
                    }
                }
            }
        ],
        "status": -1,
        "totalCount": 1
    }
}

PLAY RECAP *****************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## テナントを作成する

ここでは、Managed Objectを使ってテナントを作成してみます。  
Managed Objectは `uni` を使用します。  
テナントを作成する時に指定するパラメーター(attributes)の詳細は、[classのドキュメント](https://pubhub.devnetcloud.com/media/apic-mim-ref-401/docs/MO-fvTenant.html) の下にある `Properties Summary` の `Defined in: fv:Tenant` を参照してください。

```yaml
---
- name: cisco aci aci_rest module test
  hosts: localhost
  gather_facts: no
  vars:
    aci_hostname: sandboxapicdc.cisco.com
    aci_username: admin
    aci_password: ciscopsdt
    validate_certs: no
  tasks:
    - name: rest api demo
      aci_rest:
        host: "{{ aci_hostname }}"
        username: "{{ aci_username }}"
        password: "{{ aci_password }}"
        validate_certs: "{{ validate_certs }}"
        method: post
        path: /api/mo/uni.json
        content:
          {
            "fvTenant": {
                "attributes": {
                    "name": "TenantA",
                    "descr": "Tenant A"
                },
            },
          }
      register: rest_api_result

    - debug: var=rest_api_result
```

実行結果です。

```bash
# ansible-playbook sample-rest.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


PLAY [cisco aci aci_rest module test] **************************************************************************************************************************************

TASK [rest api demo] *******************************************************************************************************************************************************
changed: [localhost]

TASK [debug] ***************************************************************************************************************************************************************
ok: [localhost] => {
    "rest_api_result": {
        "changed": true,
        "failed": false,
        "imdata": [
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "deleteNonPresent",
                        "descr": "Tenant A",
                        "dn": "uni/tn-TenantA",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T10:57:08.127+00:00",
                        "monPolDn": "",
                        "name": "TenantA",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "rn": "",
                        "status": "modified",
                        "uid": "15374"
                    }
                }
            }
        ],
        "status": -1,
        "totalCount": 1
    }
}

PLAY RECAP *****************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Managed Objectで情報を取得する

ここでは、classではなくManaged Objectを指定して上で作ったテナント情報を取得します。  
URIは　`/api/mo` の後はdnの `/uni/tn-TenantA` を指定します。

```yaml
---
- name: cisco aci aci_rest module test
  hosts: localhost
  gather_facts: no
  vars:
    aci_hostname: sandboxapicdc.cisco.com
    aci_username: admin
    aci_password: ciscopsdt
    validate_certs: no
  tasks:
    - name: rest api demo
      aci_rest:
        host: "{{ aci_hostname }}"
        username: "{{ aci_username }}"
        password: "{{ aci_password }}"
        validate_certs: "{{ validate_certs }}"
        method: get
        path: /api/mo/uni/tn-TenantA.json
      register: rest_api_result

    - debug: var=rest_api_result
```

実行結果です。

```bash
# ansible-playbook sample-rest.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


PLAY [cisco aci aci_rest module test] **************************************************************************************************************************************

TASK [rest api demo] *******************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ***************************************************************************************************************************************************************
ok: [localhost] => {
    "rest_api_result": {
        "changed": false,
        "failed": false,
        "imdata": [
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "Tenant A",
                        "dn": "uni/tn-TenantA",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-30T10:57:08.131+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "TenantA",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "15374"
                    }
                }
            }
        ],
        "status": -1,
        "totalCount": 1
    }
}

PLAY RECAP *****************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

