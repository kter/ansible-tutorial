1. インベントリ内全ホストに対してpingモジュールを使う

    ```
    ansible all -i hosts -m ping
    ```

2. プレイブックsite.ymlをインベントリファイルhostsに対して実行

    ```
    ansible-playbook -i hosts -m ping
    ```

3. setupモジュールで収集された値(Facts)の一覧

    ```
    ansible all -i hosts -m setup
    ```

4. setupモジュール実行キャンセル

    * Python環境がないホストへのデプロイ
    * Factsを使う必要がなく、デプロイ時間を短縮したい

    ```
    - hosts: all
      gather_facts: false
    ```

5. ファイル・ディレクトリの作成

    ```
    - name: /tmp/dir1を作成
      file:
        path: /tmp/dir1
        state: directory
    ```

6. シンボリックリンクの作成

    ```
    - name: /tmp/dir1のシンボリックリンクを/tmp/link_to_dir1に作成
      file:
        src: /tmp/dir1
        dest: /tmp/link_to_dir1
        state: link
    ```

7. 権限変更

    ```
    - name: /tmp/dir1をansibleAdminユーザ／グループ所有にして全ユーザから実行可能にする
      file:
        path: /tmp/dir1
        state: directory
        owner: ansibleAdmin
        group: ansibleAdmin
        mode: "u=rwx,g=rx,o=rx"
    ```

8. ファイルを転送

    ```
    - name: Ansible稼働マシン上のファイルをリモートにコピー
      copy:
        src: original_file
        dest: ~/copied_file
    ```

9. 行単位でファイルを編集

    ```
    - name: sshd設定Play
      hosts: all
      become: true
      tasks:
        - name: rootへのパスワードSSHログインを禁止
          lineinfile:
            dest: /etc/ssh/sshd_config
            regexp: '^PermitRootLogin\s+'
            line: Permitrootlogin without-password
            validate: sshd -t -f %s # Ansibleが作成する一時ファイルを指定
          notify:
            - sshd再起動
      handlers:
        - name: sshd再起動
          service:
            name: sshd
            state: restarted
    ```

10. 任意のコマンドを実行

    * パイプやリダイレクトは使えない
    * $記号での環境変数の参照はできない ( {{ ansible_env.(変数名) | quote }}を使う, setup実行時にセットされる)
    * 変数を使う場合はquoteフィルターを使ってサニタイズする

    ```
    - name: SSH鍵生成サンプルPlay
      hosts: all
      tasks: 
        - name: tmpディレクトリ内にパスフレーズなしのSSH鍵を生成
          command: "/usr/bin/ssh-keygen -b 2048 -t rsa -N '' -f /tmp/new-id_rsa"
            args:
              creates: /tmp/new-id_rsa # ファイルが存在しない場合のみ処理が実行される
    ```

11. インベントリについて

    * グループ間で変数定義が重複しないようにする
    * ホスト名が同じ場合は同一ホストとして扱われる
    * -iオプションでディレクトリを指定すれば、そのディレクトリ内のファイルをすべて合わせてくれる
    * add_hostモジュールでPlaybook内でInventoryに新規ホストを追加できる
    * group_byモジュールでPlaybook内でグループ分けをすることができる

    ```
    [app]
    app1 ansible_host=some.app.host
    app2 ansible_host=another.app.host
    [db]
    db1 ansible_host=db.1.host
    db2 ansible_host=db.2.host
    [japaneast]
    app1
    db1
    [japanwest]
    app2
    db2
    [app:vars]
    admin_username=app_user
    admin_uid=1001
    ```

12. インベントリの変数設定ファイル切り出し

    * ホスト変数はhost_vars/ホスト名.yml
    * グループ変数はgroup_vars/グループ名.yml
    * グループ名にはallも使える

13. 変数定義

    * ansible-playbook実行時
        ```-e 'nginx_version=1.10.2 nginx_user=ng'```
        ```-e '{"nginx_port": 8080}'``` (数値型を扱うならこちら
        ```-e '@extra-vars.yml'``` (ファイルを使うならこちら
    * Playbook内での変数定義
        ```
        vars:
            nginx_http_port: 80
            mysql_port: 3306
        ```
    * Playbook内での変数定義 (ファイルから読み込む)
        ```
        vars_files:
            - some_vars.yml
            - another_vars.yml
        ```

14. テンプレート その一

    ```
    admin_user:
      name: taro
      uid: 1001
    ```

    上記変数定義は下記文で次のように取り出せる。

    ```ユーザ名{{ admin_user.name }}のUIDは{{ admin_user.uid }}です```
    ```ユーザ名taroのUIDは1001です```

15. テンプレート その二

    条件分岐が使える

    ```
    {% ansible_os_family == 'RedHat' %}
    このマシンのディストリビューションはRed hat系です
    {% elif ansible_os_family == 'Debian' %}
    このマシンのディストリビューションはDebian系です
    {% else %}
    このマシンはRed Hat系でもDebian系でもありません
    {% endif %}
    ```

16. テンプレート その三

    繰り返しが使える

    * for分の中でifを使うことで、繰り返しの対象を絞ることができる
    * 特殊変数loopを使って繰り返し数や最後まで行ったかどうかが分かる

    ```
    admin_users:
      - name: taro
        uid: 1001
      - name: jiro
        uid: 1002
      - name: hanako
        uid: 1003
    ```

    ```
    {% for user in admin_users %}
    ユーザ名{{ user.name }}のUIDは{{ user.uid }}です。
    {% endfor %}
    ```

