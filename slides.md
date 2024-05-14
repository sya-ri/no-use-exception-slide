---
theme: seriph
background: https://cover.sli.dev
title: いいから黙って例外使わず書いてみろ
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
layout: center
---

# いいから黙って例外使わず書いてみろ

@sya-ri

---

# 例外（Exception）ってなに？

- 処理の失敗を表すもの
- 言語機能の一つで、いろんな言語に存在する
  ```kotlin
  // Kotlin
  throw Exception("message")
  ```
  ```typescript
  // TypeScript
  throw "message";
  ```
- スライドにおけるエラー・失敗・例外について・・・
  - エラーや失敗の表現方法の一つとして例外を用いる。

---

# どういうことを話すか

- メソッドの戻り値の型について
- 失敗したことを呼び出し元に伝える方法について
  - 💢 Exception（例外）
  - ↩️ 戻り値
- 適当に状況を考えて話す
  - 例外以外の部分でツッコミどころがあるかもしれないが、あまり気にしないでほしい
    - アーキテクチャとか外部APIとか命名とか

<br />

私のスタンス

- ❌ ~~間違ってる！！こう書け！！~~
- 😌 こういう書き方もあるよ
- 🥰 場合によってはこっちが適してるかも？

---

# 要点まとめ

- 失敗を例外で扱うのは必ずしも正解ではない
  - むしろ、ハンドリングするなら例外を使うのは良くないかも
  - しないのであれば例外使ってもいいかも（500エラーにしてしまうみたいな）
- なぜ失敗したのかを返す必要がない場合もある
- 失敗という概念が起きないような設計や仕様にしたっていい

---
transition: slide-up
---

# ケース１「値の保存と取得」

- `set` で値を保存できる
- `get` を呼び出すと保存された値を取得できる

```kotlin
class Repository {
    fun get(): String {
        //
    }
    
    fun set(value: String) {
        //
    }
}
```

実行してみると

```kotlin
val repository = Repository()
repository.set("hello world")
println(repository.get())
```

> ```text
> hello world
> ```

---
transition: slide-up
---

# ケース１「値の保存と取得」

```kotlin
val repository = Repository()
println(repository.get())
```

🤔 保存してない状態で取得した時の挙動、どうする？

- 例外を投げる
- null
- 空文字（デフォルト値）

---
transition: slide-up
---

**例外を投げる**

```kotlin
class NoValueStoredException : Exception()

class Repository {
    private var value: String? = null

    fun get(): String {
        return value ?: throw NoValueStoredException()
        // if (value != null) {
        //     return value
        // } else {
        //     throw NoValueStoredException()
        // }
    }

    fun set(value: String) {
        this.value = value
    }
}
```

> ```text
> Exception in thread "main" org.jetbrains.kotlin.idea.scratch.generated.ScratchFileRunnerGenerated$ScratchFileRunnerGenerated$NoValueStoredException
> at org.jetbrains.kotlin.idea.scratch.generated.ScratchFileRunnerGenerated$ScratchFileRunnerGenerated$Repository.get(tmp.kt:13)
> at org.jetbrains.kotlin.idea.scratch.generated.ScratchFileRunnerGenerated$ScratchFileRunnerGenerated.generated_get_instance_res0(tmp.kt:21)
> at org.jetbrains.kotlin.idea.scratch.generated.ScratchFileRunnerGenerated.main(tmp.kt:33)
> ```

---
transition: slide-up
---

**例外を投げる**

```kotlin
class NoValueStoredException : Exception()

class Repository {
    private var value: String? = null

    fun get(): String {
        return value ?: throw NoValueStoredException()
    }

    fun set(value: String) {
        this.value = value
    }
}
```

<br />

- 特にいうことない。
- Java ならあるあるだよね〜
- エラーハンドリングをしたいのであれば、`try-catch` で `NoValueStoredException` を拾う

---
transition: slide-up
---

**null**

```kotlin
class Repository { 
    private var value: String? = null
  
    fun get(): String? { // String -> String?
        return value
    }

    fun set(value: String) {
        this.value = value
    }
}
```

> ```text
> null
> ```

<br />

- 例外投げるくらいなら `null` でも良くね？
- 型安全だし、エラーハンドリングし忘れることないよね
  - 呼び出し元目線: `null` が返ったらエラー（失敗）！

---

**空文字（デフォルト値）**

```kotlin
class Repository {
    private var value: String = "" // String? -> String

    fun get(): String {
        return value
    }

    fun set(value: String) {
        this.value = value
    }
}
```

> ```text
>  
> ```

<br />

- まず、値が初期化されていないっていう状態をやめないか？
  - 何も設定されていない（＝空が設定されている）
- エラーハンドリングが不要！未設定と空文字の区別をする必要がない

---
transition: slide-up
---

# ケース２「失敗の仕方が複数ある」

とある外部APIを使って、ユーザー名を取得するメソッドを作ろう！

```http request
# 指定したユーザーの情報を取得するAPI
GET /users/:id
```

想定しうるパターンとしては・・・

- ✅ 成功
  - ユーザー名が返ってくる
- ❌ 失敗
  - APIの接続に失敗（外部APIのサーバーが落ちてるかも？再試行してもダメそう）
  - ユーザーが存在しない（存在しないユーザーIDを指定するとユーザー情報自体取得できない）
  - 名前が設定されていない（名前を設定していないと、レスポンスに名前が含まれない仕様らしい）

---
transition: slide-up
---

**例外を使って表現してみよう**

```kotlin
// APIの接続に失敗
class ApiConnectionException : Exception()

// ユーザーが存在しない
class NotFoundUserException : Exception()

// 名前が設定されていない
class NameNotSetException : Exception()

fun getUserName(userId: String): String
```

```kotlin
try {
    val userName = getUserName(userId)
    // ...
} catch (error: ApiConnectionException) {
    // ...
} catch (error: NotFoundUserException) {
    // ...
} catch (error: NameNotSetException) {
    // ...
}
```

どうしてもエラーハンドリング大変だよね

---
transition: slide-up
---

**戻り値を使って表現してみよう**

```kotlin
sealed interface GetUserNameResult {
  class Success(val userName: String) : GetUserNameResult
  data object ApiConnectionError : GetUserNameResult
  data object NotFoundUserError : GetUserNameResult
  data object NameNotSetError : GetUserNameResult
}

fun getUserName(userId: String): GetUserNameResult
```

```kotlin
when (val result = getUserName(userId)) {
    is GetUserNameResult.Success -> {}
    GetUserNameResult.ApiConnectionError -> {}
    GetUserNameResult.NotFoundUserError -> {}
    GetUserNameResult.NameNotSetError -> {}
}
```

- `Result<T, E>` みたいな書き方をしてもいいと思う。
- 戻り値を使った方が、エラーハンドリングは簡単そう

---
transition: none
layout: center
---

✋ ちょっと待って！！！

---
transition: slide-up
layout: center
---

そのエラーパターンって必要ですか？？？？

> - APIの接続に失敗
> - ユーザーが存在しない
> - 名前が設定されていない

---
transition: slide-up
---

**そのエラーパターンって必要？？**

- APIの接続に失敗
  - この場合のハンドリングするか？？？
  - 例外投げっぱなしにして、最悪500エラーで処理とかでも良くない？？
- ユーザーが存在しない
  - まあ必要かもな・・・
- 名前が設定されていない
  - まあ必要かもな・・・
  - でも、空文字とかnullの名前でも良くない？

---

**こうしてみるのはどうですか？**

```kotlin
class ApiConnectionException : Exception()

/**
 * @return 存在しないユーザーの場合、null。名前が設定されていない場合は ""
 * @throws ApiConnectionException APIの接続に失敗
 */
fun getUserName(userId: String): String?
```

- 存在しないユーザーかどうかは判断したい時あるよね
  - `null` で表現する
  - 存在しないユーザー ≠ 名前が設定されていないユーザー
- 「名前が設定されていない = 空文字の名前が設定されている」でも問題ないよね
- `ApiConnectionException` はハンドリングする気がないから例外投げるだけ投げるよ
  - 必要ならハンドリングしてもええでの気持ちで `@throws` を書く

---

# 要点まとめ

- 失敗を例外で扱うのは必ずしも正解ではない
  - むしろ、ハンドリングするなら例外を使うのは良くないかも
  - しないのであれば例外使ってもいいかも（500エラーにしてしまうみたいな）
- なぜ失敗したのかを返す必要がない場合もある
  - `null` で置き換えできるかも？？
- 失敗という概念が起きないような設計や仕様にしたっていい
  - 文字列であれば空文字 `""` を使うとかね

---
layout: center
---

```text
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index: 18, Size: 18
```
