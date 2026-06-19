# gitops-manifest-k8s

GitOps manifests cho dự án YAS (Yet Another Shop).
Được quản lý bằng **Kustomize**, **ArgoCD** watch và sync.

> Infrastructure (Postgres, Kafka, Keycloak, Elasticsearch, Redis) do TV1 deploy bằng Helm —
> repo này **KHÔNG** chứa infra manifests.

## Cấu trúc

```
gitops-manifest-k8s/
├── base/
│   ├── kustomization.yaml          # list 57 resources (19 services × 3)
│   ├── <service>/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── serviceaccount.yaml     # BẮT BUỘC cho Istio AuthorizationPolicy (TV4)
│   └── _common/
│       └── namespace-default.yaml  # optional, overlay luôn override namespace
├── environments/
│   ├── dev/                        # namespace=dev, image tags auto-update bởi Jenkins CI
│   ├── staging/                    # namespace=staging, update khi có tag v*
│   └── developer-build/            # namespace=developer-build, type=NodePort
│       ├── kustomization.yaml
│       └── patches/nodeport-patch.yaml
└── README.md
```

## Service Ports

| # | Service | App Port | Ghi chú |
|:-:|---------|:--------:|---------|
| 1 | product | 8080 | |
| 2 | payment | 8081 | |
| 3 | media | 8083 | |
| 4 | cart | 8084 | |
| 5 | order | 8085 | |
| 6 | location | 8086 | |
| 7 | backoffice-bff | 8087 | Spring Boot, YAML config |
| 8 | storefront-bff | 8087 | Spring Boot, YAML config |
| 9 | customer | 8088 | |
| 10 | rating | 8089 | |
| 11 | inventory | 8090 | |
| 12 | tax | 8091 | |
| 13 | promotion | 8092 | conflict port (khác pod nên OK) |
| 14 | search | 8092 | conflict port (khác pod nên OK) |
| 15 | webhook | 8092 | conflict port (khác pod nên OK) |
| 16 | payment-paypal | 8093 | |
| 17 | sampledata | 8094 | |
| 18 | recommendation | 8095 | |
| 19 | delivery | 8080 | default Spring Boot port |

## Validate trước khi push

```bash
# dev — phải có 19 Deployment, 57 namespace=dev, 19 ServiceAccount
kubectl kustomize environments/dev/ > /tmp/dev-output.yaml
grep -c "kind: Deployment"     /tmp/dev-output.yaml   # = 19
grep -c "kind: ServiceAccount" /tmp/dev-output.yaml   # = 19

# developer-build — tất cả Service phải có type: NodePort
kubectl kustomize environments/developer-build/ > /tmp/devbuild-output.yaml
grep -c "type: NodePort" /tmp/devbuild-output.yaml    # = 19
```

## Cách update image tag (thủ công)

```bash
cd environments/dev
kustomize edit set image bingsu1103/product=bingsu1103/product:abc1234
```

Jenkins CI tự động chạy lệnh trên với commit-id cho từng service khi build.

## Cách thêm service mới

1. Tạo thư mục `base/<service-name>/`
2. Copy & chỉnh `deployment.yaml`, `service.yaml`, `serviceaccount.yaml` (đổi tên + port)
3. Thêm 3 dòng resource vào `base/kustomization.yaml`
4. Thêm image entry vào 3 file `environments/*/kustomization.yaml`
