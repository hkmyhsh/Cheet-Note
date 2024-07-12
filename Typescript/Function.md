# 型アノテーション
- 単純な関数
  - ```
    function fn(a) {
      return a;
    }
    ```
    - この場合、引数 a の型は `any` とみなされる
      - 設定によっては**「暗黙のanyエラー」**となる
    - **極力any型は避けるべき**
  - ```
    function fn(a: number): number{ return a; }
    ```
  - 加算結果を返す関数
    - ```
      function add(a: number, b: number): number{
        return a + b;
      }
      ```
  - **戻り値を返さない**関数
    - ```
      function fnVoid(a: number): void{
        console.log(a); 
      }
      ```
    - 戻り値を返すと**エラー**になる
