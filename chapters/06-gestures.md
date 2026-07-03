# 第6章：ジェスチャー操作

> 執筆者：カリム
> 最終更新：2026-07-03

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

この章では、SwiftUIで利用できる代表的なジェスチャー操作について学んだ。タップ・ロングプレス・ドラッグ・拡大縮小・回転の実装方法を理解し、それぞれの状態管理や複数のジェスチャーを組み合わせる方法を確認した。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ここに模範コード全体を貼る
// ============================================
// 第6章（基本）：ジェスチャーで操作するカードアプリ
// ============================================
// タップ、ロングプレス、ドラッグ、ピンチ、回転の
// 各ジェスチャーを実際に体験しながら学びます。
// ============================================

import SwiftUI

// MARK: - メインビュー

struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("タップ & ロングプレス") {
                    TapDemoView()
                }
                NavigationLink("ドラッグ") {
                    DragDemoView()
                }
                NavigationLink("ピンチ（拡大縮小）") {
                    MagnifyDemoView()
                }
                NavigationLink("回転") {
                    RotateDemoView()
                }
                NavigationLink("組み合わせ") {
                    CombinedDemoView()
                }
            }
            .navigationTitle("ジェスチャー体験")
        }
    }
}

// MARK: - タップ & ロングプレス

struct TapDemoView: View {
    @State private var tapCount = 0
    @State private var backgroundColor: Color = .blue
    @State private var isPressed = false

    var body: some View {
        VStack(spacing: 30) {
            Text("タップ回数: \(tapCount)")
                .font(.title)

            // シングルタップ
            RoundedRectangle(cornerRadius: 16)
                .fill(backgroundColor)
                .frame(width: 200, height: 200)
                .overlay {
                    Text("タップしてね")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .onTapGesture {
                    tapCount += 1
                    backgroundColor = Color(
                        hue: Double.random(in: 0...1),
                        saturation: 0.7,
                        brightness: 0.9
                    )
                }

            // ロングプレス
            Circle()
                .fill(isPressed ? .green : .orange)
                .frame(width: 120, height: 120)
                .scaleEffect(isPressed ? 1.3 : 1.0)
                .overlay {
                    Text(isPressed ? "成功!" : "長押し")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .animation(.spring(duration: 0.3), value: isPressed)
                .onLongPressGesture(minimumDuration: 1.0) {
                    isPressed = true
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        isPressed = false
                    }
                }
        }
        .navigationTitle("タップ & ロングプレス")
    }
}

// MARK: - ドラッグ

struct DragDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero

    var body: some View {
        VStack {
            Text("カードをドラッグしてみよう")
                .font(.headline)
                .padding()

            Spacer()

            RoundedRectangle(cornerRadius: 20)
                .fill(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(width: 200, height: 280)
                .shadow(radius: 8)
                .overlay {
                    VStack {
                        Image(systemName: "hand.draw")
                            .font(.system(size: 40))
                        Text("ドラッグ")
                            .font(.title3)
                    }
                    .foregroundStyle(.white)
                }
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ドラッグ")
    }
}

// MARK: - ピンチ（拡大縮小）

struct MagnifyDemoView: View {
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0

    var body: some View {
        VStack {
            Text("ピンチで拡大縮小")
                .font(.headline)
                .padding()

            Text(String(format: "倍率: %.1fx", scale))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "star.fill")
                .font(.system(size: 100))
                .foregroundStyle(.yellow)
                .scaleEffect(scale)
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    scale = 1.0
                    lastScale = 1.0
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ピンチ")
    }
}

// MARK: - 回転

struct RotateDemoView: View {
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("2本指で回転")
                .font(.headline)
                .padding()

            Text(String(format: "角度: %.0f°", angle.degrees))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "arrow.up")
                .font(.system(size: 80))
                .foregroundStyle(.red)
                .rotationEffect(angle)
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("回転")
    }
}

// MARK: - 組み合わせ

struct CombinedDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("ドラッグ・ピンチ・回転を同時に")
                .font(.headline)
                .padding()

            Spacer()

            Image(systemName: "photo.artframe")
                .font(.system(size: 120))
                .foregroundStyle(.indigo)
                .scaleEffect(scale)
                .rotationEffect(angle)
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                    scale = 1.0
                    lastScale = 1.0
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("組み合わせ")
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

（アプリの動作を自分の言葉で説明する。スクリーンショットを貼ってもよい。）
このアプリは、SwiftUIで使える基本的なジェスチャーを実際に体験できるサンプルアプリである。メニューから各デモ画面へ移動し、それぞれのジェスチャーの動きを確認できる。

タップすると色が変わり、回数が表示される
長押しすると色とサイズが変化する
ドラッグでカードを移動できる
ピンチ操作で画像を拡大・縮小できる
回転ジェスチャーで画像を回転できる
最後はドラッグ・拡大縮小・回転を同時に利用できる

## コードの詳細解説

### 基本ジェスチャー（タップ、ロングプレス）

```swift
// 該当部分のコードを抜粋して貼る
.onTapGesture { tapCount += 1 backgroundColor = Color( hue: Double.random(in: 0...1), saturation: 0.7, brightness: 0.9 ) } .onLongPressGesture(minimumDuration: 1.0) { isPressed = true DispatchQueue.main.asyncAfter(deadline: .now() + 1) { isPressed = false } }
```

**何をしているか：**
onTapGestureはユーザーが1回タップしたことを検知し、タップ回数を増やすと同時に背景色をランダムに変更している。
onLongPressGestureは1秒以上押されたことを検知し、円の色を緑色に変更してサイズを大きくし、1秒後に元へ戻している。

**なぜこう書くのか：**
SwiftUIではジェスチャーをViewへ直接追加できるため、コードがシンプルになる。また、@Stateを利用することで値が変更された瞬間に画面が自動更新される。

**もしこう書かなかったら：**
onTapGestureがなければタップしても何も起こらない。

onLongPressGestureを削除すると長押ししても反応せず、DispatchQueueを使わなければ一度緑色になったまま戻らなくなる。

---

### ドラッグジェスチャーとオフセット管理

```swift
DragGesture() .onChanged { value in offset = CGSize( width: lastOffset.width + value.translation.width, height: lastOffset.height + value.translation.height ) } .onEnded { _ in lastOffset = offset }
```

**何をしているか：**
カードを指で動かした距離を取得し、その分だけ表示位置を変更している。
ドラッグ終了後には現在位置をlastOffsetへ保存して、次回もその位置からドラッグできるようにしている。
**なぜこう書くのか：**
ドラッグ中の移動量(translation)は毎回リセットされるため、以前の位置(lastOffset)を保持して足し合わせる必要がある。
**もしこう書かなかったら：**
lastOffsetを保存しない場合、ドラッグするたびにカードが元の位置へ戻ってしまい、不自然な動きになる。
---

### 拡大縮小と回転

```swift
// 該当部分のコードを抜粋して貼る
MagnifyGesture() .onChanged { value in scale = lastScale * value.magnification } .onEnded { _ in lastScale = scale } RotateGesture() .onChanged { value in angle = lastAngle + value.rotation } .onEnded { _ in lastAngle = angle }
```

**何をしているか：**
ピンチ操作で画像の倍率を変更し、回転ジェスチャーでは画像の角度を変更している。

どちらもジェスチャー終了時に現在の状態を保存している。
**なぜこう書くのか：**
毎回現在の倍率や角度を保存することで、次の操作でも前回の状態を維持したまま続けて操作できる。
**もしこう書かなかったら：**
保存しない場合は、ジェスチャーが終わるたびに倍率や角度が初期状態へ戻ったり、不自然な動きになったりする。
---

### ジェスチャーの組み合わせとアニメーション

```swift
// 該当部分のコードを抜粋して貼る
.scaleEffect(scale)
.rotationEffect(angle)
.offset(offset)
.gesture(DragGesture())
.gesture(MagnifyGesture())
.gesture(RotateGesture())
withAnimation(.spring) {
    offset = .zero
    scale = 1.0
    angle = .zero
}
```

**何をしているか：**
1つの画像にドラッグ・拡大縮小・回転を同時に適用している。

リセットボタンではwithAnimationを利用して滑らかに初期位置へ戻している。
**なぜこう書くのか：**
現実の写真アプリなどでは、移動・拡大・回転を自由に組み合わせられるため、そのようなUIを実現できる。

アニメーションを付けることで自然な操作感になる。
**もしこう書かなかったら：**
アニメーションがない場合は一瞬で元に戻るため、動きが不自然になる。

ジェスチャーを組み合わせなければ、それぞれ別々の画面でしか操作できない。
---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

項目	　　　　　　　　　　説明	　　　　　　　　　　　　　　　　使用例
onTapGesture　　　　	タップを検知する	　　　　　　　　　　　.onTapGesture { ... }
onLongPressGesture	長押しを検知する	　　　　　　　　　　　.onLongPressGesture(minimumDuration: 1.0)
DragGesture	　　　　　ドラッグ操作を検知する	　　　　　　　.gesture(DragGesture())
MagnifyGesture	　　　ピンチによる拡大・縮小を検知する	　.gesture(MagnifyGesture())
RotateGesture	　　　　回転操作を検知する	　　　　　　　　.gesture(RotateGesture())
offset()	　　　　　　　Viewの位置を変更する	　　　　　　　.offset(offset)
scaleEffect()	　　　　　Viewを拡大・縮小する	　　　　　　.scaleEffect(scale)
rotationEffect()	　　Viewを回転する	　　　　　　　　　　.rotationEffect(angle)
withAnimation()	　　　アニメーション付きで状態を変更する	　　withAnimation(.spring){...}
@State	　　　　　　　　画面の状態を保持する	　　　　　　　　@State private var scale = 1.0

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：withAnimation(.spring)を削除した。
- 結果：リセット時に画像が一瞬で元の位置へ戻った。
- わかったこと：withAnimationを使うことで自然なアニメーションを追加できる。

**実験2：**
- やったこと：minimumDurationを1.0秒から2.0秒へ変更した。
- 結果：長押しを2秒以上続けないと反応しなくなった。
- わかったこと：minimumDurationで長押し時間を自由に設定できる。

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**@Stateはなぜ必要なのか？
   **得られた理解：**@Stateを付けることで値が変更された瞬間にSwiftUIが画面を再描画するため、UIとデータを簡単に同期できる。

2. **質問：**なぜlastOffsetやlastScaleを保存する必要があるのか？
   **得られた理解：**ジェスチャーで取得できる値は現在の操作分だけなので、以前の位置や倍率を保持しておかないと途中からリセットされてしまう。

3. **質問：**withAnimationは何をしているのか？
   **得られた理解：**値の変化を一定時間かけて表示するため、急な変化ではなく滑らかなアニメーションになる。

## この章のまとめ

（この章では、SwiftUIの基本的なジェスチャー操作を学び、タップ・長押し・ドラッグ・拡大縮小・回転の実装方法を理解した。また、@Stateを利用して状態を管理し、withAnimationによって自然な動きを実現できることも学んだ。複数のジェスチャーを組み合わせることで、実際のアプリに近い操作性を実現できることが分かった。）
