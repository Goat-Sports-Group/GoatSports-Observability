# GoatSports Observability

Bộ giám sát cho hệ thống GOAT Sports: thu số liệu (metrics), gom log và cảnh báo.

## Thành phần

| Dịch vụ | Vai trò | Cổng |
|---|---|---|
| Prometheus | Thu metrics từ `/actuator/prometheus` của gateway và 11 service | `9090` |
| Loki + Promtail | Promtail đọc log container, đẩy về Loki | `3100` |
| Grafana | Dashboard và cảnh báo | `3200` |

## Cấu trúc

```
.
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml           # cấu hình production (scrape theo tên container)
│   └── prometheus.dev.yml       # cấu hình khi chạy local
├── promtail/promtail-config.yml
└── grafana/
    ├── dashboards/              # goatsports-overview, goatsports-service
    └── provisioning/
        ├── datasources/         # Prometheus, Loki
        ├── dashboards/
        └── alerting/            # luật cảnh báo, kênh nhận cảnh báo
```

## Cảnh báo có sẵn

`grafana/provisioning/alerting/goatsports-alert-rules.yaml`:

- **SERVICE DOWN**: một service ngừng trả metrics
- **Lỗi 5xx**: lỗi phía server tăng
- **4xx bất thường**
- **401/403 tăng vọt**: dấu hiệu dò tài khoản hoặc lỗi phân quyền
- **Log ERROR (Loki)**: xuất hiện log mức ERROR

Kênh nhận cảnh báo cấu hình trong `goatsports-contact-points.yaml`.

## Chạy

Cần mạng `goatsports-network` (tạo bởi repo `GoatSports-Docker`) để Prometheus gọi được các service.

```bash
docker compose up -d
```

Grafana: http://localhost:3200. Dashboard và datasource được provision tự động khi khởi động.
audit-service đọc Prometheus và Grafana để hiển thị trang Giám sát trong app quản trị.
