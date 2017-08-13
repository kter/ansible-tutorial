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
