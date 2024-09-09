# Các bước triển khai

## 1. Mô hình triển khai Quẹt vân tay phân hệ Giảng đường

![datagram](../../images/giang-duong/trienkhai-1.png "hình 1")

## 2. Deploy lên Docker

### 2.1 Dịch vụ lấy và xử lý dữ liệu từ máy chấm công

- Được triển khai trên **Portainer** *192.168.125.155:9000* với ID: *admin*, Password: *CAIT@2020;*
- Được cài trên stacks **maychamcong**
- Nội dung file manifest để triển khai stacks **maychamcong**

```sh
version: "3.8"
services:
  receiver_attendance_teaching:
    image: dockerhub.utc.edu.vn/maychamcong-receiver:1.0
    ports:
      - "8088:8080"
    environment:
      - "BOOTSTRAP_SERVERS=192.168.125.7:9093"
      - "TOPIC=attendance_teaching_events"
    networks:
      - utc
  receiver_attendance_studying:
    image: dockerhub.utc.edu.vn/maychamcong-receiver:1.0
    ports:
      - "8089:8080"
    environment:
      - "BOOTSTRAP_SERVERS=192.168.125.7:9093"
      - "TOPIC=attendance_studying_events"
    networks:
      - utc
  processor_attendance_teaching:
    image: dockerhub.utc.edu.vn/maychamcong-processor:1.0
    environment:
      - "BOOTSTRAP_SERVERS=192.168.125.7:9093"
      - "TOPIC=attendance_teaching_events"
      - "GROUP_ID=processor_attendance_teaching"
      - "TYPE=GIANG_VIEN"
      - "ATTENDANCE_TEACHING_TOPIC=attendance_teaching_2024"
      - "ATTENDANCE_STUDYING_TOPIC=attendance_studying_2024"
      - "CONNECTION_STRING__ATTENDANCE=Server=192.168.80.205;Database=UTC_Prod_QuanLyMayChamCong;Uid=sa;Pwd=77udcn77@u7c2o2019;Connection Timeout=30;MultipleActiveResultSets=True;TrustServerCertificate=True"
      - "CONNECTION_STRING__STAFF_PROFILE=Server=192.168.125.132;Database=U202001_Prod_CanBo_V5;Uid=dhgt;Pwd=B9fd9wGEYvt3;Connection Timeout=30;MultipleActiveResultSets=True;TrustServerCertificate=True"
    networks:
      - utc
  processor_attendance_studying:
    image: dockerhub.utc.edu.vn/maychamcong-processor:1.0
    environment:
      - "BOOTSTRAP_SERVERS=192.168.125.7:9093"
      - "TOPIC=attendance_studying_events"
      - "GROUP_ID=processor_attendance_studying"
      - "TYPE=SINH_VIEN"
      - "ATTENDANCE_TEACHING_TOPIC=attendance_teaching_2024"
      - "ATTENDANCE_STUDYING_TOPIC=attendance_studying_2024"
      - "CONNECTION_STRING__ATTENDANCE=Server=192.168.80.205;Database=UTC_Prod_QuanLyMayChamCong;Uid=sa;Pwd=77udcn77@u7c2o2019;Connection Timeout=30;MultipleActiveResultSets=True;TrustServerCertificate=True"
      - "CONNECTION_STRING__STAFF_PROFILE=Server=192.168.125.132;Database=U202001_Prod_CanBo_V5;Uid=dhgt;Pwd=B9fd9wGEYvt3;Connection Timeout=30;MultipleActiveResultSets=True;TrustServerCertificate=True"
    networks:
      - utc
networks:
  utc:
    external: true
```

### 2.2 Dịch vụ xử lý dữ liệu hiển thị trên Usmart, Eutc, Tivi

- Phân hệ giảng đường được triển khai trên **Portainer** *192.168.125.85:9000* với ID: *admin*, Password: *X4xD7KBqKca9QGS5*
- Được cài đặt trên stacks **utc_prod**
- Nội dung file manifest để triển khai trên stacks **utc_prod**

```sh
version: '3.8'
services:
  giangduong_api:
    image: dockerhub.utc.edu.vn/giangduong.api:5.2.2
    environment:
      - CORE_VERSION=5.0
      - ETCD__CONFIG=authenticated=true;server=http://192.168.125.85:2379;username=root;password=Usm@rt123
      - SCOPE__ENVIRONMENT=Production
      - SCOPE__NAME=U202001
      - SCOPE__SERVICE=giangduong
      - USMART_CONFIGURATION_HOST=http://192.168.125.155:2379
      - USMART_CONFIGURATION_UID=root
      - USMART_CONFIGURATION_PWD=CAIT@2022;
      - USMART__TEACHING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__TEACHING__TOPIC=attendance_teaching_2024
      - USMART__STUDYING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__STUDYING__TOPIC=attendance_studying_2024
    networks:
      u202001_prod_v5_s1:
        aliases:
          - giangduong.api
    deploy:
      replicas: 1
      update_config:
        order: stop-first
      placement:
        constraints: [node.role == worker]
      labels:
        - traefik.constraint-label-sub=u202001_production_v5
        - traefik.docker.network=u202001_prod_v5_s1
        - traefik.enable=true
        - traefik.http.middlewares.giangduong_api_v5-stripprefix.stripprefix.prefixes=/giangduong
        - traefik.http.middlewares.giangduong_api_v5-websocket.headers.customrequestheaders.X-Forwarded-Proto=https
        - traefik.http.routers.giangduong_api_v5.entrypoints=http
        - traefik.http.routers.giangduong_api_v5.middlewares=giangduong_api_v5-stripprefix,giangduong_api_v5-websocket
        - traefik.http.routers.giangduong_api_v5.rule=PathPrefix(`/giangduong{regex:$$|/.*}`)
        - traefik.http.services.giangduong_api_v5.loadbalancer.server.port=80
        - traefik.http.services.giangduong_api_v5.loadbalancer.sticky=true
        - traefik.http.services.giangduong_api_v5.loadbalancer.sticky.cookie.name=GiangDuongStickyCookie
  giangduong_client:
    image: dockerhub.utc.edu.vn/u202001/giangduong.client:5.1.1
    ports:
      - 22049:80
    networks:
      u202001_prod_v5_s1:
        aliases:
          - giangduong.client
    environment:
      - BASE_HREF=/giangduong
      - APP_CODE=GIANGDUONG
    volumes:
      -  u202001_prod_v5_admin_client_env:/usr/share/nginx/html/assets/environments/
      -  u202001_prod_v5_admin_client_menu:/usr/share/nginx/html/assets/menus/
    deploy:
      replicas: 1
      update_config:
        order: stop-first
      placement:
        constraints: [node.role == manager]
  giangduong_workers:
    image: dockerhub.utc.edu.vn/giangduong.workers:5.1
    environment:
      - CORE_VERSION=5.0
      - ETCD__CONFIG=authenticated=true;server=http://192.168.125.85:2379;username=root;password=Usm@rt123
      - SCOPE__ENVIRONMENT=Production
      - SCOPE__NAME=U202001
      - SCOPE__SERVICE=giangduong
      - USMART_CONFIGURATION_HOST=http://192.168.125.155:2379
      - USMART_CONFIGURATION_UID=root
      - USMART_CONFIGURATION_PWD=CAIT@2022;
      - USMART__TEACHING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__TEACHING__TOPIC=attendance_teaching_2024
      - USMART__TEACHING__GROUP_ID=giangduong_teaching_worker_prod
      - USMART__TEACHING__ALLOW_DIFFERENCE=20 # 15 phút
      - USMART__STUDYING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__STUDYING__TOPIC=attendance_studying_2024
      - USMART__STUDYING__GROUP_ID=giangduong_studying_worker_prod
      - USMART__STUDYING__ALLOW_DIFFERENCE=15 # 15 phút
      - TZ=Asia/Ho_Chi_Minh
      - USMART__SIGNALR_ENDPOINT=http://giangduong.api/hubs/lich-hoc
      - USMART__TEACHING_RERUN=
      - USMART__STUDYING_RERUN=
      - STUDYING__TOTAL_CONSUMER=5
      - TEACHING__TOTAL_CONSUMER=3
    networks:
      u202001_prod_v5_s1:
        aliases:
          - giangduong.workers
    deploy:
      replicas: 1
      update_config:
        order: stop-first
      placement:
        constraints: [node.role == worker]
  giangduong_conjobs:
    image: dockerhub.utc.edu.vn/giangduong.cronjobs:5.0
    ports:
      - "21089:5000"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - CORE_VERSION=5.0
      - ETCD__CONFIG=authenticated=true;server=http://192.168.125.85:2379;username=root;password=Usm@rt123
      - SCOPE__ENVIRONMENT=Production
      - SCOPE__NAME=U202001
      - SCOPE__SERVICE=giangduong
      - USMART_CONFIGURATION_HOST=http://192.168.125.155:2379
      - USMART_CONFIGURATION_UID=root
      - USMART_CONFIGURATION_PWD=CAIT@2022;
      - USMART__TEACHING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__TEACHING__TOPIC=attendance_teaching_2024
      - USMART__TEACHING__GROUP_ID=giangduong_teaching_worker_prod
      - USMART__TEACHING__ALLOW_DIFFERENCE=20
      - USMART__STUDYING__BOOTSTRAP_SERVERS=192.168.125.7:9093
      - USMART__STUDYING__TOPIC=attendance_teaching_2024
      - USMART__STUDYING__GROUP_ID=giangduong_studying_worker_prod
      - USMART__STUDYING__ALLOW_DIFFERENCE=15
      - USMART__SIGNALR_ENDPOINT=http://giangduong.api/hubs/lich-hoc
    networks:
      u202001_prod_v5_s1:
        aliases:
          - giangduong.conjobs
    deploy:
      replicas: 1
      update_config:
        order: stop-first
volumes:
  u202001_prod_v5_admin_client_env:
    external: true
  u202001_prod_v5_admin_client_menu:
    external: true
networks:
  u202001_prod_v5_s1:
    external: true
```

- DB của phân hệ giảng đường được lưu trữ trên server: *192.168.125.132:1433* ID: *dhgt*, Password: *B9fd9wGEYvt3*, DB: *U202001_Prod_GiangDuong_V5*
