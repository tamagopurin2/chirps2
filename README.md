# CHIRPS2降水データ解析・可視化スクリプト

このPythonスクリプトは、`xarray`、`matplotlib`、`cartopy`ライブラリを用いて、CHIRPS2年降水データを処理・可視化します。NetCDFファイルを読み込み、特定の地域のデータを抽出、平均降水量を計算し、地図を作成し、処理済みデータをGeoTIFFファイルとしてエクスポートします。

## 目次

1. [はじめに](#はじめに)
2. [必要な環境](#必要な環境)
3. [使い方](#使い方)
4. [コード解説](#コード解説)
5. [ライセンス](#ライセンス)
6. [謝辞](#謝辞)

## 1. はじめに

このスクリプトは、CHIRPS2年降水データを解析・可視化します。以下の処理を行います。

* NetCDFファイルから降水データを読み込む。
* ユーザー定義の緯度経度範囲でデータを抽出する。
* 各グリッドセルについて平均降水量を計算する。
* `cartopy`を用いて平均降水量の地図を作成する。
* 処理された降水データをGeoTIFFファイルとしてエクスポートする。

## 2. 必要な環境

スクリプトを実行する前に、Python3とJupyterNotebookの環境を構築してください。
その後以下のライブラリをDLします。

* `xarray`: NetCDFデータの読み込みと操作。
* `matplotlib`: 可視化用。
* `cartopy`: 地理空間データ処理と地図プロット用。
* `netCDF4`: NetCDFファイルの読み込み (通常は`xarray`と一緒にインストールされる)。
* `rasterio`: GeoTIFFファイルの操作。

これらのライブラリは、pipでインストールできます。Annacondaの人はcondaを使ってください。

```bash
pip install xarray matplotlib cartopy netCDF4 rasterio
```

conda: 
```bash
conda install xarray matplotlib cartopy netCDF4 rasterio
```

uv:
```bash
uv add xarray matplotlib cartopy netCDF4 rasterio
```

## 3. 使い方

1. データ取得: CHIRPS2年降水NetCDFファイル (例: chirps-v2.0.annual.nc) をダウンロードし、指定したwork_dirディレクトリに配置します。  このディレクトリパスはスクリプト内で設定します。

2. スクリプト実行: 各セルごとに実行していくだけです。
スクリプトは以下の処理を行います。

* NetCDFファイルを読み込む。
* 指定した緯度経度範囲に基づいてデータを抽出する。
* 平均降水量を計算する。
* 平均降水量の地図を表示する。
* 平均降水量データをGeoTIFFファイル (average_prec.tif) としてwork_dirディレクトリに保存する。

## 4. コード解説
```Python
import os
import netCDF4
import xarray as xr


# 作業フォルダとデータの指定
work_dir = "/home/nknazs/CHIRPS/data"
data = "chirps-v2.0.annual.nc"  # 入力するファイル名を入力
output_file = "average_prec.tif"    # 出力データ

# ファイルの読み込み
fpath = os.path.join(work_dir, data)   # ファイルパスの結合

# データセットを開く
ds = xr.open_dataset(fpath)
```

データパス:
work_dir変数でNetCDFファイルが格納されているディレクトリのパスを指定します。例: work_dir = "/home/nknazs/CHIRPS/data"
data変数でNetCDFファイル名を指定します。例: data = "chirps-v2.0.annual.nc"
これらの設定が実際の環境に合っていることを確認してください。Windowsパスでは、バックスラッシュを2重にする (例: r"C:\\path\\to\\file") ことを推奨します。
データ読み込み: スクリプトはxarrayを用いてNetCDFファイルを開き、降水データ (precip変数) にアクセスします。変数名は特定のCHIRPS2ファイルによって異なる場合があるので、コマンドプロンプトからncdump -h chirps-v2.0.annual.ncを実行するか、PanoplyなどのNetCDFビューアでファイルを確認して正しい名前を調べてください。


データ抽出: latとlonのスライスで対象地域を定義します。例: lat = slice(40, 45)。必要に応じて開始値と終了値を調整して、目的の領域を選択してください。
```Python
## データの切り出し
lat = slice(40, 45)     # 緯度(データセットによっては指定方法が違うので注意．DatasetのCoordinatesを確認)
lon = slice(72, 82)    # 経度

ds  = ds.sel(latitude=lat, longitude=long)
```

データ集計: スクリプトは各グリッドセルの平均降水量を計算し、時間次元で平均化します。

```Python
# グリッドごとの降水量の平均値を計算
mean_prec_grid = ds.groupby(['latitude', 'longitude'], squeeze=False).mean()
# 時間方向の平均を計算
mean_prec_2d = mean_prec_grid['precip'].mean(dim='time')


print(mean_prec_grid)
```


可視化: cartopyとmatplotlibを用いて、平均降水量の地図を作成します。地図の範囲は、選択した緯度経度範囲とバッファによって決定されます。

```Python
## グラフとして出力
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
from cartopy.mpl.gridliner import LONGITUDE_FORMATTER, LATITUDE_FORMATTER


# Cartopyで地図を作成
plt.figure(figsize=(10, 6))
ax = plt.axes(projection=ccrs.PlateCarree())
ax.add_feature(cfeature.COASTLINE)
ax.add_feature(cfeature.BORDERS, linestyle=':')

# 降水量をカラーマップで表示 (修正箇所)
im = ax.contourf(mean_prec_grid.longitude, mean_prec_grid.latitude, mean_prec_2d, cmap='jet')

# カラーバーを表示
plt.colorbar(im, orientation='vertical', label='Precipitation (mm/year)')

# 表示範囲の設定
lat_min, lat_max = lat.start, lat.stop  # 緯度
lon_min, lon_max = lon.start, lon.stop  # 経度
# bufferの設定（最初に指定した範囲からどれくらい表示範囲を広げる or 狭くする）
lat_buffer = 2
lon_buffer = 2
# リスト化してまとめる
extent = [lon_min - lon_buffer, lon_max + lon_buffer, lat_min - lat_buffer, lat_max + lat_buffer]

# 範囲設定を図に適用
ax.set_extent(extent)

# 緯度経度グリッドを表示
gl = ax.gridlines(crs=ccrs.PlateCarree(), draw_labels=True,
                  linewidth=1, color='gray', alpha=0.5, linestyle='--')
gl.top_labels = False  # 上側のラベルを非表示
gl.right_labels = False  # 右側のラベルを非表示
gl.xformatter = LONGITUDE_FORMATTER  # 経度ラベルの書式を設定
gl.yformatter = LATITUDE_FORMATTER   # 緯度ラベルの書式を設定

# タイトルを表示
plt.title('Average Precipitation')

# グラフを表示
plt.show()
```

GeoTIFFエクスポート: 処理された降水データは、rioxarrayを用いてGeoTIFFファイルとしてエクスポートされます（座標系はWGS 84）。　


```Python
fpath_output = os.path.join(work_dir, output_file)
mean_prec_2d.rio.to_raster(fpath_output, crs="EPSG:4326")  # 座標系をWGS84に指定
```

## 5. ライセンス
このスクリプトはMITライセンスで提供されています。

## 6. 謝辞
このスクリプトはCHIRPSのデータを使用しています。

