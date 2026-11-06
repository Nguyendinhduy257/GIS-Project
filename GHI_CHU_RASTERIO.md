# Ghi chú: lỗi không tìm thấy `rasterio`

Notebook dùng kernel `evgis`, nhưng `import rasterio` báo `ModuleNotFoundError` dù pip báo đã cài. Nguyên nhân là gói trong môi trường bị thiếu hoặc cài không nhất quán.

## Cách khắc phục đã dùng

Mở Anaconda Prompt và chạy:

```bash
conda activate evgis
conda install -c conda-forge --force-reinstall rasterio
```

Chấp nhận kế hoạch cài đặt, đợi hoàn tất, rồi **restart kernel** notebook. Kiểm tra:

```python
import sys, rasterio
print(sys.executable)
print(rasterio.__version__)
```

Kết quả đúng: đường dẫn Python nằm trong `anaconda3\envs\evgis\` và `rasterio` import được.

## Để tránh lặp lại

- Chọn kernel **Python (evgis)** trong notebook.
- Cài thư viện GIS vào đúng môi trường Conda; dự án khai báo phụ thuộc trong `environment.yml`.
- Sau khi cài hoặc sửa gói, restart kernel trước khi import lại.
