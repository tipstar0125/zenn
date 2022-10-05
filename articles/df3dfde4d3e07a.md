---
title: "Rust: シリアル通信やってみた"
emoji: ⚙️
type: "tech"
topics: ["rust"]
published: true
---

今までPythonで作成していたアプリをRustへ移行する一環として共有します。

# 準備

手元にハードウェアがない場合は、仮想シリアル環境を構築して、デバッグするやり方がオススメです。
※私はコロナの影響で在宅リモートでコードを書いて、たまに現場で動作確認をしています

- Tera Term

ハードウェアの模擬として`Tera Term`を使用しますので、インストールしてください。

- com0com

仮想シリアルはcom0comがオススメです。以下の記事が分かりやすいので参考にしてください。
https://qiita.com/yaju/items/e5818c99857883a59033

インストールが完了したら、`Setup`から下図のような設定をしましょう。
接続ポートの指定は任任で構いません。
下図の例では、`COM1`と`COM2`、`COM11`と`COM12`が接続されています。

![altテキスト](/images/serial_com0com.png)

試しに`Tera Term`を2つ開いて、それぞれ`COM1`と`COM2`に接続して、通信できるか確認してみましょう！
以下の例では、`COM1`の画面を選択した状態で、キーボードを叩いて、`COM2`に送信されていることを確認しています。

![altテキスト](/images/serial_com0com_test.png)


# Dependencies

本記事で使用する`Crate`は以下になりますので、`Cargo.toml`に記載しておきましょう。
同じような`Crate`に`serial`というものもありますが、examplesがあって、使いやすそうな`serialport`を採用します。
`serial`を使用した感じだと、同じような使い方なので、そちらを使い方も参考にはなると思います。

```
[dependencies]
serialport = "4.2.0"
```

# データ送受信

コードの全容は以下の通り。
以下のような動きになります。
1. Helloという文字列をTera Term側に1秒周期で送り続ける
2. 1秒周期でTera Termからのデータを受け取り、標準出力する
※1秒待機しているだけので、厳密には1秒周期ではないです

:::details main.rs
```rust
use std::error::Error;
use std::io::prelude::*;
use std::time::Duration;

fn main() -> Result<(), Box<dyn Error>> {
    let mut port = serialport::new("COM1", 9600)
        .stop_bits(serialport::StopBits::One)
        .data_bits(serialport::DataBits::Eight)
        .parity(serialport::Parity::None)
        .timeout(Duration::from_millis(100))
        .open()?;


    let mut buf: Vec<u8> = vec![0; 1000];
    loop {
        println!("Write...");
        match port.write("Hello\r\n".as_bytes()) {
            Ok(_) => std::io::stdout().flush()?,
            Err(ref e) if e.kind() == std::io::ErrorKind::TimedOut => (),
            Err(e) => eprintln!("{:?}", e),
        }

        println!("Read...");
        match port.read(buf.as_mut_slice()) {
            Ok(t) => {
                let bytes = &buf[..t];
                let string = String::from_utf8(bytes.to_vec())?;
                println!("bytes: {:?}", bytes);
                println!("string: {:?}", string);
            }
            Err(ref e) if e.kind() == std::io::ErrorKind::TimedOut => (),
            Err(e) => eprintln!("{:?}", e),
        }
        std::thread::sleep(Duration::from_millis(1000));
    }
}

```
:::

- 接続設定

今回、`COM1`をハードウェアの模擬的な接続先として、`COM2`に`Tera Term`を接続します。
※`stop bits`や`daa bits`、`parity`はデフォルトをそのまま設定しているだけなので、省略可能

```rust
let mut port = serialport::new("COM1", 9600)
    .stop_bits(serialport::StopBits::One)
    .data_bits(serialport::DataBits::Eight)
    .parity(serialport::Parity::None)
    .timeout(Duration::from_millis(100))
    .open()?;
```

- 送信

実際に送信しているコードは、`port.write()`の部分だけです。これだけでも送信はできますが、バッファをクリアしたり、エラー処理する場合は、以下のようにします。引数は`String`型ではなく、`Vec<u8>`なので、`as_bytes()`で変換するか、`b"Hello"`とする必要があります。

```rust
match port.write("Hello\r\n".as_bytes()) {
    Ok(_) => std::io::stdout().flush()?,
    Err(ref e) if e.kind() == std::io::ErrorKind::TimedOut => (),
    Err(e) => eprintln!("{:?}", e),
}
```

- 受信

受信する場合は、以下のようなコードになります。
バイト列で扱うか、文字列で扱うかは用途により異なるので、今回は標準出力として両方を出力することにしました。

```rust
match port.read(buf.as_mut_slice()) {
    Ok(t) => {
        let bytes = &buf[..t];
        let string = String::from_utf8(bytes.to_vec())?;
        println!("bytes: {:?}", bytes);
        println!("string: {:?}", string);
    }
    Err(ref e) if e.kind() == std::io::ErrorKind::TimedOut => (),
    Err(e) => eprintln!("{:?}", e),
}
```

- 動作確認

Rustアプリ側の標準出力は以下の通り。
![altテキスト](/images/serial_rust.png)

Tera Term側の標準出力は以下の通り。
![altテキスト](/images/serial_teraterm.png)


# まとめ
今回作成したデモアプリは以下のGithubにありますので、参考にしてください。
https://github.com/TatsuyaYoke/serial-app-rs

また、以下のexamplesにその他の使い方が紹介されていますので、合わせて参考にしてください。
https://github.com/serialport/serialport-rs/tree/main/examples