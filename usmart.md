# Các bước triển khai Phân hệ, dịch vụ trên Usmart.

## Một số ký hiệu

| Ký hiệu | Mô tả | Ví dụ |
| ------- | ----- | ----- |
| [MA_DICH_VU] | Mã dịch vụ | ThiDuaKhenThuong |
| [MA_DICH_VU_VIET_THUONG] | Mã dịch vụ | thiduakhenthuong |
| [MA_DICH_VU_VIET_HOA] | Mã dịch vụ viết hoa | THIDUAKHENTHUONG |
| [TEN_DICH_VU] | Tên dịch vụ | Quản lý Thi đua  - khen thưởng |
| [DEFAULT_REDIRECT] | URL mặc định | dashboard |
| [PORT] | Port triển khai dịch vụ | 22046 |
| [ICON] | Icon của service | pi pi-home |

## Đóng gói docker image

Đóng gói docker image API

```sh
cd Microservice/[MA_DICH_VU].Api
deploy.bat
```

Đóng gói docker image client

```sh
cd ClientApp
npm run deploy-[MA_DICH_VU_VIET_THUONG]
```

## Triển khai lên docker

- Đăng nhập vào portainer: [http://192.168.125.85:9000](http://192.168.125.85:9000)
- Cập nhật stack [utc_prod](http://192.168.125.85:9000/#!/1/docker/stacks/utc_prod?id=103&type=1&regular=true&external=false&orphaned=false) chèn vào phía trên:

```sh
volumes:
  u202001_prod_v5_admin_client_env:
    external: true
networks:
  u202001_prod_v5_s1:
    external: true
```

đoạn mã sau:

```sh
[MA_DICH_VU_VIET_THUONG]_api:
    image: dockerhub.utc.edu.vn/[MA_DICH_VU_VIET_THUONG].api:5.1
    environment:
      - CORE_VERSION=5.0
      - ETCD__CONFIG=authenticated=true;server=http://192.168.125.85:2379;username=root;password=Usm@rt123
      - SCOPE__ENVIRONMENT=Production
      - SCOPE__NAME=U202001
      - SCOPE__SERVICE=[MA_DICH_VU_VIET_THUONG]
    networks:
      u202001_prod_v5_s1:
        aliases:
          - [MA_DICH_VU_VIET_THUONG].api
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
        - traefik.http.middlewares.[MA_DICH_VU_VIET_THUONG]_api_v5-stripprefix.stripprefix.prefixes=/[MA_DICH_VU_VIET_THUONG]
        - traefik.http.routers.[MA_DICH_VU_VIET_THUONG]_api_v5.entrypoints=http
        - traefik.http.routers.[MA_DICH_VU_VIET_THUONG]_api_v5.middlewares=[MA_DICH_VU_VIET_THUONG]_api_v5-stripprefix
        - traefik.http.routers.[MA_DICH_VU_VIET_THUONG]_api_v5.rule=PathPrefix(`/[MA_DICH_VU_VIET_THUONG]{regex:$$|/.*}`)
        - traefik.http.services.[MA_DICH_VU_VIET_THUONG]_api_v5.loadbalancer.server.port=80
  [MA_DICH_VU_VIET_THUONG]_client:
    image: dockerhub.utc.edu.vn/[MA_DICH_VU_VIET_THUONG].client:5.1
    ports:
      - [PORT]:80
    networks:
      u202001_prod_v5_s1:
        aliases:
          - [MA_DICH_VU_VIET_THUONG].client
    environment:
      - BASE_HREF=/[MA_DICH_VU_VIET_THUONG]
      - APP_CODE=[MA_DICH_VU_VIET_HOA]
    volumes:
      -  u202001_prod_v5_admin_client_env:/usr/share/nginx/html/assets/environments/
    deploy:
      replicas: 1
      update_config:
        order: stop-first
      placement:
        constraints: [node.role == manager]
```

- Click vào nút **Update the stack**

## Cấu hình tham số

**Đăng nhập xftd vào 2 máy chủ sau:**

- 192.168.125.85

```sh
username: root
password: phanhenckh++
```

- 192.168.125.86

```sh
username: root
password: 4n2APr3aM24q8yVH
```

**Sửa file prod.json**

- 192.168.125.85

```sh
/mnt/docker/volumes/u202001_prod_v5_admin_client_env/_data/prod.json
```

- 192.168.125.86

```sh
/var/lib/docker/volumes/dhgiaothong_user_client_env_prod/_data/prod.json
```

Thêm đoạn mã sau vào **appSwitcher**:

```sh
{
    "code": "[MA_DICH_VU_VIET_HOA]",
    "icon": "[ICON]",
    "title": "[TEN_DICH_VU]",
    "longTitle": "Phân hệ [TEN_DICH_VU]",
    "url": "/[MA_DICH_VU_VIET_THUONG]",
    "defaultRedirect": "[DEFAULT_REDIRECT]",
    "target": "_blank",
    "visible": true,
    "showDefaultSetting": false,
    "background": "#87b1c5",
    "colorTitle": "#fff",
    "colorIcon": "#fff",
    "column": 1
}
```

**Sửa file nginx.conf**

- 192.168.125.86

```sh
/var/lib/docker/volumes/nginx_config_dhgiaothong_prod/_data/nginx.conf
```

Chèn phía trên:

```sh
location / {
    proxy_http_version 1.1;
    proxy_pass http://user.client/user/;
}
```

Đoạn mã sau:

```sh
location ^~ /[MA_DICH_VU_VIET_THUONG] {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_pass http://192.168.125.85:[PORT]/[MA_DICH_VU_VIET_THUONG]/;
}
```

*Khởi động lại dịch vụ* **nginx** *trong stack* **dhgiaothong_prod**

*Copy tập tin* **commonworkspacev5/App/shared/shared-assets/menus/[MA_DICH_VU_VIET_THUONG].json** vào thư mục **/mnt/docker/volumes/u202001_prod_v5_admin_client_menu/_data** trên máy chủ **192.168.125.85**

## Thêm phân hệ trên USMART

- Đăng nhập vào hệ thống [USMART](https://usmart.utc.edu.vn) bằng tài khoản admin.

- Thêm phân hệ trong phần [Quản lý phân hệ](https://usmart.utc.edu.vn/admin/authorization/module).

```sh
Mã: [MA_DICH_VU_VIET_HOA]
Tên: [TEN_DICH_VU]
```

- Sử dụng chức năng **Import** (Chọn Import client) trong phần [Quản lý quyền](https://usmart.utc.edu.vn/admin/authorization/base-permission) để import danh sách quyền cho dịch vụ.

## Phân quyền ứng dụng

## Chạy thử dịch vụ

Truy cập vào <https://usmart.utc.edu.vn/[MA_DICH_VU_VIET_THUONG]/[DEFAULT_REDIRECT>]

## Khác

- Add git

```sh
dotnet nuget add source https://git.dttt.vn/api/v4/projects/8/packages/nuget/index.json -n "Tri Nam" -u nuget -p PcDjH5AuX8TstvzxnaDx
```
