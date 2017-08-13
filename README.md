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
