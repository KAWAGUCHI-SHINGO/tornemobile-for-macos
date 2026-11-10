# macOSでiOS版 torne mobile を快適に利用するガイド

Apple Silicon搭載macOSでiOS版「torne mobile」を利用する際、購入情報の復旧やアプリの完全終了に関して発生する不完全な挙動を回避し、正常に動作させるための環境構築および運用の手順です。

---

## ⚠️ 免責事項 (Disclaimer)

- **自己責任での利用**: 本リポジトリで公開している手順およびスクリプトは、個人の検証結果に基づくものであり、動作を完全保証するものではありません。本手順を実行したことによるデータ損失、アプリ動作の不具合、アカウントの停止、その他いかなる不利益や損害についても、作成者は一切の責任を負いません。
- **公式非対応**: 本手順はソニー・インタラクティブエンタテインメント（SIE）および torne mobile の公式サポート対象外の操作を含みます。アプリやOSのアップデートにより、予告なく利用できなくなる可能性があります。
- **権利の帰属**: 「torne」「torne mobile」は株式会社ソニー・インタラクティブエンタテインメントの登録商標または商標です。

---

## 概要・注意点

* **購入情報復旧に関するトラブルシューティング**: 起動時に特定の権限処理を行うことで、課金アイテムの復旧を正常に行います。
* **アプリの完全終了**: torne mobileは `Cmd + Q` や通常の終了メニューではプロセス（`torneplus`）が完全に終了しない場合があります。ショートカット機能を利用した完全終了用コマンドの設定が必要です。

---

## 初期設定・購入情報の復旧手順

### STEP 1. torne mobile を完全に終了

ターミナルを開き、以下のコマンドで動作中の `torneplus` プロセスを強制終了します。

```bash
killall -9 torneplus
```

### STEP 2. ディレクトリの検索 (初回)

torne mobile 用のコンテナディレクトリを検索します。

```bash
find ~/Library/Containers -maxdepth 4 -path "*StoreKit/receipt" 2>/dev/null | while read -r r; do
  if strings -n 6 "$r" 2>/dev/null | grep -q "torne.function"; then
    echo "$r"
  fi
done
```

**出力結果の例:**
```text
/Users/<<username>>/Library/Containers/0127D990-B7B1-4566-B373-7C0C3A230A25/Data/StoreKit/receipt
```
> [!NOTE]
> `0127D990-B7B1-4566-B373-7C0C3A230A25` の部分は環境ごとに異なります。出力された文字列を確認してください。

### STEP 3. ディレクトリIDを環境変数にセット

STEP 2 で取得したID（英数字の文字列）を環境変数 `TORNEMOBILE` に代入します。

```bash
TORNEMOBILE=0127D990-B7B1-4566-B373-7C0C3A230A25
```

### STEP 4. 既存コンテナの削除

設定をクリアするため、該当のコンテナディレクトリを削除します。

```bash
rm -rf ~/Library/Containers/${TORNEMOBILE}
```

### STEP 5. 初期設定の実行

1. **torne mobile** を起動します。
2. アプリの初期設定を行います。

> [!CAUTION]
> **ここでは絶対に「購入情報の復旧」を行わないでください。**
> 誤って操作した場合は、STEP 1 からやり直す必要があります。

### STEP 6. ディレクトリの再検索

再生成された torne mobile 用の新しいコンテナディレクトリを検索します。

```bash
find ~/Library/Containers -maxdepth 4 -path "*StoreKit/receipt" 2>/dev/null | while read -r r; do
  if strings -n 6 "$r" 2>/dev/null | grep -q "torne.function"; then
    echo "$r"
  fi
done
```

> [!NOTE]
> 出力されるディレクトリIDは **STEP 2 で検索した文字列とは異なります**。

### STEP 7. 新しいディレクトリIDをセット

STEP 6 で出力された**新しいID**を環境変数に再設定します。

```bash
TORNEMOBILE=<STEP 6で出力された新しいディレクトリID>
```

### STEP 8. レシート権限の監視ルーチンを実行

レシートファイルのパーミッションを固定化するループ処理を実行します。

```bash
while true; do
  find ~/Library/Containers/${TORNEMOBILE} -name "receipt" -exec chmod 600 {} + 2>/dev/null
  sleep 0.1
done
```

> [!IMPORTANT]
> **このターミナルウィンドウは閉じずに、スクリプトを実行したままにしてください。**

### STEP 9. 購入情報の復旧

1. **torne mobile** のアプリ画面に戻ります。
2. アプリ内で **「購入情報の復旧」** を実行します。

### STEP 10. 監視ルーチンの停止

購入情報が正しく反映されたことを確認したら、STEP 8 で実行中のターミナル画面に戻り、`Ctrl + C` を押してスクリプトを強制終了します。

---

## STEP 11. 終了用ショートカットの作成

torne mobile を確実に終了させるため、macOS の「ショートカット.app」を利用して強制終了ショートカットを作成します。

### 1. ショートカットの作成
1. **ショートカット.app**（`/Applications/Shortcuts.app`）を開きます。
2. 新しいショートカットを作成し、「シェルスクリプトを実行」アクションを追加します。
3. 実行内容に以下を入力します：
   ```bash
   killall -9 torneplus
   ```
4. ショートカット名を「**torneplusを終了**」などの分かりやすい名称で保存します。

### 2. キーボードショートカットの割り当て
1. macOSの **システム設定 > キーボード > キーボードショートカット...** を開きます。
2. サイドメニューから **サービス**（または **ショートカット**）を選択します。
3. 先ほど作成した「torneplusを終了」を探し、キーボードショートカット（例: `Option + Control + Q`）を割り当てます。

---

## torne mobile の適切な終了方法

torne mobile は、`Cmd + Q` やメニューバーの「torne mobile を終了」などの標準的な終了手順ではバックグラウンドプロセス（`torneplus`）が完全に終了しない場合があります。

そのため、終了する際は **STEP 11 で作成したショートカット** を利用してください。

ショートカットを作成したくないor作成できなかった場合は、ターミナルで以下のコマンドを実行してください。
```bash
killall -9 torneplus
```

### 実行時の注意点
* **アプリがアクティブな状態では動作しない場合があります:**
  torne mobile が最前面にある場合、ショートカットキーが反応しないことがあります。その場合はデスクトップや他のアプリ（Finderなど）をクリックしてフォーカスを外してからショートカットキーを入力してください。
* **メニューバーから終了する場合:**
  メニューバー左上の `{アクティブなアプリ名}`（Finderやターミナルなど） > **サービス** > **torneplusを終了** を選択することでも終了可能です。
