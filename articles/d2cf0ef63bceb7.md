---
title: "ヒューリスティックコンテスト用Visualizer(Rust, SVG) チートシート集"
emoji: "🏃"
type: "tech"
topics: ["rust", "AtCoder", "svg"]
published: true
---

Rustのsvgクレートを使って、ヒューリスティックコンテストのビジュアライザを作成するためのチートシート集を紹介します。

# Cargo.toml

```toml
[dependencies]
svg = "0.9.0"
```

# html出力

```rust
let svg = vis();  // svg.to_string()
let vis = format!("<html><body>{}</body></html>", svg);
std::fs::write("vis.html", vis).unwrap();

pub fn vis() -> String {
    let mut svg = Document::new()
        .set("viewBox", (0, 0, 600, 600))
        .set("width", 600)
        .set("height", 600);
    // ここに絵を上書き追加していく
    svg.to_string()
}
```

# Document

`Document`は、絵を描画するキャンバスみたいなもので、描画範囲を指定し、この中に絵を上書き追加していくことで、絵を描くことができる。コンテストにおける描画範囲の目安は`600 x 600`ぐらい。
※下記の`style`は、描画範囲が分かるように色をつけているだけなので、通常は不要

```rust
use svg::Document;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set("viewBox", (0, 0, 600, 600))
        .set("width", 600)
        .set("height", 600)
        .set("style", "background-color:#F2F3F5");  // 通常不要

    svg.to_string()
}
```
![Document](/images/Document.png)

# Rectangle

四角形を追加する場合は`Rectangle`を使用する。使いやすいように、座標・サイズ・色を引数として関数にしておくとよい。座標は四角形の左上となる。

```rust
use svg::node::element::Rectangle;
use svg::Document;

pub fn rect(x: usize, y: usize, w: usize, h: usize, fill: &str) -> Rectangle {
    Rectangle::new()
        .set("x", x)
        .set("y", y)
        .set("width", w)
        .set("height", h)
        .set("fill", fill)
}

pub fn vis() -> String {
    let mut svg = Document::new()
        .set("viewBox", (0, 0, 600, 600))
        .set("width", 600)
        .set("height", 600)
        .set("style", "background-color:#F2F3F5");  // 通常不要

    svg = svg.add(rect(0, 0, 30, 30, "red"));

    svg.to_string()
}
```
![Document](/images/Rectangle_single.png)

グリッド系の場合は、以下のように1グリッド当たりのサイズを計算すればよい。

```rust
const SVG_SIZE: usize = 600;
const N: usize = 20;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set("viewBox", (0, 0, SVG_SIZE, SVG_SIZE))
        .set("width", SVG_SIZE)
        .set("height", SVG_SIZE)
        .set("style", "background-color:#F2F3F5"); // 通常不要

    let d = SVG_SIZE / N;
    for i in 0..N {
        for j in 0..N {
            if (i + j) % 2 == 0 {
                svg = svg.add(rect(i * d, j * d, d, d, "red"));
            } else {
                svg = svg.add(rect(i * d, j * d, d, d, "blue"));
            }
        }
    }

    svg.to_string()
}
```

![Document](/images/Rectangle_grid.png)

四角形の枠線を色付けする場合は、`stroke`や`stroke-width`を設定する。
※外側の枠線が描画範囲を超えるので、マージンを設けている

```rust
use svg::node::element::Rectangle;
use svg::Document;

pub fn rect(x: usize, y: usize, w: usize, h: usize, fill: &str) -> Rectangle {
    Rectangle::new()
        .set("x", x)
        .set("y", y)
        .set("width", w)
        .set("height", h)
        .set("fill", fill)
        .set("stroke", "gray")
        .set("stroke-width", 3)
}

const SVG_SIZE: usize = 600;
const MARGIN: isize = 10;
const N: usize = 20;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set(
            "viewBox",
            (
                -MARGIN,
                -MARGIN,
                SVG_SIZE + 2 * MARGIN as usize,
                SVG_SIZE + 2 * MARGIN as usize,
            ),
        )
        .set("width", SVG_SIZE + MARGIN as usize)
        .set("height", SVG_SIZE + MARGIN as usize)
        .set("style", "background-color:#F2F3F5"); // 通常不要

    let d = SVG_SIZE / N;
    for i in 0..N {
        for j in 0..N {
            if (i + j) % 2 == 0 {
                svg = svg.add(rect(i * d, j * d, d, d, "red"));
            } else {
                svg = svg.add(rect(i * d, j * d, d, d, "blue"));
            }
        }
    }

    svg.to_string()
}
```

![Document](/images/Rectangle_stroke.png)

# Circle

円を追加する場合は`Circle`を使用する。使いやすいように中心座標・半径・色を引数として関数にしておくとよい。座標は円の中心となる。枠線は四角形と同様に設定することができる。

```rust
use svg::node::element::Circle;
use svg::Document;

pub fn cir(x: usize, y: usize, r: usize, fill: &str) -> Circle {
    Circle::new()
        .set("cx", x)
        .set("cy", y)
        .set("r", r)
        .set("fill", fill)
}

const SVG_SIZE: usize = 600;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set("viewBox", (0, 0, SVG_SIZE, SVG_SIZE))
        .set("width", SVG_SIZE)
        .set("height", SVG_SIZE)
        .set("style", "background-color:#F2F3F5"); // 通常不要

    svg = svg.add(cir(30, 30, 30, "red"));

    svg.to_string()
}
```
![Document](/images/Circle.png)


# Line

線を追加する場合は`Line`を使用する。使いやすいように始点座標・終点座標・色を引数として関数にしておくとよい。例えば、以下のように2次元グリッド上の経路の軌跡をなどを表現する場合に使うことができる。

```rust
use svg::node::element::{Line, Rectangle};
use svg::Document;

pub fn rect(x: usize, y: usize, w: usize, h: usize, fill: &str) -> Rectangle {
    Rectangle::new()
        .set("x", x)
        .set("y", y)
        .set("width", w)
        .set("height", h)
        .set("fill", fill)
        .set("stroke", "gray")
        .set("stroke-width", 3)
}

pub fn lin(x1: usize, y1: usize, x2: usize, y2: usize, color: &str) -> Line {
    Line::new()
        .set("x1", x1)
        .set("y1", y1)
        .set("x2", x2)
        .set("y2", y2)
        .set("stroke", color)
        .set("stroke-width", 3)
        .set("stroke-linecap", "round")
}

const SVG_SIZE: usize = 600;
const MARGIN: isize = 10;
const N: usize = 20;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set(
            "viewBox",
            (
                -MARGIN,
                -MARGIN,
                SVG_SIZE + 2 * MARGIN as usize,
                SVG_SIZE + 2 * MARGIN as usize,
            ),
        )
        .set("width", SVG_SIZE + MARGIN as usize)
        .set("height", SVG_SIZE + MARGIN as usize)
        .set("style", "background-color:#F2F3F5"); // 通常不要

    let d = SVG_SIZE / N;
    for i in 0..N {
        for j in 0..N {
            if (i + j) % 2 == 0 {
                svg = svg.add(rect(i * d, j * d, d, d, "red"));
            } else {
                svg = svg.add(rect(i * d, j * d, d, d, "blue"));
            }
        }
    }

    let roots = vec![(0, 0), (0, 2), (2, 4), (5, 4)];
    for nodes in roots.windows(2) {
        let (x1, y1) = nodes[0];
        let (x2, y2) = nodes[1];
        svg = svg.add(lin(
            x1 * d + d / 2,
            y1 * d + d / 2,
            x2 * d + d / 2,
            y2 * d + d / 2,
            "lightgray",
        ));
    }

    svg.to_string()
}
```

![Document](/images/Line_roots.png)


破線を表現する場合は、`stroke-dasharray`を設定する。

```rust
pub fn lin(x1: usize, y1: usize, x2: usize, y2: usize, color: &str) -> Line {
    Line::new()
        .set("x1", x1)
        .set("y1", y1)
        .set("x2", x2)
        .set("y2", y2)
        .set("stroke", color)
        .set("stroke-width", 3)
        .set("stroke-linecap", "round")
        .set("stroke-dasharray", 5)
}
```

![Document](/images/Line_roots_dash.png)

# Text

テキストを追加する場合は`Text`を使用する。使いやすいように座標・テキストを引数として関数にしておくとよい。
※`svg::node::element::Text`と`svg::node::Text`の両方を用いるが、同じ名称であるため、テキストを設定する後者を`TextContent`と再定義して設定している。
※座標は上下及び左右中央揃えしている場合は、文字列の中心（していない場合は文字列の左下）

```rust
use svg::node::element::{Rectangle, Text};
use svg::node::Text as TextContent;
use svg::Document;

pub fn rect(x: usize, y: usize, w: usize, h: usize, fill: &str) -> Rectangle {
    Rectangle::new()
        .set("x", x)
        .set("y", y)
        .set("width", w)
        .set("height", h)
        .set("fill", fill)
        .set("stroke", "gray")
        .set("stroke-width", 3)
}

pub fn txt(x: usize, y: usize, text: &str) -> Text {
    Text::new()
        .add(TextContent::new(text))
        .set("x", x)
        .set("y", y)
        .set("fill", "black")
        .set("font-size", 20)
        .set("dominant-baseline", "central") // 上下中央揃え
        .set("text-anchor", "middle") // 左右中央揃え
}

const SVG_SIZE: usize = 600;
const MARGIN: isize = 10;
const N: usize = 20;

pub fn vis() -> String {
    let mut svg = Document::new()
        .set(
            "viewBox",
            (
                -MARGIN,
                -MARGIN,
                SVG_SIZE + 2 * MARGIN as usize,
                SVG_SIZE + 2 * MARGIN as usize,
            ),
        )
        .set("width", SVG_SIZE + MARGIN as usize)
        .set("height", SVG_SIZE + MARGIN as usize)
        .set("style", "background-color:#F2F3F5"); // 通常不要

    let d = SVG_SIZE / N;
    for i in 0..N {
        for j in 0..N {
            if (i + j) % 2 == 0 {
                svg = svg.add(rect(i * d, j * d, d, d, "red"));
            } else {
                svg = svg.add(rect(i * d, j * d, d, d, "blue"));
            }
            // グリッドの中心に調整
            svg = svg.add(txt(i * d + d / 2, j * d + d / 2, "a"));
        }
    }

    svg.to_string()
}
```
![Document](/images/Text.png)

文字列は基本的に、上下及び左右中央揃えで使用する場合が多いので、`Text`毎に設定せずに、`Style`を用いて、以下のようにまとめて設定することもできる。

```rust
use svg::node::element::{Rectangle, Style, Text};
use svg::node::Text as TextContent;
use svg::Document;

svg = svg.add(Style::new(format!(
    "text {{text-anchor: middle; dominant-baseline: central; font-size: {}}}",
    20
)));
```

# Color

図形の色付けは、以下に示すカラーコードの`Color Name`または#で始まるRGBを16進数で表した文字列を`fill`に設定すればよい。
https://itsakura.com/html-color-codes

## RGB

寒色～暖色で数字の大きさをを表現するには、以下の関数を使用する。引数は0から1の値。

```rust
// 0 <= val <= 1
pub fn color(mut val: f64) -> String {
    val = val.min(1.0);
    val = val.max(0.0);
    let (r, g, b) = if val < 0.5 {
        let x = val * 2.0;
        (
            30. * (1.0 - x) + 144. * x,
            144. * (1.0 - x) + 255. * x,
            255. * (1.0 - x) + 30. * x,
        )
    } else {
        let x = val * 2.0 - 1.0;
        (
            144. * (1.0 - x) + 255. * x,
            255. * (1.0 - x) + 30. * x,
            30. * (1.0 - x) + 70. * x,
        )
    };
    format!(
        "#{:02x}{:02x}{:02x}",
        r.round() as i32,
        g.round() as i32,
        b.round() as i32
    )
}
```

例えば、0から50の数字に色を割り当てると以下のようになる。

```rust
let MAX = 50;
let GRID_SIZE = 10;
for i in 0..=MAX {
    svg = svg.add(rect(
        i * GRID_SIZE,
        0,
        GRID_SIZE,
        GRID_SIZE * 4,
        &color(i as f64 / MAX as f64),
    ));
}
```

![Document](/images/Color.png)

## Opacity

透明度で数字の大きさをを表現するには、`opacity`を以下のように設定する。

```rust
let MAX = 50;
let GRID_SIZE = 10;
for i in 0..=MAX {
    svg = svg.add(
        rect(i * GRID_SIZE, 0, GRID_SIZE, GRID_SIZE * 4, "green")
            .set("opacity", i as f64 / MAX as f64),
    );
}
```

![Document](/images/Opacity.png)

`opacity`設定の場合、枠線の透明度も変わってしまうが、`fill-opacity`を使えば、塗りつぶし箇所のみ透明度を変更できる。

```rust
let MAX = 50;
let GRID_SIZE = 10;
for i in 0..=MAX {
    svg = svg.add(
        rect(i * GRID_SIZE, 0, GRID_SIZE, GRID_SIZE * 4, "green")
            .set("fill-opacity", i as f64 / MAX as f64)
            .set("stroke", "black"),
    );
}
```

![Document](/images/FillOpacity.png)

# Tooltip

図形にマウスオーバーした際に、補足情報を表示するツールチップは、以下の`Group`を使用する。補足情報は複雑になりやすいので、`format`マクロを使用すると使いやすい。
※以下を見るとわかるよに、`Group`に`Rectangle`などの図形を`add`している。逆に設定しがちなので注意

```rust
use svg::node::element::{Group, Title};
use svg::node::Text as TextContent;
use svg::Document;

pub fn group(title: String) -> Group {
    Group::new().add(Title::new().add(TextContent::new(title)))
}

let d = SVG_SIZE / N;
for i in 0..N {
    for j in 0..N {
        let mut grp = group(format!("x:{}\ny:{}", i, j));
        if (i + j) % 2 == 0 {
            let rec = rect(i * d, j * d, d, d, "red");
            grp = grp.add(rec);
        } else {
            let rec = rect(i * d, j * d, d, d, "blue");
            grp = grp.add(rec);
        }
        svg = svg.add(grp);
    }
}
```
![Document](/images/Tooltip.png)

# 複雑な図形

四角形や円以外の複雑な図形に関しては、`Data`と`Path`を組み合わせて、設定することができる。例えば三角形を描く場合は以下の通り。
`move_to`で、始点の座標に移動し、`line_by`で今いる箇所を始点として、移動先の相対座標を指定する。終点が始点と同じ場合は、`close`を使えば勝手にやってくれる。

```rust
use svg::node::element::path::Data;
use svg::node::element::Path;

let data = Data::new()
    .move_to((50, 50))
    .line_by((30, 0))
    .line_by((-15, -15))
    .close();
let p = Path::new().set("d", data).set("fill", "green");
svg = svg.add(p);
```
![Document](/images/Triangle.png)

# Rotate

あまり使用する場面はなさそうだが、図形を回転する方法を紹介する。
`transform`で、`rotate`を設定する。
第1引数が回転角度で、第2・第3引数が、回転の中心座標。

```rust
let size = 30.0;
let rec = Rectangle::new()
    .set("x", size * 3.0)
    .set("y", size)
    .set("width", size)
    .set("height", size)
    .set("fill", "green")
    .set(
        "transform",
        format!("rotate({},{},{})", 45, size * 3.5, size * 1.5),
    );
svg = svg.add(rec);
```

![Document](/images/Rotate.png)

# 番外編

ビジュアライザに関連して、AHCで用いられている乱数クレートを用いた入力生成について紹介する。

```toml
[dependencies]
rand = "0.8.5"
rand_chacha = "0.3.1"
rand_distr = "0.4.3"
```

## 入力生成

```rust
let mut rng = ChaCha20Rng::seed_from_u64(0);
println!("{}", rng.gen_range(0..10)); // 0～9
println!("{}", rng.gen_range(-10..=10)); // -10～10
println!("{}", rng.gen_range(0.0..=10.0)); // 0.0～10.0
println!("{}", rng.gen_ratio(50, 100)); // 50/100の確率でtrue
```

## 正規分布

正規分布から値を生成する場合は`Normal`を使用する。
値の上限、下限がある場合は、`max`, `min`, `clamp`で制限する。

```rust
let mut rng = ChaCha20Rng::seed_from_u64(0);
let mean = 50.0;
let std = 5.0;
let normal_dist = Normal::<f64>::new(mean, std).unwrap();
let n = normal_dist.sample(&mut rng).clamp(40.0, 50.0);
println!("{}", n);
```

## 重み付き乱数

以下の例は、50%の確率で'a'、25%の確率で'b'、25%の確率で'c'が出力される。

```rust
let mut rng = ChaCha20Rng::seed_from_u64(0);
let choices = ['a', 'b', 'c'];
let weights = [2, 1, 1];
let dist = WeightedIndex::new(weights).unwrap();
for _ in 0..100 {
    println!("{}", choices[dist.sample(&mut rng)]);
}
```

# まとめ

AHCで提供されているビジュアライザは上記を組み合わせれば、ほぼ作ることができます（3Dに関しては知らんので、そのときは諦めましょう）。マスターズまでビジュアライザ筋トレがんばりましょう！！

AHCのビジュアライザと同様に、ブラウザ表示する方法に関しては、yunixさんの記事を参照してください。Reactがあるので、触ったことがない人には、ちょっと難しそうに見えますが、機能追加したいとかじゃない限り、React部分はそのまま使えそうです。Vercelへのデプロイも簡単でした。
https://yunix-kyopro.hatenablog.com/entry/2023/12/17/150534