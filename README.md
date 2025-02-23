# はじめに

「2/22(土)ICPのサンプルキャニスター(スマコン)をデプロイする会 参加すればICPハッカソンWave3が提出できる！」イベント
へ参加、講師TMさんからステップバイステップで教わった後に復習したまとめです。

## **[Internet Computer（ICP）](https://internetcomputer.org/)のサンプルキャニスター(スマコン)をデプロイします。**

Akindo ♾️「ICPハッカソン2025」
[**Wave3：サンプルキャニスター・デプロイ**](https://app.akindo.io/wave-hacks/0nvK8Rd9dfzj63PZV)

イベントで教わった内容から初期設定に関する情報を加え、この手順でサンプルキャニスターをPlaygroundへデプロイしました。
すでに基本的な開発環境が整っている方や開発に慣れている方にとっては回りくどい内容です。

参考ドキュメント：
- [Deploying your first full-stack dapp](https://internetcomputer.org/docs/current/tutorials/hackathon-prep-course/deploying-first-fullstack-dapp)
- [Installing developer tools](https://internetcomputer.org/docs/current/developer-docs/getting-started/install)
- [Internet Computer Canister開発 Rust編](https://zenn.dev/halifax/books/icpbook-rust)

開発環境：

- OS：masOS Monterey
- CPU：Intel Core

---

# i. 開発環境の構築



### i-1. Command Line Developer Tools のインストール

macOSでは開発ツールとして、Appleの「Command Line Developer Tools」が必要です。

```
xcode-select --install
```


### i-2. Command Line Developer Toolsのインストール確認

インストールが正常に完了しているか確認します。

```
xcode-select -p
```

🖥️ 出力結果に `/Library/Developer/CommandLineTools` が表示されれば、正しくインストールされています。

⚠️ ARM Core（Apple Siliconチップ）では、Rosetta 2 のインストールが必要です。

```
softwareupdate --install-rosetta
```


### i-3. Homebrewのインストール

HomebrewはmacOS向けのパッケージ管理ツールです。この後`dfx` や `wasmtime` などの依存ツールをインストールするために必要です。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

📝 しばらく開発に利用していなかった端末だったため、`Updating Homebrew...`  となり、アップデイト終了まで5分ほどかかりました…




### i-4. Homebrewのインストール確認

Homebrewのインストールが正常に完了しているか確認します。バージョン情報が表示されればOKです。

```
brew --version
```

🖥️ 出力結果

`Homebrew 4.4.21` 




### i-5.NVM（Node Version Manager）の設定

NVM（Node Version Manager）は、複数の Node.js バージョンを簡単に管理・切り替え するツールです。

```
brew install nvm
```

NVM のインストールが正常に完了しているか確認します。バージョン情報が表示されればOKです。

```
nvm version
```

🖥️ 出力結果

`nvm 0.40.1`




### i-6. Node.js のインストール

Node.js の **最新の安定版 (LTS)** を `nvm` を使ってインストールします。

```
nvm install --lts
```

**LTSをデフォルトのバージョンとして設定する場合**

インストールした LTS バージョンを デフォルト にするには、次のコマンドを実行します。

```
nvm alias default lts
```

Node.jsのバージョンが正しくインストールされているか確認できます。

```
node -v
```

🖥️ 出力結果

`v22.14.0`  




### i-7. ICP開発 SDK `dfx` のインストール

Dfinityが提供するICP開発用のツール `dfx` をインストールします。

```
sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
```

インストールのオプションを聞かれたら、`Proceed with installation(default)`を選択します。

```
Current installation options:

            dfx version: latest
   modify PATH variable: yes

Proceed with installation?:
> Proceed with installation (default)
  Customize installation
  Cancel installation
```




### i-8. `dfx` の利用設定

インストールした `dfx` をターミナルから利用できるようにするため、PATHを設定します。

```
echo 'export PATH=$HOME/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```




### i-9. `dfx` の動作確認

`dfx` を利用できる状態になったか確認します。バージョンが表示されればOKです。
失敗した時には、PATHの設定に誤りがないか確認するか、`exit` による再起動を試しましょう。

```
dfx --version
```

🖥️ 出力結果

`dfx 0.25.0` 




### i-10.**Rust のインストール (`rustup` 使用)**

Canister開発にはRustが必要です。`rustup` を使ってインストールします。

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

インストールのオプションを聞かれたら、`Proceed with installation(default - just press enter)`を選択します。

```
Current installation options:

   default host triple: x86_64-apple-darwin
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with standard installation (default - just press enter)
2) Customize installation
3) Cancel installation
```




### i-11. WebAssemblyのインストール

ICPのCanisterはWebAssembly（Wasm）を使用するため、対応するターゲットを追加します。

```
rustup target add wasm32-unknown-unknown
```




### i-12. `cargo`（Rustパッケージマネージャー）のインストール確認

Rustが正しくインストールされているか確認します。

```
cargo --version
```

🖥️ 出力結果

`cargo 1.85.0` 




---

# ii. サンプルキャニスターをデプロイする



### ii-1.Canisterのプロジェクト作成

`dfx new` コマンドで新しいCanisterのプロジェクトを作成します。<my_canister_name>の部分に好きなプロジェクト名を入れます。

```
dfx new <my_canister_name>
cd <my_canister_name>
```



今回<my_canister_name>を`bypp_canister` で作成しました。

```
dfx new bypp_canister
```

バックエンドの開発言語を選択します。

```
? Select a backend language: ›  
❯ Motoko
  Rust
  Python (Kybra)
  Typescript (Azle)
```

 ✔ `Motoko` を選択

📝 **Motoko と Rust** 
Motoko は、DFINITY 財団 によって開発された ICP 専用のプログラミング言語 であり、Rust をベースとしています。名前の由来は『攻殻機動隊』の草薙素子🚀
Motoko と Rust のどちらも使用して、ICP 上で Canister を開発できます。

フロントエンドのフレームワークを選択します。

```
? Select a frontend framework: ›  
  SvelteKit
❯ React
  Vue
  Vanilla JS
  No JS template
  None
```

 ✔ `React` を選択

```
⬚ Internet Identity
⬚ Bitcoin (Regtest)
⬚ Frontend tests
```

 ✔ `Unchecked` を選択

---

### ii-2.【 local環境 】 `dfx` のローカルネットワーク起動

ICPのローカル開発環境を起動します。`--background` をつけることでバックグラウンドで実行されます。

```
dfx start --clean --background
```

📝 `--clean`

以前のデータやキャッシュを削除し、クリーンな状態で起動 します。

📝 `--background`

ターミナルをブロックせず、バックグラウンドで実行します。

### ii-3. 【 local 環境 】`dfx` の作成したCanisterデプロイ

作成したCanisterをlocal環境にデプロイします。

```
dfx deploy
```



### ii-4. 【 local 環境 】デプロイの確認

このコマンドを実行すると、現在の環境に設定されているIdentityの一覧 が表示されます。

```
dfx identity list
```

設定した<my_canister_name>（今回`bypp_canister` ）があるか確認します。

```
default
alice
bypp_canister
charlie
```

【 local環境 】へのデプロイが完了すると、Frontend用とBackend用のURLが生成されます。
ブラウザで画面を確認すると、URLが見つかります。

<img width="1203" alt="Image" src="https://github.com/user-attachments/assets/87b1dde8-4229-49e7-9471-3e17394a9f37" />

### 【 local環境 】 Frontend

生成されたFrontend URLを開いた画面です。
このサンプルキャニスターは、入力された名前に挨拶を付けて返答する機能を持っています。**「Click me!」**ボタンを押すと、**「Enter your name」**に入力した名前の前に「Hello」を付けたメッセージが表示されます。

🖥️ Enter your name:に入力し`Click Me!` をクリックした状態

<img width="1680" alt="Image" src="https://github.com/user-attachments/assets/74e73548-850b-4ec3-a979-28d0ce2e653d" />

### 【 local環境 】 Backend

生成されたBackend URLを開いた画面です。

このサンプルキャニスターは、入力された名前に挨拶を付けて返答する機能を持っています。「QUERY」ボタンを押すとテキストボックスに入力した名前の前に「Hello,」を付けたメッセージが表示されます。

🖥️ テキストボックスに入力し`Query` をクリックした状態

<img width="1680" alt="Image" src="https://github.com/user-attachments/assets/d652bee8-be9a-41d9-8086-369dcae3e26f" />

### ii-5. 【 Playground環境 】作成したCanisterのデプロイ

**identityを指定し、Canisterを「Playground」環境にデプロイ** します。

```
dfx deploy --identity <my_canister_name> --playground
```

今回の`bypp_canister` でデプロイ。

```
dfx deploy --identity bypp-canister --playground
```

📝 【Playground環境】について
- `--playground` は、**開発用の環境** のため20分で利用できなくなる
- Playground環境 を利用するとEVMのガス代にあたる `cycles` トークンを消費せずにテストできる


【 Playground環境 】へのデプロイが完了すると、Frontend用とBackend用のURLが生成されます。
ブラウザで画面を確認すると、URLが見つかります。

<img width="1203" alt="Image" src="https://github.com/user-attachments/assets/65d6dc60-3266-4549-87ba-d9009f740851" />

---

# iii. 🚀【Playground環境】デプロイ完了


生成されたFrontend 用と Backend用のURLを開くと、【Playground環境】のサンプルキャニスターにアクセスできます。見た目と機能は【 local環境 】と同じものです。

### 【 Playground環境 】Frontend

生成されたのFrontend URLを開いた画面です。

🖥️ Enter your name:に入力し`Click Me!` をクリックした状態

<img width="1680" alt="Image" src="https://github.com/user-attachments/assets/dde1a4d2-4643-4186-8474-e256e6cbc31a" />

### 【 Playground環境 】Backend

生成されたのBackend URLを開いた画面です。

🖥️ テキストボックスに入力し`Query` をクリックした状態

<img width="1680" alt="Image" src="https://github.com/user-attachments/assets/a15e65ab-1841-4606-9fa4-421e8bcfc41d" />



