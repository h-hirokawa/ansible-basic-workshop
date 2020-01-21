# RoleによるPlaybookの共通部品化

今までの演習では、1ファイルでplaybookを作成して、全ての要素のそのファイルの中に定義してきました。もちろん、1ファイル構成のplaybookでも実用的なものを作ることは可能ですが、実際の運用で色々なplaybookを実装し活用するようになると以下のようなことをしたくなってくるでしょう。

* 作業工程を適切に分割して管理したい
* playbook間で共通で実施する作業を再利用可能にしたい
* 組織内で使用されているplaybookをコモディティ化したい

このようなニーズに応えてくれるのがroleです。
roleを作成する事でplaybookをパーツとして分解し、構造化されたディレクトリに格納し、role単位で独立して管理することができるようになります。

様々な作業単位で自動化をパーツ化して再利用可能な部品とすることができます。`Role` は完全にインベントリーと切り離されており、様々な playbook から呼び出して利用することが可能です。このような playbook の開発・管理方法を Ansible では [best practice](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) と呼んでいます。

この演習では、まずAnsible Galaxyで共有されている公開Roleについて紹介した後で、実際にPlaybookのRole化を進めていきます。

## Ansible Galaxyで公開Roleを見つける

Ansibleは公式に[Ansible Galaxy](https://galaxy.ansible.com/)というRole共有のためのプラットフォームを提供しており、世界中の人が作ったRoleを手軽に検索し、自由に使用することができます。

実際に、現時点での[ダウンロード数トップの人気Role](https://galaxy.ansible.com/search?keywords=&order_by=-download_count&deprecated=false&type=role&page=1)を見てみましょう。

並べてみてみると、各種ミドルウェア導入やOS内設定のRoleが広く使われているようです。もちろん、他にもたくさんのRoleが揃っていますので、何かのplaybookを作成する際にそのまま使えたり、実装のベースにできそうなRoleがないか、まずはAnsible Galaxyで探してみるというのは良い手でしょう。特に人気上位Roleの多くを提供している[geerlingguy](https://galaxy.ansible.com/geerlingguy)氏はクオリティの高いRoleを幅広く提供しており、Roleの作りとしても参考になるものが多いです。

注意点として、いずれのRoleも動作保証がされている訳ではなく、また同種のロールを色々な人が提供しているため、使用に際してはそのRoleが安心して使えるものかどうかの見極めが必要となります。DL数やユーザースコア(5点満点)も大きな指標になりますが、実際に中身を見たり、使い捨てできるVMやコンテナ環境で動かしてみて自身で確認を行うことが重要です。

## Role の構造
`Role` はディレクトリ内に予め決められた構成でファイルを配置して利用します。するとそのディレクトリが `Role` として Ansible から呼び出せるようになります。

代表的なロール構造を以下に掲載します。
```
site.yml        # 呼び出し元のplaybook
roles/          # playbook と同じ階層の roles ディレクトリに
                # ロールが格納されていると Ansible が判断します。 
  your_role/    # your_role という role を格納するディレクトリ
                # (ディレクトリ名 = role 名となります)
    tasks/      #
      main.yml  #  ロールの中で実行するタスクを記述します。
    handlers/   #
      main.yml  #  ロールの中で使用するハンドラーを記述します。
    templates/  #
      ntp.conf.j2  # ロールで利用するテンプレートを配置します。
    files/      #
      bar.txt   #  ロールの中で利用するファイルを配置します。
      foo.sh    #
    defaults/   #
      main.yml  #  ロールの中で利用する変数の一覧と
                #  デフォルト値を記述します。
  your_2nd_role/   # your_2nd_role というロールになります。
```

上記のようなディレクトリ構造を作った時に、`site.yml` では以下のようにロールを呼び出すことができます。


```yaml
---
- hosts: all
  tasks:
  - import_role:
      name: your_role

  - include_role:
      name: your_2nd_role
```

このように `import_role`  `include_role` というモジュールを使ってロールの名前を指定するだけで処理が呼びだせるようになります。この2つのモジュールは両方ともロールを呼出しますが、違いは以下です。

- [`import_role`](https://docs.ansible.com/ansible/latest/modules/import_role_module.html) playbook の実行前にロールを読み込む（先読み）
- [`include_role`](https://docs.ansible.com/ansible/latest/modules/include_role_module.html) タスクの実行時にロールが読み込まれる（後読み）

> Note: 現時点でこの2つの使い分けは意識する必要はありません。基本的には `import_role` を使う方が安全でシンプルです。`include_role` は処理によって呼び出すロールを動的に変更するような、複雑な処理を記述する際に利用します。

## Role の作成

実際にロールを作成してみます。と言っても難しいことはありません。今まで書いた処理を決められたディレクトリに分割していくだけです。

今回の演習ではWebサーバーを設定する `web_setup` ロールを作成します。以下のようなディレクトリ構造になります。
```
role_playbook.yml     # 実際にロールを呼び出す playbook
roles
└── web_setup              # ロール名
    ├── defaults
    │   └── main.yml       # 変数のデフォルト値を格納
    ├── files
    │   └── httpd.conf     # 配布するファイルを格納
    ├── handlers
    │   └── main.yml       # ハンドラーを定義
    ├── tasks
    │   └── main.yml       # タスクを記述
    └── templates
        └── index.html.j2  # テンプレートファイルを配置
```

このディレクトリ構造の作成には `ansible-galaxy` コマンドを用いることが可能です。

```shell
$ cd ~/ansible-files
$ mkdir roles
$ cd roles
$ ansible-galaxy init web_setup
$ cd web_setup
$ rm -r meta tests vars  # 今回は使用しないディレクトリ
```

各ファイルを作成していきます。

### `~/ansible-files/roles/web_setup/tasks/main.yml`

```yaml
---
- name: install httpd
  yum:
    name: httpd
    state: latest

- name: start & enabled httpd
  service:
    name: httpd
    state: started
    enabled: yes

- name: Put index.html from template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html

- name: Copy Apache configuration file
  copy:
    src: httpd.conf
    dest: /etc/httpd/conf/
  notify:
    - restart_apache
```

ロールには `play` パートを記述する必要はありませんのでタスクのみを列挙していきます。また、ロール内の `templates` `files` ディレクトリは、明示的にパスを指定しなくてもモジュールからファイルを参照できるようになっています。そのため、`copy` と `template` モジュールの `src` にはファイル名しか記述されていません。

### `~/ansible-files/roles/web_setup/handlers/main.yml`

```yaml
---
- name: restart_apache
  service:
    name: httpd
    state: restarted
```

### `~/ansible-files/roles/web_setup/defaults/main.yml`

```yaml
---
LANG: JP
```

### `~/ansible-files/roles/web_setup/templates/index.html.j2`

```jinja2
<html><body>
<h1>This server is running on {{ inventory_hostname }}.</h1>

{% if LANG == "JP" %}
     Konnichiwa!
{% else %}
     Hello!
{% endif %}
</body></html>
```

### `~/ansible-files/roles/web_setup/files/httpd.conf`

このファイルはサーバー側から取得して、以下のように編集してください。

```shell
$ cd ~/ansible-files/roles/web_setup
$ ansible node1 -b -m yum -a 'name=httpd state=latest'
$ ansible node1 -m fetch -a 'src=/etc/httpd/conf/httpd.conf dest=files/httpd.conf flat=yes'
```

ファイルが取得できていることを確認して、以下のようにファイルを書き換えます。

```shell
$ ls -l files/
```

```
ServerAdmin root@localhost
      ↓
ServerAdmin centos_role@localhost
```

### `~/ansible-files/role_playbook.yml`

実際にロールを呼び出す playbook を作成します。

```yaml
---
- name: using role
  hosts: web
  become: yes
  tasks:
    - import_role:
        name: web_setup
```

### 全体の確認

作成したロールを確認します。

```shell
$ cd ~/ansible-files
$ tree roles
```

以下のような構造になっていれば必要なファイルの準備が整っています。
```bash
roles
└── web_setup
    ├── defaults
    │   └── main.yml
    ├── files
    │   └── httpd.conf
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
        └── index.html.j2
```

## 実行

作成した playbook を実行します。

```shell
$ ansible-playbook role_playbook.yml
```

```bash
(省略)
TASK [web_setup : install httpd] *********************
ok: [node1]
ok: [node3]
ok: [node2]

TASK [web_setup : start & enabled httpd] *************
ok: [node1]
ok: [node3]
ok: [node2]

TASK [web_setup : Put index.html from template] ******
ok: [node3]
ok: [node2]
ok: [node1]

TASK [web_setup : Copy Apache configuration file] ****
changed: [node3]
changed: [node2]
changed: [node1]

RUNNING HANDLER [web_setup : restart_apache] *********
changed: [node2]
changed: [node3]
changed: [node1]
(省略)
```

実行に成功したら、ブラウザで各サーバーにアクセスして結果を確認してください。

ロールを使うことで飛躍的に自動化の再利用性が高まります。これはタスクとインベントリーが完全に切り離されるためです。それと同時に、自動度の高い playbook の記述方法に[best practice](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)という一定のルールを設けることで、「どこに何が定義されているのか」の見通しが良くなり、他のメンバーからも安心してロールを再利用してもらえるようになります。
