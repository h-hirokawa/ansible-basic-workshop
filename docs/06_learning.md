# Ansibleの学習と開発環境

## Ansibleの学習について
### 書籍
Ansible関連書籍は日本語の様々ありますが、2020年1月現在、学習を進めるに際しては新しい情報が含まれており、内容も手厚い以下の二冊がおすすめです。

* [Ansible構築・運用ガイドブック](https://book.mynavi.jp/ec/products/detail/id=112246)
* [Ansible実践ガイド 第三版](https://book.impress.co.jp/books/1118101094)

どちらも取り扱っている範囲に大きな違いはありませんが、前者の方がより初学者に親しみやすい内容であるように思われます。

### Web上学習環境

* https://www.katacoda.com/irixjp

Katacodaという無料でブラウザ上で実環境を動かしながら学習ができるサービスで、Red Hatのソリューションアーキテクトである中島倫明氏が日本語のAnsible学習カリキュラムを公開しています。
Ansible 101, Ansible 102, Ansible 103（Step 3まで）を学習すれば、Ansible Playbook実装に必要となる基礎知識を掴むことができるでしょう。

## 開発環境
### インストール方法
AnsibleはLinuxやmacOSを含めた多くのUNIX上で動作しますが、残念ながらWindowsをAnsibleのコントロールノードとすることはできません（AnsibleからWindowsを操作することは可能）。
Windowsがメインの開発環境となる人は、各種VMや[Windows Subsystem for Linux](https://docs.microsoft.com/ja-jp/learn/modules/get-started-with-windows-subsystem-for-linux/) (Windows 10必須)を使用してLinux環境を用意してください。

Ansibleの稼働にはPython2.7もしくは3.5以上が必要となりますが、Python 2.7は2019年を持ってEOLとなっていますので、デフォルトでPython3を使用するようになっているRHEL 8やCentOS 8環境を使うことを推奨します。
インターネットにさえ接続されていれば、以下のように簡単な手順でAnsibleをインストールすることができます。

#### RHEL 8の場合

```shell
$ sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
$ sudo yum install ansible
```
(2020年1月現在の最新版であるAnsible 2.9がインストールされる)

#### CentOS 8の場合

```shell
$ sudo yum install epel-release
$ sudo yum install ansible
```

インストールに関する詳細情報は公式ドキュメントの[インストール解説](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)を参照してください。

### エディタ
Playbookの開発に際しては、OS標準のテキストエディタなどではなく、コードハイライトや補完、文法チェックなどまでを行ってくれるものを使用するようにしましょう。
特にMicrosoft製のエディタ [Visual Studio Code(VS Code)](https://code.visualstudio.com) を使用することをお勧めします（Windowsの場合、User Installerを使うと管理者権限がなくともインストール可能なようです）。

下記のブログにてAnsible用プラグインの導入や活用方法について説明されていますので、参照ください。
https://tekunabe.hatenablog.jp/entry/2019/09/21/vscode_playbook

特に[Ansible](https://marketplace.visualstudio.com/items?itemName=vscoss.vscode-ansible)プラグインはAnsible Playbook開発時には導入必須と言えます。

VS Code自体の基本的な使い方は下記の記事などをご参照ください。
https://www.atmarkit.co.jp/ait/articles/1507/10/news028.html
