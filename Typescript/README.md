# 型指定
- number型（2 の 53乗まで）
  - `const d: number = 255;`
  - 浮動小数点型
    - `const f: number = 1.5;`
  - 整数リテラル
    - 0b: 2進数; 0o: 8進数; 0x: 16進数
    - `const b: number = 0b11111111;`
    - `const o: number = 0o377;`
    - `const h: number = 0xff;`
- bigit型（2 の 53乗より大きい整数値）
  - `const big: bigint = 500n;`
- boolean型（true / false）
  - `const isStart: boolean = false;`
- string型（" または ' でくくる）
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
- 配列型
  - `type[]` または `Array<type>` となる
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

- any型（すべての型を代入できる）
  - ```
    let d: any = 255;
    d = 100;　// 100 を代入
    d = 'Hello'; // 違う型でもエラーにならない
    ```
- 初期値がない状態（あたいが入っていない状態で使おうとすると**エラー**）
  - ```
    let d: number;
    console.log(d); // エラーになる
    ```

# 判定用演算子
- 型の判定
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
- インスタンスの判定
  - `instanceof` 演算子
  - ```
    class User { }
    const user = new User();
    console.log(user instanceof User); // true
    ```
- nullの判定
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
