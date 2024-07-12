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
    ```
