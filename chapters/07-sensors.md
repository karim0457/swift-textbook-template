# 第7章：センサーの活用

> 執筆者：カリム
> 最終更新：2026-07-03

## この章で学ぶこと

この章では、CoreMotionを利用してiPhoneに搭載されているモーションセンサーの情報を取得する方法を学んだ。端末の傾き（Pitch・Roll・Yaw）をリアルタイムで取得し、水平器アプリとして表示する方法や、センサーデータを画面へ反映する仕組みを理解した。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第7章（基本）：加速度センサーで動く水平器アプリ
// ============================================
// CoreMotionを使って端末の傾きをリアルタイムで取得し、
// 水平器（水準器）として表示するアプリです。
//
// 【注意】シミュレータではセンサーが使えません。
//         実機（iPhone / iPad）でテストしてください。
// ============================================

import SwiftUI
import CoreMotion

// MARK: - モーションマネージャー

@Observable
class MotionManager {
    private let motionManager = CMMotionManager()

    var pitch: Double = 0    // 前後の傾き
    var roll: Double = 0     // 左右の傾き
    var yaw: Double = 0      // 水平方向の回転
    var isAvailable: Bool = false

    func startUpdates() {
        guard motionManager.isDeviceMotionAvailable else {
            isAvailable = false
            return
        }

        isAvailable = true
        motionManager.deviceMotionUpdateInterval = 1.0 / 60.0

        motionManager.startDeviceMotionUpdates(to: .main) { [weak self] motion, error in
            guard let self = self, let motion = motion else { return }

            self.pitch = motion.attitude.pitch
            self.roll = motion.attitude.roll
            self.yaw = motion.attitude.yaw
        }
    }

    func stopUpdates() {
        motionManager.stopDeviceMotionUpdates()
    }
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var motionManager = MotionManager()

    var body: some View {
        NavigationStack {
            if motionManager.isAvailable {
                VStack(spacing: 30) {
                    // 水平器の円
                    LevelIndicator(
                        pitch: motionManager.pitch,
                        roll: motionManager.roll
                    )

                    // 数値表示
                    DataDisplay(
                        pitch: motionManager.pitch,
                        roll: motionManager.roll,
                        yaw: motionManager.yaw
                    )
                }
                .padding()
                .navigationTitle("水平器")
            } else {
                ContentUnavailableView(
                    "センサーが利用できません",
                    systemImage: "iphone.slash",
                    description: Text("このアプリは実機（iPhone）で動作します。\nシミュレータではセンサーが使えません。")
                )
            }
        }
        .onAppear {
            motionManager.startUpdates()
        }
        .onDisappear {
            motionManager.stopUpdates()
        }
    }
}

// MARK: - 水平器インジケーター

struct LevelIndicator: View {
    let pitch: Double
    let roll: Double

    private let maxOffset: CGFloat = 100

    private var xOffset: CGFloat {
        CGFloat(roll) * maxOffset
    }

    private var yOffset: CGFloat {
        CGFloat(pitch) * maxOffset
    }

    private var isLevel: Bool {
        abs(pitch) < 0.03 && abs(roll) < 0.03
    }

    var body: some View {
        ZStack {
            // 外側の円
            Circle()
                .stroke(.gray.opacity(0.3), lineWidth: 2)
                .frame(width: 250, height: 250)

            // 中心の十字線
            Path { path in
                path.move(to: CGPoint(x: 125, y: 0))
                path.addLine(to: CGPoint(x: 125, y: 250))
                path.move(to: CGPoint(x: 0, y: 125))
                path.addLine(to: CGPoint(x: 250, y: 125))
            }
            .stroke(.gray.opacity(0.2), lineWidth: 1)
            .frame(width: 250, height: 250)

            // 中間の円
            Circle()
                .stroke(.gray.opacity(0.2), lineWidth: 1)
                .frame(width: 125, height: 125)

            // バブル（傾きに応じて移動）
            Circle()
                .fill(isLevel ? .green : .red)
                .frame(width: 40, height: 40)
                .opacity(0.8)
                .shadow(color: isLevel ? .green : .red, radius: 8)
                .offset(
                    x: max(-maxOffset, min(maxOffset, xOffset)),
                    y: max(-maxOffset, min(maxOffset, yOffset))
                )
                .animation(.spring(duration: 0.1), value: xOffset)
                .animation(.spring(duration: 0.1), value: yOffset)

            // 水平時の表示
            if isLevel {
                Text("水平!")
                    .font(.headline)
                    .foregroundStyle(.green)
                    .offset(y: 140)
            }
        }
    }
}

// MARK: - 数値データ表示

struct DataDisplay: View {
    let pitch: Double
    let roll: Double
    let yaw: Double

    var body: some View {
        VStack(spacing: 12) {
            DataRow(
                label: "前後の傾き（Pitch）",
                value: pitch,
                icon: "arrow.up.and.down"
            )
            DataRow(
                label: "左右の傾き（Roll）",
                value: roll,
                icon: "arrow.left.and.right"
            )
            DataRow(
                label: "水平回転（Yaw）",
                value: yaw,
                icon: "arrow.triangle.2.circlepath"
            )
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 12)
                .fill(.gray.opacity(0.05))
        )
    }
}

struct DataRow: View {
    let label: String
    let value: Double
    let icon: String

    var body: some View {
        HStack {
            Image(systemName: icon)
                .frame(width: 30)
                .foregroundStyle(.blue)

            Text(label)
                .font(.caption)

            Spacer()

            Text(String(format: "%.3f rad", value))
                .font(.system(.caption, design: .monospaced))
                .foregroundStyle(.secondary)

            Text(String(format: "(%.1f°)", value * 180 / .pi))
                .font(.system(.caption, design: .monospaced))
                .foregroundStyle(.secondary)
                .frame(width: 60, alignment: .trailing)
        }
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、iPhoneのモーションセンサーを利用した水平器アプリである。端末を傾けると、画面上の丸いバブルが傾きに応じて動き、水平になると緑色に変化して「水平！」と表示される。また、Pitch・Roll・Yawの数値もリアルタイムで確認できる。

## コードの詳細解説

### CoreMotionの基本（CMMotionManager）

```swift
let motionManager = CMMotionManager() motionManager.startDeviceMotionUpdates(to: .main) { [weak self] motion, error in guard let self = self, let motion = motion else { return } self.pitch = motion.attitude.pitch self.roll = motion.attitude.roll self.yaw = motion.attitude.yaw }
```

**何をしているか：**
CMMotionManagerを使って端末のモーションセンサーを起動し、リアルタイムで傾きの情報を取得している。

**なぜこう書くのか：**
（CoreMotionの機能はCMMotionManagerが管理しているため、このクラスを利用してセンサー情報を取得する必要がある。また、.mainで更新することで取得した値をすぐ画面へ反映できる。

**もしこう書かなかったら：**
センサーが起動しないため、Pitch・Roll・Yawの値は更新されず、水平器として動作しない。

---

### デバイスの姿勢データ（pitch/roll/yaw）

```swift
self.pitch = motion.attitude.pitch self.roll = motion.attitude.roll self.yaw = motion.attitude.yaw
```

**何をしているか：**
端末の前後・左右・回転方向の傾きを取得している。

Pitch：前後の傾き
Roll：左右の傾き
Yaw：水平方向の回転
**なぜこう書くのか：**
取得した値を@Observableの変数へ保存することで、値が変化すると画面も自動で更新される。
**もしこう書かなかったら：**
センサーの値を取得できても画面へ表示されず、水平器は動かない。
---

### 歩数計（CMPedometer）

```swift
private let pedometer = CMPedometer() if isPedometerAvailable { pedometer.startUpdates(from: Date()) { [weak self] data, error in guard let self = self, let data = data else { return } DispatchQueue.main.async { self.stepCount = data.numberOfSteps.intValue if let dist = data.distance { self.distance = dist.doubleValue } } } }
```

**何をしているか：**
CMPedometerを利用して、歩数と移動距離をリアルタイムで取得している。取得した歩数はstepCountに、移動距離はdistanceに保存し、画面へ表示している。
**なぜこう書くのか：**
CMPedometerはiPhoneのモーションセンサーを利用して歩数や歩行距離を取得するためのクラスである。startUpdates()を使うことで、歩数が増えるたびに最新のデータを受け取ることができる。また、DispatchQueue.main.asyncを利用してUIを安全に更新している。
**もしこう書かなかったら：**
CMPedometerを使用しなければ歩数や移動距離を取得できず、活動トラッカーとして機能しない。また、startUpdates()を呼ばなければデータは更新されず、歩数は常に0のままになる。
---

### CoreLocationとの連携

```swift
private let locationManager = CLLocationManager() override init() { super.init() locationManager.delegate = self locationManager.desiredAccuracy = kCLLocationAccuracyBest locationManager.requestWhenInUseAuthorization() } locationManager.startUpdatingLocation() func locationManager( _ manager: CLLocationManager, didUpdateLocations newLocations: [CLLocation] ) { guard let location = newLocations.last else { return } currentSpeed = max(0, location.speed) locations.append(location.coordinate) }
```

**何をしているか：**
CoreLocationを利用して現在地や移動速度を取得している。位置情報が更新されるたびに最新の位置を受け取り、移動速度をcurrentSpeedへ保存するとともに、移動した地点の座標をlocationsへ記録している。
**なぜこう書くのか：**
歩数だけでは実際の移動状況を正確に把握できないためである。CoreLocationを組み合わせることで、現在の移動速度や移動経路を取得できる。また、CLLocationManagerDelegateを利用することで位置情報が更新されるたびに自動で処理を実行できる。
**もしこう書かなかったら：**
位置情報を取得できなくなり、現在の速度を表示できない。また、移動経路も記録できなくなるため、活動トラッカーとして取得できる情報が歩数のみになってしまう。
---

（必要に応じてセクションを増やす）
## 新しく学んだSwiftの文法・API

| 項目                          | 説明                           | 使用例                                                         |
| --------------------------- | ---------------------------- | ----------------------------------------------------------- |
| `CMMotionManager`           | デバイスの姿勢や加速度などのモーションセンサーを取得する | `motionManager.startDeviceMotionUpdates(to: .main) { ... }` |
| `CMPedometer`               | 歩数や歩行距離を取得する                 | `pedometer.startUpdates(from: Date()) { ... }`              |
| `CLLocationManager`         | 現在地や速度などの位置情報を取得する           | `locationManager.startUpdatingLocation()`                   |
| `CLLocationManagerDelegate` | 位置情報が更新されたときの処理を実装する         | `func locationManager(_:didUpdateLocations:)`               |
| `startUpdatingLocation()`   | GPSによる位置情報の取得を開始する           | `locationManager.startUpdatingLocation()`                   |
| `stopUpdatingLocation()`    | 位置情報の取得を停止する                 | `locationManager.stopUpdatingLocation()`                    |
| `DispatchQueue.main.async`  | メインスレッドでUIを安全に更新する           | `DispatchQueue.main.async { ... }`                          |
| `Timer.publish()`           | 一定時間ごとに処理を実行するタイマーを作成する      | `Timer.publish(every: 1, on: .main, in: .common)`           |
| `onReceive()`               | タイマーなどから受け取ったイベントを処理する       | `.onReceive(timer) { _ in ... }`                            |
| `@Observable`               | データの変更を画面へ自動反映する             | `@Observable class ActivityTracker`                         |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：歩数計測を開始したまま実際に少し歩いてみた。
- 結果：歩数・移動距離・消費カロリーがリアルタイムで更新された。
- わかったこと：CMPedometerは歩くたびにデータを取得し、自動で画面へ反映できることが分かった。

**実験2：**
- やったこと：locationManager.stopUpdatingLocation()をコメントアウトして実行した。
- 結果：計測を停止しても位置情報の取得が続いた。
- わかったこと：不要な位置情報の取得を止めることで、バッテリー消費を抑えることが重要だと理解した。

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**なぜCMPedometerとCoreLocationを両方使うのですか？
   **得られた理解：**
CMPedometerは歩数や歩行距離を取得し、CoreLocationは現在地や移動速度を取得する。それぞれ役割が異なるため、組み合わせることでより詳しい活動データを記録できる。
2. **質問：**なぜDispatchQueue.main.asyncでUIを更新する必要がありますか？
   **得られた理解：**
歩数データはバックグラウンドで取得されるため、そのまま画面を更新すると問題が起こる可能性がある。メインスレッドで更新することで、安全に画面へ反映できる。
3. **質問：**なぜ位置情報はCLLocationManagerDelegateで受け取るのですか？
   **得られた理解：**
位置情報はいつ更新されるか分からないため、Delegateを利用して更新されたタイミングで自動的に受け取る仕組みになっていることが分かった。
## この章のまとめ

この章では、CoreMotionのCMPedometerとCoreLocationを組み合わせて、歩数・移動距離・速度・消費カロリーを記録する活動トラッカーを作成した。それぞれのセンサーには異なる役割があり、組み合わせることで実用的なアプリを開発できることを学んだ。また、位置情報やモーションセンサーは適切なタイミングで開始・停止することが、バッテリー消費を抑えるために重要であることも理解できた。
