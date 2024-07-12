# 関数
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
  - **最後まで到達されることはない**関数
    - ```
      // スローにより、到達される事は無い
      function fnError(): never{
        throw new Error('Error');
      }
      // 永久ループにより、到達される事は無い
      function fnInfinite(): never{
        for (;;) {}
      } 
      // 型推論によりnever型
      function fnReturnInfinite(){
        return fnInfinite();
      }
      ```
  - **引数を省略可能にする**
    - `?`をつけた関数は省略可能になる
      - 省略された場合は`undefined`になる
      - ただし、順番として**必須引数が後に来るのはエラー**になる
    - ```
      function add(a: number, b?: number): number{
        if(b === undefined){
          b = 0;
        }
        return a + b;
      }
      ```
  - **デフォルト値**の設定
    - ただし、順番として**必須引数が後に来るのはエラー**になる
    - ```
      function add(a: number, b: number = 0): number{
        return a + b;
      }
      ```
  - **可変長引数**の設定
    - `...`（ドット3つ）を前につけると**可変長**となる`配列`として受け取る
    - ```
      function join(...strs: string[]): string{
        let res = "";
        for(let str of strs){
          res += str;
        }
        return res;
      }
      console.log(join('ab', 'cd', 'ef')); // "abcdef"
      ```

# 無名関数
- 関数式
  - ```
    const add = function(a: number, b: number): number{
      return a + b;
    };
    console.log(typeof add); // "function" 
    console.log(add(1, 2)); // 3
    ```
- アロー関数
  - `=>（アロー）`を使った関数式
  - ```
    const add = (a: number, b: number): number => {
      return a + b;
    };
    ```
  - **処理が一文の場合**は`{・・・}`を省略できる
  - 式そのものが戻り値とみなされ、`return` の省略もできる
    - ```
      const add = (a: number, b: number): number =>　a + b;
      ```
- 関数コンストラクタ
  - TypeScript による型のサポートはされていないので**使うべきではない**
  - ```
    const add = new Function('a', 'b', 'return a + b;');
    ```

# オーバーロード
- サンプルコード
  - ```
    function isPositiveNumber(a: any ): boolean{
      if(typeof a === "number"){
        return a > 0;
      }else{
        return a;
      }
    }
    ```
    - number型やboolean型を受け取った場合は順当な結果を返すが、**string**型を受け取った際に**予期せぬ結果が返ってきてしまう
      - ```
        console.log(isPositiveNumber(-1)); // false
        console.log(isPositiveNumber(true)); // true
        console.log(isPositiveNumber("abc")); // "abc"
        ```
- 関数シグニチャ
  - TypeScript では**引数、戻り値の型チェックをする**ため関数シグニチャを定義しておく
    - ```
      function isPositiveNumber(a: number): boolean;
      function isPositiveNumber(a: boolean): boolean;
      function isPositiveNumber(a: any ): boolean{ …… // 先程の関数本体
      ```
      - **型チェックがされ、number型やboolean型以外だとエラーになる**
        - ```
          console.log(isPositiveNumber("abc")); // エラー
          ```
  - オーバーロードではなく、合併型でも型チェックされる
    - ```
      function isPositiveNumber(a: number | boolean ): boolean{ ……
      ```
- 型エイリアスによるオーバーロード
  - **説明用**
    - ```
      type IsPositiveNumber = {
        (a: number): boolean;
        (a: boolean): boolean;
      }
      const isPositiveNumber: IsPositiveNumber = (a: any ): boolean => {
        if(typeof a === "number"){
          return a > 0;
        }else{
          return a;
        }
      }
      ```
  - **本来の書き方**
    - ```
      type IsPositiveNumber = {
        (a: number): boolean;
        (a: boolean): boolean;
      }
      const isPositiveNumber: IsPositiveNumber = (a: number | boolean): boolean => { ……
      ```
