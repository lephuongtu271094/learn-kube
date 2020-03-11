Do cấu hình mặc định của Kubernetes Cluster, cổng được mở ra ngoài phải chọn trong khoảng 30000-32767 sau này có thể sửa cấu hình này với tham số chạy Cluster --service-node-portrange trong file dashboard-v2-rc5.yaml

# see more https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ and https://github.com/kubernetes/dashboard/releases


# Tạo kubernetes-dashboard-certs, xác thực SSL

Ta sẽ dùng OpenSSL để sinh certificates tự xác thực SSL (ngoài ra khi triển khai thực tế có thể lấy các certificate do mua hoặc miễn phí từ https://letsencrypt.org/ theo Domain), chạy các lệnh sau:

sudo mkdir certs
sudo chmod 777 -R certs
openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
sudo chmod -R 777 certs

Sau khi hoàn thành thì các thông tin certificate lưu ở thư mục certs, chạy lệnh sau để tạo Secret

kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard

Giờ truy cập địa chỉ https://192.168.10.100:31000 sẽ vào màn hình đăng nhập

# Tiếp theo chạy lệnh sau để lấy Token

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

# Triển khai metrics server
metrics server trong kubernetes (metrics server) giám sát về tài nguyên sử dụng trên cluster, cung cấp các API để các thành phần khác truy vấn đến biết được và mức độ sử dụng tài nguyên (CPU, Memory) của Pod, Node ... Cần có Metric Server để HPA hoạt động chính xác

Để triển khai, lấy về mã nguồn từ GitHub

git clone git@github.com:kubernetes-sigs/metrics-server.git

Mở file metrics-server/deploy/kubernetes/metrics-server-deployment.yaml tìm đến dòng args: sửa lại thành

        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
Sau đó triển khai

kubectl apply -f metrics-server/deploy/kubernetes/
Khi có Metric server thì trong Dashboard có thêm thống kê về CPU, Memory của các POD, và có thêm lệnh xem tài nguyên về Node, Pod