# タイプ（型）
- `交差`による拡張可
  - ```
    type Computer = {
      name: string
    }
    type Tablet = Computer & {
      size: number
    }
    ```
- プロパティの**変更不可**
  - **エラー：Duplicate identifier 'Article'.**
  - ```
    // エラーになります
    type Article = {
      title: string
    }
    type Article = { 
      body: string
    }
    ```

# インターフェイス
- `extends` による拡張可
  - ```
    interface Computer{
      name: string
    }
    interface Tablet extends Computer {
      size: number
    }
    ```
- プロパティーの追加
  - ```
    interface Article{
      title: string
    }
    interface Article{
      body: string
    }
    ```

