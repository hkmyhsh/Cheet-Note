# ジョブにおけるコンテナの利用
- ジョブコンテナ
  - `runs-on` と同じ階層で `container` 以下に設定を記述する
  - ```
    jobs:
      container-test-job:
        runs-on: ubuntsu-latest
        container:
         image: golang:1.22.2
        steps:
          - run: (..略..)
    ```
- サービスコンテナ
  - ランナーがジョブの開始時に立ち上げるコンテナ
    - サービスコンテナはDBへのアクセスを必要とするようなテストを実施する際に便利
  - ```
    jobs:
      test:
        runs-on: ubuntu-latest
        services:
          postgres:
            image: postgres
            ports:
              - 5432:5432
            credentials:
              username: ${{ secrets.registry_username }}
              password: ${{ secrets.registry_password }}
        steps:
          run: (..略..)
    ```
  
