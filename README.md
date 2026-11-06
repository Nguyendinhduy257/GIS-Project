# Chọn vị trí trạm sạc xe điện bằng GIS — 31 khu vực Hà Nội

Đồ án cuối kỳ môn **Hệ thống thông tin không gian GIS**.

Dự án xây dựng quy trình chấm điểm đa tiêu chí (AHP) để tìm vị trí đặt trạm sạc xe điện
trên địa bàn Hà Nội, sử dụng lưới ô vuông 250 m và 5 tiêu chí không gian: dân số,
chung cư, điểm đến (POI), đường trục và khoảng cách tới trạm sạc hiện có.

Phạm vi: **31 polygon phân tích** = 30 đơn vị cấp huyện cũ (12 quận, 17 huyện, 1 thị xã)
+ phường Lý Thái Tổ cũ.

| Chỉ số | Giá trị |
|---|---|
| Ô lưới 250 m toàn vùng | 54.779 |
| Ô bị loại (biên, sông, hồ) | 2.684 (4,9%) |
| Ô hợp lệ để chấm điểm | 52.095 |
| Diện tích vùng hợp lệ | 3.351,5 km² |
| Dân số trong ô hợp lệ | 8.340.509 |
| Trạm sạc hiện có | 332 (256 nằm trong vùng) |
| Ô đạt từ 65 điểm | 3.717 (7,1%) |
| Điểm cao nhất | 94,8 |

Trọng số AHP: dân số 0,3689 · chung cư 0,2063 · điểm đến 0,1093 · đường trục 0,1093 ·
trạm sạc 0,2063 (tỷ số nhất quán CR = 0,003).

---

## 1. Cấu trúc dự án

```
tram_sac_xe_dien/
├─ tram_sac_xe_dien_toan_bo_quan_huyen_Ha_Noi_v2.ipynb   # NOTEBOOK CHÍNH
├─ tram_sac_xe_dien_toan_bo_quan_huyen_Ha_Noi (1).ipynb  # bản trước v2 (tham khảo)
├─ tram_sac_xe_dien.ipynb                                # bản 2 quận (lịch sử)
├─ tram_sac_xe_dien_2.ipynb                              # bản nhiều quận (lịch sử)
├─ tram_sac_xe_dien_mo_rong*.ipynb                       # bản thử nghiệm (lịch sử)
│
├─ du_lieu/
│  ├─ vnm_ppp_2020.tif          # raster dân số WorldPop 2020 (187,8 MB, lưu bằng Git LFS)
│  ├─ charging_manual.csv       # trạm sạc nhập tay
│  ├─ area.gpkg
│  └─ hn/
│     ├─ <slug>/                # mỗi khu vực: area, roads_main, charging, apartments, poi, water
│     ├─ charging_all.gpkg      # toàn bộ trạm sạc đã gộp
│     ├─ pop_clip.tif           # dân số đã cắt theo vùng nghiên cứu
│     └─ pop_clip.json          # dấu vết kiểm tra pop_clip còn mới hay không
│
├─ ket_qua/                     # bảng, bản đồ, biểu đồ (tự sinh khi chạy)
├─ tro_choi_hn/
│  ├─ index.html                # trò chơi chọn vị trí trạm sạc
│  └─ data/                     # 31 .js khu vực + hn.js + 34 .geojson (tự sinh)
│
├─ bao_cao/
│  ├─ bao_cao.md                # báo cáo đầy đủ
│  └─ khung_bao_cao.md
│
├─ environment.yml              # môi trường Conda (tên: evgis)
├─ GHI_CHU_RASTERIO.md          # ghi chú sửa lỗi rasterio
└─ README.md
```

> Các notebook không có đuôi `_v2` là bản cũ, chỉ để tham khảo.
> **Chỉ cần chạy `tram_sac_xe_dien_toan_bo_quan_huyen_Ha_Noi_v2.ipynb`.**

---

## 2. Yêu cầu hệ thống

| Thành phần | Ghi chú |
|---|---|
| Hệ điều hành | Windows 10/11 (khuyến nghị). macOS/Linux chạy được nhưng bỏ qua ô sửa DLL |
| Anaconda | Đã cài Anaconda hoặc Miniconda, có Anaconda Navigator |
| Git + Git LFS | **Bắt buộc** để lấy đúng file raster 187,8 MB |
| Dung lượng trống | ~4 GB (môi trường ~3 GB + dữ liệu và `.git` ~1 GB) |
| Internet | Cần khi clone và khi xem tile bản đồ; chạy lại thì không bắt buộc |

Tải Git (kèm Git LFS): https://git-scm.com/download/win

---

## 3. Cài đặt môi trường `evgis`

### 3.1. Clone dự án (có Git LFS)

Mở **PowerShell** tại thư mục muốn lưu dự án (nên dùng đường dẫn ngắn, không dấu):

```powershell
git lfs install
git clone https://github.com/Nguyendinhduy257/GIS-Project.git
cd GIS-Project
git lfs pull
```

Kiểm tra file raster đã tải đủ:

```powershell
Get-Item du_lieu\vnm_ppp_2020.tif | Select-Object Length
```

Kết quả đúng: `196897622` (≈187,8 MB). Nếu chỉ vài trăm byte thì file mới chỉ là
"con trỏ" LFS → chạy lại `git lfs pull`.

> Nếu repo để **Private**, bạn cần được chủ repo thêm làm collaborator
> (Settings → Collaborators) rồi mới clone được.

### 3.2. Import bằng Anaconda Navigator

1. Mở **Anaconda Navigator**.
2. Chọn tab **Environments** (bên trái).
3. Bấm **Import** (thanh dưới cùng).
4. Chọn **Local file** → trỏ tới `environment.yml` trong thư mục dự án.
5. Ô tên môi trường sẽ tự điền **`evgis`** (lấy từ trường `name:` trong file) — giữ nguyên.
6. Bấm **Import** và đợi 5–15 phút (tùy mạng).
7. Khi xong, chọn môi trường `evgis` → bấm **▶** → **Open Terminal**
   (hoặc **Open with JupyterLab**).

Nếu Navigator hỏi điều khoản sử dụng kênh `conda-forge`, chọn đồng ý để tiếp tục.

Nếu máy đã có môi trường tên `evgis`, Import sẽ báo trùng tên — hãy xoá môi trường cũ
(hoặc đổi `name:` trong file) trước khi Import.

### 3.3. Cách khác: tạo bằng dòng lệnh

```powershell
conda env create -f environment.yml
conda activate evgis
```

### 3.4. Kiểm tra môi trường

Trong terminal của `evgis` (hoặc một cell notebook), chạy:

```python
import sys
import geopandas, osmnx, rasterio, rasterstats, shapely, pyproj, folium, mapclassify

print(sys.executable)
print("geopandas ", geopandas.__version__)
print("osmnx     ", osmnx.__version__)
print("rasterio  ", rasterio.__version__)
print("shapely   ", shapely.__version__)
```

Kết quả đúng: `sys.executable` phải nằm trong `...\anaconda3\envs\evgis\`.

Nếu `rasterio` báo lỗi, xem `GHI_CHU_RASTERIO.md` hoặc chạy:

```powershell
conda install -c conda-forge --force-reinstall rasterio
```

### 3.5. Đăng ký và chọn kernel

Môi trường `evgis` cần xuất hiện trong danh sách kernel của Jupyter:

```powershell
conda activate evgis
python -m ipykernel install --user --name evgis --display-name "Python (evgis)"
```

Trong notebook: **Kernel → Change kernel → Python (evgis)**.

> Bắt buộc chọn đúng kernel. Nếu để kernel `python3` mặc định, notebook có thể chạy
> bằng Python base và báo `ModuleNotFoundError`.

---

## 4. Chạy notebook

1. Mở **JupyterLab** từ môi trường `evgis`.
2. Điều hướng tới **thư mục gốc dự án** (nơi có `environment.yml`).
   Không mở notebook từ thư mục cha — mọi đường dẫn trong notebook là tương đối.
3. Mở `tram_sac_xe_dien_toan_bo_quan_huyen_Ha_Noi_v2.ipynb`.
4. Chọn kernel **Python (evgis)**.
5. **Kernel → Restart Kernel and Run All Cells**.

### 4.1. Các cờ cấu hình (ô "Cài đặt", phần 0)

| Biến | Mặc định | Ý nghĩa |
|---|---|---|
| `SIZE` | `250` | Cạnh ô lưới (m) |
| `BUFFER_M` | `1000` | Vùng đệm quanh mỗi khu vực khi tải OSM |
| `NORM_SCOPE` | `"all"` | Chuẩn hóa chung toàn thành phố để so sánh giữa các quận |
| `FORCE_AREA` | `False` | `True`: tìm lại ranh giới dù đã có `area.gpkg` |
| `FORCE_OSM` | `False` | `True`: tải lại dữ liệu OSM dù đã có |
| `FORCE_POP` | `False` | `True`: cắt lại dân số dù đã có `pop_clip.tif` |
| `DOWNLOAD_WORLDPOP` | `False` | `True`: tự tải raster WorldPop bằng code |

### 4.2. Thời gian chạy

| Tình huống | Thời gian |
|---|---|
| Đã có đủ dữ liệu trong repo (bình thường) | ~10–20 phút |
| Phải tải lại OSM / Overpass lỗi phải thử lại | 30–60 phút |
| Đã có `cache/` từ lần chạy trước | nhanh hơn đáng kể |

Notebook tự bỏ qua bước tải nếu `du_lieu/hn/<slug>/*.gpkg` đã tồn tại và hợp lệ.
Overpass hay trả lỗi 502 — notebook có cơ chế thử lại và đổi máy chủ, cứ để chạy tiếp.

---

## 5. Kết quả

### 5.1. Tệp kết quả

| Tệp | Nội dung |
|---|---|
| `ket_qua/hn_tong_hop_quan.csv` | Tổng hợp 31 khu vực: số ô, điểm trung bình, ô tốt, dân số, trạm |
| `ket_qua/hn_top10_toan_thanh.csv` | 10 vị trí tốt nhất toàn thành phố |
| `ket_qua/hn_top3_moi_quan.csv` | 3 vị trí tốt nhất mỗi khu vực (93 dòng) |
| `ket_qua/hn_ban_do.html` | Bản đồ tương tác (51 MB — GitHub không xem trước được) |
| `ket_qua/hn_so_sanh_quan.png` | Biểu đồ so sánh giữa các khu vực |
| `ket_qua/hn_do_nhay.png` | Biểu đồ phân tích độ nhạy |
| `tro_choi_hn/data/*.js`, `*.geojson` | Dữ liệu cho trò chơi (ô 27) |

> `ket_qua/ban_do.html`, `top10.csv`, `do_nhay.png` là kết quả của bản 2 quận cũ,
> không thuộc phạm vi báo cáo hiện tại.

### 5.2. Mở bản đồ và trò chơi

- Bản đồ: mở trực tiếp `ket_qua/hn_ban_do.html` bằng Chrome/Edge (cần Internet để tải tile).
- Trò chơi: mở `tro_choi_hn/index.html`.

Nếu trình duyệt chặn file local, chạy một web server tạm:

```powershell
python -m http.server 8000
```

rồi mở http://localhost:8000/tro_choi_hn/

---

## 6. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách sửa |
|---|---|---|
| `ModuleNotFoundError: No module named 'rasterio'` | Chạy nhầm kernel/env | Chọn kernel `Python (evgis)`; xem `GHI_CHU_RASTERIO.md` |
| Kernel chết, tự restart khi import | Xung đột DLL PostgreSQL/QGIS trên Windows | Chạy **ô 2** (sửa DLL) trước tiên, rồi restart kernel |
| `Kernel not found` khi mở notebook | Chưa đăng ký kernel | `python -m ipykernel install --user --name evgis --display-name "Python (evgis)"` |
| `FileNotFoundError: du_lieu/vnm_ppp_2020.tif` | Chưa tải LFS | `git lfs pull`, hoặc tải WorldPop, hoặc đặt `DOWNLOAD_WORLDPOP = True` |
| `RasterioIOError` khi mở file `.tif` | File chỉ là con trỏ LFS (~130 byte) | `git lfs pull` |
| `Warning: Cannot find header.dxf (GDAL_DATA is not defined)` | Thiếu biến môi trường GDAL | Vô hại; ô 2 đã thiết lập `GDAL_DATA` |
| Overpass trả 502 / timeout | Máy chủ OSM quá tải | Để notebook tự thử lại; nếu vẫn lỗi thì chạy lại ô 9 |
| Bản đồ trắng, không có tile | Không có Internet | Kết nối mạng rồi tải lại trang |
| `thiếu data/hn.js` | Chưa chạy phần 9 (ô 27) | Chạy ô 27 để xuất dữ liệu cho trò chơi |

---

## 7. Dữ liệu và phiên bản

### 7.1. Nguồn dữ liệu

| Dữ liệu | Nguồn | Giấy phép |
|---|---|---|
| Dân số raster 2020 | [WorldPop](https://www.worldpop.org/) – `vnm_ppp_2020.tif` | CC BY 4.0 |
| Ranh giới, đường, POI, chung cư, trạm sạc | OpenStreetMap qua Overpass API | ODbL |
| Trạm sạc nhập tay | Nhóm thu thập, `du_lieu/charging_manual.csv` | — |

### 7.2. Phiên bản thư viện

Phiên bản được ghim trong `environment.yml` (Python 3.11, kênh `conda-forge`):
geopandas 1.2.0 · osmnx 2.1.1 · shapely 2.2.0 · pyproj 3.7.2 · rasterio 1.4.4 ·
rasterstats 0.21.0 · numpy 2.4.6 · pandas 3.0.6 · scipy 1.17.1 · matplotlib 3.11.2 ·
mapclassify 2.10.0 · folium 0.20.0.

> Máy tác giả dùng rasterio 1.3.10 cài bằng pip. Bản 1.3.10 không cài được cùng
> numpy 2.x từ conda-forge, nên file môi trường dùng 1.4.4 — kết quả không thay đổi
> vì notebook chỉ dùng các API cơ bản (`open`, `mask`, `meta`, `write`) và `zonal_stats`.

### 7.3. Git LFS

Hai tệp lớn được lưu bằng Git LFS: `du_lieu/vnm_ppp_2020.tif` (187,8 MB) và
`ket_qua/hn_ban_do.html` (51,3 MB), tổng ~240 MB.

Nếu chỉ cần code và bảng kết quả, có thể clone bỏ qua LFS để tiết kiệm băng thông:

```powershell
$env:GIT_LFS_SKIP_SMUDGE=1
git clone https://github.com/Nguyendinhduy257/GIS-Project.git
```

Khi đó `du_lieu/hn/pop_clip.tif` (đã có sẵn trong repo) vẫn đủ để chạy phần lớn
quy trình. Chỉ khi notebook báo cần `vnm_ppp_2020.tif` thì mới phải tải thêm.

---

## 8. Báo cáo

Báo cáo đầy đủ (phương pháp, kết quả, thảo luận, phụ lục) nằm tại
[`bao_cao/bao_cao.md`](bao_cao/bao_cao.md).
