# 第1章：WebAPIの基本

> 執筆者：カリム
> 最終更新：26/04/15

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、インターネット上のサービス（API）からデータを取得して、アプリ内に表示する方法を学ぶ。具体的にはiTunes Search APIを使って音楽を検索し、その結果をリスト表示するアプリを題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
import SwiftUI

// MARK: - データモデル

struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var songs: [Song] = []
    @State private var searchText: String = ""
    @State private var isLoading: Bool = false

    var body: some View {
        NavigationStack {
            VStack {
                // 検索バー
                HStack {
                    TextField("アーティスト名を入力", text: $searchText)
                        .textFieldStyle(.roundedBorder)

                    Button("検索") {
                        Task {
                            await searchMusic()
                        }
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(searchText.isEmpty)
                }
                .padding(.horizontal)

                // 検索結果リスト
                if isLoading {
                    ProgressView("検索中...")
                        .padding()
                    Spacer()
                } else if songs.isEmpty {
                    ContentUnavailableView(
                        "曲を検索してみよう",
                        systemImage: "music.note",
                        description: Text("アーティスト名を入力して検索ボタンを押してください")
                    )
                } else {
                    List(songs) { song in
                        SongRow(song: song)
                    }
                }
            }
            .navigationTitle("Music Search")
        }
    }

    // MARK: - API通信

    func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        guard let url = URL(string: urlString) else { return }

        isLoading = true

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")
            songs = []
        }

        isLoading = false
    }
}

// MARK: - 曲の行ビュー

struct SongRow: View {
    let song: Song

    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: song.artworkUrl100)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.gray.opacity(0.3)
            }
            .frame(width: 60, height: 60)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 4) {
                Text(song.trackName)
                    .font(.headline)
                    .lineLimit(1)

                Text(song.artistName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

音楽を無料で探せるアプリ

## コードの詳細解説

### データモデル（Codable構造体）

```swift
struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}
```

**何をしているか：**
このコードでは、APIから取得したJSONデータをSwiftで扱える形に変換するためのデータモデルを定義していま

**なぜこう書くのか：**
APIから返されるデータはJSON形式であるため、そのままではSwiftで扱うことができません。
そのため、Codableプロトコルを使うことで、JSONとSwiftの構造体を自動的に変換できるようにしています。

**もしこう書かなかったら：**
Codableを使わない場合、JSONを手動で解析する必要があり、コードが複雑になります。

---

### API通信の処理

```swift
func searchMusic() async {
    guard let encodedText = searchText.addingPercentEncoding(
        withAllowedCharacters: .urlQueryAllowed
    ) else { return }

    let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

    guard let url = URL(string: urlString) else { return }

    isLoading = true

    do {
        let (data, _) = try await URLSession.shared.data(from: url)
        let response = try JSONDecoder().decode(SearchResponse.self, from: data)
        songs = response.results
    } catch {
        print("エラー: \(error.localizedDescription)")
        songs = []
    }

    isLoading = false
}
```

**何をしているか：**
このコードでは、iTunes APIにリクエストを送り、音楽データを取得してsongsに格納しています。
**なぜこう書くのか：**
ネットワーク通信は時間がかかる処理であるため、async/awaitを使って非同期処理として実装しています。
**もしこう書かなかったら：**
非同期処理を使わない場合、アプリがフリーズしたように見えてしまい、ユーザー体験が悪くなります。
---

### ビューの構成

```swift
NavigationStack {
    VStack {
        HStack {
            TextField("アーティスト名を入力", text: $searchText)

            Button("検索") {
                Task {
                    await searchMusic()
                }
            }
        }

        if isLoading {
            ProgressView("検索中...")
        } else if songs.isEmpty {
            ContentUnavailableView(
                "曲を検索してみよう",
                systemImage: "music.note"
            )
        } else {
            List(songs) { song in
                SongRow(song: song)
            }
        }
    }
}
```

**何をしているか：**
このコードでは、検索画面のUIを構築しています。
**なぜこう書くのか：**
SwiftUIでは、状態に応じてUIを動的に変更する「宣言的UI」の考え方が採用されています。
**もしこう書かなかったら：**
条件分岐を使わない場合、ローディング表示や検索結果などが同時に表示され、UIが分かりにくくなります。
---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Codable` | JSONデータとSwiftの構造体を相互変換するプロトコル | `struct Song: Codable { ... }` |
| 例：`async/await` | 非同期処理を同期的に書ける構文 | `let data = try await URLSession.shared.data(from: url)` |
| 例：`async/await` | 非同期処理を同期的に書ける構文 | `let data = try await URLSession.shared.data(from: url)` |
| 例：`Identifiable` | 一意なIDを持つことでList表示が可能になる  | `struct Song: Identifiable`         |
| 例：`@State`       | View内で状態を管理し、変更時にUIを更新する | `@State private var songs: [Song]`  |
| 例：`URLSession`   | API通信を行うためのクラス           | `URLSession.shared.data(from: url)` |
| 例：`JSONDecoder`  | JSONをSwiftの構造体に変換する      | `JSONDecoder().decode(...)`         |
| 例：`AsyncImage`   | URLから画像を非同期で読み込む         | `AsyncImage(url: URL(...))`         |


## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
やったこと：検索ワードを空にして検索ボタンを押した
結果：ボタンが無効化されており、検索できなかった
わかったこと：.disabled(searchText.isEmpty)で入力チェックができる

**実験2：**
- やったこと：存在しないアーティスト名を入力
- 結果：検索したとき何も出てこなかった、画面が変わらなかった
- わかったこと：不明な名前書けば対応できない

## AIに聞いて特に理解が深まった質問 TOP3

1. 質問：なぜCodableが必要なのか？
得られた理解：JSONとSwiftのデータを自動変換するために必要である。

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
