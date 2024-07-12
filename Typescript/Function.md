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
