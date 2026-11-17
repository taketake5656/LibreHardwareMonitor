# LibreHardwareMonitor（taketake5656 フォーク版）
[![GitHub license](https://img.shields.io/github/license/LibreHardwareMonitor/LibreHardwareMonitor)](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/blob/master/LICENSE) [![Nuget](https://img.shields.io/nuget/v/LibreHardwareMonitorLib)](https://www.nuget.org/packages/LibreHardwareMonitorLib/) [![Nuget (with prereleases)](https://img.shields.io/nuget/vpre/LibreHardwareMonitorLib?label=nuget-pre)](https://www.nuget.org/packages/LibreHardwareMonitorLib/) [![Nuget](https://img.shields.io/nuget/dt/LibreHardwareMonitorLib?label=nuget-downloads)](https://www.nuget.org/packages/LibreHardwareMonitorLib/)

> **このリポジトリについて**
> 本家 [LibreHardwareMonitor/LibreHardwareMonitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor) の v0.9.6 をベースにした、自分用の修正版です。

## このフォークでの変更点
### デスクトップガジェットがスリープ復帰時にクラッシュする問題の修正
「Show Gadget」を有効にしていると、スリープからの復帰時にときどき次の例外で落ちる問題を修正しました。

```
System.InvalidOperationException: Object is currently in use elsewhere.
   at LibreHardwareMonitor.UI.SensorGadget.OnPaint(...)
```

- **原因**：復帰直後にハードウェアが作り直されると、センサーの追加・削除イベントがバックグラウンドの更新スレッドで発生します。このときガジェット（`SensorGadget`）がそのスレッドでサイズ変更と再描画を行い、UI スレッドのタイマー描画と同じ GDI+ の `Graphics` を同時に使っていました。
- **修正内容**：`SensorGadget` で受け取るハードウェア・センサーの追加・削除イベントを、`SynchronizationContext.Post` で UI スレッドに回して処理するようにしました（`LibreHardwareMonitor/UI/SensorGadget.cs`）。

### GitHub Actions でのビルド
`.github/workflows/fork-build.yml` を追加しました。`fix/gadget-thread-safety` ブランチへの push、または手動実行（workflow_dispatch）でビルドされます。

- ビルド結果は [Actions](https://github.com/taketake5656/LibreHardwareMonitor/actions) の各実行ページの「Artifacts」からダウンロードできます（保持期間 90 日）。
  - `LibreHardwareMonitor-gadgetfix`：.NET Framework 4.7.2 版（通常はこちら）
  - `LibreHardwareMonitor-gadgetfix (.NET 10)`：.NET 10 版

---

以下は本家 README の日本語訳です。

## 概要
Libre Hardware Monitor は、コンピューターの温度センサー、ファン回転数、電圧、負荷、クロック周波数を監視できるフリーソフトウェアです。

## 含まれるもの
| 名前 | .NET | ビルド状況 |
| --- | --- | --- |
| **LibreHardwareMonitor** <br /> すべてのデータをグラフィカルな画面で表示する Windows Forms アプリケーション | .NET Framework 4.7.2 <br/> .NET 10.0 | [![Build status](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/actions/workflows/master.yml/badge.svg)](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/actions) |
| **LibreHardwareMonitorLib** <br /> すべての機能を自分のアプリケーションで使えるようにするライブラリ | .NET Framework 4.7.2 <br/> .NET Standard 2.0 <br/> .NET 8.0、.NET 9.0、.NET 10.0 | [![Build status](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/actions/workflows/master.yml/badge.svg)](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/actions) |

## できること
次のようなデバイスから情報を読み取れます。
- マザーボード
- Intel と AMD のプロセッサー
- NVIDIA、AMD、Intel のグラフィックカード
- HDD、SSD、NVMe ドライブ
- ネットワークカード

## ダウンロード
本家の最新リリースは[こちら](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/releases)からダウンロードできます。WinGet でもインストールできます。

```
winget install LibreHardwareMonitor.LibreHardwareMonitor
```

### ナイトリービルド
**GitHub** アカウントがあれば、ナイトリービルドを[こちら](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/actions)からダウンロードできます。アカウントがない場合は、*最新の*ナイトリービルドを[こちら](https://nightly.link/LibreHardwareMonitor/LibreHardwareMonitor/workflows/master/master/LibreHardwareMonitor.zip)からダウンロードできます。

## 改善に協力するには
LibreHardwareMonitor チームはフィードバックや貢献を歓迎しています。<br/>
お使いのマザーボードで正しく動作するか確認してみてください。メーカーによってデータの読み取り方が少しずつ異なるため、不正確な点に気づいたらプルリクエストを送ってください。提案や改善案があれば、気軽に Issue を作成してください。

## 開発者向け情報
**自分のアプリケーションにライブラリを組み込む**
1. [LibreHardwareMonitorLib](https://www.nuget.org/packages/LibreHardwareMonitorLib/) NuGet パッケージをアプリケーションに追加します。
2. 下のサンプルコードを使います。


**サンプルコード**
```c#
public class UpdateVisitor : IVisitor
{
    public void VisitComputer(IComputer computer)
    {
        computer.Traverse(this);
    }
    public void VisitHardware(IHardware hardware)
    {
        hardware.Update();
        foreach (IHardware subHardware in hardware.SubHardware) subHardware.Accept(this);
    }
    public void VisitSensor(ISensor sensor) { }
    public void VisitParameter(IParameter parameter) { }
}

public void Monitor()
{
    Computer computer = new Computer
    {
        IsCpuEnabled = true,
        IsGpuEnabled = true,
        IsMemoryEnabled = true,
        IsMotherboardEnabled = true,
        IsControllerEnabled = true,
        IsNetworkEnabled = true,
        IsStorageEnabled = true
    };

    computer.Open();
    computer.Accept(new UpdateVisitor());

    foreach (IHardware hardware in computer.Hardware)
    {
        Console.WriteLine("Hardware: {0}", hardware.Name);
        
        foreach (IHardware subhardware in hardware.SubHardware)
        {
            Console.WriteLine("\tSubhardware: {0}", subhardware.Name);
            
            foreach (ISensor sensor in subhardware.Sensors)
            {
                Console.WriteLine("\t\tSensor: {0}, value: {1}", sensor.Name, sensor.Value);
            }
        }

        foreach (ISensor sensor in hardware.Sensors)
        {
            Console.WriteLine("\tSensor: {0}, value: {1}", sensor.Name, sensor.Value);
        }
    }
    
    computer.Close();
}
```

**管理者権限**

一部のセンサーは、データの取得に管理者権限が必要です。IDE を管理者権限で再起動するか、プロジェクトに [app.manifest](https://learn.microsoft.com/en-us/windows/win32/sbscs/application-manifests) ファイルを追加して requestedExecutionLevel を requireAdministrator に設定してください。


## ライセンス
LibreHardwareMonitor は MPL 2.0 でライセンスされた、自由でオープンソースのソフトウェアです。一部は別の条件でライセンスされています。[THIRD-PARTY-LICENSES](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/blob/master/THIRD-PARTY-NOTICES.txt) を参照してください。
