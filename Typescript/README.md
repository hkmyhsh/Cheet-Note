# 型指定
- **number**型（2 の 53乗まで）
  - `const d: number = 255;`
  - 浮動小数点型
    - `const f: number = 1.5;`
  - 整数リテラル
    - 0b: 2進数; 0o: 8進数; 0x: 16進数
    - `const b: number = 0b11111111;`
    - `const o: number = 0o377;`
    - `const h: number = 0xff;`
- **bigit**型（2 の 53乗より大きい整数値）
  - `const big: bigint = 500n;`
- **boolean**型（true / false）
  - `const isStart: boolean = false;`
- **string**型（" または ' でくくる）
  - ```
    let s: string = 'Hello';
    s = "TypeScript";
    const word: string = "TypeScript";
    const version: number = 3;
    const message: string = `Hello
    ${word} ${version + 1}`;
    console.log(message);
    /*
    Hello
    TypeScript 4
    */
    ```
- **配列**型
  - `type[]` または `Array<type>` となる
    - 要素は一つの型で統一される
  - ```
    const nums: number[] = [10, 20 , 30];
    const strs: string[] = ["foo", "bar" , "hoge"];
    ```
  - **コピー**
    - **参照**
      - ```
        const numsA = [1, 2];
        const numsB = numsA; // numsB には numsA の参照が入る
        numsB[0] = 100;
        console.log(numsA); // [100, 2]
        ```
    - **参照ではなく、値を入れるだけ**
      - ```
        const numsA = [1, 2];
        const numsB = [...numsA]; 
        numsB[0] = 100;
        console.log(numsA); // [1, 2]
        ```
- **合併（プリミティブ）型**
  - 
  - `n:1` の場合
    - ```
      let sns: (string | number)[] = [10, 'abc', 100];
      for(let sn of sns){
        if(typeof sn === 'number'){
          console.log(sn * 10);
        }else{
          console.log(sn);
        }
      }
      /* 100 
　        "abc" 
　        1000 */
      ```
- **タプル**型
  - `[]` となる
    - 複数の型を格納できる
    - 型が混在するので使用には注意
  - ```
    let tup: [string, number, boolean];
    tup = ["abc", 10, true];
    tup = [10, "abc", true]; // エラー
    console.log(tup[0].toUpperCase());
    console.log(tup[1].toUpperCase()); // エラー
    ```
- **enum**（列挙）型
  - 列挙値は**0**から始まる（`number`型）
    - 列挙型[列挙値] で列挙子（`string`型）を取得できる
    - ```
      enum Color {
        Red ,
        Green,
        Blue,
      }
      let g: Color = Color.Green;
      console.log(g); // 1
      console.log(Color[1]); // "Green"
      ```
    - **列挙値は任意で設定できる**
      - ```
        enum Color {
          Red = 5 ,
          Green = 10,
          Blue,
        }
        ```
      - この場合 Blue の値は 11 となります
- **any**型（すべての型を代入できる）
  - **制限なく操作ができてしまう分危険**
  - ```
    let d: any = 255;
    d = 100;　// 100 を代入
    d = 'Hello'; // 違う型でもエラーにならない
    ```
- **unknown**型（何のわからない）
  - any型と同じく、すべての型を受け取ることができる
  - 何の型か「わからない」ため**何もすることができない**
    - `Object is of type 'unknown'.` というエラー
  - ```
    let a: unknown = 'abc';
    console.log(a.toLocaleUpperCase()); // エラー
    console.log(a + 10); // エラー 
    a = 20;
    console.log(a.toLocaleUpperCase); // エラー
    ```
  - 使い方① **型ガード**
    - ```
      let a: unknown = 'abc';
      if(typeof a === 'string'){
        console.log(a.toLocaleUpperCase()); // "ABC" 
      }
      if(typeof a === 'number'){
      // 判定が偽となるので、このブロックは処理されない
        console.log(a + 10);
      }
      ```
  - 使い方② **any型を返す関数の受け取りに使う** / **外部から任意の値を受け取る**
    - ```
      const fn = (): any =>　"abc";
      let a:unknown = fn();
      if(typeof a === 'string'){
        console.log(a.toLocaleUpperCase()); // "ABC" 
      }
      ```

- **初期値がない状態**（あたいが入っていない状態で使おうとすると**エラー**）
  - ```
    let d: number;
    console.log(d); // エラーになる
    ```

# オブジェクト
- 辞書オブジェクト
  - インデックスシグネクチャの書き方
    - `[キー名: キーの型]: 値の型`
    - 例: キーがstring型、値がstring型の場合
      - `[key: string]: string`
  - ```
    const tools: { [code: string]: string } = {};
    tools['clean_0001'] = 'broom';
    tools['clean_0002'] = 'dustpan';
    console.log(tools['clean_0001']);// "broom" 
    console.log(tools['clean_0003']); // undefined
    tools['clean_0002'] = 10; // エラー
    // Type 'number' is not assignable to type 'string'.
    ```
  - **ループで回してみる**
    - ```
      const tools: { [code: string]: string } = {};
      tools['clean_0001'] = 'broom';
      tools['clean_0002'] = 'dustpan';
      Object.entries(tools).forEach(([key, value]) => console.log(key + ':' + value));
      // "clean_0001:broom" 
      // "clean_0002:dustpan"
      ```
  - **`delete` キーワードで削除**
    - ```
      delete tools['clean_0001'];
      Object.entries(tools).forEach(([key, value]) => console.log(key + ':' + value));
      // "clean_0002:dustpan"
      ```
  - **合併（プリミティブ）型**
    - `n:1` の場合
      - ```
        let sn: string | number = 10;
        sn = 'abc';
        sn = 100;
        sn = true; // エラー
        ```

# 判定用演算子
- **型の判定**
  - `typeof` 演算子
  - **null の場合は `object` を返す**
  - ```
    let d;
    console.log(typeof d); // "undefined"
    ```
  - ```
    const d = 255;
    if(typeof d === "number"){
      console.log("d は number型です");
    }else{
      console.log("d は number型ではありません");
    }
    // "d は number型です" と出力
    ```
- **インスタンスの判定**
  - `instanceof` 演算子
  - ```
    class User { }
    const user = new User();
    console.log(user instanceof User); // true
    ```
- **nullの判定**
  - 初期値が null なら any 型だと判断される
  - ```
    //let a: any = null; と同じ
    let a = null;
    console.log(typeof a); // "object"
    a = 1;
    console.log(typeof a); // "number"
    a = 'Hello';
    console.log(typeof a); // "string"
    ```

# 各種メソッド
- **配列の結合**
  - `concat` メソッド
    - ```
      const numsA = [1, 2];
      const numsB = numsA.concat(); 
      numsB[0] = 100;
      console.log(numsA); // [1, 2]
      ```
- **型アサーション**
  - コンパイラに対し**型を表明（アサーション）する機能**
    - ```
      let u: unknown = '100'; // 数字を代入
      console.log((u as string).length); // 3
      ```
    - 関数の引き渡しでも利用可能
      - ```
        function fn(s:string){
          console.log(s); // 100
          console.log(s.length); // undefined 
        }
        let n = 100;
        fn(n as any);
        // 下記の場合エラー
        // fn(n as string);
      
