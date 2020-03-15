DevNetにあるACI Sandbox LabをAnsibleで操作できるか確認してみました。

# Cisco ACI Guide

[https://docs.ansible.com/ansible/latest/scenario_guides/guide_aci.html](https://docs.ansible.com/ansible/latest/scenario_guides/guide_aci.html)

# 環境準備

virtualenv作ってAnsible諸々をインストールします。

```bash
% python3 -m venv venv
% . venv/bin/activate
(venv) % pip install ansible requests python-dateutil
```

# テスト用ベースPlaybook

ACI上に存在するテナント情報一覧を取得するPlaybookです。

```yaml
---
- name: ansible aci module test
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Gather tenants
      aci_tenant:
        hostname: host name
        username: user name
        password: user password
        validate_certs: no
        state: query
      register: r

    - debug: msg="{{ r }}"
```

# Playbook実行

実際にPlaybookを動作させて確認してみます。

## ALWAYS-ON環境

**使ったPlaybook**

```yaml
---
- name: ansible aci module test
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Gather tenants
      aci_tenant:
        hostname: sandboxapicdc.cisco.com
        username: admin
        password: ciscopsdt
        validate_certs: no
        state: query
      register: r

    - debug: msg="{{ r }}"
```

**実行結果**

```bash
(venv) % ansible-playbook main.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'

PLAY [ansible aci module test] ****************************************************************************

TASK [Query all users] ************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "current": [
            {
                "fvTenant": {
                    "attributes": {
                        "annotation": "",
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-infra",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-12T07:11:35.032+00:00",
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
                        "modTs": "2020-03-12T07:11:35.142+00:00",
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
                        "modTs": "2020-03-12T07:11:28.172+00:00",
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
                        "modTs": "2020-03-12T07:14:36.481+00:00",
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
                        "descr": "",
                        "dn": "uni/tn-SnV",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-12T07:14:36.863+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "SnV",
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
                        "descr": "",
                        "dn": "uni/tn-TESTA",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-12T08:28:57.703+00:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "TESTA",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "15374"
                    }
                }
            }
        ],
        "failed": false
    }
}

PLAY RECAP ************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

問題なし :)

## RESERVE環境

RESERVE環境はAnyConnectでVPNを接続した状態が必要です。
ただ、インストーラーがWindows/Macしかありません。
そのため、Macを汚すのは嫌だったので、ここでは仮想VMにWindows(ネットからダウンロードしたISOを使った)を入れてポートフォワードしてアクセスしてみました。

**ポートフォワード設定**

```powerhell
netsh interface portproxy add v4tov4 listenport=443 listenaddr=WindowsIP connectport=443 connectaddress=CiscoACIIP or RemoteDesktop
```

**connectaddress** はSandbox Labによりけり。

**使ったPlaybook**

```yaml
---
- name: ansible aci module test
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Gather tenants
      aci_tenant:
        hostname: ポートフォワードのIP
        username: RESERVEのCiscoACIのユーザー(SANDBOXページ参照)
        password: RESERVEのCiscoACIのユーザーのパスワード(SANDBOXページ参照)
        validate_certs: no
        state: query
      register: r

    - debug: msg="{{ r }}"
```

**実行結果**

```bash
(venv) aci % ansible-playbook main.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'

PLAY [ansible aci module test] ****************************************************************************

TASK [Gather tenants] *************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "current": [
            {
                "fvTenant": {
                    "attributes": {
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-infra",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2018-03-25T09:56:29.241-07:00",
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
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-mgmt",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2018-03-25T09:56:28.612-07:00",
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
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-common",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2018-03-25T09:56:25.733-07:00",
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
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-kubesbx00",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2018-08-09T02:13:56.093-07:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "kubesbx00",
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
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-kubesbx30",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2019-04-29T03:16:12.831-07:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "kubesbx30",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "14673"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-kubesbx31",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-08T04:06:53.964-07:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "kubesbx31",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "14673"
                    }
                }
            },
            {
                "fvTenant": {
                    "attributes": {
                        "childAction": "",
                        "descr": "",
                        "dn": "uni/tn-kubesbx06",
                        "extMngdBy": "",
                        "lcOwn": "local",
                        "modTs": "2020-03-11T12:22:07.008-07:00",
                        "monPolDn": "uni/tn-common/monepg-default",
                        "name": "kubesbx06",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": "",
                        "status": "",
                        "uid": "14673"
                    }
                }
            }
        ],
        "failed": false
    }
}

PLAY RECAP ************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

ポートフォワードでも問題ない :)
